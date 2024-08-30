Muscular Jade Hedgehog

Medium

# Discrepancy between spec and code: Vault admin cannot update `tokensReceiver`.


## Summary

Vault admin cannot update `tokensReceiver`. However, the specs claim that admin should be able to update it.

## Vulnerability Detail

In the [specs](https://ludicrous-rate-748.notion.site/8060186191934380800b669406f4d83c?v=35634cda6b084e2191b83f295433efdf&p=42afde9e098b42ef8296e43286b73299&pm=s), we can see that the "Investment recipient address" is an address the admin can set and **adjust**.

> This is the list of parameters that the admin can set and adjust

> Investment recipient address

> This is the address that receives the funds sent by investors when requesting a subscription (either instant or standard)

However, the `tokensReceiver` variable can only be set during the initializer, and there is no set function for it. Note that all other variables listed in the spec has a setter function, except for `tokensReceiver`.

This is a discrepancy between code and specs where the admin does not have the ability to update the investment recipient address as expected.

```solidity
    /**
     * @notice address to which tokens and mTokens will be sent
     */
    address public tokensReceiver;

    function __ManageableVault_init(
        address _ac,
        MTokenInitParams calldata _mTokenInitParams,
        ReceiversInitParams calldata _receiversInitParams,
        InstantInitParams calldata _instantInitParams,
        address _sanctionsList,
        uint256 _variationTolerance,
        uint256 _minAmount
    ) internal onlyInitializing {
        _validateAddress(_mTokenInitParams.mToken, false);
        _validateAddress(_mTokenInitParams.mTokenDataFeed, false);
@>      _validateAddress(_receiversInitParams.tokensReceiver, true);
        _validateAddress(_receiversInitParams.feeReceiver, true);
        require(_instantInitParams.instantDailyLimit > 0, "zero limit");
        _validateFee(_variationTolerance, true);
        _validateFee(_instantInitParams.instantFee, false);

        mToken = IMTbill(_mTokenInitParams.mToken);
        __Pausable_init(_ac);
        __Greenlistable_init_unchained();
        __Blacklistable_init_unchained();
        __WithSanctionsList_init_unchained(_sanctionsList);

@>      tokensReceiver = _receiversInitParams.tokensReceiver;
        feeReceiver = _receiversInitParams.feeReceiver;
        instantFee = _instantInitParams.instantFee;
        instantDailyLimit = _instantInitParams.instantDailyLimit;
        minAmount = _minAmount;
        variationTolerance = _variationTolerance;
        mTokenDataFeed = IDataFeed(_mTokenInitParams.mTokenDataFeed);
    }
```

## Impact

1. Admin cannot set `tokensReceiver` after contract initialization.
2. Discrepancy between spec and code.

## Code Snippet

- https://github.com/sherlock-audit/2024-08-midas-minter-redeemer/blob/main/midas-contracts/contracts/abstract/ManageableVault.sol#L76

## Tool used

Manual Review

## Recommendation

Create a setter function for `tokensReceiver`.