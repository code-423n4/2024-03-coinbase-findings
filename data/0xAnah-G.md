#   COINBASE SMART WALLET GAS OPTIMIZATIONS



## INTRODUCTION
Highlighted below are optimizations exclusively targeting state-mutating functions and view/pure functions invoked by state-mutating functions. In the discussion that follows, only runtime gas is emphasized, given its inevitable dominance over deployment gas costs throughout the protocol's lifetime. 

Please be aware that some code snippets may be shortened to conserve space, and certain code snippets may include @audit tags in comments to facilitate issue explanations."



## [G-01] Declare and use a constant variable rather than a using a pure function to return a constant value in the `CoinbaseSmartWallet` and `MagicSpend` contracts
- https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWallet.sol#L20-#L366
- https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L18-#L341

The `CoinbaseSmartWallet` and `MagicSpend` contracts both declares a function `entryPoint()` whose purpose is to return a constant address value `0x5FF137D4b0FDCD49DcA30c7CF57E578a026d2789` (address of the EntryPoint v0.6), this function is called in modifiers and other functions in both contracts. Since the value returned by this function is a constant it would be better gas efficient to replace this function call in the modifiers and functions that call the `entryPoint()` function to read a constant variable instead. This is because having to invoke a function would cost around 40 gas unit as it invloves 2 `JUMP` instruction and the stack set-up while reading a constant variable is much cheaper as it is just a stack read
which cost around 3 gas units. The diff below shows how the `CoinbaseSmartWallet` contract should be refactored similar procedure should be carried out for the `MagicSpend` contract.


```solidity
file: src/SmartWallet/CoinbaseSmartWallet.sol

20:  contract CoinbaseSmartWallet is MultiOwnable, UUPSUpgradeable, Receiver, ERC1271 {
.
.
.
65:     modifier onlyEntryPoint() virtual {
66:         if (msg.sender != entryPoint()) {
67:             revert Unauthorized();
68:         }
69: 
70:         _;
71:     }
.
.
.
74:    modifier onlyEntryPointOrOwner() virtual {
75:        if (msg.sender != entryPoint()) {
76:            _checkOwner();
77:        }
78:
79:        _;
80:    }
.
.
.
217:    function entryPoint() public view virtual returns (address) {
218:        return 0x5FF137D4b0FDCD49DcA30c7CF57E578a026d2789;
219:    }
.
.
.
229:    function getUserOpHashWithoutChainId(UserOperation calldata userOp)
230:        public
231:        view
232:        virtual
233:        returns (bytes32 userOpHash)
234;    {
235:        return keccak256(abi.encode(UserOperationLib.hash(userOp), entryPoint()));
236:    }
.
.
.
366: }
```

```diff
diff --git a/src/SmartWallet/CoinbaseSmartWallet.sol b/src/SmartWallet/CoinbaseSmartWallet.sol
index 2278a5a..b4edffc 100644
--- a/src/SmartWallet/CoinbaseSmartWallet.sol
+++ b/src/SmartWallet/CoinbaseSmartWallet.sol
@@ -42,6 +42,8 @@ contract CoinbaseSmartWallet is MultiOwnable, UUPSUpgradeable, Receiver, ERC1271
     /// @dev Helps enforce sequential sequencing of replayable transactions.
     uint256 public constant REPLAYABLE_NONCE_KEY = 8453;

+    address private constant ENTRY_POINT_ADDRESS = 0x5FF137D4b0FDCD49DcA30c7CF57E578a026d2789;
+
     /// @notice Thrown when trying to re-initialize an account.
     error Initialized();

@@ -63,7 +65,7 @@ contract CoinbaseSmartWallet is MultiOwnable, UUPSUpgradeable, Receiver, ERC1271

     /// @notice Reverts if the caller is not the EntryPoint.
     modifier onlyEntryPoint() virtual {
-        if (msg.sender != entryPoint()) {
+        if (msg.sender != ENTRY_POINT_ADDRESS) {
             revert Unauthorized();
         }

@@ -72,7 +74,7 @@ contract CoinbaseSmartWallet is MultiOwnable, UUPSUpgradeable, Receiver, ERC1271

     /// @notice Reverts if the caller is neither the EntryPoint, the owner, nor the account itself.
     modifier onlyEntryPointOrOwner() virtual {
-        if (msg.sender != entryPoint()) {
+        if (msg.sender != ENTRY_POINT_ADDRESS) {
             _checkOwner();
         }

@@ -232,7 +234,7 @@ contract CoinbaseSmartWallet is MultiOwnable, UUPSUpgradeable, Receiver, ERC1271
         virtual
         returns (bytes32 userOpHash)
     {
-        return keccak256(abi.encode(UserOperationLib.hash(userOp), entryPoint()));
+        return keccak256(abi.encode(UserOperationLib.hash(userOp), ENTRY_POINT_ADDRESS));
     }

     /// @notice Returns the implementation of the ERC1967 proxy.
```



