# Chimera Remote Configuration

Public runtime configuration for the [Chimera wallet](https://github.com/Emurgo/chimera). Edits to the JSON files in this repo change wallet behaviour without an app release.

## Files

| File | URL | Used by |
|---|---|---|
| `dev.json` | `…/refs/heads/main/dev.json` | Development builds and EAS internal dev variant |
| `preprod.json` | `…/refs/heads/main/preprod.json` | Pre-production / QA builds |
| `staging.json` | `…/refs/heads/main/staging.json` | Staging builds |
| `prod.json` | `…/refs/heads/main/prod.json` | Released production builds |

Each file shares the same schema. The wallet picks the URL based on the build variant (`--env=dev|preprod|staging|prod`).

Banner copy lives under `banners/content/<contentId>/<locale>.md`. Banner images live under `banners/images/`.

## Fetch + cache behaviour

| Step | Behaviour |
|---|---|
| Cache TTL | 5 minutes |
| Stale-while-revalidate | App boots from the last cached JSON; a background fetch refreshes it |
| Fetch failure | App keeps using the last cached JSON; no user-visible error |
| First launch | Built-in code defaults apply until the first fetch lands |
| Apply cadence | UI consumers re-render automatically when the cache updates |

## Precedence — code defaults vs. remote config

Feature flags (the `features` block) are evaluated through six precedence layers, **highest wins**:

1. **Runtime override** — set by dev-tools 5-tap menu (persisted on device)
2. **Remote config** — this repo
3. **Brand defaults** — `secondfi` in `packages/config/feature-flags-defaults.ts`
4. **Environment defaults** — `development` vs `production`
5. **Platform defaults** — `mobile` / `web` / `extension`
6. **Code defaults** — the safest baseline

A flag omitted from a remote JSON keeps whichever lower-precedence value applied. Setting `features.earn: false` in `prod.json` is the canonical kill-switch for the Earn tab.

Non-feature-flag fields (`hardwareWallets`, `swap`, `ramp`, `earn`, `dapps`, `cards`, `secondfi`, `allowedLinks`, `banners`, `disableSigning`, `maintenance`) have no code-defaults layer — they are read directly from the remote JSON.

---

# Schema reference

Each section below describes:
- **Wire format** — the JSON shape
- **Read by** — the wallet code paths that consume the field
- **Effect** — what flipping / setting / omitting the field does to the running app

---

## `disableSigning`

```json
{ "disableSigning": false }
```

| | |
|---|---|
| **Wire format** | `boolean` (default `false` when omitted) |
| **Read by** | `useDisableSigning()` → `RemoteBanners` component; signing flows |
| **Effect when `true`** | Read-only mode. A warning banner ("disable signing") is rendered at the top of the dashboard. Send / swap / receive flows that build or submit transactions surface the banner and refuse to sign. Used as an emergency switch when chimera's signing backend is suspected of producing bad transactions. |
| **Effect when `false` / omitted** | Normal signing. |

---

## `hardwareWallets`

```json
{
  "hardwareWallets": {
    "ledger": {
      "usb": { "extension": true, "android": true, "ios": false },
      "ble": { "extension": false, "android": true, "ios": true }
    },
    "trezor": {
      "usb": { "extension": true, "android": false, "ios": false },
      "ble": { "extension": false, "android": false, "ios": false }
    }
  }
}
```

| | |
|---|---|
| **Wire format** | `{ ledger?: DeviceTransports, trezor?: DeviceTransports }` where `DeviceTransports = { usb?: PlatformCells, ble?: PlatformCells }` and `PlatformCells = { extension?, android?, ios? }`. Each leaf cell is a `boolean`. Web shares the `extension` cell. |
| **Read by** | `useHardwareTransport(device, transport, platform)` → hardware-wallet onboarding screen (`app/hardware/index.tsx`) |
| **Effect when cell is `true`** | The matching pill is shown on the device list; the user can connect that device over that transport on that platform. |
| **Effect when cell is `false` / unset** | Strict opt-in — the pill is hidden. If a device has no transport enabled for the current platform, the entire device entry is hidden. |
| **Notes** | A device-local dev-tools override (5-tap menu) can flip individual cells without redeploying remote config. The override is per-cell — unset override cells fall through to remote config. |

---

## `features`

```json
{
  "features": {
    "earn": true,
    "swap": true,
    "cardanoStaking": true,
    "earnDefi": false
  }
}
```

| | |
|---|---|
| **Wire format** | `Partial<Record<FeatureFlagId, boolean \| { enabled: boolean, rolloutPercentage?: number }>>` |
| **Read by** | `useFeatureFlag(id)` hook across the entire app — tab visibility, screen entrypoints, action buttons, settings rows |
| **Effect when `true`** | The associated UI surfaces. |
| **Effect when `false`** | The associated UI is hidden / disabled. If the flag controls a navigation tab, the route is removed entirely. |
| **Effect when omitted** | Whatever lower-precedence layer dictates wins (usually the brand default for `secondfi`). |

### Percentage rollout

```json
"midnightAirdrop": { "enabled": true, "rolloutPercentage": 25 }
```

Each device gets a stable installation ID hashed to a `0..99` bucket. The flag evaluates `true` when the bucket is below `rolloutPercentage`. A given installation always sees the same answer until the percentage changes.

### Flag inventory

#### Navigation tabs
| Flag | Effect when `true` |
|---|---|
| `earn` | Earn tab visible in the bottom tab bar. Also gated by the remote `earn.gauntlet.enable` field (see below). |
| `card` | Card tab visible. |
| `portfolio` | Portfolio tab visible. |

#### Card sub-features (inside the Card tab)
| Flag | Effect when `true` |
|---|---|
| `cardSpendingInsights` | Spending-insights screen unlocked from the card dashboard. |
| `cardLimits` | Card limits screen unlocked. |
| `cardEarn` | Earn-on-card affordance unlocked. |

#### Chain-specific
| Flag | Effect when `true` |
|---|---|
| `cardanoStaking` | Delegation + governance entry points unlocked on Cardano. |
| `earnDefi` | DeFi earn opportunities surface in the Earn tab (Liqwid, Lenfi, etc.). |

#### Earn strategies
| Flag | Effect when `true` |
|---|---|
| `earnStrategyCurveRusdy` | Curve rUSDY vault visible. |
| `earnStrategyGtUsda` | Aera gtUSDa vault visible. **Additionally requires** `earn.gauntlet.enable: true` because the vault is powered by the Gauntlet backend. |

#### Core wallet
| Flag | Effect when `true` |
|---|---|
| `send` | Send button + flow available on the dashboard. |
| `receive` | Receive button + flow available. |
| `addWallet` | "Add wallet" affordance in the wallet picker. |
| `swap` | Swap quick-action + flow available. |
| `buyCrypto` | Ramps on-ramp ("Buy") quick-action available. |
| `sellCrypto` | Ramps off-ramp ("Sell") flow available. |
| `activity` | Activity tab / list available. |

#### Onboarding
| Flag | Effect when `true` |
|---|---|
| `createWallet` | "Create a new wallet" flow visible during onboarding. |
| `restoreWallet` | "Restore a wallet" (seed phrase) flow visible. |
| `socialLogin` | Web3Auth social-login flow visible. |
| `connectHardwareWallet` | Hardware-wallet onboarding flow visible. |

#### Additional
| Flag | Effect when `true` |
|---|---|
| `dappConnector` | DApp connector (CIP-30) is injected. On the extension, the content script reads this flag from chrome.storage before injecting any provider. |
| `discover` | Discover (DApp browser) tab visible. |
| `midnightAirdrop` | Midnight NIGHT-claim entry point surfaces in Settings. |
| `escrowRedeemPrimary` | NIGHT redeem builds via the self-built escrow path first, falling back to Midnight `/build` on error. Default off — Midnight is primary. |
| `submitNightClaimViaMidnight` | NIGHT claim submissions go through Midnight's `/thaws/{addr}/transactions` endpoint regardless of which build path produced the tx. |
| `addressBook` | Address-book entries in settings + send flow. |
| `transactionFilters` | Filter pills on the activity / transaction history view. |
| `pushNotifications` | Push-notification permission prompt + handlers active. Defaults: on for mobile, off for web/extension. |
| `portfolioChart` | Time-series chart on the portfolio screen. |

#### Developer tools (local-only — not honoured from remote config in prod)
| Flag | Effect when `true` |
|---|---|
| `devTools` | Settings → Developer Tools menu visible. Enabled in dev builds; in prod, requires the 5-tap unlock. |
| `cardMockData` | Card screens render fixture data instead of hitting the card backend. |
| `browserExtraIdentities` | Browser's "Show as" identity picker exposes Eternl + Lace in addition to SecondFi + Yoroi. |
| `txSimulation` | Transaction-simulation step runs before signing. |
| `encryptedBackupExport` | Encrypted-backup export affordance on the recovery-phrase screen. (The import side via deep link is always on.) |

---

## `maintenance`

```json
{
  "maintenance": {
    "swap": false,
    "buyCrypto": false,
    "cardano": false,
    "bitcoin": false,
    "ethereum": false
  }
}
```

| | |
|---|---|
| **Wire format** | `Partial<Record<string, boolean>>` where keys are either feature names (`send`, `receive`, `swap`, `buyCrypto`, `sellCrypto`, `cardanoStaking`, `earnDefi`) or chain ids (`cardano`, `bitcoin`, `ethereum`, `base`, `avalanche`, `binance`, `tron`, `solana`, `ripple`, `midnight`). |
| **Read by** | `useMaintenanceMode(key)` → `RemoteBanners`, `DashboardQuickActions`, `DashboardChainPage` |
| **Effect when `true`** | The feature / chain is visible but disabled. A "we're working on it" banner renders inline on the dashboard chain page and the quick-action button for the feature is greyed out. |
| **Effect when `false` / omitted** | Normal behaviour. |
| **Difference vs. `features.<x>: false`** | A feature flag *hides* the surface. Maintenance *shows the surface with a warning*. Use maintenance when you want users to know the feature exists but is temporarily unavailable. |

---

## `swap`

```json
{
  "swap": {
    "enabledProviders": ["dexhunter", "minswap", "muesliswap", "steelswap"],
    "partners": {
      "dexhunter": "<partner-id>",
      "muesliswap": "<partner-id>",
      "minswap": "<partner-id>",
      "steelswap": "<partner-id>"
    },
    "authTokens": {
      "dexhunter": "<auth-token>"
    },
    "initialPair": {
      "tokenIn": ".",
      "tokenOut": "fe7c786ab321f41c654ef6c1af7b3250a613c24e4213e0425a7ae456.55534441"
    },
    "excludedTokens": [
      "<policyId>.<assetNameHex>"
    ],
    "verifiedTokens": [
      "<policyId>.<assetNameHex>"
    ]
  }
}
```

| Field | Wire format | Read by | Effect |
|---|---|---|---|
| `enabledProviders` | `string[]` (provider ids: `dexhunter`, `minswap`, `muesliswap`, `steelswap`) | Swap routing layer (`@chimera/swap`) | Only the listed providers participate in quote aggregation. Omit one to silently drop it from the comparator. Empty array → no swap providers. |
| `partners` | `{ [providerId]: string }` | `SwapProvider` → swap SDK initialisation | Partner-id / referrer string passed to each DEX for revenue-share attribution. |
| `authTokens` | `{ [providerId]: string }` | `SwapProvider` → swap SDK initialisation | Per-provider API auth token. Only set for providers that require one (e.g. `dexhunter`). |
| `initialPair.tokenIn` / `initialPair.tokenOut` | Routing token id (`.` = native ADA; `<policyId>.<assetNameHex>` for Cardano tokens) | `SwapProvider` | Pre-fills the swap form's "from" / "to" tokens on first open. Once the user changes either side manually, the pre-fill is sticky for the rest of the session. |
| `excludedTokens` | `string[]` (routing token ids) | Token picker | Listed tokens are hidden from the swap token picker. Used to suppress scam / known-broken tokens. |
| `verifiedTokens` | `string[]` (routing token ids) | Token picker | Listed tokens render with a "verified" checkmark badge in the picker. Visual hint only — does not affect swap routing. |

---

## `ramp`

```json
{
  "ramp": {
    "enabledProviders": ["banxa", "encryptus", "moonpay", "ramp_network", "revolut"]
  }
}
```

| | |
|---|---|
| **Wire format** | `{ enabledProviders?: string[] }` (ids: `banxa`, `encryptus`, `moonpay`, `ramp_network`, `revolut`) |
| **Read by** | Ramp flow (`/ramp` Expo Router route) and `RampsService` |
| **Effect when populated** | Only the listed providers are offered on the buy / sell crypto screen. The order in the array drives the display order. |
| **Effect when empty / omitted** | No ramp providers offered — Buy / Sell buttons show "no providers available". |

---

## `earn`

```json
{
  "earn": {
    "gauntlet": {
      "enable": true
    }
  }
}
```

| | |
|---|---|
| **Wire format** | `{ gauntlet?: { enable?: boolean } }` |
| **Read by** | `getEarnGauntletEnabled()` → `ConfigProvider` pushes the value into the earn-backend client module (`isEarnGauntletEnabled()`) and folds it into the `features.earn` evaluation layer. |
| **Effect when `true`** | The Earn tab is allowed (subject to other gates), `earn-backend-client` calls hit `https://earn.api.dev.secondfi.io`, and the Aera gtUSDa vault becomes eligible (still needs `features.earnStrategyGtUsda: true`). |
| **Effect when `false` / omitted** | The Earn tab is hidden (overrides the brand default `earn: true`), and every earn-backend call short-circuits with `EarnBackendDisabledError`. **`undefined` is treated as `false`** — explicit opt-in is required. |

---

## `dapps`

```json
{
  "dapps": {
    "banned": ["https://malicious.example"],
    "recommended": [
      {
        "id": "minswap",
        "name": "Minswap",
        "description": "...",
        "category": "DEX",
        "logo": "minswap.png",
        "uri": "https://app.minswap.org",
        "origins": ["https://minswap.org", "https://app.minswap.org"],
        "isSingleAddress": false
      }
    ],
    "filters": {
      "Investment": ["DeFi", "DEX", "Stablecoin"],
      "Trading": ["DEX", "Trading Tools", "Stablecoin"]
    }
  }
}
```

| Field | Wire format | Effect |
|---|---|---|
| `banned` | `string[]` of URL prefixes | DApp browser refuses to load any URL that starts with one of these prefixes. Used as a kill-switch for known-phishing sites. |
| `recommended` | Array of recommended-dapp records | Surface in the Discover tab as featured DApps. `logo` is a filename under `banners/images/` (path relative to this repo). `origins` is the set of URLs that should be treated as the same DApp. `isSingleAddress: true` makes the connector only inject the active account (no multi-address enumeration). |
| `filters` | `{ <filter-label>: <category[]> }` | Defines the category pills in the Discover tab and which `category` values they each match. |

---

## `secondfi`

```json
{
  "secondfi": {
    "smartAccount": {
      "paymasterProvider": "wirex",
      "usdcDepositRequired": false
    }
  }
}
```

| Field | Wire format | Effect |
|---|---|---|
| `smartAccount.paymasterProvider` | `"emurgo" \| "wirex"` (any other value → undefined → fallback) | Picks the ZeroDev paymaster that sponsors SecondFi smart-account gas. Switching between `wirex` and `emurgo` changes which provider absorbs gas fees for ERC-4337 user-ops. |
| `smartAccount.usdcDepositRequired` | `boolean` | When `true`, smart-account creation gates the user on depositing USDC first (the deposit funds the account's initial state). When `false`, the account is created without a deposit prerequisite. |

---

## `cards`

```json
{
  "cards": {
    "topUp": {
      "tokenIds": [
        "BASE.0x833589fcd6edb6e08f4c7c32d4f71b54bda02913",
        "ETH.0xdac17f958d2ee523a2206206994597c13d831ec7"
      ],
      "chainIds": [
        "eip155:8453",
        "eip155:1",
        "solana:5eykt4UsFv8P8NJdTREpY1vzqKqZKvdp"
      ]
    }
  }
}
```

| Field | Wire format | Effect |
|---|---|---|
| `topUp.tokenIds` | `string[]` of routing token ids (`<CHAIN>.<address>`) | Exact tokens allowed as the source asset when topping up the card. The picker only shows tokens in this list. |
| `topUp.chainIds` | `string[]` of CAIP-2 chain ids | Additionally allows the **native** asset on each listed chain (ETH on `eip155:1`, SOL on `solana:…`, etc.). Non-native ERC-20s on those chains must still be in `tokenIds` to surface. |

Both lists are read by the card top-up asset picker. Omitting the section disables the picker (no fundable assets).

---

## `allowedLinks`

```json
{
  "allowedLinks": {
    "cardano-card-io": "https://cardanocard.io"
  }
}
```

| | |
|---|---|
| **Wire format** | `Partial<Record<string, string>>` — opaque link id → URL prefix |
| **Read by** | `notification-navigation.ts` — push-notification deep-link handler |
| **Effect** | When a push notification arrives with an `open_url` action, the URL is matched against this allowlist by prefix. Unmatched URLs are dropped with a `Notification open_url dropped — not in allowedLinks` log line. **Trust boundary** — push payloads originate outside the remote-config control plane, so the allowlist is the trust gate. |
| **Not for banner CTAs** | Banners live in the same JSON, so an allowlist check against them would be circular. Banner CTAs are *not* validated against this list. |

---

## `banners`

```json
{
  "banners": [
    {
      "id": "midnight-airdrop-2026",
      "contentId": "midnight-airdrop",
      "type": "info",
      "imageUrl": "banners/images/midnight-airdrop.svg",
      "ctaLink": "/(tabs)/portfolio",
      "startDate": "2026-04-16T00:00:00Z",
      "endDate": "2026-05-16T00:00:00Z",
      "isDismissible": true,
      "priorityWeight": 10,
      "slot": "home",
      "platformOverrides": {
        "ios": { "priorityWeight": 20 },
        "android": { "priorityWeight": 20 }
      }
    }
  ]
}
```

| Field | Wire format | Effect |
|---|---|---|
| `id` | `string` | Stable identifier. Used as the dismissal key — re-using an id on a new banner means returning users won't see it if they dismissed the previous one. |
| `contentId` | `string` | Folder name under `banners/content/<contentId>/`. The app loads `<contentId>/<locale>.md`, falling back to `en-US.md`. The Markdown body becomes the banner copy: first heading → title, body → description, `[cta]: <label>` → button text. |
| `type` | `"info" \| "warning" \| "error" \| "success"` | Drives the banner colour / icon. |
| `imageUrl` | repository-relative path | Asset under `banners/images/`. Loaded via the same raw-githubusercontent URL as the JSON. |
| `ctaLink` | `string` | Either an Expo Router path (must start with `/`, e.g. `/ramp`, `/(tabs)/portfolio`) or a full external URL (`https://…`). Internal paths navigate via `router.push`; external URLs open the browser. |
| `startDate` / `endDate` | ISO 8601 UTC timestamp | Optional. Banner is filtered out before `startDate` and after `endDate`. |
| `isDismissible` | `boolean` | If `false`, the banner has no close button and stays visible until removed from remote config. |
| `priorityWeight` | `number` | Banners are sorted descending by weight, so higher numbers appear first. |
| `slot` | `"home"` (currently the only value) | Which mount point renders the banner. Surfaces opt in by mounting `<RemoteBanners slot="..."/>` and filtering. Defaults to `"home"` when omitted. |
| `platformOverrides` | `{ ios? \| android? \| web? \| extension?: Partial<RemoteBanner> }` | Per-platform overrides for any of: `contentId`, `type`, `imageUrl`, `ctaLink`, `startDate`, `endDate`, `isDismissible`, `priorityWeight`, `slot`. Overrides are shallow-merged on top of the base banner. Useful for platform-specific copy or weight tweaks. |

### Banner Markdown format

`banners/content/<contentId>/<locale>.md`:

```md
# Banner title

Banner body text.

[cta]: CTA label
```

Locales matching the user's selected language are tried first; falls back to `en-US.md`.

---

# Common operations

### Enable a feature for everyone
```diff
{
  "features": {
-   "earn": false,
+   "earn": true,
  }
}
```

### Gradual rollout (25% of installs)
```diff
{
  "features": {
-   "midnightAirdrop": true,
+   "midnightAirdrop": { "enabled": true, "rolloutPercentage": 25 },
  }
}
```

Bump `rolloutPercentage` to ramp; set to `100` (or replace with a bare `true`) when fully shipped.

### Emergency kill-switch
```diff
{
  "features": {
-   "swap": true,
+   "swap": false,
  }
}
```

### Maintenance mode (keep the surface visible with a banner)
```diff
{
  "maintenance": {
-   "swap": false,
+   "swap": true,
  }
}
```

### Add a banned DApp
```diff
{
  "dapps": {
    "banned": [
+     "https://phishing.example/"
    ]
  }
}
```

### Add a swap partner
```diff
{
  "swap": {
    "partners": {
+     "newdex": "yoroi-partner-id"
    },
    "enabledProviders": [
+     "newdex"
    ]
  }
}
```

A change takes effect within ~5 minutes on each device (cache TTL). A user-triggered cold start applies it immediately.

---

# Verifying changes

### Fetch current config
```bash
curl https://raw.githubusercontent.com/Emurgo/chimera-config/refs/heads/main/prod.json | jq .
```

### Test in-app

1. Trigger a 5-tap on the About screen to unlock dev tools.
2. Settings → Developer Tools → Feature Flags.
3. Every flag is listed with its evaluated value and source (`runtime` / `remote` / `brand` / `environment` / `platform` / `code`).
4. Long-press a row to set a per-device runtime override (highest precedence; persists across launches).

---

# Security considerations

- **Public repo** — never commit secrets that grant write access to backend services. Partner ids, API keys with read-only or referrer-only scope, and rate-limited public tokens are fine.
- **Trust boundary** — `allowedLinks` is the only field that gates content originating outside this repo (push notifications). Treat changes to that field carefully.
- **Banner CTAs** — links inside `banners[].ctaLink` are *not* validated against `allowedLinks` (they originate from the same writer). Anyone with write access here can route users to arbitrary URLs via a banner.
- **Rate limit** — each device fetches ~12 times per hour (5-minute cache).

---

# Versioning

Use Conventional Commits so the history reads as a changelog:

```
feat: enable earn.gauntlet on prod
fix: disable swap kill-switch on staging
perf: bump midnightAirdrop rollout to 50%
revert: roll back gauntlet enable on prod
```

---

# Support

- **This repo:** https://github.com/Emurgo/chimera-config
- **App repo:** https://github.com/Emurgo/chimera
- **Issues:** file on the [chimera repo](https://github.com/Emurgo/chimera/issues)

## License

Same as the Chimera wallet project.
