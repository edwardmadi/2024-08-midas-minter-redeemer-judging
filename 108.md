Muscular Jade Hedgehog

Medium

# Discrepancy between spec and code: admins can waive deposit/redemption min amount limit for specific user.


## Summary

In the code, the vault admins can waive the min amount limit that is set for a specific user. However, this is not listed in the specs, which is a discrepancy between the spec and code.

## Vulnerability Detail

Please see this doc for the list of parameters the admin can set and adjust: https://ludicrous-rate-748.notion.site/8060186191934380800b669406f4d83c?v=35634cda6b084e2191b83f295433efdf&p=42afde9e098b42ef8296e43286b73299&pm=s.

There is a minimum limit of mTokens that users need to fulfill when depositing/redeeming tokens. Additionally, there may be a independent minimum for user's first deposit, and for user redeeming to fiat.

> The admin can adjust separately those minimums

> - 1st deposit. When we don’t have a prospectus, there is a minimum that needs to be enforced for the first deposit. This rule only applies for the first mintings (not for the first redemption). This can be set in TOKEN.
> - Minting. Some Products may have some operational headaches, and we may want to limit very small deposits (regardless if this is the first one or not). This can be set in TOKEN
> - for redemptions
>    - crypto_redemption. Similarly to the minting minimum, we want to limit redemptions to a minimum. This can be set in TOKEN.
>    - fiat_redemption. FIAT can be more costly, and requires its dedicated minimum. This can be set in TOKEN.

The issue is that vault admin can freely set `isFreeFromMinAmount[user]` to waive the user of this limit. However, this feature is not listed in the specs. This is considered as a discrepancy between spec and code, which the contest readme allows to submit: "Please note that discrepancies between the spec and the code can be reported as issues".

To make a comparison, the ability to waive fees is listed in the specs, and is implemented in code.

ManageableVault.sol
```solidity
    function freeFromMinAmount(address user, bool enable)
        external
        onlyVaultAdmin
    {
        require(isFreeFromMinAmount[user] != enable, "DV: already free");

@>      isFreeFromMinAmount[user] = enable;

        emit FreeFromMinAmount(user, enable);
    }

    function addWaivedFeeAccount(address account) external onlyVaultAdmin {
        require(!waivedFeeRestriction[account], "MV: already added");
        waivedFeeRestriction[account] = true;
        emit AddWaivedFeeAccount(account, msg.sender);
    }

    function removeWaivedFeeAccount(address account) external onlyVaultAdmin {
        require(waivedFeeRestriction[account], "MV: not found");
        waivedFeeRestriction[account] = false;
        emit RemoveWaivedFeeAccount(account, msg.sender);
    }
```

## Impact

Admins have larger permission than documented. Discrepancy between spec and code: admins can waive deposit/redemption min amount limit for specific user.

## Code Snippet

- https://github.com/sherlock-audit/2024-08-midas-minter-redeemer/blob/main/midas-contracts/contracts/abstract/ManageableVault.sol#L337
- https://github.com/sherlock-audit/2024-08-midas-minter-redeemer/blob/main/midas-contracts/contracts/RedemptionVault.sol#L496
- https://github.com/sherlock-audit/2024-08-midas-minter-redeemer/blob/main/midas-contracts/contracts/DepositVault.sol#L393

## Tool used

Manual Review

## Recommendation

Remove this feature, or add it in specs.