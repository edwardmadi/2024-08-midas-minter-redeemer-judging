Winning Jetblack Salmon

Medium

# Contradiction between the Specification and the Code in the `MBasisRedemptionVaultWithSwapper` contract.

## Summary

The instant redemption fee should be given to the LP if the redemption is performed via swap functionality. However, in the current code, the fee is transferred to the `feeReceiver` in all cases of redemption.

## Vulnerability Details

There is a contradiction between the code and the specification: https://ludicrous-rate-748.notion.site/Special-Contract-Version-User-can-instantly-redeem-mBASIS-for-USDC-via-a-mBASIS-mTBILL-swap-and-an-57e8d19b1c2242e8af50db5c8592532b

```text
# Specific features

- ...
- ... , the liquidity provider will receive most of the fees (instant redemption feature applied on mBASIS).
```

In the current code, fees are transferred to the `feeReceiver` in all cases:

```solidity
File: MBasisRedemptionVaultWithSwapper.sol
131:         if (feeAmount > 0)
132:             _tokenTransferFromUser(address(mToken), feeReceiver, feeAmount, 18); // <== contradiction with specification, incorrect fee distribution
```

This issue results in incorrect fee distribution in the `MBasisRedemptionVaultWithSwapper` contract.

## Impact

Contradiction with the specification. Incorrect fee distribution.

As stated by the protocol in the README: `Please note that discrepancies between the spec and the code can be reported as issues`.

## Code Snippet

https://github.com/sherlock-audit/2024-08-midas-minter-redeemer/blob/main/midas-contracts/contracts/mBasis/MBasisRedemptionVaultWithSwapper.sol#L131-L132

## Tools used

Manual review.

## Recommendations

Update the specification or the code to ensure consistency.