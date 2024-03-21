# [L-01] some reverts in testcases.
File:
https://github.com/code-423n4/2024-03-coinbase/blob/e0573369b865d47fed778de00a7b6df65ab1744e/src/FreshCryptoLib/FCL.sol#L344-L369

According to the [audit report](https://github.com/base-org/FCL-ecdsa-verify-audit/blob/main/docs/secp256r1-ecdsa-verify-solidity-review.pdf), I re-run the fuzzing test after the patches in [README.md](https://github.com/code-423n4/2024-03-coinbase/blob/main/README.md).
But the testcases raise some assert failure, please check the following example
```bash
forge test --ffi
[⠢] Compiling...
No files changed, compilation skipped

Ran 1 test for test/IsOnCurve.t.sol:IsOnCurve
[PASS] test_ecAff_isOnCurve() (gas: 776584361)
Suite result: ok. 1 passed; 0 failed; 0 skipped; finished in 10.30s (10.30s CPU time)


Ran 13 tests for test/FCL.t.sol:FCLTest
[PASS] test_basecasesFC_nModInv() (gas: 19780)
[PASS] test_basecasesFC_pModInv() (gas: 20002)
[PASS] test_ecAff_add(uint256,uint256) (runs: 10007, μ: 1151740, ~: 1030564)
[FAIL. Reason: assertion failed; counterexample: calldata=0x8857cfbb00000000000000000000000000000000000000000000000000000000000005900000000000000000000000000000000000000000000000000000000000000d3cffffffff00000001000000000000000000000000ffffffffffffffffffffffff args=[1424, 3388, 115792089210356248762697446949407573530086143415290314195533631308867097853951 [1.157e77]]] test_ecZZ_AddN(uint256,uint256,uint256) (runs: 6, μ: 1031438, ~: 904248)
[FAIL. Reason: assertion failed; counterexample: calldata=0x81f7fd2300000000000000000000000000000000000000000000000000000000000005900000000000000000000000000000000000000000000000000000000000000d3cffffffff00000001000000000000000000000000ffffffffffffffffffffffff args=[1424, 3388, 115792089210356248762697446949407573530086143415290314195533631308867097853951 [1.157e77]]] test_ecZZ_AddN_inline(uint256,uint256,uint256) (runs: 6, μ: 1021375, ~: 893891)
[FAIL. Reason: assertion failed; counterexample: calldata=0xff980cf100000000000000000000000000000000000000000000000000000000000003fbffffffff00000001000000000000000000000000ffffffffffffffffffffffff args=[1019, 115792089210356248762697446949407573530086143415290314195533631308867097853951 [1.157e77]]] test_ecZZ_Dbl(uint256,uint256) (runs: 7, μ: 527197, ~: 458785)
[FAIL. Reason: assertion failed; counterexample: calldata=0x511b911f00000000000000000000000000000000000000000000000000000000000003fbffffffff00000001000000000000000000000000ffffffffffffffffffffffff args=[1019, 115792089210356248762697446949407573530086143415290314195533631308867097853951 [1.157e77]]] test_ecZZ_Dbl_impl1(uint256,uint256) (runs: 7, μ: 509781, ~: 448807)
[PASS] test_ecZZ_Dbl_impl2(uint256,uint256) (runs: 10007, μ: 647476, ~: 655457)
[FAIL. Reason: assertion failed; counterexample: calldata=0x58d28bc700000000000000000000000000000000000000000000000000000000000005900000000000000000000000000000000000000000000000000000000000000d3cffffffff00000001000000000000000000000000ffffffffffffffffffffffff args=[1424, 3388, 115792089210356248762697446949407573530086143415290314195533631308867097853951 [1.157e77]]] test_ecZZ_mulmod(uint256,uint256,uint256) (runs: 6, μ: 498687, ~: 454944)
[PASS] test_ecZZ_mulmuladd_S_asm(uint256,uint256,uint256) (runs: 10007, μ: 1130643, ~: 1009492)
[PASS] test_fuzzFC_nModInv(uint256) (runs: 10007, μ: 1928, ~: 1929)
[PASS] test_fuzzFC_pModInv(uint256) (runs: 10007, μ: 2392, ~: 2393)
[PASS] test_mulmuladd_S_bugcheck() (gas: 85228)
Suite result: FAILED. 8 passed; 5 failed; 0 skipped; finished in 197.21s (474.56s CPU time)

Ran 2 test suites in 197.21s (207.50s CPU time): 9 tests passed, 5 failed, 0 skipped (14 total tests)

Failing tests:
Encountered 5 failing tests in test/FCL.t.sol:FCLTest
[FAIL. Reason: assertion failed; counterexample: calldata=0x8857cfbb00000000000000000000000000000000000000000000000000000000000005900000000000000000000000000000000000000000000000000000000000000d3cffffffff00000001000000000000000000000000ffffffffffffffffffffffff args=[1424, 3388, 115792089210356248762697446949407573530086143415290314195533631308867097853951 [1.157e77]]] test_ecZZ_AddN(uint256,uint256,uint256) (runs: 6, μ: 1031438, ~: 904248)
[FAIL. Reason: assertion failed; counterexample: calldata=0x81f7fd2300000000000000000000000000000000000000000000000000000000000005900000000000000000000000000000000000000000000000000000000000000d3cffffffff00000001000000000000000000000000ffffffffffffffffffffffff args=[1424, 3388, 115792089210356248762697446949407573530086143415290314195533631308867097853951 [1.157e77]]] test_ecZZ_AddN_inline(uint256,uint256,uint256) (runs: 6, μ: 1021375, ~: 893891)
[FAIL. Reason: assertion failed; counterexample: calldata=0xff980cf100000000000000000000000000000000000000000000000000000000000003fbffffffff00000001000000000000000000000000ffffffffffffffffffffffff args=[1019, 115792089210356248762697446949407573530086143415290314195533631308867097853951 [1.157e77]]] test_ecZZ_Dbl(uint256,uint256) (runs: 7, μ: 527197, ~: 458785)
[FAIL. Reason: assertion failed; counterexample: calldata=0x511b911f00000000000000000000000000000000000000000000000000000000000003fbffffffff00000001000000000000000000000000ffffffffffffffffffffffff args=[1019, 115792089210356248762697446949407573530086143415290314195533631308867097853951 [1.157e77]]] test_ecZZ_Dbl_impl1(uint256,uint256) (runs: 7, μ: 509781, ~: 448807)
[FAIL. Reason: assertion failed; counterexample: calldata=0x58d28bc700000000000000000000000000000000000000000000000000000000000005900000000000000000000000000000000000000000000000000000000000000d3cffffffff00000001000000000000000000000000ffffffffffffffffffffffff args=[1424, 3388, 115792089210356248762697446949407573530086143415290314195533631308867097853951 [1.157e77]]] test_ecZZ_mulmod(uint256,uint256,uint256) (runs: 6, μ: 498687, ~: 454944)

Encountered a total of 5 failing tests, 9 tests succeeded
```