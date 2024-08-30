Muscular Jade Hedgehog

Medium

# No slippage parameter for standard deposit request.


## Summary

No slippage parameter for standard deposit request. Users may receive less mTokens than they expect.

## Vulnerability Detail

When users are depositing tokens to acquire mTokens, there are two paths: 1) instant deposit, 2) standard deposit.

- During instant deposit, users will receive mTokens immediately, based on the inToken and mToken rate at the time.
- During standard deposit, users will need to wait for admins to approve the request.

Users may fallback to using a standard deposit if the daily limit is hit for instant deposit.

The issue here is that there is a slippage parameter limiting the amount of mTokens users receive for instant deposit, but not for standard deposit. This is an issue, especially considering the tokenIn rate is calculated during the deposit request transaction, and that there is no slippage parameter to control it. Example:

1. User calls `depositRequest()` to deposit WBTC for mToken.
2. The deposit transaction takes a while to process, and WBTC price crashes during the time (and recovers later).
3. The transaction is processed, but the `tokenInRate` is now much lower than when the user initiated the deposit. This will lead to users receiving less mToken in the end.

During the process, there is no mechanism for user to set a slippage parameter nor cancel the request.

Note that the specs did not mention slippage is required for standard deposit. However, this doesn't mean it doesn't pose a vulnerability.

```solidity
    function depositRequest(
        address tokenIn,
        uint256 amountToken,
        bytes32 referrerId
    )
        external
        whenFnNotPaused(this.depositRequest.selector)
        onlyGreenlisted(msg.sender)
        onlyNotBlacklisted(msg.sender)
        onlyNotSanctioned(msg.sender)
        returns (uint256 requestId)
    {
        address user = msg.sender;

        address tokenInCopy = tokenIn;
        uint256 amountTokenCopy = amountToken;
        bytes32 referrerIdCopy = referrerId;

        uint256 currentId = currentRequestId.current();
        requestId = currentId;
        currentRequestId.increment();

        (
            uint256 tokenAmountInUsd,
            uint256 feeAmount,
            uint256 amountTokenWithoutFee,
            ,
@>          uint256 tokenInRate,
            uint256 tokenOutRate,
            uint256 tokenDecimals
        ) = _calcAndValidateDeposit(user, tokenInCopy, amountTokenCopy, false);

        _tokenTransferFromUser(
            tokenInCopy,
            tokensReceiver,
            amountTokenWithoutFee,
            tokenDecimals
        );

        if (feeAmount > 0)
            _tokenTransferFromUser(
                tokenInCopy,
                feeReceiver,
                feeAmount,
                tokenDecimals
            );

        mintRequests[currentId] = Request({
            sender: user,
            tokenIn: tokenInCopy,
            status: RequestStatus.Pending,
            depositedUsdAmount: tokenAmountInUsd,
@>          usdAmountWithoutFees: (amountTokenWithoutFee * tokenInRate) /
                10**18,
            tokenOutRate: tokenOutRate
        });

        emit DepositRequest(
            currentId,
            user,
            tokenInCopy,
            tokenAmountInUsd,
            feeAmount,
            tokenOutRate,
            referrerIdCopy
        );
    }
```

## Impact

Users may receive less mTokens than they expect.

## Code Snippet

- https://github.com/sherlock-audit/2024-08-midas-minter-redeemer/blob/main/midas-contracts/contracts/DepositVault.sol#L148

## Tool used

Manual Review

## Recommendation

Also add a slippage parameter for standard deposits to limit the minimum amount of mToken received.

When admins approve the deposit request, check the amount of mTokens against the number. If it is impossible to fulfill the request, the admin can just reject the request.