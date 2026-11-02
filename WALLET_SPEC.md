# SilentVault — non-custodial TON wallet Mini App (v1 spec)

## Choice made (2026-11-02)
Non-custodial via TON Connect. The app never holds funds, never sees a seed
phrase, never signs anything itself. Every transaction is approved by the
user inside their own wallet (Tonkeeper, MyTonWallet, Telegram's built-in
wallet). That removes custody, licensing and KYC from v1 entirely.

## What v1 does
- Connect wallet (TON Connect UI, jsBridge inside Telegram + universal link outside)
- Balances: TON (Gram) via Toncenter, USDT (Jetton) via TonAPI
- Send TON (native) and USDT (Jetton transfer cell, 0.05 TON gas envelope)
- Receive: address + QR code
- Recent activity: last 10 account events via TonAPI
- Telegram WebApp theming, expand, signed-in display name (display only)

## What v1 deliberately does NOT do
- No backend server, no database, no accounts system. Balances/history come
  from public TON indexers; nothing about the user is stored anywhere.
- No KYC, no card buys, no P2P market, no swaps, no price charts. Those
  belong to a custodial v2 and need the legal work from the requirements
  answer (licensing, KYC vendor, custody infrastructure).
- No seed-phrase import, no key generation in-app. If a future version
  ever asks for a seed phrase, that is a scam pattern — v1 never will.

## Security model
- Trust boundary: the user's wallet app. This Mini App is untrusted UI.
- initData is used for display only. There is no server action to protect,
  so there is nothing to forge: an attacker faking initData gains nothing
  (they cannot move funds without the victim's wallet signature).
- Transaction validity window: 600 seconds. Amounts use integer math
  (nanoTON / jetton base units via BigInt), never floats, on the send path.
- Recipient addresses are passed to the wallet for the user's own
  confirmation — the wallet is the final check, and the UI says so.

## Files
- `index.html` — the whole app (single file, CDN: telegram-web-app.js,
  @tonconnect/ui@2.4.4, @ton/core via esm.sh, QR via api.qrserver.com)
- `tonconnect-manifest.json` — MUST be updated with the real host URL
  before launch (both `url` and `iconUrl`); TON Connect refuses a
  manifest whose URL does not match the serving origin.
- `icon.svg` — app icon (gold diamond on navy).

## Launch checklist (owner steps)
1. BotFather: `/newbot` → name "SilentVault" (or Zesh's pick), copy token
   → send token to Muse via the secure card (never in chat).
2. Host these files over HTTPS (recommended: GitHub Pages, free; any
   static host works). Note the final URL.
3. Update `tonconnect-manifest.json` with that URL, redeploy.
4. BotFather: `/newapp` on the bot → title, description, icon, the hosted
   URL → gives `t.me/<bot>/<app>` link.
5. BotFather: `/setmenubutton` so the app opens from the bot's menu.
6. Test inside Telegram: connect wallet → balance shows → send a tiny
   TON amount to a second wallet you control → receive QR scans.
7. Only then announce it; never announce a wallet app that hasn't moved
   real funds end-to-end in a test.

## Possible v2 (only after v1 is live and tested)
- In-app swap via a TON DEX aggregator (still non-custodial: user signs).
- Multi-asset balances (all Jettons, not just USDT).
- Contacts: send to a Telegram username (needs an address registry — the
  hard part; TON DNS or a voluntarily-registered mapping, with spoofing
  warnings).
- A backend ONLY if a feature needs it (e.g. price alerts, saved
  contacts) — and then initData HMAC validation becomes mandatory.
