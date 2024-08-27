Muscular Jade Hedgehog

Medium

# RedemptionVaults should not include fees when updating allowance.


## Summary

RedemptionVaults should not include fees when updating allowance. Allowance would be deducted more than expected.

## Vulnerability Detail

When redeeming mTokens, the user needs to specify which `tokenOut` he wants to receive. The allowance limit for the `tokenOut` is updated accordingly.

The issue here is, when users are redeeming mTokens, a portion of mToken is taken as fees (which sent to `feeReceiver`). But when reducting the allowance of `tokenOut`, the mToken fees are also included. An example is:

1. User redeems 100k mTBill (say it is worth 200k USDC).
2. The fee is set to 2%, so 2k mTBill is sent to `feeReceiver`.
3. 98k mTBill is burned, and 196k USDC is sent to user.

The issue here is, 200k USDC is deducted from the allowance instead of 196k. Users only received 196k of USDC, so it does not make sense to deduct 200k USDC. Admins could have used the allowance parameter to limit the total amount of USDC sent to users, but now they can't.

Note this issue occurs in all redemption vaults, including `RedemptionVaultWithBUIDL` and `MBasisRedemptionVaultWithSwapper`.

```solidity
    function redeemInstant(
        address tokenOut,
        uint256 amountMTokenIn,
        uint256 minReceiveAmount
    )
        external
        virtual
        whenFnNotPaused(this.redeemInstant.selector)
        onlyGreenlisted(msg.sender)
        onlyNotBlacklisted(msg.sender)
        onlyNotSanctioned(msg.sender)
    {
        address user = msg.sender;

        (
            uint256 feeAmount,
            uint256 amountMTokenWithoutFee
        ) = _calcAndValidateRedeem(user, tokenOut, amountMTokenIn, true, false);

        _requireAndUpdateLimit(amountMTokenIn);

        uint256 tokenDecimals = _tokenDecimals(tokenOut);

        uint256 amountMTokenInCopy = amountMTokenIn;
        address tokenOutCopy = tokenOut;
        uint256 minReceiveAmountCopy = minReceiveAmount;

        (uint256 amountMTokenInUsd, uint256 mTokenRate) = _convertMTokenToUsd(
            amountMTokenInCopy
        );
        (uint256 amountTokenOut, uint256 tokenOutRate) = _convertUsdToToken(
            amountMTokenInUsd,
            tokenOutCopy
        );

        uint256 amountTokenOutWithoutFee = _truncate(
            (amountMTokenWithoutFee * mTokenRate) / tokenOutRate,
            tokenDecimals
        );

        require(
            amountTokenOutWithoutFee >= minReceiveAmountCopy,
            "RV: minReceiveAmount > actual"
        );

@>      _requireAndUpdateAllowance(tokenOutCopy, amountTokenOut);

        mToken.burn(user, amountMTokenWithoutFee);
        if (feeAmount > 0)
@>          _tokenTransferFromUser(address(mToken), feeReceiver, feeAmount, 18);

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

Allowance would be deducted more than expected.

## Code Snippet

- https://github.com/sherlock-audit/2024-08-midas-minter-redeemer/blob/main/midas-contracts/contracts/RedemptionVault.sol#L169
- https://github.com/sherlock-audit/2024-08-midas-minter-redeemer/blob/main/midas-contracts/contracts/RedemptionVaultWithBUIDL.sol#L123
- https://github.com/sherlock-audit/2024-08-midas-minter-redeemer/blob/main/midas-contracts/contracts/mBasis/MBasisRedemptionVaultWithSwapper.sol#L143

## Tool used

Manual Review

## Recommendation

Change to `_requireAndUpdateAllowance(tokenOutCopy, amountTokenOutWithoutFee);`.