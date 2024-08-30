Refined Aquamarine Barracuda

High

# Lack of refund mechanism will bring financial losses to owners of rejected deposit requests

### Summary

The deposit request process lacks a refund mechanism. Specifically, if a user requests a deposit via the depositRequest() function but the request is rejected, the user will not get refunded with the deposited value.

### Root Cause

The rejectRequest function doesn't refund the deposited tokens.

https://github.com/sherlock-audit/2024-08-midas-minter-redeemer/blob/main/midas-contracts/contracts/DepositVault.sol#L243-L255

### Internal pre-conditions

_No response_

### External pre-conditions

_No response_

### Attack Path

_No response_

### Impact

_No response_

### PoC

_No response_

### Mitigation

_No response_