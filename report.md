---
sponsor: "Coinbase"
slug: "2024-03-coinbase"
date: "2024-05-01"
title: "Smart Wallet"
findings: "https://github.com/code-423n4/2024-03-coinbase-findings/issues"
contest: 350
---

# Overview

## About C4

Code4rena (C4) is an open organization consisting of security researchers, auditors, developers, and individuals with domain expertise in smart contracts.

A C4 audit is an event in which community participants, referred to as Wardens, review, audit, or analyze smart contract logic in exchange for a bounty provided by sponsoring projects.

During the audit outlined in this document, C4 conducted an analysis of the Smart Wallet smart contract system written in Solidity. The audit took place between March 14 — March 21, 2024.

Following the C4 audit, 3 wardens ([McToady](https://code4rena.com/@McToady), [imare](https://code4rena.com/@imare), and [cheatc0d3](https://code4rena.com/@cheatc0d3)) reviewed the mitigations for all identified issues; the [mitigation review report](#mitigation-review) is appended below the audit report.

## Wardens

53 Wardens contributed reports to Coinbase Smart Wallet:

  1. [Circolors](https://code4rena.com/@Circolors) ([McToady](https://code4rena.com/@McToady) and [irreverent](https://code4rena.com/@irreverent))
  2. [lsaudit](https://code4rena.com/@lsaudit)
  3. [doublespending](https://code4rena.com/@doublespending)
  4. [Jorgect](https://code4rena.com/@Jorgect)
  5. [imare](https://code4rena.com/@imare)
  6. [cheatc0d3](https://code4rena.com/@cheatc0d3)
  7. [roguereggiant](https://code4rena.com/@roguereggiant)
  8. [0xepley](https://code4rena.com/@0xepley)
  9. [popeye](https://code4rena.com/@popeye)
  10. [emerald7017](https://code4rena.com/@emerald7017)
  11. [K42](https://code4rena.com/@K42)
  12. [albahaca](https://code4rena.com/@albahaca)
  13. [0x11singh99](https://code4rena.com/@0x11singh99)
  14. [naman1778](https://code4rena.com/@naman1778)
  15. [0xAnah](https://code4rena.com/@0xAnah)
  16. [foxb868](https://code4rena.com/@foxb868)
  17. [0xbrett8571](https://code4rena.com/@0xbrett8571)
  18. [0xmystery](https://code4rena.com/@0xmystery)
  19. [d3e4](https://code4rena.com/@d3e4)
  20. [Limbooo](https://code4rena.com/@Limbooo)
  21. [0xhacksmithh](https://code4rena.com/@0xhacksmithh)
  22. [Koala](https://code4rena.com/@Koala)
  23. [shealtielanz](https://code4rena.com/@shealtielanz)
  24. [7ashraf](https://code4rena.com/@7ashraf)
  25. [Bigsam](https://code4rena.com/@Bigsam)
  26. [IceBear](https://code4rena.com/@IceBear)
  27. [Tigerfrake](https://code4rena.com/@Tigerfrake)
  28. [ZanyBonzy](https://code4rena.com/@ZanyBonzy)
  29. [gpersoon](https://code4rena.com/@gpersoon)
  30. [y4y](https://code4rena.com/@y4y)
  31. [hunter\_w3b](https://code4rena.com/@hunter_w3b)
  32. [SAQ](https://code4rena.com/@SAQ)
  33. [clara](https://code4rena.com/@clara)
  34. [SBSecurity](https://code4rena.com/@SBSecurity) ([Slavcheww](https://code4rena.com/@Slavcheww) and [Blckhv](https://code4rena.com/@Blckhv))
  35. [MSK](https://code4rena.com/@MSK)
  36. [JCK](https://code4rena.com/@JCK)
  37. [fouzantanveer](https://code4rena.com/@fouzantanveer)
  38. [kaveyjoe](https://code4rena.com/@kaveyjoe)
  39. [unique](https://code4rena.com/@unique)
  40. [LinKenji](https://code4rena.com/@LinKenji)
  41. [JcFichtner](https://code4rena.com/@JcFichtner)
  42. [Myd](https://code4rena.com/@Myd)
  43. [SM3\_SS](https://code4rena.com/@SM3_SS)
  44. [shamsulhaq123](https://code4rena.com/@shamsulhaq123)
  45. [slvDev](https://code4rena.com/@slvDev)
  46. [dharma09](https://code4rena.com/@dharma09)
  47. [Hajime](https://code4rena.com/@Hajime)
  48. [robriks](https://code4rena.com/@robriks)
  49. [cryptphi](https://code4rena.com/@cryptphi)
  50. [aycozynfada](https://code4rena.com/@aycozynfada)
  51. [jesjupyter](https://code4rena.com/@jesjupyter)

This audit was judged by [3docSec](https://twitter.com/judge).

Final report assembled by PaperParachute and [liveactionllama](https://twitter.com/liveactionllama).

# Summary

The C4 analysis yielded an aggregated total of 3 unique vulnerabilities. Of these vulnerabilities, 1 received a risk rating in the category of HIGH severity and 2 received a risk rating in the category of MEDIUM severity.

Additionally, C4 analysis included 26 reports detailing issues with a risk rating of LOW severity or non-critical. There were also 13 reports recommending gas optimizations.

All of the issues presented here are linked back to their original finding.

# Scope

The code under review can be found within the [C4 Smart Wallet repository](https://github.com/code-423n4/2024-03-coinbase), and is composed of 7 smart contracts written in the Solidity programming language and includes 786 lines of Solidity code.

In addition to the known issues identified by the project team, a Code4rena bot race was conducted at the start of the audit. The winning bot, **Pechenkata** from wardens [bozho](https://code4rena.com/@bozho) and [radev_sw](https://code4rena.com/@radev_sw), generated the [Automated Findings report](https://github.com/code-423n4/2024-03-coinbase/blob/main/bot-report.md) and all findings therein were classified as out of scope.

# Severity Criteria

C4 assesses the severity of disclosed vulnerabilities based on three primary risk categories: high, medium, and low/non-critical.

High-level considerations for vulnerabilities span the following key areas when conducting assessments:

- Malicious Input Handling
- Escalation of privileges
- Arithmetic
- Gas use

For more information regarding the severity criteria referenced throughout the submission review process, please refer to the documentation provided on [the C4 website](https://code4rena.com), specifically our section on [Severity Categorization](https://docs.code4rena.com/awarding/judging-criteria/severity-categorization).

# High Risk Findings (1)
## [[H-01] Remove owner calls can be replayed to remove a different owner at the same index, leading to severe issues when combined with lack of last owner guard](https://github.com/code-423n4/2024-03-coinbase-findings/issues/114)
*Submitted by [Circolors](https://github.com/code-423n4/2024-03-coinbase-findings/issues/114), also found by [lsaudit](https://github.com/code-423n4/2024-03-coinbase-findings/issues/88)*

Users are able to upgrade their account's owners via either directly onto the contract with a regular transaction or via an ERC-4337 EntryPoint transaction calling `executeWithoutChainIdValidation`. If a user chooses to use a combination of these methods it's very likely that the addresses at a particular ownership index differ across chain. Therefore if a user later calls `removeOwnerAtIndex` on another chain will end up removing different addresses on different chains. It is unlikely this would be the user's intention. The severity of this ranges from minimal (the user can just add the mistakenly removed owner back) or critical (the user mistakenly removes their only accessible owner on a specific chain, permanently locking the account).

### Proof Of Concept

Scenario A: Alice permanently bricks her account on an unused chain:

1.  Alice has a CoinbaseSmartWallet, and uses it on Base, Ethereum & Optimism
2.  Alice later decides to add a new owner using a cross-chain `executeWithoutChainIdValidation`
3.  Alice later wants to remove the initial owner (index 0) and does so by signing another cross-chain replayable signature
4.  Despite it not being her intention anyone could take that signature and replay it on Arbitrum, Avalanche, etc as there is no check to stop the user from removing the final owner

Scenario A: Alice adds owners using both methods and ends up with undesired results:

1.  Alice has a CoinbaseSmartWallet and uses it across all chains
2.  She has Gnosis Safe's and ERC-6551 token bound accounts on different chains so adds them as owners on those specific chains using `execute`
3.  She then adds a secondary EOA address on all chains using `executeWithoutChainIdValidation`
4.  Now if she uses `executeWithoutChainIdValidation` to call `removeOwnerAtInde` she will be removing different owners on different chains, which is likely not her intention

While more complex scenarios than this might sound bizarre it's important to remember that Alice could be using this smart account for the next N years, only making changes sporadically, and as her ownership mappings across different chains become more out of sync the likelihood of a significant error occurring increases.

### Recommended Mitigation Steps

As `MultiOwnableStorage` uses a mapping to track owner information rather than a conventional array, it might be simpler to do away with the indexes entirely and have a `removeOwner(bytes calldata _ownerToRemove)` function. This would avoid the situations outlined above where when calling `removeOwnerAtIndex` removes different owners on different chains. To ensure replayability and avoid having a stuck nonce on chains where `_ownerToRemove` is not an owner the function should not revert in the case the owner is not there, but instead return a bool `removed` to indicate whether an owner was removed or not.

This would make it significantly less likely that users run into the issues stated above, without having to limit their freedom to make ownership changes manually or via ERC-4337 EntryPoint transactions.

**[3docSec (judge) increased severity to High and commented](https://github.com/code-423n4/2024-03-coinbase-findings/issues/114#issuecomment-2022115164):**
 > Root cause: `removeOwnerAtIndex` can be replayed cross-chain despite the same index may point to a different owner. The issue was confirmed by the sponsor in the [issue #57](https://github.com/code-423n4/2024-03-coinbase-findings/issues/57) thread and addressed [in this PR](https://github.com/coinbase/smart-wallet/pull/42).
 >
 > After consulting with a fellow judge, I'm upgrading this one as high, as there is a well-defined attack path that causes the user to lose ownership of their wallet.

**[Coinbase mitigated](https://github.com/code-423n4/2024-04-coinbase-mitigation/blob/main/README.md#scope):**
> The issue is remediated by updating the parameterization of `removeOwnerAtIndex` to also take an `owner` argument. We then check that the `owner` passed matches the owner found at the index. In this way, we prevent a replayable transaction removing a different owner at the same index. (PR [here](https://github.com/coinbase/smart-wallet/pull/42))

**Status:** Mitigation confirmed. Full details in reports from [McToady](https://github.com/code-423n4/2024-04-coinbase-mitigation-findings/issues/6), [cheatc0d3](https://github.com/code-423n4/2024-04-coinbase-mitigation-findings/issues/13), and [imare](https://github.com/code-423n4/2024-04-coinbase-mitigation-findings/issues/8).



***
 
# Medium Risk Findings (2)
## [[M-01] Balance check during `MagicSpend` validation cannot ensure that `MagicSpend` has enough balance to cover the requested fund](https://github.com/code-423n4/2024-03-coinbase-findings/issues/110)
*Submitted by [doublespending](https://github.com/code-423n4/2024-03-coinbase-findings/issues/110), also found by [imare](https://github.com/code-423n4/2024-03-coinbase-findings/issues/133)*

[Balance check](https://github.com/code-423n4/2024-03-coinbase/blob/e0573369b865d47fed778de00a7b6df65ab1744e/src/MagicSpend/MagicSpend.sol#L130-L135) during `MagicSpend` validation cannot ensure that `MagicSpend` has enough balance to cover the requested fund.

```solidity
        // Ensure at validation that the contract has enough balance to cover the requested funds.
        // NOTE: This check is necessary to enforce that the contract will be able to transfer the remaining funds
        //       when `postOp()` is called back after the `UserOperation` has been executed.
        if (address(this).balance < withdrawAmount) {
            revert InsufficientBalance(withdrawAmount, address(this).balance);
        }
```

### Proof of Concept

1.  EntryPoint executes userOps by two loops: [validation loop](https://github.com/eth-infinitism/account-abstraction/blob/abff2aca61a8f0934e533d0d352978055fddbd96/contracts/core/EntryPoint.sol#L98-L102) and [execution loop](https://github.com/eth-infinitism/account-abstraction/blob/abff2aca61a8f0934e533d0d352978055fddbd96/contracts/core/EntryPoint.sol#L107-L109).
2.  Suppose we have two userOps using `MagicSpend`:
    *   userOp1 with `withdrawAmount` 100
    *   userOp2 with `withdrawAmount` 150

3.  userOp1 and userOp2 are packed together as `UserOperation[]` [ops](https://github.com/eth-infinitism/account-abstraction/blob/abff2aca61a8f0934e533d0d352978055fddbd96/contracts/core/EntryPoint.sol#L92) and ops will be executed by entryPoint through [the `handleOps` method](https://github.com/eth-infinitism/account-abstraction/blob/abff2aca61a8f0934e533d0d352978055fddbd96/contracts/core/EntryPoint.sol#L92)
4.  Suppose the balance of MagicSpend is 200.
5.  During the validation loop:
    *   `validatePaymasterUserOp` will be called during userOp1 validation and pass for  address(this).balance (=200) is larger than withdrawAmount (=100)
    *   `validatePaymasterUserOp` will be called during userOp1 validation and pass for  address(this).balance (=200) is larger than withdrawAmount (=150)

```solidity
        // https://github.com/eth-infinitism/account-abstraction/blob/abff2aca61a8f0934e533d0d352978055fddbd96/contracts/core/EntryPoint.sol#L98C1-L102C10
        for (uint256 i = 0; i < opslen; i++) {
            UserOpInfo memory opInfo = opInfos[i];
            (uint256 validationData, uint256 pmValidationData) = _validatePrepayment(i, ops[i], opInfo);
            _validateAccountAndPaymasterValidationData(i, validationData, pmValidationData, address(0));
        }
```

6.  During the execution loop:
    *   The balance of MagicSpend is 200
    *   The request fund of userOp1 and userOp2 is 100+150=250 > 200

```solidity
        // https://github.com/eth-infinitism/account-abstraction/blob/abff2aca61a8f0934e533d0d352978055fddbd96/contracts/core/EntryPoint.sol#L107-L109
        for (uint256 i = 0; i < opslen; i++) {
            collected += _executeUserOp(i, ops[i], opInfos[i]);
        }
```

*   After executing `postOp` of userOp1, the balance of MagicSpend is 100.

```solidity
        // https://github.com/code-423n4/2024-03-coinbase/blob/e0573369b865d47fed778de00a7b6df65ab1744e/src/MagicSpend/MagicSpend.sol#L160C1-L162C10
        if (withdrawable > 0) {
            SafeTransferLib.forceSafeTransferETH(account, withdrawable, SafeTransferLib.GAS_STIPEND_NO_STORAGE_WRITES);
        }
```

*   When executing `postOp` of userOp2, MagicSpend does not have enough balance to cover the requested fund (=150) of userOp2 and the execution will fail.

```solidity
        // https://github.com/code-423n4/2024-03-coinbase/blob/e0573369b865d47fed778de00a7b6df65ab1744e/src/MagicSpend/MagicSpend.sol#L160C1-L162C10
        if (withdrawable > 0) {
            SafeTransferLib.forceSafeTransferETH(account, withdrawable, SafeTransferLib.GAS_STIPEND_NO_STORAGE_WRITES);
        }
```

*   So, a balance check during MagicSpend validation cannot ensure that MagicSpend has enough balance to cover the requested fund.

```solidity
        // https://github.com/code-423n4/2024-03-coinbase/blob/e0573369b865d47fed778de00a7b6df65ab1744e/src/MagicSpend/MagicSpend.sol#L130-L135
        // Ensure at validation that the contract has enough balance to cover the requested funds.
        // NOTE: This check is necessary to enforce that the contract will be able to transfer the remaining funds
        //       when `postOp()` is called back after the `UserOperation` has been executed.
        if (address(this).balance < withdrawAmount) {
            revert InsufficientBalance(withdrawAmount, address(this).balance);
        }
```

### Recommended Mitigation Steps

Use a state instead of `address(this).balance` to record the remaining balance.

```solidity
        if (remain < withdrawAmount) {
            revert InsufficientBalance(withdrawAmount, address(this).balance);
        }
        remain -= withdrawAmount;

```

**[raymondfam (lookout) commented](https://github.com/code-423n4/2024-03-coinbase-findings/issues/110#issuecomment-2015397482):**
 > This would happen only when the entrypoint batches the transactions. Additionally, this is kind of related to the known issue from the readme: When acting as a paymaster, EntryPoint will debit MagicSpend slightly more than actualGasCost, meaning what is withheld on a gas-paying withdrawal will not cover 100\% of MagicSpend's balance decrease in the EntryPoint.

**[wilsoncusack (Coinbase) confirmed and commented](https://github.com/code-423n4/2024-03-coinbase-findings/issues/110#issuecomment-2020511843):**
 > Ah right I feel like we discussed this @xenoliss and then perhaps forgot in a later conversation. The only way to avoid this would be to have validation keep some tally of all the expected coming withdraws.

**[3docSec (judge) commented](https://github.com/code-423n4/2024-03-coinbase-findings/issues/110#issuecomment-2022193681):**
 > To be fair, one could argue that the bundlers should simulate the execution of their bundles before submission to avoid reverts; however, this is a valid way of grieving the reputation of the MagicSpend contract.

**[wilsoncusack (Coinbase) commented](https://github.com/code-423n4/2024-03-coinbase-findings/issues/110#issuecomment-2022736533):**
> @3docSec - it is correct that the bundler would be expected to see this and revert the whole bundle. However, the point is valid that the guard is not fully satisfactory as it is written. It should probably be removed or fixed.

**[Coinbase mitigated](https://github.com/code-423n4/2024-04-coinbase-mitigation/blob/main/README.md#scope):**
> This issue is complex to address. The warden suggested adding a variable to track in flight withdraws, and we [pursued this](https://github.com/coinbase/magic-spend/pull/16). However, we realized that bundlers penalize paymasters when the UserOp behaves differently when simulated in isolation vs. in the bundle, and this would not fix this. Instead, we give the owner a tool to address this probabilistically: the owner can set a `maxWithdrawDenominator` and we enforce that native asset withdraws must be `<= address(this).balance / maxWithdrawDenominator`. For example, if `maxWithdrawDenominator` is set to 20, it would take 20 native asset withdraws (each withdrawing max allowed) + 1 native asset withdraw in the same transaction to cause a revert. It is of course known that this doesn't entirely solve the issue, and the efficacy depends the value chosen and usage. (PR [here](https://github.com/coinbase/magic-spend/pull/17))

**Status:** Mitigation confirmed. Full details in reports from [imare](https://github.com/code-423n4/2024-04-coinbase-mitigation-findings/issues/7) and [McToady](https://github.com/code-423n4/2024-04-coinbase-mitigation-findings/issues/5).



***

## [[M-02] Users can front run the signature of the paymaster operation leading to some problems](https://github.com/code-423n4/2024-03-coinbase-findings/issues/39)
*Submitted by [Jorgect](https://github.com/code-423n4/2024-03-coinbase-findings/issues/39), also found by [cheatc0d3](https://github.com/code-423n4/2024-03-coinbase-findings/issues/164)*

The paymaster is an extension of the eip-4337, normally the paymaster is willing to pay a user transaction if the account can return the amount of gas at the final of the transaction.

In the context of the coinbase smart wallet, the paymaster is the contract call magicSpend.sol, This contract exposes the normal function needed to be a paymaster:

    function validatePaymasterUserOp
        (PackedUserOperation calldata userOp, bytes32 userOpHash, uint256 maxCost)
        external returns (bytes memory context, uint256 validationData);

    function postOp
        (PostOpMode mode, bytes calldata context, uint256 actualGasCost, uint256 actualUserOpFeePerGas)
        external;

The magic spend is also implementing the entry point deposit, unlock and withdraw functions as required.

Addionally of this the magicSpend is implementing a withdraw functions for users:

    file:https://src/MagicSpend/MagicSpend.sol#L181
     function withdraw(WithdrawRequest memory withdrawRequest) external {
            _validateRequest(msg.sender, withdrawRequest);

            if (!isValidWithdrawSignature(msg.sender, withdrawRequest)) {
                revert InvalidSignature();
            }

            if (block.timestamp > withdrawRequest.expiry) {
                revert Expired();
            }

            // reserve funds for gas, will credit user with difference in post op
            _withdraw(withdrawRequest.asset, msg.sender, withdrawRequest.amount);
        }

[Link](https://github.com/code-423n4/2024-03-coinbase/blob/e0573369b865d47fed778de00a7b6df65ab1744e/src/MagicSpend/MagicSpend.sol#L181)

The problem is that [validatePaymasterUserOp](https://github.com/code-423n4/2024-03-coinbase/blob/e0573369b865d47fed778de00a7b6df65ab1744e/src/MagicSpend/MagicSpend.sol#L109) is consuming the same signature of the withdraw function, so user can request a transaction through the paymaster, then front runt this transaction calling the withdraw function in the magicSpend (as you notice this transaction is not being processed through the bundler so user can get this withdraw transaction first if he send the correct amount of gas to be included first) making the validatePaymasterUserOp revert because the nonce was already consumed.

### Impact

`Are there any griefing attacks that could cause this paymaster to be banned by bundlers?`

1.  Paymaster can be banned by bundlers because the user can trigger revert transactions which is one of the reason because bundlers can ban paymasters.

I consider that this has to be another vulnerability, but I decided to put it here because the main problem is the same.

2.  Paymaster can be drained if user front runs the signature given to pay an operation, withdrawing directly funds in the withdraw function, user can do this repeated times until the Paymaster is completely drained.

### Proof of Concept

Run this test in `file:/test/MagicSpend/Withdraw.t.sol` :

```

    function test_frontRuning() public {  //@audit POC HIGH 1
        MagicSpend.WithdrawRequest memory WithdrawRequest = MagicSpend.WithdrawRequest({
            asset: asset,
            amount: amount, 
            nonce: nonce,
            expiry: expiry,
            signature: _getSignature()
        });

        UserOperation memory userOp = UserOperation({
            sender: withdrawer,
            nonce: 0,
            initCode: "",
            callData: "",
            callGasLimit: 49152,
            verificationGasLimit: 378989,
            preVerificationGas: 273196043,
            maxFeePerGas: 1000304,
            maxPriorityFeePerGas: 1000000,
            paymasterAndData: abi.encodePacked(address(magic), abi.encode(WithdrawRequest)),
            signature: ""
        });

        magic.withdraw(WithdrawRequest); //Front runing the validatePaymasterUserOp operation

        bytes32 userOpHash = sha256("hi");
        uint256 maxCost = amount - 10;

        vm.expectRevert();
        magic.validatePaymasterUserOp(userOp, userOpHash, maxCost);
    }
```

### Tools Used

Foundry

### Recommended Mitigation Steps

Consider adding another signer for the withdraw function different from the validatePaymasterUserOp signer.

**[wilsoncusack (Coinbase) acknowledged, but disagreed with severity and commented](https://github.com/code-423n4/2024-03-coinbase-findings/issues/39#issuecomment-2020737885):**
 > To be clear, the same signature cannot be used twice. The front run is interesting: requires a user to submit a userOp with the withdraw signature in `paymasterAndData`, and then call from their SCW (via another user op or direct call from an EOA owner) and directly call withdraw. There's a race condition here, but it could indeed hurt the paymaster's reputation if pulled off. 

**[3docSec (judge) decreased severity to Medium and commented](https://github.com/code-423n4/2024-03-coinbase-findings/issues/39#issuecomment-2022177047):**
 > Because the front-run signature would DoS the second one, tokens would be spent only once (invalidating point #2 of the impact), so it looks more like a grieving attack on the MS reputation with no tokens at risk => medium seems more appropriate.



***

# Low Risk and Non-Critical Issues

For this audit, 26 reports were submitted by wardens detailing low risk and non-critical issues. The [report highlighted below](https://github.com/code-423n4/2024-03-coinbase-findings/issues/65) by **0xmystery** received the top score from the judge.

*The following wardens also submitted reports: [d3e4](https://github.com/code-423n4/2024-03-coinbase-findings/issues/183), [Limbooo](https://github.com/code-423n4/2024-03-coinbase-findings/issues/181), [0xhacksmithh](https://github.com/code-423n4/2024-03-coinbase-findings/issues/178), [imare](https://github.com/code-423n4/2024-03-coinbase-findings/issues/135), [Circolors](https://github.com/code-423n4/2024-03-coinbase-findings/issues/117), [Koala](https://github.com/code-423n4/2024-03-coinbase-findings/issues/115), [doublespending](https://github.com/code-423n4/2024-03-coinbase-findings/issues/108), [shealtielanz](https://github.com/code-423n4/2024-03-coinbase-findings/issues/98), [7ashraf](https://github.com/code-423n4/2024-03-coinbase-findings/issues/91), [lsaudit](https://github.com/code-423n4/2024-03-coinbase-findings/issues/87), [Bigsam](https://github.com/code-423n4/2024-03-coinbase-findings/issues/68), [IceBear](https://github.com/code-423n4/2024-03-coinbase-findings/issues/67), [Tigerfrake](https://github.com/code-423n4/2024-03-coinbase-findings/issues/64), [ZanyBonzy](https://github.com/code-423n4/2024-03-coinbase-findings/issues/61), [gpersoon](https://github.com/code-423n4/2024-03-coinbase-findings/issues/51), [foxb868](https://github.com/code-423n4/2024-03-coinbase-findings/issues/33), [0xbrett8571](https://github.com/code-423n4/2024-03-coinbase-findings/issues/23), [y4y](https://github.com/code-423n4/2024-03-coinbase-findings/issues/18), [cheatc0d3](https://github.com/code-423n4/2024-03-coinbase-findings/issues/198), [robriks](https://github.com/code-423n4/2024-03-coinbase-findings/issues/184), [cryptphi](https://github.com/code-423n4/2024-03-coinbase-findings/issues/170), [aycozynfada](https://github.com/code-423n4/2024-03-coinbase-findings/issues/162), [SBSecurity](https://github.com/code-423n4/2024-03-coinbase-findings/issues/147), [jesjupyter](https://github.com/code-423n4/2024-03-coinbase-findings/issues/74), and [Jorgect](https://github.com/code-423n4/2024-03-coinbase-findings/issues/46).*

## [L-01] NatSpec comment on 255 owner limitation in `_addOwner()` being ambiguous
The discrepancy between the NatSpec comment in the `_addOwner` function of the MultiOwnable contract and its actual implementation could lead to misunderstandings about the contract's owner limit. The comment suggests a limit of 255 owners:

https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/MultiOwnable.sol#L176

```solidity
    /// @notice Convenience function used to add the first 255 owners.
```
But the code, utilizing `uint256` for indexing, does not enforce any such limit, allowing for a significantly larger number of owners. 

https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/MultiOwnable.sol#L8-L9

```solidity
    /// @dev Tracks the index of the next owner to add.
    uint256 nextOwnerIndex;
```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/MultiOwnable.sol#L179-L181

```solidity
    function _addOwner(bytes memory owner) internal virtual {
        _addOwnerAtIndex(owner, _getMultiOwnableStorage().nextOwnerIndex++);
    }
```
This issue, while not affecting the contract's functionality or security directly, could cause confusion among developers and users, leading to potential misalignments in expectations and implementations. Addressing this through clarification in the documentation or by introducing an explicit limit, if desired, is recommended to ensure clear and accurate communication of the contract's capabilities. 

```diff
    function _addOwner(bytes memory owner) internal virtual {
+        require(_getMultiOwnableStorage().nextOwnerIndex < 255, "Owner limit reached");
        _addOwnerAtIndex(owner, _getMultiOwnableStorage().nextOwnerIndex++);
    }
```

## [L-02] Risks associated with forceful ETH transfers to unprepared contracts
Forcing ETH transfers to contracts that are not designed to receive them, due to the absence of a `payable fallback` or `receive` function, can lead to permanently locked funds within these recipient contracts. This situation, facilitated by low-level operations like `selfdestruct`, presents a significant severity issue that undermines the predictability and safety of smart contract interactions in the Ethereum ecosystem. To mitigate such risks, it is recommended that sending contracts verify the recipient's ability to handle ETH in a standard manner and that developers design contracts with interoperability and safe ETH handling in mind, ensuring the broader reliability and trustworthiness of the ecosystem.

https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L158-L162

```solidity
        // Send the all remaining funds to the user accout.
        delete _withdrawableETH[account];
        if (withdrawable > 0) {
            SafeTransferLib.forceSafeTransferETH(account, withdrawable, SafeTransferLib.GAS_STIPEND_NO_STORAGE_WRITES);
        }
```

## [L-03] Limitations in fund withdrawal for non-gas and multi-asset transactions
The contract `MagicSpend` presents two primary limitations impacting its flexibility and utility in broader transaction contexts. First, there's a procedural limitation where, after the invocation of `validatePaymasterUserOp()` by the EntryPoint, the subsequent need for non-gas related fund withdrawals (e.g., for swaps or mints) within the same transaction is constrained to using `withdrawGasExcess()`. Attempting to invoke `withdraw()` would result in failure due to the nonce already being marked as used, restricting access to additional funds beyond the initial gas validation context. Second, there's a functional limitation where `withdrawGasExcess()` is strictly designed for ETH withdrawals, lacking the capability to handle ERC-20 tokens. This is in contrast to `withdraw()`, which is designed with the flexibility to manage both ETH and ERC-20 token withdrawals. Together, these limitations could significantly impact the contract's applicability in complex transaction scenarios requiring multi-asset interactions and nuanced fund management.

1. A user operation requires validation for gas coverage via [validatePaymasterUserOp()](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L109-L140), which also marks the operation's nonce as used.
2. Post-validation, the operation requires additional funds for non-gas related activities (e.g., token swaps). However, due to the nonce already being marked as used, [withdraw()](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L178-L194) cannot be invoked for this purpose.
3. Furthermore, should these additional activities require assets other than ETH, the contract's design restricts the user to [withdrawGasExcess()](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L165-L176), which only supports ETH, thereby limiting multi-asset transaction capabilities within the same operation context.

Consider the following fix:<br>
Enhance `withdrawGasExcess()` to support ERC-20 tokens in addition to ETH. This could involve introducing additional parameters or overloading the function to handle different asset types, ensuring broader applicability in multi-asset transaction contexts.

## [L-04] Missing check for passkey associated with `CoinbaseSmartWallet._validateSignature()`
Neither passkey nor an address goes through checks as implemented in [MultiOwnable._initializeOwners()](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/MultiOwnable.sol#L162-L174) when added through `addOwnerAddress()` and `addOwnerPublicKey()` that have `_addOwner()` invoked,

https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/MultiOwnable.sol#L179-L181

```solidity
    function _addOwner(bytes memory owner) internal virtual {
        _addOwnerAtIndex(owner, _getMultiOwnableStorage().nextOwnerIndex++);
    }
```
When `CoinbaseSmartWallet._validateSignature()` is called, a needed check is performed when `ownerBytes.length == 32` but not so when `ownerBytes.length == 64`. Consider adding the needed check for a consistent earlier revert as follows:

https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWallet.sol#L316-L324

```diff
-        if (ownerBytes.length == 64) {
+        if (owners[i].length != 64) {
+                revert InvalidOwnerBytesLength(owners[i]);
+            }
            (uint256 x, uint256 y) = abi.decode(ownerBytes, (uint256, uint256));

            WebAuthn.WebAuthnAuth memory auth = abi.decode(sigWrapper.signatureData, (WebAuthn.WebAuthnAuth));

            return WebAuthn.verify({challenge: abi.encode(message), requireUV: false, webAuthnAuth: auth, x: x, y: y});
-        }

-        revert InvalidOwnerBytesLength(ownerBytes);
```

## [L-05] Possible deployment and functional failure of Contracts on L2s due to Dencun Opcodes
With the protocol intending to operate on various L2 chains as is inferred from [executeWithoutChainIdValidation()](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWallet.sol#L170-L187), it may encounter deployment and operational challenges if utilizing any of the new opcodes that enhance Ethereum's functionality, such as those related to shard blob transactions (EIP-4844) and others as described in the link below:

https://www.coinlive.com/news/ethereum-dencun-hard-fork-content-introduction

Devoid of `Dencun upgrade`, contracts relying on the new opcodes will likely fail because the Ethereum Virtual Machine (EVM) on these L2s would not recognize or know how to execute the new instructions, leading to reverts or other unexpected behaviors.  

As of todate, Optimism, Arbitrum, and Base have implemented the Dencun upgrade. Consider implementing conditional logic in contracts or hold off deploying to L2 that hasn't had the upgrade updated.

## [N-01] Typographical error in constant variable name
The constant variable `MUTLI_OWNABLE_STORAGE_LOCATION` in the MultiOwnable contract contains a typographical error, where "MULTI" is misspelled as "MUTLI". While being non-critical and not impacting the functionality, security, or performance of the contract, the typo could potentially cause mild confusion or readability issues for developers/users reviewing or interacting with the code. Correcting the spelling would enhance code clarity and maintain naming consistency without affecting the contract's execution or behavior.

https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/MultiOwnable.sol#L36-L37

```diff
-    bytes32 private constant MUTLI_OWNABLE_STORAGE_LOCATION =
+    bytes32 private constant MULTI_OWNABLE_STORAGE_LOCATION =
        0x97e2c6aad4ce5d562ebfaa00db6b9e0fb66ea5d8162ed5b243f51a2e03086f00;
```

## [N-02] Comment mismatch on future enhanced asset support in MagicSpend
MagicSpend.sol, initially documented to support only ETH withdrawals, inherently possesses a broader capability to manage multiple asset types, including ERC-20 tokens, through its versatile `_withdraw()` function. 

https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L326-L340

```solidity
    /// @notice Withdraws funds from this contract.
    ///
    /// @dev Callers MUST validate that the withdraw is legitimate before calling this method as
    ///      no validation is performed here.
    ///
    /// @param asset  The asset to withdraw.
    /// @param to     The beneficiary address.
    /// @param amount The amount to withdraw.
    function _withdraw(address asset, address to, uint256 amount) internal {
        if (asset == address(0)) {
            SafeTransferLib.safeTransferETH(to, amount);
        } else {
            SafeTransferLib.safeTransfer(asset, to, amount);
        }
    }
```
This flexibility underscores the contract's design for comprehensive asset management, necessitating an update in the documentation to accurately reflect its multi-asset support capabilities. Such an update would clarify the contract's functionality, aligning the documentation with the implemented code and ensuring a clear understanding of its capabilities for developers and auditors alike. 

https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L23

```diff
-        /// @dev The asset to withdraw. NOTE: Only ETH (associated with zero address) is supported for now.
+        /// @dev The asset to withdraw. Initially designed to support only ETH (associated with zero address),
+        /// but the contract is equipped to handle a broader range of assets, including ERC-20 tokens, as evidenced
+        /// by the flexibility of the _withdraw() function.
```

## [N-03] Incorrect comment associated with `CoinbaseSmartWallet._validateSignature()`
The comment below is inaccurate,

https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWallet.sol#L301-L306

```solidity
        if (ownerBytes.length == 32) {
            if (uint256(bytes32(ownerBytes)) > type(uint160).max) {
                // technically should be impossible given owners can only be added with @audit
                // addOwnerAddress and addOwnerPublicKey, but we leave incase of future changes. @audit
                revert InvalidEthereumAddressOwner(ownerBytes);
            }
```
given the fact that `addOwnerAddress()` and `addOwnerPublicKey()` both invoking `_addOwner()`,

https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/MultiOwnable.sol#L179-L181

```solidity
    function _addOwner(bytes memory owner) internal virtual {
        _addOwnerAtIndex(owner, _getMultiOwnableStorage().nextOwnerIndex++);
    }
```
do not have checks as implemented in `_initializeOwners()`,

https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/MultiOwnable.sol#L168-L170

```solidity
            if (owners[i].length == 32 && uint256(bytes32(owners[i])) > type(uint160).max) {
                revert InvalidEthereumAddressOwner(owners[i]);
            }
```

## [N-04] Private function with embedded modifier reduces contract size
Consider having the logic of a modifier embedded through a private function to reduce contract size if need be. A `private` visibility that is more efficient on function calls than the `internal` visibility is adopted because the modifier will only be making this call inside the contract.

For instance, the modifier below may be refactored as follows:

https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L92-L96

```diff
+    function _onlyEntryPoint() private view {
+        if (msg.sender != entryPoint()) revert Unauthorized();
+    }

     modifier onlyEntryPoint() {
-        if (msg.sender != entryPoint()) revert Unauthorized();
+        _onlyEntryPoint();
     _;
     }
```

## [N-05] Activate the Optimizer
Before deploying your contract, activate the optimizer when compiling using `solc --optimize --bin sourceFile.sol`. By default, the optimizer will optimize the contract assuming it is called 200 times across its lifetime. If you want the initial contract deployment to be cheaper and the later function executions to be more expensive, set it to ` --optimize-runs=1`. Conversely, if you expect many transactions and do not care for higher deployment cost and output size, set `--optimize-runs` to a high number.

```
module.exports = {
solidity: {
version: "0.8.23",
settings: {
optimizer: {
  enabled: true,
  runs: 1000,
},
},
},
};
```
Please visit the following site for further information:

https://docs.soliditylang.org/en/v0.5.4/using-the-compiler.html#using-the-commandline-compiler

Here's one example of instance on opcode comparison that delineates the gas saving mechanism:

```
for !=0 before optimization
PUSH1 0x00
DUP2
EQ
ISZERO
PUSH1 [cont offset]
JUMPI

after optimization
DUP1
PUSH1 [revert offset]
JUMPI
```
Disclaimer: There have been several bugs with security implications related to optimizations. For this reason, Solidity compiler optimizations are disabled by default, and it is unclear how many contracts in the wild actually use them. Therefore, it is unclear how well they are being tested and exercised. High-severity security issues due to optimization bugs have occurred in the past . A high-severity bug in the emscripten -generated solc-js compiler used by Truffle and Remix persisted until late 2018. The fix for this bug was not reported in the Solidity CHANGELOG. Another high-severity optimization bug resulting in incorrect bit shift results was patched in Solidity 0.5.6. Please measure the gas savings from optimizations, and carefully weigh them against the possibility of an optimization-related bug. Also, monitor the development and adoption of Solidity compiler optimizations to assess their maturity.   

**[wilsoncusack (Coinbase) confirmed](https://github.com/code-423n4/2024-03-coinbase-findings/issues/65#issuecomment-2020668704)**

**[3docSec (judge) commented](https://github.com/code-423n4/2024-03-coinbase-findings/issues/65#issuecomment-2023873685):**
 > While this was selected for report inclusion for quantity and quality, an honorable mention goes to [#183](https://github.com/code-423n4/2024-03-coinbase-findings/issues/183), [#171](https://github.com/code-423n4/2024-03-coinbase-findings/issues/171), and [#181](https://github.com/code-423n4/2024-03-coinbase-findings/issues/181) - I would recommend these ones for report inclusion too, to be sure the sponsor won't miss out on remediating them.



***

# Gas Optimizations

For this audit, 13 reports were submitted by wardens detailing gas optimizations. The [report highlighted below](https://github.com/code-423n4/2024-03-coinbase-findings/issues/82) by **K42** received the top score from the judge.

*The following wardens also submitted reports: [0x11singh99](https://github.com/code-423n4/2024-03-coinbase-findings/issues/195), [naman1778](https://github.com/code-423n4/2024-03-coinbase-findings/issues/143), [albahaca](https://github.com/code-423n4/2024-03-coinbase-findings/issues/137), [0xAnah](https://github.com/code-423n4/2024-03-coinbase-findings/issues/37), [SM3\_SS](https://github.com/code-423n4/2024-03-coinbase-findings/issues/191), [hunter\_w3b](https://github.com/code-423n4/2024-03-coinbase-findings/issues/188), [SAQ](https://github.com/code-423n4/2024-03-coinbase-findings/issues/185), [clara](https://github.com/code-423n4/2024-03-coinbase-findings/issues/161), [shamsulhaq123](https://github.com/code-423n4/2024-03-coinbase-findings/issues/158), [slvDev](https://github.com/code-423n4/2024-03-coinbase-findings/issues/127), [dharma09](https://github.com/code-423n4/2024-03-coinbase-findings/issues/84), and [Hajime](https://github.com/code-423n4/2024-03-coinbase-findings/issues/38).*

## Possible Optimization in [MultiOwnable.sol](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/MultiOwnable.sol)

### Possible Optimization 
- In the [_checkOwner()](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/MultiOwnable.sol#L201C1-L207C6) function, the contract checks if `msg.sender` is an owner or the contract itself. By rearranging the conditions to check the cheaper condition first (direct address comparison), we can save gas when the sender is the contract itself, avoiding the more expensive storage lookup.

Here is the optimized code: 

```solidity
function _checkOwner() internal view virtual {
    if (msg.sender == address(this) || isOwnerAddress(msg.sender)) {
        return;
    }
    revert Unauthorized();
}
```

- Estimated gas saved = This optimization may save a small amount of gas in scenarios where `msg.sender` is the contract itself, as it avoids the potentially more expensive `isOwnerAddress` call. The exact savings depend on the EVM's current gas pricing for storage reads and condition checks but could be around 200 gas for each invocation that short-circuits.

## Possible Optimizations in [ERC1271.sol](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/ERC1271.sol)

### Possible Optimization 1
- The [_hashStruct()](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/ERC1271.sol#L133C2-L135C6) function is a simple wrapper around a `keccak256` computation. Inlining this operation in [_eip712Hash()](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/ERC1271.sol#L121C1-L123C6) can save the overhead of an external function call.

Here is the optimized code snippet: 

```solidity
function _eip712Hash(bytes32 hash) internal view virtual returns (bytes32) {
    // Inlined _hashStruct operation
    bytes32 hashStruct = keccak256(abi.encode(_MESSAGE_TYPEHASH, hash));
    return keccak256(abi.encodePacked("\x19\x01", domainSeparator(), hashStruct));
}

```

- Estimated gas saved = Inlining can save around 200 or more gas per call by avoiding the external function call overhead.

### Possible Optimization 2
- The [domainSeparator()](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/ERC1271.sol#L100C1-L111C6) function computes the `EIP-712` domain separator every time it's called, which involves multiple `keccak256` hash computations and `abi.encode`. Since the domain separator only changes with contract deployment (due to its dependency on `chainId` and `address(this)`), it can be cached after the first computation to save gas on subsequent calls.

Here is the optimized code: 

```solidity
bytes32 private cachedDomainSeparator;
bool private isDomainSeparatorCached;

function domainSeparator() public view returns (bytes32) {
    if (isDomainSeparatorCached) {
        return cachedDomainSeparator;
    } else {
        (string memory name, string memory version) = _domainNameAndVersion();
        bytes32 separator = keccak256(
            abi.encode(
                keccak256("EIP712Domain(string name,string version,uint256 chainId,address verifyingContract)"),
                keccak256(bytes(name)),
                keccak256(bytes(version)),
                block.chainid,
                address(this)
            )
        );
        return separator;
    }
}
```

- Estimated gas saved = This optimization can save approximately 500 or more gas for each call after the first, depending on the complexity of [\_domainNameAndVersion()](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/ERC1271.sol#L143C1-L143C112) and the size of the strings involved.

## Possible Optimizations in [CoinbaseSmartWalletFactory.sol](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWalletFactory.sol)

### Possible Optimization 1
- The [\_getSalt()](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWalletFactory.sol#L81C2-L83C6) function is used to generate a unique salt for each account creation, based on the owners and nonce. Inlining this function within [createAccount()](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWalletFactory.sol#L38C1-L56C6) and [getAddress()](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWalletFactory.sol#L64C1-L66C6) can reduce the overhead of an `external` function call.

Here is the optimized code snippet: 

```solidity
function createAccount(bytes[] calldata owners, uint256 nonce)
    public
    payable
    returns (CoinbaseSmartWallet account)
{
    if (owners.length == 0) {
        revert OwnerRequired();
    }

    // Inline _getSalt operation
    bytes32 salt = keccak256(abi.encode(owners, nonce));

    (bool alreadyDeployed, address accountAddress) =
        LibClone.createDeterministicERC1967(msg.value, implementation, salt);

    account = CoinbaseSmartWallet(payable(accountAddress));

    if (!alreadyDeployed) {
        account.initialize(owners);
    }
}

function getAddress(bytes[] calldata owners, uint256 nonce) external view returns (address predicted) {
    // Inline _getSalt operation
    bytes32 salt = keccak256(abi.encode(owners, nonce));

    predicted = LibClone.predictDeterministicAddress(initCodeHash(), salt, address(this));
}

```

- Estimated gas saved = Inlining can save around 200 gas per call by avoiding the `external` function call overhead.

### Possible Optimization 2
- The [initCodeHash()](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWalletFactory.sol#L71C2-L73C6) function computes the hash of the initialization code for the `ERC1967` proxy used in account creation. This value depends solely on the immutable implementation address and thus remains constant. Caching this value can save gas for each account creation after the first.

Here is the optimized code: 

```solidity
bytes32 private cachedInitCodeHash;
bool private isInitCodeHashCached;

function initCodeHash() public view returns (bytes32 result) {
    if (isInitCodeHashCached) {
        return cachedInitCodeHash;
    } else {
        cachedInitCodeHash = LibClone.initCodeHashERC1967(implementation);
        isInitCodeHashCached = true;
        return cachedInitCodeHash;
    }
}

```

- Estimated gas saved = This optimization can save approximately 800 or so gas for each call after the first, depending on the EVM's current gas pricing for storage reads and the `SLOAD` operation.

## Possible Optimizations in [CoinbaseSmartWallet.sol](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWallet.sol)

### Possible Optimization 1
- The [\_validateSignature()](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWallet.sol#L291C1-L325C6) function decodes the signature parameter multiple times to extract `SignatureWrapper` and then again for `WebAuthn.WebAuthnAuth` or `address`. This redundancy can be minimized by decoding once and passing the necessary components to other functions as needed.

Here is the optimized code snippet: 

```solidity
function _validateSignature(bytes32 message, bytes calldata signature)
    internal
    view
    override
    returns (bool)
{
    SignatureWrapper memory sigWrapper = abi.decode(signature, (SignatureWrapper));
    bytes memory ownerBytes = ownerAtIndex(sigWrapper.ownerIndex);

    // Directly pass decoded values to reduce redundant operations
    return _processSignature(message, ownerBytes, sigWrapper.signatureData);
}

function _processSignature(bytes32 message, bytes memory ownerBytes, bytes memory signatureData)
    internal
    view
    returns (bool)
{
    if (ownerBytes.length == 32) {
        address owner = abi.decode(ownerBytes, (address));
        return SignatureCheckerLib.isValidSignatureNow(owner, message, signatureData);
    } else if (ownerBytes.length == 64) {
        (uint256 x, uint256 y) = abi.decode(ownerBytes, (uint256, uint256));
        WebAuthn.WebAuthnAuth memory auth = abi.decode(signatureData, (WebAuthn.WebAuthnAuth));
        return WebAuthn.verify({challenge: abi.encode(message), requireUV: false, webAuthnAuth: auth, x: x, y: y});
    }
    revert InvalidOwnerBytesLength(ownerBytes);
}
```

- Estimated gas saved = This optimization can save around 200 or so gas per validation by reducing the overhead of multiple `abi.decode` calls.

### Possible Optimization 2
- The [onlyEntryPointOrOwner()](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWallet.sol#L74C1-L81C1) modifier checks if `msg.sender` is the entry point and then calls `_checkOwner` which again checks if `msg.sender` is an owner. This can be optimized by consolidating the checks to avoid redundant `msg.sender` comparisons and storage reads.

Here is the optimized code: 

```solidity
modifier onlyEntryPointOrOwner() {
    if (msg.sender != entryPoint() && !isOwnerAddress(msg.sender) && msg.sender != address(this)) {
        revert Unauthorized();
    }
    _;
}

```

- Estimated gas saved = This change can save approximately 100 or so gas per transaction by reducing the number of conditional checks and potentially avoiding a storage read in `_checkOwner`.

## Possible Optimizations in [WebAuthn.sol](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/WebAuthnSol/WebAuthn.sol)

### Possible Optimization 1
- The `clientDataJSONHash` is computed every time the [verify()](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/WebAuthnSol/WebAuthn.sol#L104C1-L161C6) function is called. This hash computation involves the entire `clientDataJSON` string, which can be relatively large. By computing this hash once and passing it as a parameter to functions that require it, we can save gas.

Here is the optimized code snippet: 

```solidity
function verify(
    bytes memory challenge,
    bool requireUV,
    WebAuthnAuth memory webAuthnAuth,
    uint256 x,
    uint256 y
) internal view returns (bool) {
    bytes32 clientDataJSONHash = sha256(bytes(webAuthnAuth.clientDataJSON));
    // Use clientDataJSONHash in subsequent operations...
}

```

- Estimated gas saved = This optimization can save approximately 200 or so gas depending on the size of `clientDataJSON`. The exact savings depend on the input size and the current gas price for the `SHA256` precompile.

### Possible Optimization 2
- The current implementation attempts to use a precompiled contract for signature verification and falls back to `FCL.ecdsa_verify` if the precompile call fails. This `fallback` mechanism can be optimized by checking the success of the precompile call more efficiently and avoiding unnecessary operations if the precompile verification is successful.

Here is the optimized code: 

```solidity
function verify(
    bytes memory challenge,
    bool requireUV,
    WebAuthnAuth memory webAuthnAuth,
    uint256 x,
    uint256 y
) internal view returns (bool) {
    // Compute messageHash and prepare args for precompile call...
    (bool success, bytes memory ret) = VERIFIER.staticcall(args);
    if (success && ret.length > 0) {
        return abi.decode(ret, (bool));
    }
    // Fallback to FCL.ecdsa_verify only if precompile call was not successful
    return FCL.ecdsa_verify(messageHash, webAuthnAuth.r, webAuthnAuth.s, x, y);
}
```

- Estimated gas saved = This optimization can save gas by avoiding unnecessary decoding and logic when the precompile call is successful. The savings could be around 500 or more gas, depending on the EVM's current gas pricing for `staticcall` and the efficiency of the precompile.

## Possible Optimizations in [FCL.sol](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol)

### Possible Optimization 1
- The [FCL_nModInv()](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L94C1-L110C6) and [FCL_pModInv()](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L374C1-L390C6) functions use the `MODEXP` precompile for modular inversion, which involves setting up memory for the call. This setup can be optimized by reducing memory operations and directly using assembly for the entire operation.

Here is the optimized code snippet: 

```solidity
function optimizedModInv(uint256 u, uint256 modulus) internal view returns (uint256 result) {
    assembly {
        // Setup for MODEXP precompile in a compact form
        let freemem_pointer := mload(0x40)
        mstore(freemem_pointer, 0x20) // Length of Base
        mstore(add(freemem_pointer, 0x20), 0x20) // Length of Exponent
        mstore(add(freemem_pointer, 0x40), 0x20) // Length of Modulus
        mstore(add(freemem_pointer, 0x60), u) // Base: u
        mstore(add(freemem_pointer, 0x80), minus_2modn) // Exponent: p-2 for nModInv, minus_2 for pModInv
        mstore(add(freemem_pointer, 0xA0), modulus) // Modulus: n or p
        // Call MODEXP precompile
        if iszero(staticcall(not(0), MODEXP_PRECOMPILE, freemem_pointer, 0xC0, freemem_pointer, 0x20)) {
            revert(0, 0)
        }
        result := mload(freemem_pointer)
    }
}

```

- Estimated gas saved = This optimization could save around 200 or so gas per call by streamlining the memory setup and reducing the overhead associated with preparing the call data for the `MODEXP` precompile.

### Possible Optimization 2
- The [ecdsa_verify()](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L50C1-L70C6) function performs several checks that could be streamlined. The check for `r` and `s` being within valid ranges and the point being on the curve could be optimized by reordering operations or combining conditions.

Here is the optimized code: 

```solidity
function optimizedEcdsaVerify(bytes32 message, uint256 r, uint256 s, uint256 Qx, uint256 Qy) internal view returns (bool) {
    // Combine checks for efficiency
    if ((r == 0 || r >= n || s == 0 || s >= n) || !ecAff_isOnCurve(Qx, Qy)) {
        return false;
    }

    // Continue with the rest of the verification as before
    ...
}

```

- Estimated gas saved = This change could save a few hundred gas by avoiding unnecessary condition evaluations and leveraging short-circuiting behaviour in logical operations.

## Possible Optimizations in [MagicSpend.sol](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol)

### Possible Optimization 1
- The [isValidWithdrawSignature()](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L260C1-L268C6) function is called multiple times for the same ``WithdrawRequest``. Caching the result of the first call and reusing it can save gas.

Here is the optimized code snippet: 

```solidity
mapping(bytes32 => bool) private _signatureVerificationCache;

function isValidWithdrawSignature(address account, WithdrawRequest memory withdrawRequest) public returns (bool) {
    bytes32 requestHash = getHash(account, withdrawRequest);
    if (_signatureVerificationCache[requestHash]) {
        return true;
    }
    bool isValid = SignatureCheckerLib.isValidSignatureNow(owner(), requestHash, withdrawRequest.signature);
    if (isValid) {
        _signatureVerificationCache[requestHash] = true;
    }
    return isValid;
}
```

- Estimated gas saved = This optimization could save around 5,000 or so gas, for each subsequent verification of the same request, depending on the complexity of the signature verification process.

### Possible Optimization 2
- The [\_validateRequest()](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L315C1-L325C1) function checks if a nonce is used and then marks it as used. This operation can be optimized by checking and setting the nonce in a single operation to reduce the gas cost associated with storage access.

Here is the optimized code: 

```solidity
function _validateRequest(address account, WithdrawRequest memory withdrawRequest) internal {
    bytes32 nonceKey = keccak256(abi.encodePacked(account, withdrawRequest.nonce));
    require(!_nonceUsed[nonceKey], "InvalidNonce");
    _nonceUsed[nonceKey] = true;
    emit MagicSpendWithdrawal(account, withdrawRequest.asset, withdrawRequest.amount, withdrawRequest.nonce);
}

```

- Estimated gas saved = This change could save a couple thousand gas by optimizing the current storage access patterns, as it reduces the number of `SLOAD` and `SSTORE` operations.

### Possible Optimization 3
- In the [validatePaymasterUserOp()](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L109C1-L140C6) function, the contract updates `_withdrawableETH` mapping. This operation can be optimized by deferring updates until necessary, potentially batching them to save gas on storage operations.

Here is the optimized code snippet: 

```solidity
// Assume existence of a batch processing system
function _batchUpdateWithdrawableETH(address account, uint256 amount, bool increase) internal {
    if (increase) {
        _withdrawableETH[account] += amount;
    } else {
        require(_withdrawableETH[account] >= amount, "InsufficientBalance");
        _withdrawableETH[account] -= amount;
    }
}
```

- Estimated gas saved = While the exact savings depend on the frequency and pattern of updates, deferring and batching updates could save thousands of gas by reducing the number of storage operations.

**[raymondfam (Lookout) commented](https://github.com/code-423n4/2024-03-coinbase-findings/issues/82#issuecomment-2015994332):**
 > Well-elaborated optimization suggested for each of the in-scope contracts.

**[3docSec (Judge) commented](https://github.com/code-423n4/2024-03-coinbase-findings/issues/82#issuecomment-2022791400):**
 > Best value on top of the gas findings reported in the automated findings report.

**[wilsoncusack (Coinbase) acknowledged](https://github.com/code-423n4/2024-03-coinbase-findings/issues/82#issuecomment-2020743978)**



***

# Audit Analysis

For this audit, 21 analysis reports were submitted by wardens. An analysis report examines the codebase as a whole, providing observations and advice on such topics as architecture, mechanism, or approach. The [report highlighted below](https://github.com/code-423n4/2024-03-coinbase-findings/issues/130) by **roguereggiant** received the top score from the judge.

*The following wardens also submitted reports: [0xepley](https://github.com/code-423n4/2024-03-coinbase-findings/issues/176), [popeye](https://github.com/code-423n4/2024-03-coinbase-findings/issues/173), [emerald7017](https://github.com/code-423n4/2024-03-coinbase-findings/issues/36), [MSK](https://github.com/code-423n4/2024-03-coinbase-findings/issues/200), [cheatc0d3](https://github.com/code-423n4/2024-03-coinbase-findings/issues/196), [JCK](https://github.com/code-423n4/2024-03-coinbase-findings/issues/190), [fouzantanveer](https://github.com/code-423n4/2024-03-coinbase-findings/issues/189), [SBSecurity](https://github.com/code-423n4/2024-03-coinbase-findings/issues/166), [clara](https://github.com/code-423n4/2024-03-coinbase-findings/issues/156), [kaveyjoe](https://github.com/code-423n4/2024-03-coinbase-findings/issues/152), [unique](https://github.com/code-423n4/2024-03-coinbase-findings/issues/121), [hunter\_w3b](https://github.com/code-423n4/2024-03-coinbase-findings/issues/120), [Circolors](https://github.com/code-423n4/2024-03-coinbase-findings/issues/118), [LinKenji](https://github.com/code-423n4/2024-03-coinbase-findings/issues/112), [JcFichtner](https://github.com/code-423n4/2024-03-coinbase-findings/issues/73), [SAQ](https://github.com/code-423n4/2024-03-coinbase-findings/issues/54), [foxb868](https://github.com/code-423n4/2024-03-coinbase-findings/issues/35), [0xbrett8571](https://github.com/code-423n4/2024-03-coinbase-findings/issues/26), [Myd](https://github.com/code-423n4/2024-03-coinbase-findings/issues/19), and [albahaca](https://github.com/code-423n4/2024-03-coinbase-findings/issues/15).*

## Introduction

SmartWallet is a smart contract wallet that enhances user authentication and operational flexibility on Ethereum-based networks. It supports Ethereum address and passkey owners, utilizing WebAuthnSol for signature validation. The wallet allows multiple owners and facilitates account-changing operations across any EVM chain with the same address. It adheres to ERC-4337 standards and integrates with paymasters like MagicSpend for broader utility.

| File Name                        | Description                       |
|----------------------------------|-----------------------------------|
| MultiOwnable.sol                |    This code establishes a blockchain-based authorization system enabling multiple entities to own and manage a contract, with ownership identifiable via Ethereum addresses or public key coordinates. It features mechanisms for adding, removing, and verifying owners, alongside initializing owner lists, guided by custom error handling and events for ownership changes.                               |
| ERC1271.sol                     |     This code outlines an abstract implementation of the ERC-1271 standard with enhanced security measures to prevent signature replay across different accounts, by incorporating an EIP-712 compliant hashing layer that includes the contract's address and chain ID in the domain separator.                              |
| CoinbaseSmartWalletFactory.sol  |      This code represents a factory for creating and managing smart wallet accounts, utilizing a cloning technique for efficient deployment, and enables setting initial ownership through a deterministic approach. It leverages an ERC-4337 implementation for account functionality, allowing multiple accounts with unique owners and nonces.                             |
| CoinbaseSmartWallet.sol         |     This code introduces a smart contract wallet compatible with ERC-4337 account abstraction, integrating multiple ownership and upgradeability features. It supports signature validation for operations and offers enhanced security through WebAuthn and ERC-1271 standards, allowing for both traditional and public key-based owner authentication.                              |
| WebAuthn.sol                    |   This code introduces a library for authenticating WebAuthn assertions, verifying signatures against public keys using either a blockchain precompile or a cryptographic library. It checks for user presence and, if required, user verification, while validating the challenge and type in the client data JSON.                                |
| FCL.sol                         |  This code provides an optimized library for verifying ECDSA signatures on curves with a prime number of points, specifically tailored for curves with a -3 coefficient. It employs precompiled contracts for efficient computation and is designed for high security and performance, explicitly supporting the secp256r1 curve.                                 |
| MagicSpend.sol                  |  This contract is an ERC-4337 paymaster implementation that allows accounts to withdraw ETH funds. It supports signing withdraw requests with unique nonces and expiry dates, includes validations for signatures and nonces, and integrates with the EntryPoint contract for account abstraction operations.                                 |

## Architecture Diagram 

The system described through the provided code snippets outlines a comprehensive architecture incorporating account abstraction, signature verification, and smart contract interactions for managing ownership, authentication, and transactions. Here is an architecture diagram in a descriptive format that illustrates the overall system interactions:

*Note: to view the provided image, please see the original submission [here](https://github.com/code-423n4/2024-03-coinbase-findings/issues/130).*

## Overview of the Whole System Interactions

1. **User Interaction**: Users interact with their accounts to initiate transactions, which may include operations like transferring assets or invoking specific functionalities of a contract.

2. **User Account**: Represents an ERC-4337 compatible smart contract wallet, enabling account abstraction and allowing transactions to be executed without ETH for gas, relying on the EntryPoint and Paymaster for execution and funding.

3. **Signature Verification**: A crucial component ensuring that operations requested by the user account are duly signed by the rightful owners, employing ECDSA and WebAuthn for robust authentication mechanisms.

4. **EntryPoint (EP)**: Serves as the gateway for user operations under the account abstraction model, coordinating the execution of transactions by interacting with Paymasters for gas sponsorship and ensuring the system's integrity.

5. **Paymaster (PM)**: Manages the funding for transactions, deciding whether to sponsor a transaction based on the available funds and the legitimacy of the operation, as validated through signature checks and nonce verification.

6. **Fund Management (FM)**: Handles the logistics of fund withdrawal requests from users, including validation of signatures and ensuring the security and authorization of the withdrawal operations.

7. **Stake Management (SM)**: Interacts with the EntryPoint for managing stakes required for the operation of Paymasters within the account abstraction framework, ensuring the Paymaster has sufficient stake for transaction sponsorship.

8. **MultiOwnable (MO)**: Manages ownership of the user account, allowing for multiple owners to control and manage the account through collective or individual permissions.

9. **WebAuthn (WA)**: Provides an additional layer of security for authentication, allowing users to utilize WebAuthn as a method for verifying their identity during transactions or ownership management operations.

This architecture provides a comprehensive system for managing smart contract-based accounts, ensuring secure transactions through robust authentication mechanisms, facilitating account abstraction, and managing funds and ownership through a decentralized framework.

### SequenceDiagram

*Note: to view the provided image, please see the original submission [here](https://github.com/code-423n4/2024-03-coinbase-findings/issues/130).*

### Sequence Diagram Overview:

1. **User Initiates a Transaction**: The user starts the process by attempting to perform a transaction through their User Account (smart contract wallet).

2. **Signature Validation**: The User Account requests signature validation, which can involve traditional ECDSA checks or WebAuthn for more sophisticated authentication mechanisms.

3. **Transaction Submission**: Once the signature is validated, the User Account submits the transaction to the EntryPoint, which acts as a hub for account abstraction operations.

4. **Gas Sponsorship Request**: The EntryPoint forwards the transaction to the Paymaster for gas sponsorship, essential for executing the transaction.

5. **Withdraw Request Verification**: If the transaction involves withdrawing funds, the Paymaster verifies the signature associated with the withdraw request to ensure its legitimacy.

6. **Funds Check**: The Paymaster checks available funds to cover the transaction and potentially the withdraw request, interacting with the Fund Management component.

7. **Sponsorship Confirmation & Execution**: Once the Paymaster confirms sponsorship, the EntryPoint proceeds to execute the transaction.

8. **Ownership Operation**: The User Account can perform ownership-related operations, managed by the MultiOwnable component for cases involving multiple owners.

9. **Authentication**: For certain operations, additional user authentication might be required, which is where WebAuthn comes into play, providing a secure method of verifying the user's identity.

10. **Stake Management**: The Paymaster interacts with the Stake Management to handle stakes associated with the EntryPoint, ensuring there are sufficient funds to cover potential transactions.

This sequence outlines a comprehensive interaction flow within a system designed to leverage account abstraction, ensuring transactions can be executed securely and efficiently while providing mechanisms for ownership management and advanced authentication.

### MultiOwnable.sol

*Note: to view the provided image, please see the original submission [here](https://github.com/code-423n4/2024-03-coinbase-findings/issues/130).*

### Functionality of Functions in the MultiOwnable Smart Contract

**addOwnerAddress**
- *Purpose*: Allows adding a new owner to the contract using an Ethereum address. This facilitates straightforward ownership management.
- *Usage*: Can only be called by an existing owner (enforced through the `onlyOwner` modifier). The input is an Ethereum address, which is then encoded and added as a new owner.

**addOwnerPublicKey**
- *Purpose*: Facilitates adding a new owner using a public key, specifically targeting use cases where owners are identified through public keys rather than Ethereum addresses.
- *Usage*: Similar to `addOwnerAddress`, this function is restricted to existing owners. It accepts the `x` and `y` coordinates of a public key, encodes them, and adds the result as a new owner.

**removeOwnerAtIndex**
- *Purpose*: Removes an owner from the contract based on a specified index. This is useful for managing the dynamic nature of ownership.
- *Usage*: Only callable by an existing owner. It checks if the specified index has a registered owner and, if so, deletes the owner from storage.

**isOwnerAddress**
- *Purpose*: Checks if a given Ethereum address is registered as an owner of the contract.
- *Usage*: Publicly accessible. Returns true if the input address is an owner, false otherwise.

**isOwnerPublicKey**
- *Purpose*: Determines if a public key (given by its `x` and `y` coordinates) is registered as an owner.
- *Usage*: Public function that returns true if the public key is an owner, otherwise false.

**isOwnerBytes**
- *Purpose*: Verifies if the provided raw bytes (representing either an Ethereum address or public key coordinates) correspond to a registered owner.
- *Usage*: Public visibility. Useful for cases where the owner identity format is not predetermined.

**ownerAtIndex**
- *Purpose*: Retrieves the raw bytes representing an owner (could be an Ethereum address or public key coordinates) based on the given index.
- *Usage*: Publicly accessible, returns the owner's raw bytes at the specified index, or an empty byte array if no owner is registered there.

**nextOwnerIndex**
- *Purpose*: Provides the next available index for adding a new owner. This helps in determining where the next owner will be added in the contract's storage.
- *Usage*: Public function that returns an integer representing the next index for a new owner.

**\_initializeOwners**
- *Purpose*: Initializes the contract with a list of owners. Intended for use at contract deployment to set initial ownership.
- *Usage*: Internal function that takes an array of raw bytes (each representing an owner) and registers them as owners. Validates the length of the input to ensure it matches expected formats for addresses or public keys.

**\_addOwner**
- *Purpose*: A convenience function that encapsulates the process of adding a new owner, abstracting away the indexing logic.
- *Usage*: Internal function called within `addOwnerAddress` and `addOwnerPublicKey` to add the provided owner to the contract.

**\_addOwnerAtIndex**
- *Purpose*: Directly adds an owner at a specific index, performing checks to ensure the owner is not already registered.
- *Usage*: Internal function that facilitates adding owners with more control over storage indices.

**\_checkOwner**
- *Purpose*: Validates whether the caller of a function is an authorized owner or the contract itself, providing a basic access control mechanism.
- *Usage*: An internal function used by the `onlyOwner` modifier to restrict access to certain contract functions.

**\_getMultiOwnableStorage**
- *Purpose*: Retrieves a reference to the contract's storage space dedicated to managing ownership information.
- *Usage*: Internal function providing low-level access to ownership data storage, utilizing Solidity's assembly language for direct memory manipulation.

Each function in the `MultiOwnable` contract plays a specific role in managing ownership, from adding and removing owners to validating ownership and managing access control. The use of raw bytes for owner representation allows for flexibility in identifying owners, accommodating both Ethereum addresses and public key coordinates.

*Note: to view the provided image, please see the original submission [here](https://github.com/code-423n4/2024-03-coinbase-findings/issues/130).*

### ERC1271.sol

*Note: to view the provided image, please see the original submission [here](https://github.com/code-423n4/2024-03-coinbase-findings/issues/130).*

### ERC-1271 With Cross Account Replay Protection Smart Contract Function Descriptions

**eip712Domain**
- *Purpose*: Provides detailed information about the EIP-712 domain used to create compliant hashes. It specifies the domain fields used to construct a domain separator for signing data.
- *Details*: Returns a set of values defining the EIP-712 domain, including the domain's name, version, chain ID, verifying contract address, salt, and any extensions to the standard EIP-712 domain fields.

**isValidSignature**
- *Purpose*: Validates a given signature against a specified hash, ensuring it matches the expected signer's signature. This function specifically checks against a "replay-safe" version of the hash, adding an extra layer of security.
- *Details*: Takes an original hash and its corresponding signature, generates a "replay-safe" hash, and checks if the signature is valid for this modified hash. Returns a specific byte code indicating success or failure.

**replaySafeHash**
- *Purpose*: Creates a modified version of a given hash to prevent cross-account replay attacks. It does this by incorporating the contract's address and chain ID into the hash, making it unique to each account and chain.
- *Details*: Accepts an original hash and returns a new hash that is compliant with EIP-712, including additional data to ensure it cannot be reused across different accounts or chains.

**domainSeparator**
- *Purpose*: Calculates and returns the domain separator for the contract, which is a unique identifier used in the EIP-712 signing process to prevent certain types of replay attacks.
- *Details*: Constructs the domain separator based on the current EIP-712 domain information, including the contract's name, version, chain ID, and address.

**\_eip712Hash**
- *Purpose*: Generates an EIP-712 compliant hash from a given input, using the contract's domain separator and the structured data's hash.
- *Details*: Internal function that takes a structured data hash (specifically for the "CoinbaseSmartWalletMessage" type) and returns its EIP-712 compliant hash by combining it with the contract's domain separator.

**\_hashStruct**
- *Purpose*: Computes the EIP-712 hash of a structured data piece, specifically for "CoinbaseSmartWalletMessage", which includes a given hash as its content.
- *Details*: Internal function that encodes the structured data (using its type hash and the input hash) and then hashes the result to produce the EIP-712 hashStruct value.

**\_domainNameAndVersion**
- *Purpose*: Abstract function that must be defined by the implementing contract. It should return the name and version of the signing domain used in EIP-712 domain separation.
- *Details*: Must be overridden by child contracts to provide specific domain name and version strings relevant to the particular implementation.

**\_validateSignature**
- *Purpose*: Abstract function that validates a provided signature against a given message. It must be implemented by the contract to define how signature validation is performed.
- *Details*: Takes a message and its associated signature, and returns a boolean indicating whether the signature is valid. The actual validation logic is to be defined in the implementation.

*Note: to view the provided image, please see the original submission [here](https://github.com/code-423n4/2024-03-coinbase-findings/issues/130).*

### CoinbaseSmartWalletFactory.sol

*Note: to view the provided image, please see the original submission [here](https://github.com/code-423n4/2024-03-coinbase-findings/issues/130).*

### Coinbase Smart Wallet Factory Smart Contract Function Descriptions

**Constructor**
- *Purpose*: Initializes the smart contract with the address of the ERC-4337 implementation that will be used for deploying new account instances.
- *Details*: Sets the `implementation` variable to the address of the ERC-4337 implementation provided during the contract deployment.

**createAccount**
- *Purpose*: Deploys a new ERC-4337 compatible account using a minimal ERC1967 proxy that points to a predefined implementation. This function also initializes the newly created account with a set of owners.
- *Details*: Accepts an array of owners and a nonce to create a deterministic address for the new account. If the account with the given parameters hasn't been deployed before, it initializes the account with the provided owners. The function returns the instance of the created account.

**getAddress**
- *Purpose*: Computes and returns the deterministic address for an account that would be created with the specified set of owners and nonce. This address is based on the implementation address and the provided parameters.
- *Details*: Utilizes the `LibClone` library to predict the address of the account that would be created by `createAccount` with the same parameters. This allows external entities to know the address of an account before it's actually created.

**initCodeHash**
- *Purpose*: Returns the hash of the initialization code used by the ERC1967 proxy. This hash is critical for computing deterministic addresses.
- *Details*: Calls `LibClone.initCodeHashERC1967` with the address of the ERC-4337 implementation to obtain the hash of the proxy's initialization code.

**\_getSalt**
- *Purpose*: Generates a deterministic salt value based on the provided owners and nonce. This salt is used in the creation of deterministic addresses for new accounts.
- *Details*: Encodes the array of owners and the nonce into a single byte array and hashes it to produce a salt value. This internal function supports the deterministic creation process by providing a unique salt for each set of parameters.

Each function within the Coinbase Smart Wallet Factory is designed to facilitate the deployment and management of ERC-4337 accounts. By leveraging the ERC-4337 standard and the ERC1967 proxy pattern, this contract provides a scalable and efficient mechanism for creating and initializing smart contract-based accounts with multiple ownership configurations.

*Note: to view the provided image, please see the original submission [here](https://github.com/code-423n4/2024-03-coinbase-findings/issues/130).*

### CoinbaseSmartWallet.sol

*Note: to view the provided image, please see the original submission [here](https://github.com/code-423n4/2024-03-coinbase-findings/issues/130).*

### Coinbase Smart Wallet Smart Contract Function Descriptions

**Constructor**
- Initializes the contract, setting up initial states or configurations that do not rely on external inputs. It prepares the contract for use, following deployment.

**initialize**
- Prepares the wallet with initial settings, particularly the assignment of owners if it has not been previously initialized. This ensures the wallet is ready for operation with designated control.

**validateUserOp**
- Processes and validates a `UserOperation`, ensuring it adheres to specified criteria such as signature validity and nonce correctness. This step is critical for transaction execution through account abstraction.

**executeWithoutChainIdValidation**
- Allows execution of transactions without validating the chain ID. This functionality supports operations that must be consistent across different chains, enhancing interoperability.

**execute**
- Facilitates the execution of specified calls or transactions from the wallet. This function is pivotal for carrying out intended actions on the blockchain.

**executeBatch**
- Executes multiple calls or transactions in a single operation, optimizing transaction execution by batching together several operations.

**entryPoint**
- Identifies the EntryPoint contract address, which is essential for interacting with the account abstraction layer and facilitating user operations.

**getUserOpHashWithoutChainId**
- Generates a hash for a `UserOperation` excluding the chain ID, supporting operations that are chain-agnostic.

**implementation**
- Provides the address of the contract's implementation, relevant for upgradeable contracts using the proxy pattern.

**canSkipChainIdValidation**
- Determines whether a function call can bypass chain ID validation based on its selector, adding flexibility to operations that require cross-chain compatibility.

**\_call**
- A low-level function to execute calls to other contracts or addresses. This is a foundational operation for interacting with the broader Ethereum ecosystem.

**\_validateSignature**
- Validates the signature associated with a transaction or operation, ensuring it originates from an authorized source. This function is crucial for security and preventing unauthorized actions.

**\_authorizeUpgrade**
- Restricts upgrade operations to authorized entities, ensuring only designated owners can modify the contract's logic.

**\_domainNameAndVersion**
- Supplies information about the domain and version used in signature verification, aiding in the creation of EIP-712 compliant messages.

*Note: to view the provided image, please see the original submission [here](https://github.com/code-423n4/2024-03-coinbase-findings/issues/130).*

### WebAuthn.sol

*Note: to view the provided image, please see the original submission [here](https://github.com/code-423n4/2024-03-coinbase-findings/issues/130).*

### WebAuthn Library Function Descriptions

**verify**
- *Purpose*: Confirms the authenticity of a WebAuthn authentication assertion, following a specific subset of verification steps from the WebAuthn specification. It checks the integrity and origin of the authentication assertion using the given challenge, user verification requirement, signature, and the public key coordinates.
- *Operation*: 
    - Validates that the "user present" and, if required, "user verified" flags are set within the authenticator data.
    - Confirms that the client data JSON indicates a "webauthn.get" operation and contains the expected challenge.
    - Verifies the signature over the concatenated authenticator data and client data JSON hash, ensuring it matches the provided public key.
    - Utilizes the RIP-7212 precompiled contract for secp256r1 signature verification if available; otherwise, falls back to the FreshCryptoLib implementation.
    - Guards against signature malleability by checking the 's' value of the signature.

The `verify` function is designed to ensure the integrity and origin of WebAuthn authentication assertions in a blockchain context, incorporating elements such as signature verification and challenge confirmation while accommodating the specific operational requirements and assumptions of blockchain applications.

*Note: to view the provided image, please see the original submission [here](https://github.com/code-423n4/2024-03-coinbase-findings/issues/130).*

### FCL.sol

*Note: to view the provided image, please see the original submission [here](https://github.com/code-423n4/2024-03-coinbase-findings/issues/130).*

### Fresh CryptoLib (FCL) Functionality Explained

**Constants**
Defines constants related to the curve and operations:
- **MODEXP\_PRECOMPILE**: Address for the modular exponentiation precompile used for efficient mathematical operations.
- **Curve Parameters**: Includes the prime field modulus (`p`), curve coefficients (`a` and `b`), base point coordinates (`gx` and `gy`), curve order (`n`), and constants for optimization (`minus_2`, `minus_2modn`, `minus_1`).

**ecdsa\_verify**
- *Purpose*: Verifies an ECDSA signature given a message hash, signature components (`r` and `s`), and the public key coordinates (`Qx` and `Qy`) of the signer.
- *Operation*: Checks the signature against the curve-defined parameters and the message, ensuring it was signed by the holder of the private key corresponding to the provided public key.

**ecAff\_isOnCurve**
- *Purpose*: Determines if a point, defined by its `x` and `y` coordinates, lies on the defined elliptic curve.
- *Operation*: Utilizes the elliptic curve equation to verify if the point satisfies the curve's equation.

**FCL\_nModInv**
- *Purpose*: Computes the modular inverse of a number under the curve's order (`n`) using the modular exponentiation precompile for efficiency.
- *Operation*: Employs Fermat's Little Theorem for the modular inverse calculation, essential for cryptographic operations like signature verification.

**ecZZ\_mulmuladd\_S\_asm**
- *Purpose*: Efficiently computes the combination of scalar multiplications (`scalar_u * G + scalar_v * Q`) using Strauss-Shamir's trick for ECDSA signature verification.
- *Operation*: Optimizes ECDSA signature verification through simultaneous multiple scalar multiplication, crucial for verifying signatures with less computational overhead.

**ecAff\_add**
- *Purpose*: Adds two points on the elliptic curve in affine coordinates, handling edge cases like point doubling and the identity element.
- *Operation*: Implements elliptic curve point addition according to curve algebra, ensuring correct addition results within the curve's group structure.

**ecAff\_IsZero**
- *Purpose*: Checks if a given elliptic curve point, represented in affine coordinates, is the identity element (zero point) of the curve.
- *Operation*: Identifies the curve's identity element, facilitating operations like point addition and scalar multiplication where the identity element has special rules.

**ecZZ\_SetAff**
- *Purpose*: Converts a point from projective (or Jacobian) coordinates back to affine coordinates, which are more straightforward to work with for certain operations.
- *Operation*: Utilizes modular inversion to normalize projective coordinates to affine, necessary for finalizing cryptographic operations like signature verification.

**ecZZ\_Dbl**
- *Purpose*: Doubles a point on the elliptic curve, given in projective coordinates, as part of elliptic curve point multiplication algorithms.
- *Operation*: Implements the point doubling formula specific to elliptic curves, optimizing the doubling operation in the context of ECDSA verification and other cryptographic algorithms.

**ecZZ\_AddN**
- *Purpose*: Adds two elliptic curve points given in mixed coordinates, optimizing the addition for cryptographic applications like ECDSA.
- *Operation*: Efficiently combines points on the elliptic curve, respecting the algebraic structure of the curve, and optimizing for cases encountered in cryptographic routines.

**FCL\_pModInv**
- *Purpose*: Calculates the modular inverse of a number under the elliptic curve's prime field modulus (`p`) using the modular exponentiation precompile.
- *Operation*: Applies Fermat's Little Theorem for calculating the modular inverse, a crucial step in cryptographic operations like signature verification and key generation.

The FCL library provides foundational operations for elliptic curve cryptography, focusing on efficiency and security, essential for implementing ECDSA signature verification and other cryptographic protocols on Ethereum.

*Note: to view the provided image, please see the original submission [here](https://github.com/code-423n4/2024-03-coinbase-findings/issues/130).*

### MagicSpend.so

*Note: to view the provided image, please see the original submission [here](https://github.com/code-423n4/2024-03-coinbase-findings/issues/130).*

### Magic Spend Smart Contract Functionality Explained

**Constructor**
- Initializes the contract by setting the initial owner through the `Ownable` pattern.

**Receive ETH**
- Allows the contract to receive Ether directly without a function call.

**Validate Paymaster User Operation**
- Validates an ERC-4337 `UserOperation` for withdrawal requests, ensuring the request does not exceed the gas cost, is funded sufficiently, and has a valid signature. This method integrates with the ERC-4337 EntryPoint as a Paymaster.

**Post Operation Handling**- Finalizes the transaction after user operation execution, handling the transfer of funds back to the user or managing gas costs.

**Withdraw Gas Excess**
- Enables users to withdraw any excess funds that were allocated for gas but not used during transaction execution.

**Withdraw Funds**
- Allows users to withdraw specified assets from the contract based on a signed withdrawal request.

**Owner Withdraw**
- Provides an exclusive function for the contract owner to withdraw assets to a specified beneficiary.

**Deposit to EntryPoint**
- Allows the owner to deposit ETH from the contract into the EntryPoint contract, facilitating future transactions.

**Withdraw from EntryPoint**
- Enables the contract owner to withdraw deposited funds from the EntryPoint to a specified beneficiary.

**Add Stake to EntryPoint**
- Allows the contract owner to add a stake to the EntryPoint, locking funds to meet the EntryPoint's staking requirements.

**Unlock Stake from EntryPoint**
- Permits the contract owner to initiate the unstaking process, making funds available for withdrawal after a cooldown period.

**Withdraw Stake from EntryPoint**
- Enables the contract owner to withdraw previously staked funds from the EntryPoint after they have been unlocked.

**Validate Withdraw Signature**
- Validates the signature associated with a withdrawal request, ensuring it matches the expected format and signer.

**Get Hash for Withdraw Request**
- Generates a hash for a withdrawal request, which is used to validate the request's signature.

**Check Nonce Usage**
- Determines if a specific nonce has already been used by an account to prevent replay attacks.

**EntryPoint Address**
- Provides the canonical address of the ERC-4337 EntryPoint contract the Paymaster interacts with.

**Internal Functions:**
- *\_validateRequest*: Performs preliminary checks on a withdrawal request before processing.
- *\_withdraw*: Handles the withdrawal of specified assets to a designated beneficiary.

This smart contract serves as a Paymaster for ERC-4337 transactions, managing funds, ensuring transactions are properly funded, and facilitating withdrawals. It implements security checks such as signature validation and nonce management to prevent unauthorized access and replay attacks.

*Note: to view the provided image, please see the original submission [here](https://github.com/code-423n4/2024-03-coinbase-findings/issues/130).*

## Risk Assessment of the Smart Contract System

**Centralization Risks**

1. **Owner Privileges**: With specific functions exclusively available to the contract's owner (e.g., `ownerWithdraw`, `entryPointDeposit`), there's a risk tied to the centralization of control. Malicious actions or security breaches involving the owner's account could lead to unauthorized asset transfers or manipulation of contract settings.
2. **Single Point of Failure**: The reliance on specific addresses (e.g., the EntryPoint address in `MagicSpend`) introduces central points of failure. Should these addresses be compromised or become inactive, it could disrupt the system's functionality.

**Systemic Risks**

1. **External Dependency**: The contracts rely heavily on external contracts and libraries (e.g., `SafeTransferLib`, `SignatureCheckerLib`). Changes or vulnerabilities in these dependencies could adversely affect the system's security and operability.
2. **Blockchain-Specific Risks**: The contracts are exposed to risks inherent to the blockchain they are deployed on, including but not limited to, network congestion, changes in gas prices, and major forks. These factors could impact transaction costs and execution times, affecting user experience and contract reliability.

**Architecture Risks**

1. **Upgradeability**: While upgradeability (via UUPS pattern in `CoinbaseSmartWallet`) ensures flexibility and the ability to fix bugs, it also introduces risks. Improperly managed upgrades could introduce vulnerabilities, disrupt service, or inadvertently change critical business logic.
2. **Cross-Contract Interactions**: The system's reliance on interactions between multiple contracts (e.g., the interaction between `MagicSpend` and the EntryPoint contract) increases complexity and the potential for unintended behaviors, especially when contracts evolve independently over time.

**Complexity Risks**

1. **Smart Contract Interactions**: The architecture involves multiple smart contracts with intricate interactions, including fund management, stake management, and execution of operations. The complexity of these interactions increases the likelihood of bugs or vulnerabilities, potentially leading to loss of funds or unauthorized access.
2. **Signature and Authentication Mechanisms**: The system's security heavily relies on signature verification and authentication mechanisms (e.g., `ERC1271`, `WebAuthn` in verification). Complexity in these areas could lead to vulnerabilities, especially with non-standard signature formats or when integrating with external authentication systems.

Each of these risk areas requires careful consideration and ongoing monitoring to mitigate potential adverse impacts on the system and its users. Regular audits, thorough testing, and a robust security framework are critical to managing these risks effectively.

## Conclusion

The smart contract system presents a sophisticated and flexible infrastructure for managing digital assets, authentication, and execution of operations within a blockchain environment. While it leverages advanced features like upgradeability and external dependencies to enhance functionality, it also introduces centralization, systematic, architectural, and complexity risks that necessitate rigorous security measures, including audits and testing. Overall, the system's success hinges on balancing innovation with robust risk management practices to ensure security and reliability.

### Time spent

13 hours

**[wilsoncusack (sponsor) acknowledged](https://github.com/code-423n4/2024-03-coinbase-findings/issues/130#issuecomment-2020513310)**

***

# [Mitigation Review](#mitigation-review)

## Introduction

Following the C4 audit, 3 wardens ([McToady](https://code4rena.com/@McToady), [imare](https://code4rena.com/@imare), and [cheatc0d3](https://code4rena.com/@cheatc0d3)) reviewed the mitigations for all identified issues. Additional details can be found within the [C4 Coinbase Mitigation Review repository](https://github.com/code-423n4/2024-04-coinbase-mitigation).

## Mitigation Review Scope

### Mitigations of High & Medium Severity Issues

**[Summary from the Sponsor](https://github.com/code-423n4/2024-04-coinbase-mitigation/blob/main/README.md#scope):**

- [H-01 Fix](https://github.com/coinbase/smart-wallet/pull/42): The issue is remediated by updating the parameterization of `removeOwnerAtIndex` to also take an `owner` argument. We then check that the `owner` passed matches the owner found at the index. In this way, we prevent a replayable transaction removing a different owner at the same index. 
- [M-01 Fix](https://github.com/coinbase/magic-spend/pull/17): This issue is complex to address. The warden suggested adding a variable to track in flight withdraws, and we [pursued this](https://github.com/coinbase/magic-spend/pull/16). However, we realized that bundlers penalize paymasters when the UserOp behaves differently when simulated in isolation vs. in the bundle, and this would not fix this. Instead, we give the owner a tool to address this probabilistically: the owner can set a `maxWithdrawDenominator` and we enforce that native asset withdraws must be `<= address(this).balance / maxWithdrawDenominator`. For example, if `maxWithdrawDenominator` is set to 20, it would take 20 native asset withdraws (each withdrawing max allowed) + 1 native asset withdraw in the same transaction to cause a revert. It is of course known that this doesn't entirely solve the issue, and the efficacy depends the value chosen and usage. 

### Additional Scope to be Reviewed

These are additional changes that will be in scope.  

| Reference ID | URL | Original Issue | Notes |
| ----------- | ------------- | ----------- | ----------- |
| QA-01 | https://github.com/coinbase/smart-wallet/pull/43 | [181](https://github.com/code-423n4/2024-03-coinbase-findings/issues/181) | We decided to take action here, changing `removeOwnerAtIndex` to revert if the owner is the last owner and adding `removeLastOwner`. |
| ADD-01 | https://github.com/coinbase/magic-spend/pull/18#pullrequestreview-1981188702 | [195](https://github.com/code-423n4/2024-03-coinbase-findings/issues/195) and [137](https://github.com/code-423n4/2024-03-coinbase-findings/issues/195) | Gas fixes. |
| ADD-02| https://github.com/coinbase/smart-wallet/pull/45 | [195](https://github.com/code-423n4/2024-03-coinbase-findings/issues/195) and [38](https://github.com/code-423n4/2024-03-coinbase-findings/issues/38) | Gas fixes. |

### Out of Scope

We are not taking action on [M-02](https://github.com/code-423n4/2024-03-coinbase-findings/issues/39).

## Mitigation Review Summary

During the mitigation review, the wardens confirmed that all in-scope findings were mitigated.

| Original Issue | Status | Full Details |
| --- | --- | --- |
| H-01 |  🟢 Mitigation Confirmed | Reports from  [McToady](https://github.com/code-423n4/2024-04-coinbase-mitigation-findings/issues/6), [cheatc0d3](https://github.com/code-423n4/2024-04-coinbase-mitigation-findings/issues/13), and [imare](https://github.com/code-423n4/2024-04-coinbase-mitigation-findings/issues/8) |
| M-01 |  🟢 Mitigation Confirmed | Reports from  [imare](https://github.com/code-423n4/2024-04-coinbase-mitigation-findings/issues/7) and [McToady](https://github.com/code-423n4/2024-04-coinbase-mitigation-findings/issues/5) |
| QA-01 |  🟢 Mitigation Confirmed | Reports from  [imare](https://github.com/code-423n4/2024-04-coinbase-mitigation-findings/issues/9), [cheatc0d3](https://github.com/code-423n4/2024-04-coinbase-mitigation-findings/issues/15), and [McToady](https://github.com/code-423n4/2024-04-coinbase-mitigation-findings/issues/4) |
| ADD-01 |  🟢 Mitigation Confirmed | Reports from  [cheatc0d3](https://github.com/code-423n4/2024-04-coinbase-mitigation-findings/issues/16), [imare](https://github.com/code-423n4/2024-04-coinbase-mitigation-findings/issues/10), and [McToady](https://github.com/code-423n4/2024-04-coinbase-mitigation-findings/issues/3) |
| ADD-02 |  🟢 Mitigation Confirmed | Reports from  [cheatc0d3](https://github.com/code-423n4/2024-04-coinbase-mitigation-findings/issues/17), [imare](https://github.com/code-423n4/2024-04-coinbase-mitigation-findings/issues/11), and [McToady](https://github.com/code-423n4/2024-04-coinbase-mitigation-findings/issues/2) |



***

# Disclosures

C4 is an open organization governed by participants in the community.

C4 audits incentivize the discovery of exploits, vulnerabilities, and bugs in smart contracts. Security researchers are rewarded at an increasing rate for finding higher-risk issues. Audit submissions are judged by a knowledgeable security researcher and solidity developer and disclosed to sponsoring developers. C4 does not conduct formal verification regarding the provided code but instead provides final verification.

C4 does not provide any guarantee or warranty regarding the security of this project. All smart contract software should be used at the sole risk and responsibility of users.
