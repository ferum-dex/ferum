# Ferum Contracts

## Commands

Easiest way to interact with Ferum is through the Aptos CLI, and the easiest way to get the CLI is by downloading the latest & greatest 

### Publish to New Account

If you renamed or removed a module, most likely your contract will not be backwards compatible and you will need to publish to a new account.

A couple of quick steps you can follow to publish to a new acccount. 

1. Use [Petra Wallet](https://chrome.google.com/webstore/detail/petra-aptos-wallet/ejjladinnckdgjemekebdpeokbikhfci) to create a new account. 
2. Use an account with some test APT to fund it (1 APT is usually enough) — you can try to use the faucet, but 9/10, it's broken. 
4. Update the contract address in `Move.toml` to match the new account address. 
5. Export the private key from Petra and use the publish command below.

### Publish (if compatible) 

```
 aptos move publish \
    --private-key <...> \
    --max-gas 100000 --url https://fullnode.testnet.aptoslabs.com/v1 \
    --included-artifacts none
    --private-key <..> 
```

### Initialize

```
aptos move run \
  --function-id 0xb9a9606eeeb416eef9cd8650c34c0d16b7a650f207830a22d77e743c53d9902a::admin::init_ferum \
  --args u128:0 u128:0 u128:0 u128:0 \
  --url https://fullnode.testnet.aptoslabs.com/v1 \
  --private-key <..>
```

### Register

**APT / USDF**

```
aptos move run \
  --function-id 0xb9a9606eeeb416eef9cd8650c34c0d16b7a650f207830a22d77e743c53d9902a::admin::init_market_entry \
  --type-args 0x1::aptos_coin::AptosCoin 0xb9a9606eeeb416eef9cd8650c34c0d16b7a650f207830a22d77e743c53d9902a::test_coins::USDF \
  --args u8:4 u8:4 \
  --url https://fullnode.testnet.aptoslabs.com/v1 \
  --private-key <..>
```

**APT / ETHF**

```
aptos move run \
  --function-id 0xb9a9606eeeb416eef9cd8650c34c0d16b7a650f207830a22d77e743c53d9902a::market::init_market_entry \
  --type-args 0x1::aptos_coin::AptosCoin 0xb9a9606eeeb416eef9cd8650c34c0d16b7a650f207830a22d77e743c53d9902a::test_coins::ETHF \
  --args u8:4 u8:4 \
  --url https://fullnode.testnet.aptoslabs.com/v1 \
  --private-key <..>
```


### Add an order

```
aptos move run \
  --function-id 0xb9a9606eeeb416eef9cd8650c34c0d16b7a650f207830a22d77e743c53d9902a::market::add_order_entry \
  --type-args 0x1::aptos_coin::AptosCoin 0xb9a9606eeeb416eef9cd8650c34c0d16b7a650f207830a22d77e743c53d9902a::test_coins::USDF \
  --args u8:1 u8:1 u64:220000 u64:40000 string: \
  --url https://fullnode.testnet.aptoslabs.com/v1 \
  --private-key <..>
```

### Mint Fake Coins 


**USDF**

```
aptos move run \
  --function-id 0xb9a9606eeeb416eef9cd8650c34c0d16b7a650f207830a22d77e743c53d9902a::test_coins::create_usdf \
  --url https://fullnode.testnet.aptoslabs.com/v1 \
  --private-key <..>
```


```
aptos move run \
  --function-id 0xb9a9606eeeb416eef9cd8650c34c0d16b7a650f207830a22d77e743c53d9902a::test_coins::mint_usdf \
  --args u64:10000000000 \
  --url https://fullnode.testnet.aptoslabs.com/v1 \
  --private-key <..>
```

**ETHF**

```
aptos move run \
  --function-id 0xb9a9606eeeb416eef9cd8650c34c0d16b7a650f207830a22d77e743c53d9902a::test_coins::create_ethf \
  --url https://fullnode.testnet.aptoslabs.com/v1 \
  --private-key <..>
```

```
aptos move run \
  --function-id 0xb9a9606eeeb416eef9cd8650c34c0d16b7a650f207830a22d77e743c53d9902a::test_coins::mint_ethf \
  --args u64:10000000000 \
  --url https://fullnode.testnet.aptoslabs.com/v1 \
  --private-key <..>
```

## Troubleshooting 

### EPACKAGE_DEP_MISSING
```
{
  "Error": "API error: Unknown error Transaction committed on chain, but failed execution: Move abort in 0x1::code: EPACKAGE_DEP_MISSING(0x60005): Dependency could not be resolved to any published package."
}
```

Probably something happening in `Move.toml`. 

1. Check that you're not mixing `testnet` vs. `devent`, and that you're consistent i.e. if you're publishing to testnet, all dependencies must be on testnet as well.
2. Make sure that any custom dependencies are published to testnet / devnet. 


### EMODULE_MISSING
```
{
  "Error": "API error: Unknown error Transaction committed on chain, but failed execution: Move abort in 0x1::code: EMODULE_MISSING(0x4): Cannot delete a module that was published in the same package"
}
```

Contracts are upgradable, but they are [heavily restricted](https://aptos.dev/guides/move-guides/upgrading-move-code/). So if you deleted a public function or a module, you will need to publish to a new account. See **Publish to New Account** above.

### EXCEEDED_MAX_TRANSACTION_SIZE
```
{
  "Error": "API error: API error Error(VmError): Invalid transaction: Type: Validation Code: EXCEEDED_MAX_TRANSACTION_SIZE"
}
```

There is a rule around how large the module can be in Aptos; forgot the exact reason why, but until we wait for them to increase it, use `--included-artifacts none` to shave down non-binary shit that gets uploaded.

### ECOIN_STORE_NOT_PUBLISHED
```
{
  "Error": "Simulation failed with status: Move abort in 0x1::coin: ECOIN_STORE_NOT_PUBLISHED(0x60005): Account hasn't registered `CoinStore` for `CoinType`"
}
```
You forgot to call `coin::register<type>(destination);` remember that in Aptos/Move, address need to register to receive a particular asset.

### AccountNotFound
```
{
  "Error": "API error: API error Error(AccountNotFound): Account not found by Address(0x753285d1314620bf3abd9819d10c148cf2526b18620190b42303a62930b47c53) and Ledger version(386010360)"
}
```

Check that you're using the right private key; it's probably mapping to an account that doesn't exist.


### FUNCTION_RESOLUTION_FAILURE
```
{
  "Error": "Simulation failed with status: Transaction Executed and Committed with Error FUNCTION_RESOLUTION_FAILURE"
}
```
Check that your module name is correct i.e. `market::function` vs. `admin::function`.