## [G-02] Refactor the `MagicSpend.withdraw()` function such that checks for `withdrawRequest.expiry` comes before validting the withdraw request.
- https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L181-#L194

We can make the `MagicSpend.withdraw()` function better gas efficient if the `withdrawRequest.expiry` check `if (block.timestamp > withdrawRequest.expiry) {revert Expired()}` is move to the top of the function and done before validating the request (i.e before calling the `_validateRequest()` function). This is because the gas cost of the `withdrawRequest.expiry` check `if (block.timestamp > withdrawRequest.expiry) {revert Expired()}` is cheaper (as it only involves comparing a global varable and a memory variable) than the gas cost of validating the request which involves reading from state and writing to state so that in scenarios where the `withdrawRequest.expiry` check fails the function would revert without having to perform the gas consuming operations of the `_validateRequest()` function. The function should be refactored as shown by the diff below: 

```solidity
file: src/MagicSpend/MagicSpend.sol

181:    function withdraw(WithdrawRequest memory withdrawRequest) external {
182:        _validateRequest(msg.sender, withdrawRequest);
183:
184:        if (!isValidWithdrawSignature(msg.sender, withdrawRequest)) {
185:            revert InvalidSignature();
186:        }
187:
188:        if (block.timestamp > withdrawRequest.expiry) {
189:            revert Expired();
190:        }
191:
192:        // reserve funds for gas, will credit user with difference in post op
193:        _withdraw(withdrawRequest.asset, msg.sender, withdrawRequest.amount);
194:    }
```

```diff
diff --git a/src/MagicSpend/MagicSpend.sol b/src/MagicSpend/MagicSpend.sol
index be2bed1..33b48c3 100644
--- a/src/MagicSpend/MagicSpend.sol
+++ b/src/MagicSpend/MagicSpend.sol
@@ -179,16 +179,15 @@ contract MagicSpend is Ownable, IPaymaster {
     ///
     /// @param withdrawRequest The withdraw request.
     function withdraw(WithdrawRequest memory withdrawRequest) external {
+        if (block.timestamp > withdrawRequest.expiry) {
+            revert Expired();
+        }
         _validateRequest(msg.sender, withdrawRequest);

         if (!isValidWithdrawSignature(msg.sender, withdrawRequest)) {
             revert InvalidSignature();
         }

-        if (block.timestamp > withdrawRequest.expiry) {
-            revert Expired();
-        }
-
         // reserve funds for gas, will credit user with difference in post op
         _withdraw(withdrawRequest.asset, msg.sender, withdrawRequest.amount);
     }
```
```
Estimated gas saved: 5000 gas units
```



## [G-03]  Pre-calculate `keccak256("EIP712Domain(string name,string version,uint256 chainId,address verifyingContract)")` and assign it to a constant variable.
- https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/ERC1271.sol#L104

Since `"EIP712Domain(string name,string version,uint256 chainId,address verifyingContract)"` is a constant value its keccak would always resolve to the same hash. Hence it should be pre-calculted and the hash saved to constant variable. Having to read the constant value any time the fuction is called would make the function cheaper than having to compute the hash everytime the function is called. The diff below shows how the code should be re-factored:

