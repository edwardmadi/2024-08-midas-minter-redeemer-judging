Muscular Jade Hedgehog

Medium

# Standard redemption in `RedemptionVault` does not update token allowance.


## Summary

Standard redemption in `RedemptionVault` does not update token allowance.

## Vulnerability Detail

From the specs, we can know that there is a token allowance for redeeming.

- [Users can request a standard redemption](https://ludicrous-rate-748.notion.site/8060186191934380800b669406f4d83c?v=35634cda6b084e2191b83f295433efdf&p=b4fc4f41370a40629c2b2b8d767eb7e3&pm=s)
- [Admin manages the list of token_in and token_out](https://ludicrous-rate-748.notion.site/8060186191934380800b669406f4d83c?v=35634cda6b084e2191b83f295433efdf&p=477e48c7688b443aa496a669134725f0&pm=s)

> **Caps**

> - Yes

> redeem cap (allowance based, set in token_out)

However, this allowance is not respected during standard redemption process.

```solidity
    function redeemRequest(address tokenOut, uint256 amountMTokenIn)
        external
        whenFnNotPaused(this.redeemRequest.selector)
        onlyGreenlisted(msg.sender)
        onlyNotBlacklisted(msg.sender)
        onlyNotSanctioned(msg.sender)
        returns (uint256 requestId)
    {
        require(tokenOut != MANUAL_FULLFILMENT_TOKEN, "RV: tokenOut == fiat");
        return _redeemRequest(tokenOut, amountMTokenIn);
    }

    function _redeemRequest(address tokenOut, uint256 amountMTokenIn)
        internal
        returns (uint256)
    {
        address user = msg.sender;

        bool isFiat = tokenOut == MANUAL_FULLFILMENT_TOKEN;

        (
            uint256 feeAmount,
            uint256 amountMTokenWithoutFee
        ) = _calcAndValidateRedeem(
                user,
                tokenOut,
                amountMTokenIn,
                false,
                isFiat
            );

        address tokenOutCopy = tokenOut;

        uint256 tokenOutRate;
        if (!isFiat) {
            TokenConfig storage config = tokensConfig[tokenOutCopy];
            tokenOutRate = _getTokenRate(config.dataFeed, config.stable);
        }

        uint256 amountMTokenInCopy = amountMTokenIn;

@>      // BUG: _requireAndUpdateAllowance is not called anywhere.

        uint256 mTokenRate = mTokenDataFeed.getDataInBase18();

        _tokenTransferFromUser(
            address(mToken),
            address(this),
            amountMTokenWithoutFee,
            18 // mToken always have 18 decimals
        );
        if (feeAmount > 0)
            _tokenTransferFromUser(address(mToken), feeReceiver, feeAmount, 18);

        uint256 requestId = currentRequestId.current();
        currentRequestId.increment();

        redeemRequests[requestId] = Request({
            sender: user,
            tokenOut: tokenOutCopy,
            status: RequestStatus.Pending,
            amountMToken: amountMTokenWithoutFee,
            mTokenRate: mTokenRate,
            tokenOutRate: tokenOutRate
        });

        emit RedeemRequest(requestId, user, tokenOutCopy, amountMTokenInCopy);

        return requestId;
    }
```

To make a comparison, the standard deposit updates the allowance for tokenIn.

```solidity
    function _calcAndValidateDeposit(
        address user,
        address tokenIn,
        uint256 amountToken,
        bool isInstant
    )
        internal
        returns (
            uint256 tokenAmountInUsd,
            uint256 feeTokenAmount,
            uint256 amountTokenWithoutFee,
            uint256 mintAmount,
            uint256 tokenInRate,
            uint256 tokenOutRate,
            uint256 tokenDecimals
        )
    {
        ...
        _requireAndUpdateAllowance(tokenIn, amountToken);
        ...
    }
```


## Impact

1. User can bypass the allowance of token allowance during standard redemption.
2. Code does not align with spec (Note that contest readme claims "discrepancies between the spec and the code can be reported as issues").

## Code Snippet

- https://github.com/sherlock-audit/2024-08-midas-minter-redeemer/blob/main/midas-contracts/contracts/RedemptionVault.sol#L372-L428

## Tool used

Manual Review

## Recommendation

Call `_requireAndUpdateAllowance(tokenIn, amountToken);` in `_redeemRequest`. Also check that is it a non-fiat redeem.