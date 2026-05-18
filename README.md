# Chimera Remote Configuration

Public configuration repository for the Chimera wallet application.

## Overview

This repository contains runtime configuration for Chimera, including feature flags, swap partner keys, and other dynamic settings that can be updated without requiring an app release.

## Files

### `prod.json`
Production configuration used by released Chimera apps.

**URL:** `https://raw.githubusercontent.com/Emurgo/chimera-config/refs/heads/main/prod.json`

**Features:**
- Stable features enabled for all users
- Beta features with percentage rollouts (e.g., portfolioPerformance at 25%)
- Experimental features disabled
- Dev tools disabled by default (requires 5-tap to enable)

### `dev.json`
Development configuration used during development and testing.

**URL:** `https://raw.githubusercontent.com/Emurgo/chimera-config/refs/heads/main/dev.json`

**Features:**
- All features enabled for comprehensive testing
- Dev tools always accessible
- Test API keys for swap partners

## Feature Flags

### Core Wallet Features
| Flag | Production | Development | Description |
|------|-----------|-------------|-------------|
| `staking` | ✅ | ✅ | Cardano staking delegation |
| `swap` | ✅ | ✅ | DEX aggregator token swapping |
| `governance` | ✅ | ✅ | Cardano governance voting (CIP-1694) |
| `nfts` | ✅ | ✅ | NFT gallery and management |
| `defi` | ✅ | ✅ | DeFi protocol integrations |
| `bridge` | ❌ | ✅ | Cross-chain bridging (experimental) |
| `dappConnector` | ✅ | ✅ | DApp connector (CIP-30) |
| `hardwareWallets` | ✅ | ✅ | Ledger/Trezor hardware wallet support |
| `buyCrypto` | ✅ | ✅ | Fiat on-ramp integration |

### Additional Features
| Flag | Production | Development | Description |
|------|-----------|-------------|-------------|
| `portfolioPerformance` | 🔄 25% | ✅ | Portfolio performance tracking (beta) |
| `midnightAirdrop` | ✅ | ✅ | Midnight TGE NIGHT token claiming |
| `advancedCoinSelection` | ❌ | ✅ | Advanced UTXO coin selection |
| `multiWallet` | ✅ | ✅ | Multiple wallet support |
| `addressBook` | ❌ | ✅ | Saved addresses management |
| `transactionFilters` | ❌ | ✅ | Transaction history filtering |
| `exportHistory` | ❌ | ✅ | Export transaction history (CSV/JSON) |
| `biometricAuth` | ✅ | ✅ | Face ID/Touch ID authentication |
| `pushNotifications` | ❌ | ✅ | Push notifications |
| `priceAlerts` | ❌ | ✅ | Price alert system |
| `devTools` | ❌ | ✅ | Developer tools menu |

**Legend:**
- ✅ Enabled for all users
- ❌ Disabled
- 🔄 X% = Percentage rollout (X% of users)

## Percentage Rollouts

Features can be gradually rolled out using percentage-based activation:

```json
{
  "features": {
    "portfolioPerformance": {
      "enabled": true,
      "rolloutPercentage": 25
    }
  }
}
```

This enables the feature for 25% of users based on their installation ID. The same user will always see the same result (consistent hashing).

## Updating Configuration

### Enable a Feature

```diff
{
  "features": {
-   "bridge": false,
+   "bridge": true,
  }
}
```

### Increase Rollout Percentage

```diff
{
  "features": {
    "portfolioPerformance": {
      "enabled": true,
-     "rolloutPercentage": 25
+     "rolloutPercentage": 50
    }
  }
}
```

### Emergency Disable

```diff
{
  "features": {
-   "problematicFeature": true,
+   "problematicFeature": false,
  }
}
```

**Note:** Changes take effect within 5 minutes (app cache TTL).

## Swap Partners

API keys for DEX aggregator partners:

```json
{
  "swap": {
    "partners": {
      "dexhunter": "your-api-key",
      "minswap": "your-api-key",
      "muesliswap": "your-api-key"
    }
  }
}
```

**Security:** API keys in this public repo should be:
- Rate-limited per key
- Restricted to specific domains/IPs if possible
- Rotated regularly
- Not contain sensitive credentials

## Card Top-Up Sources

The card top-up asset picker reads `cards.topUp` from remote config.
`tokenIds` contains exact routing token IDs. `chainIds` contains CAIP-2
chain IDs and allows only the native token for each listed chain. The current
config allows native assets for the supported chains and the main stablecoins
available through routing on those chains.

