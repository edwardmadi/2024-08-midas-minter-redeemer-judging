Muscular Jade Hedgehog

Medium

# MBasisRedemptionVaultWithSwapper does not update mBasis daily limit or allowance when conducting mBasis->mTBill swap.


## Summary

MBasisRedemptionVaultWithSwapper does not update mBasis daily limit or allowance when conducting mBasis->mTBill swap.

## Vulnerability Detail

When users are redeeming from `MBasisRedemptionVaultWithSwapper`, if there is not enough `tokenOut` balance, it will first conduct a mBasis->mTBill swap and try to redeem from the mTBill redemption vault.

The issue here is, when this happens, the daily limit and token allowance is only deducted from the mTBill redemption vault, and not the original `MBasisRedemptionVaultWithSwapper` contract.

```solidity
    function redeemInstant(
        address tokenOut,
        uint256 amountMTokenIn,
        uint256 minReceiveAmount
    )
        external
        override(IRedemptionVault, RedemptionVault)
        whenFnNotPaused(this.redeemInstant.selector)
        onlyGreenlisted(msg.sender)
        onlyNotBlacklisted(msg.sender)
        onlyNotSanctioned(msg.sender)
    {
        ...
        uint256 contractTokenOutBalance = IERC20(tokenOutCopy).balanceOf(
            address(this)
        );

        if (
            contractTokenOutBalance >=
            amountTokenOutWithoutFee.convertFromBase18(tokenDecimals)
        ) {
@>          _requireAndUpdateLimit(amountMTokenInCopy);
            _requireAndUpdateAllowance(tokenOutCopy, amountTokenOut);

            mToken.burn(user, amountMTokenWithoutFee);
        } else {
            uint256 mTbillAmount = _swapMBasisToMToken(amountMTokenWithoutFee);

            IERC20(mTbillRedemptionVault.mToken()).safeIncreaseAllowance(
                address(mTbillRedemptionVault),
                mTbillAmount
            );

            mTbillRedemptionVault.redeemInstant(
                tokenOutCopy,
                mTbillAmount,
                minReceiveAmountCopy
            );

            uint256 contractTokenOutBalanceAfterRedeem = IERC20(tokenOutCopy)
                .balanceOf(address(this));
            amountTokenOutWithoutFee = (contractTokenOutBalanceAfterRedeem -
                contractTokenOutBalance).convertToBase18(tokenDecimals);
        }

        _tokenTransferToUser(
            tokenOutCopy,
            user,
            amountTokenOutWithoutFee,
            tokenDecimals
        );

        emit RedeemInstant(
            user,
            tokenOutCopy,
            amountMTokenInCopy,
            feeAmount,
            amountTokenOutWithoutFee
        );
    }
```

## Impact

mBasis daily limit and token allowance does not deduct as expected.

## Code Snippet

- https://github.com/sherlock-audit/2024-08-midas-minter-redeemer/blob/main/midas-contracts/contracts/mBasis/MBasisRedemptionVaultWithSwapper.sol#L146-L164

## Tool used

Manual Review

## Recommendation

Also update mBasis daily limit and allowance when conducting mBasis->mTBill swap for `MBasisRedemptionVaultWithSwapper`.