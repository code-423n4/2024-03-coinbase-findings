
# L-2: `_validateRequest()` doesn't check against the TX expiration.
# L-3:  Make use of `non-reentrant` guards on functions sending eth or other tokens.
# L-4: Operations that allow replayability on multiple chains can be replayed multiple times on a specific chain if not using the MagicSpender as its paymaster. 
# L-1: Griefing as an attacker can front-run the call to create the CSW Account leading to bad user UX.
# L-5: Solady's `isValidSignatureNow()` is prone to signature malleability.
# L-6: Newly created accounts should depend on user's signature.

# Info-1: `safeTransfer()` doesn't check contracts existence.