```json
{
  "cards": {
    "topUp": {
      "tokenIds": [
        "CARDANO.25c5de5f5b286073c593edfd77b48abc7a48e5a4f3d4cd9d428ff935.55534443",
        "CARDANO.25c5de5f5b286073c593edfd77b48abc7a48e5a4f3d4cd9d428ff935.55534454",
        "BASE.0x833589fcd6edb6e08f4c7c32d4f71b54bda02913",
        "BASE.0xfde4c96c8593536e31f229ea8f37b2ada2699bb2",
        "BASE.0x60a3e35cc302bfa44cb288bc5a4f316fdb1adb42",
        "ETH.0xa0b86991c6218b36c1d19d4a2e9eb0ce3606eb48",
        "ETH.0xdac17f958d2ee523a2206206994597c13d831ec7",
        "ETH.0x1abaea1f7c830bd89acc67ec4af516284b1bc33c",
        "AVAX.0xb97ef9ef8734c71904d8002f8b6bc66dd9c48a6e",
        "AVAX.0xa7d7079b0fead91f3e65f86e8915cb59c1a4c664",
        "AVAX.0x9702230a8ea53601f5cd2dc00fdbc13d4df4a8c7",
        "AVAX.0xc891eb4cbdeff6e073e859e987815ed1505c2acd",
        "BSC.0x8ac76a51cc950d9822d68b83fe1ad97b32cd580d",
        "BSC.0x55d398326f99059ff775485246999027b3197955",
        "SOL.EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v",
        "SOL.Es9vMFrzaCERmJfrF4H2FYD4KCoNkY11McCe8BenwNYB",
        "SOL.HzwqbKZw8HxMN6bF2yFZNrht3c2iXXzpKcFu7uBEDKtr",
        "TRON.TEkxiTehnzSmSe2XqrBj4w32RUN966rdz8",
        "TRON.TR7NHqjeKQxGTCi8q8ZY4pL8otSzgjLj6t"
      ],
      "chainIds": [
        "cardano:mainnet",
        "eip155:8453",
        "bip122:000000000019d6689c085ae165831e93",
        "eip155:1",
        "eip155:43114",
        "eip155:56",
        "solana:5eykt4UsFv8P8NJdTREpY1vzqKqZKvdp",
        "tron:mainnet"
      ]
    }
  }
}
```

## Banners

Display in-app banners for announcements. The JSON manifest controls metadata, while translated copy lives in
Markdown files under `banners/content`.

```json
{
  "banners": [
    {
      "id": "dev-mode",
      "contentId": "dev-mode",
      "type": "warning",
      "imageUrl": "banners/images/dev-mode.svg",
      "ctaLink": "https://emurgo.io",
      "startDate": "2026-04-16T00:00:00Z",
      "endDate": "2026-05-16T00:00:00Z",
      "isDismissible": true,
      "priorityWeight": 10,
      "platformOverrides": {
        "ios": {
          "priorityWeight": 20
        }
      }
    }
  ]
}
```

Copy is fetched from `banners/content/{contentId}/{locale}.md`, falling back to
`banners/content/{contentId}/en-US.md` when the selected locale is not available.

Markdown files use this format:

```md
# Banner title

Banner body text.

[cta]: CTA label
```

Images should be stored under `banners/images/` and referenced by repository-relative path in `imageUrl`.

CTA links use a cross-platform convention:
- App routes start with `/` and must match the Expo Router path used by the app, for example `/ramp` or `/(tabs)/portfolio`.
- External links use a full URL such as `https://emurgo.io`.

## Cache Behavior

- **Cache Duration:** 5 minutes
- **Stale-While-Revalidate:** Config fetched in background on app launch
- **Fallback:** App uses last cached config if fetch fails
- **First Launch:** Uses built-in defaults until config loads

## Testing

### Verify Configuration

```bash
# Production
curl https://raw.githubusercontent.com/Emurgo/chimera-config/refs/heads/main/prod.json | jq

# Development
curl https://raw.githubusercontent.com/Emurgo/chimera-config/refs/heads/main/dev.json | jq
```

### Test Feature Flags in App

1. Enable dev mode (5-tap on About screen)
2. Go to Settings > Developer Tools > Developer Menu
3. Open Feature Flags Debug
4. View all flags with their sources
5. Toggle runtime overrides for local testing

## Versioning

Configuration changes are tracked via git history. Use semantic commit messages:

```
feat: enable bridge feature for all users
fix: disable problematic swap partner
perf: increase portfolioPerformance rollout to 50%
revert: rollback bridge feature due to issues
```

## Monitoring

### Metrics to Track
- Feature flag evaluation count
- Remote config fetch success rate
- Time to fetch config
- Percentage rollout distribution

### Logging
- All config fetches logged in app
- Feature flag evaluations logged at debug level
- Runtime overrides logged at info level

## Security Considerations

### Public Repository
- ✅ No secrets or API keys with write access
- ✅ No user data
- ✅ No internal infrastructure details
- ✅ Safe for public consumption

### API Keys
- Use separate keys for different environments
- Rotate keys if compromised
- Monitor usage for abuse
- Consider IP restrictions if available

### Rate Limiting
Apps cache config for 5 minutes to minimize fetches:
- ~12 fetches per hour per device
- ~288 fetches per day per device

## Support

For issues or questions about this configuration:
- **Repository:** https://github.com/Emurgo/chimera-config
- **Main Project:** https://github.com/Emurgo/chimera
- **Issues:** Open an issue in the main Chimera repository

## License

Same as the Chimera wallet project.