```solidity
file: src/SmartWallet/ERC1271.sol

100:    function domainSeparator() public view returns (bytes32) {
101:        (string memory name, string memory version) = _domainNameAndVersion();
102:        return keccak256(
103:            abi.encode(
104:                keccak256("EIP712Domain(string name,string version,uint256 chainId,address verifyingContract)"),
105:                keccak256(bytes(name)),
106:                keccak256(bytes(version)),
107:                block.chainid,
108:                address(this)
109:            )
110:        );
111:    }
```

```diff
diff --git a/src/SmartWallet/ERC1271.sol b/src/SmartWallet/ERC1271.sol
index c3144f3..0dad17b 100644
--- a/src/SmartWallet/ERC1271.sol
+++ b/src/SmartWallet/ERC1271.sol
@@ -22,6 +22,8 @@ abstract contract ERC1271 {
     ///         - An EIP-712 hash: keccak256("\x19\x01" || someDomainSeparator || hashStruct(someStruct))
     bytes32 private constant _MESSAGE_TYPEHASH = keccak256("CoinbaseSmartWalletMessage(bytes32 hash)");

+    bytes32 private constant _EIP712_DOMAIN_HASH = keccak256("EIP712Domain(string name,string version,uint256 chainId,address verifyingContract)");
+
     /// @notice Returns information about the `EIP712Domain` used to create EIP-712 compliant hashes.
     ///
     /// @dev Follows ERC-5267 (see https://eips.ethereum.org/EIPS/eip-5267).
@@ -101,7 +103,7 @@ abstract contract ERC1271 {
         (string memory name, string memory version) = _domainNameAndVersion();
         return keccak256(
             abi.encode(
-                keccak256("EIP712Domain(string name,string version,uint256 chainId,address verifyingContract)"),
+                _EIP712_DOMAIN_HASH,
                 keccak256(bytes(name)),
                 keccak256(bytes(version)),
                 block.chainid,
```
```
Estimated gas saved: 37 gas units
```




## [G-04] Multiple accesses of a array should use a local variable cache
The instances below point to the second+ access of a value inside an array, within a function. Caching an array's struct avoids re-calculating the array offsets into memory.

### Proof of concept 
```solidity
struct Person {
    string name;
    uint age;
    uint id;
}

contract NoCacheArrayElement {

    Person[] students;

    function createStudents() external  {
        Person[] memory arrayOfPersons = new Person[](3); 
        Person memory newPerson1 = Person("Emmanuel", 15,1);
        Person memory newPerson2 = Person("Faustina", 16,2);
        Person memory newPerson3 = Person("Emmanuela", 14,3);

        arrayOfPersons[0] = newPerson1;
        arrayOfPersons[1] = newPerson2;
        arrayOfPersons[2] = newPerson3;

        _addNewSet(arrayOfPersons);
    }

    function _addNewSet(Person[] memory _persons) internal {
        uint len = _persons.length;
        unchecked {
            for(uint i; i < len; ++i) {
                Person memory newStudent = Person(_persons[i].name, _persons[i].age, _persons[i].id);
                students.push(newStudent);
            }
        }

    }
}
```
```
test for test/NoCacheArrayElement.t.sol:NoCacheArrayElementTest
[PASS] test_createStudents() (gas: 230357)
```

```solidity

struct Person {
    string name;
    uint age;
    uint id;
}

contract CacheArrayElement {

    Person[] students;

    function createStudents() external  {
        Person[] memory arrayOfPersons = new Person[](3); 
        Person memory newPerson1 = Person("Emmanuel", 15,1);
        Person memory newPerson2 = Person("Faustina", 16,2);
        Person memory newPerson3 = Person("Emmanuela", 14,3);

        arrayOfPersons[0] = newPerson1;
        arrayOfPersons[1] = newPerson2;
        arrayOfPersons[2] = newPerson3;

        _addNewSet(arrayOfPersons);
    }

    function _addNewSet(Person[] memory _persons) internal {
        uint len = _persons.length;
        unchecked {
            for(uint i; i < len; ++i) {
                Person memory myPerson = _persons[i];
                Person memory newStudent = Person(myPerson.name, myPerson.age, myPerson.id);
                students.push(newStudent);
            }
        }

    }
}
```
```
test for test/Counter.t.sol:CacheArrayElementTest
[PASS] test_createStudents() (gas: 230096)
```

### Instances 

