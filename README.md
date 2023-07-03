# Alien Worlds MSIG Helper

UI to allow custodians of planets to create new proposals.

If the action requested accepts an array of values then use JSON notification, eg `[1,2,3,4]`

## Examples

### Claim DTAL

**Contract** : awlndratings

**Action** : claimpay

**Receiver** : DAO account (eg. magor.dac)

### Transfer NFTs

**Contract** : atomicassets

**Action** : transfer

**From** : DAO account (eg. magor.dac)

**To** : receiving account

**Asset_ids** : JSON of all the assets to transfer, eg `["127343453645"]` or `["12387124343246", "122387463643"]` for multiple assets

**Memo** : Required memo