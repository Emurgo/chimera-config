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

## Banners

Display in-app banners for announcements:

```json
{
  "banners": [
    {
      "id": "maintenance-2024-02",
      "type": "warning",
      "title": "Scheduled Maintenance",
      "message": "System maintenance on Feb 15, 10:00 AM UTC",
      "startDate": "2024-02-10T00:00:00Z",
      "endDate": "2024-02-16T00:00:00Z",
      "dismissible": true
    }
  ]
}
```

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