1. #### Cache `owners[i]`
- https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/MultiOwnable.sol#L164-#L169


```solidity
file: src/SmartWallet/MultiOwnable.sol

162:    function _initializeOwners(bytes[] memory owners) internal virtual {
163:        for (uint256 i; i < owners.length; i++) {
164:            if (owners[i].length != 32 && owners[i].length != 64) {
165:                revert InvalidOwnerBytesLength(owners[i]);
166:            }
167:
168:            if (owners[i].length == 32 && uint256(bytes32(owners[i])) > type(uint160).max) {
169:                revert InvalidEthereumAddressOwner(owners[i]);
170:            }
171:
172:            _addOwnerAtIndex(owners[i], _getMultiOwnableStorage().nextOwnerIndex++);
173:        }
174:    }
```

```diff
diff --git a/src/SmartWallet/MultiOwnable.sol b/src/SmartWallet/MultiOwnable.sol
index 64fd19a..da8ad64 100644
--- a/src/SmartWallet/MultiOwnable.sol
+++ b/src/SmartWallet/MultiOwnable.sol
@@ -160,16 +160,20 @@ contract MultiOwnable {
     ///
     /// @param owners The intiial list of owners to register.
     function _initializeOwners(bytes[] memory owners) internal virtual {
+        bytes memory owner;
+        uint256 ownerLength;
         for (uint256 i; i < owners.length; i++) {
-            if (owners[i].length != 32 && owners[i].length != 64) {
+            owner = owners[i];
+            ownerLength = owner.length;
+            if (ownerLength != 32 && ownerLength != 64) {
                 revert InvalidOwnerBytesLength(owners[i]);
             }

-            if (owners[i].length == 32 && uint256(bytes32(owners[i])) > type(uint160).max) {
-                revert InvalidEthereumAddressOwner(owners[i]);
+            if (ownerLength == 32 && uint256(bytes32(owner)) > type(uint160).max) {
+                revert InvalidEthereumAddressOwner(owner);
             }

-            _addOwnerAtIndex(owners[i], _getMultiOwnableStorage().nextOwnerIndex++);
+            _addOwnerAtIndex(owner, _getMultiOwnableStorage().nextOwnerIndex++);
         }
     }
```


2. #### Cache `calls[i]`
- https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWallet.sol#L207


```solidity
file: src/SmartWallet/CoinbaseSmartWallet.sol

205:    function executeBatch(Call[] calldata calls) public payable virtual onlyEntryPointOrOwner {
206:        for (uint256 i; i < calls.length;) {
207:            _call(calls[i].target, calls[i].value, calls[i].data);
208:            unchecked {
209:                ++i;
210:            }
211:        }
212:    }
```

```diff
diff --git a/src/SmartWallet/CoinbaseSmartWallet.sol b/src/SmartWallet/CoinbaseSmartWallet.sol
index 2278a5a..124fb34 100644
--- a/src/SmartWallet/CoinbaseSmartWallet.sol
+++ b/src/SmartWallet/CoinbaseSmartWallet.sol
@@ -203,8 +203,10 @@ contract CoinbaseSmartWallet is MultiOwnable, UUPSUpgradeable, Receiver, ERC1271
     ///
     /// @param calls The list of `Call`s to execute.
     function executeBatch(Call[] calldata calls) public payable virtual onlyEntryPointOrOwner {
+        Call calldata call;
         for (uint256 i; i < calls.length;) {
-            _call(calls[i].target, calls[i].value, calls[i].data);
+            call = calls[i];
+            _call(call.target, call.value, call.data);
             unchecked {
                 ++i;
             }
```




## [G-05]  Use custom error instead of assert
In using the assert() function, when the conditions results to false all the changes made to the contract would be reverted all the remaining gas of the transaction would be consumed.
Meanwhile, in using an if revert custom error, when the condition in the if-statement results to true would revert all the changes made to the contract and does a refund all the remaining gas fees of the transaction.

### Instances
1. #### Refactor `assert(mode != PostOpMode.postOpReverted)` to `if (amount > balance) revert InsufficientAmount()`.
- https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L150

