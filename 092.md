Winning Jetblack Salmon

Medium

# Multiple instances of calculations with incorrect rounding directions that are unfavorable to the protocol.

## Summary

Within all the contracts, all calculations are rounded down, even when such rounding favors the user. This is incorrect, as all roundings in calculations should favor the protocol.

## Vulnerability Details

There are multiple instances of this issue:

1. Within the `_getFeeAmount()` function, the rounding should be up.

```solidity
File: ManageableVault.sol
540:         return (amount * feePercent) / ONE_HUNDRED_PERCENT; // <== should be rounding up
```

The fee calculations should always round the value up in favor of the protocol.

2. `feeTokenAmount` is truncated prematurely, leading to rounding in favor of the user.

```solidity
File: DepositVault.sol
379:         feeTokenAmount = _truncate( // <== truncated prematurely as feeTokenAmount is used as a multiplier for feeInUsd calculation
380:             _getFeeAmount(userCopy, tokenIn, amountToken, isInstant, 0),
381:             tokenDecimals
382:         );
383:         amountTokenWithoutFee = amountToken - feeTokenAmount; // <== should _truncate() here only
384: 
385:         uint256 feeInUsd = (feeTokenAmount * tokenInRate) / 10**18; // <== feeTokenAmount used as a multiplier in fee amount calculation
```

3. Rounding down in the `feeInUsd` calculation.

```solidity
File: DepositVault.sol
385:         uint256 feeInUsd = (feeTokenAmount * tokenInRate) / 10**18; // <== should be rounding up in favor of the protocol.
```

## Impact

Incorrect rounding will result in higher output amounts, favoring the user over the protocol.

## Code Snippet

https://github.com/sherlock-audit/2024-08-midas-minter-redeemer/blob/main/midas-contracts/contracts/abstract/ManageableVault.sol#L540  
https://github.com/sherlock-audit/2024-08-midas-minter-redeemer/blob/main/midas-contracts/contracts/DepositVault.sol#L379-L385  

## Tools Used

Manual review.

## Recommendations

Round the value up instead of down in all mentioned instances.