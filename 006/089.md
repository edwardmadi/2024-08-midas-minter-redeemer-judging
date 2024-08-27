Winning Jetblack Salmon

High

# Outdated `tokenOutRate` used during admin fulfillment of a standard redemption request.

## Summary

During the fulfillment of a standard redemption request, two separate exchange rates are used to calculate the withdrawal amount. 

However, only one of them (`mTokenRate`) is updated by the admin when performing the fulfillment of a standard redemption request.

Failing to update the `tokenOutRate` can lead to a loss of funds for the user or the protocol, depending on the price change of the redemption token that can occur between the time of when redemption request was made and its fulfillment.

## Vulnerability Details

In contrast to the `DepositVault`, where only one exchange rate is needed during the deposit request fulfillment, the `RedemptionVault` uses two exchange rates to calculate the withdrawal amount. 

This can be seen in the following piece of code:

```solidity
File: RedemptionVault.sol
331:             uint256 amountTokenOutWithoutFee = _truncate(
332:                 (request.amountMToken * newMTokenRate) / request.tokenOutRate, // <== outdated tokenOutRate used
333:                 tokenDecimals
334:             );
```

As shown, only the `mTokenRate` was updated, as `newMTokenRate` is used for calculations. 

The missing update of the `tokenOutRate` can lead to a loss of funds due to under- or over-redemption of the output token.

Consider the following scenario:
1. Alice requests a standard redemption of $100,000 worth of mTokens (mTBILL or mBASIS) into WBTC.
2. There is a delay of a few hours between the redemption request and the fulfillment of the request.
3. During that time, the `MTokenRate` does not change, but the price of WBTC increases by 10%.
4. During the fulfillment, the admin provides the `newMTokenRate`, he can even use `safeApproveRequest()` since the redemption is within the `variation tolerance` of the mToken exchange rate. However, due to the use of the outdated `tokenOutRate`, the withdrawal amount is now 10% greater than the current valuation of the mToken/WBTC.

As demonstrated, the protocol loses funds due to not updating the `tokenOutRate` during fulfillment. 

A similar loss can also occur for the user if the price of the output token drops between the time of the request and the redemption.

## Impact

Using an incorrect exchange rate during withdrawal amount calculation leads to fund losses for the protocol or the user.

## Code Snippet

https://github.com/sherlock-audit/2024-08-midas-minter-redeemer/blob/main/midas-contracts/contracts/RedemptionVault.sol#L331-L334

## Tools used

Manual review.

## Recommendations

In addition to providing the `newMTokenRate` as a parameter for `_approveRequest()`, an additional `newTokenOutRate` parameter should be introduced, and similarly to the `safeApproveRequest()` and `approveRequest()` functions.

In the case of `safeApproveRequest()`, the `newTokenOutRate` should also be validated for `variation tolerance`.