```solidity
file: src/MagicSpend/MagicSpend.sol

143:    function postOp(IPaymaster.PostOpMode mode, bytes calldata context, uint256 actualGasCost)
144:        external
145:        onlyEntryPoint
146:    {
147:        // `PostOpMode.postOpReverted` should be impossible.
148:        // Only possible cause would be if this contract does not own enough ETH to transfer
149:        // but this is checked at the validation step.
150:        assert(mode != PostOpMode.postOpReverted);
151:
152:        (uint256 maxGasCost, address account) = abi.decode(context, (uint256, address));
153:
154:        // Compute the total remaining funds available for the user accout.
155:        // NOTE: Take into account the user operation gas that was not consummed.
156:        uint256 withdrawable = _withdrawableETH[account] + (maxGasCost - actualGasCost);
157:
158:        // Send the all remaining funds to the user accout.
159:        delete _withdrawableETH[account];
160:        if (withdrawable > 0) {
161:            SafeTransferLib.forceSafeTransferETH(account, withdrawable, SafeTransferLib.GAS_STIPEND_NO_STORAGE_WRITES);
162:        }
163:    }
```

```diff
diff --git a/src/MagicSpend/MagicSpend.sol b/src/MagicSpend/MagicSpend.sol
index be2bed1..022b1ba 100644
--- a/src/MagicSpend/MagicSpend.sol
+++ b/src/MagicSpend/MagicSpend.sol
@@ -147,7 +147,7 @@ contract MagicSpend is Ownable, IPaymaster {
         // `PostOpMode.postOpReverted` should be impossible.
         // Only possible cause would be if this contract does not own enough ETH to transfer
         // but this is checked at the validation step.
-        assert(mode != PostOpMode.postOpReverted);
+        if (mode == PostOpMode.postOpReverted) revert UnexpectedPostOpRevertedMode();

         (uint256 maxGasCost, address account) = abi.decode(context, (uint256, address));
```





## [G-06]  Avoid zero transfers
In Solidity, performing unnecessary operations can consume more gas than needed, leading to cost inefficiencies. For instance, if a transfer function doesn't have a zero amount check and someone calls it with a zero amount, unnecessary gas will be consumed in executing the function, even though the state of the contract remains the same. By implementing a zero amount check, such unnecessary function calls can be avoided, thereby saving gas and making the contract more efficient. Amounts should be checked for 0 before calling a transfer
Checking non-zero transfer values can avoid an expensive external call and save gas.

#### Please note this instance was not included in the bots reports


### 1 Instance
1. #### Refactor `MagicSpend.entryPointDeposit()` to avoid zero amount transfers
- https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L213

```solidity
file: src/MagicSpend/MagicSpend.sol

212:    function entryPointDeposit(uint256 amount) external payable onlyOwner {
213:        SafeTransferLib.safeTransferETH(entryPoint(), amount);
214:    }
```

```diff
diff --git a/src/MagicSpend/MagicSpend.sol b/src/MagicSpend/MagicSpend.sol
index be2bed1..799c360 100644
--- a/src/MagicSpend/MagicSpend.sol
+++ b/src/MagicSpend/MagicSpend.sol
@@ -210,7 +210,9 @@ contract MagicSpend is Ownable, IPaymaster {
     ///
     /// @param amount The amount to deposit on the the Entrypoint.
     function entryPointDeposit(uint256 amount) external payable onlyOwner {
-        SafeTransferLib.safeTransferETH(entryPoint(), amount);
+        if (amount > 0){
+            SafeTransferLib.safeTransferETH(entryPoint(), amount);
+        }
     }
```






## CONCLUSION
As you embark on incorporating the recommended optimizations, we want to emphasize the utmost importance of proceeding with vigilance and dedicating thorough efforts to comprehensive testing. It is of paramount significance to ensure that the proposed alterations do not inadvertently introduce fresh vulnerabilities, while also successfully achieving the anticipated enhancements in performance.

We strongly advise conducting a meticulous and exhaustive evaluation of the modifications made to the codebase. This rigorous scrutiny and exhaustive assessment will play a pivotal role in affirming both the security and efficacy of the refactored code. Your careful attention to detail, coupled with the implementation of a robust testing framework, will provide the necessary assurance that the refined code aligns with your security objectives and effectively fulfills the intended performance optimizations.
