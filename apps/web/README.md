# Starlog Web

Starlog is a community travel prototype for the Stellar Meridian hackathon. The Spanish-first home page presents six trip and event examples, an interactive schematic map for meetups and community-listed Bitcoin merchants, and a bilingual LiveKit guide.

The interface follows the supplied Starlog palette and neo-brutalist editorial direction: square corners, hard shadows, dotted petroleum background, oversized DM Serif Display headings, Inter body text, and IBM Plex Mono labels. It uses no stock photography.

## Run locally

From this directory:

```bash
pnpm dev
```

Open `http://localhost:3000`.

## Current scope

- Browse six trip/event examples linking to official experience or event pages.
- Explore meetup pins and community directory listings for Bitcoin merchants in three cities.
- Create one local meetup preview and see its pending Luma and Eventbrite publication status.
- Switch the interface between Spanish and English, and use the Pollar wallet on Stellar testnet.
- Open the bilingual LiveKit guide for recommendations.
- Search Musement activity inventory and check sandbox availability when partner test credentials are configured.
- Create an explicitly labeled Musement sandbox order for integration testing; it does not charge Stellar assets or issue real tickets.

Displayed amounts on the home page are prototype references, not live quotes or confirmed bookings. Merchant entries come from third-party Bitcoin directories and must be checked with each business; they do not indicate support for Stellar assets. The static cards do not complete payments. An optional Estela MCP connection lets the LiveKit agent search flights, request paid x402 quotes, and book in Stellar testnet after explicit traveler confirmation. Card processing and external Luma/Eventbrite publishing are not connected in this prototype.

## Musement activities sandbox

The web app includes a server-side Musement integration pinned to the sandbox host (`https://sandbox.musement.com/api/v3`) and API version 3.4.0. It supports catalog search, date availability, product options, sandbox order creation and order-status polling. Pending orders refresh every 10 seconds, and voucher links appear when Musement returns them. It never calls Musement production. Musement requires a partner contract and issues the sandbox client ID, client secret and application value after onboarding; add those values to `apps/web/.env.local` as `MUSEMENT_CLIENT_ID`, `MUSEMENT_CLIENT_SECRET` and `MUSEMENT_APPLICATION`. Set the market and currency to the values assigned to the partner account. Sandbox orders are disabled by default; set `MUSEMENT_SANDBOX_ORDERS_ENABLED=true` to enable explicit test order creation.

The sandbox requires its test lead-booker values (`TEST TEST`, `test@test.com`) and does not email customers or issue real tickets. Test order items normally remain `PENDING`; Musement's API team can update a test order to confirm the status flow. The no-payment endpoint is attempted only against the sandbox and needs Musement's merchant permission for non-zero orders. This test flow is not a Stellar payment integration: production must verify an on-chain payment server-side before creating an order, then use the approved merchant flow and handle refunds, cancellations and customer support.

## LiveKit assistant setup

The assistant worker lives in `../agent`. It uses LiveKit Inference for speech recognition, language generation and speech synthesis. Create a LiveKit Cloud project, then copy the example files and configure the project URL, API key and API secret:

```powershell
# Terminal 1: from apps/web
Copy-Item .env.example .env.local
pnpm install
pnpm dev

# Terminal 2: from apps/agent
Copy-Item .env.example .env.local
pnpm install
pnpm dev
```

Set the same private `STARLOG_INTERNAL_API_TOKEN` in both `.env.local` files. In the agent file, set `STARLOG_WEB_API_URL=http://localhost:3000`. The worker name is `starlog-guide` and must match the browser session and token endpoint.

The local draft endpoint stores submissions in `apps/web/.data/meetup-drafts.json`. Replace this file-backed store with a database before deploying to serverless or running multiple web instances. Drafts remain pending human review; no event is automatically published.

### Estela MCP flight flow

Place the Starlog backend and its `mcp` directory beside this project. Configure and start the backend using its own ignored `apps/web/.env.local`; the local demo can run it on port 3002. In `Starlog/mcp`, run `npm ci`, create a dedicated testnet wallet with `npm run setup:wallet`, then run `npm run setup:testnet`. Fund that wallet with test USDC through the Circle faucet and set `ESTELA_URL=http://localhost:3002` in `mcp/.env.local`. Point this agent's `STARLOG_MCP_DIR` at the absolute MCP directory and restart the LiveKit worker.

The browser app can run on port 3001 (`pnpm dev -- -p 3001`). Open `http://localhost:3001/?assistant=flights` to search Duffel sandbox offers, select one, and send it to the agent for an Estela quote. The `buscar_vuelos` and `cotizar` MCP calls each charge a small amount of **testnet** USDC from the agent wallet. Selecting or quoting a flight does not book it. `reservar` requires confirmed passenger details and the traveler's explicit approval of the exact total. Keep all `.env.local` files out of Git.

## Pollar wallet (Stellar testnet)

The top-bar wallet control uses `@pollar/react` and `@pollar/core` 0.12.0. It supports Pollar login, wallet creation when the app is configured for on-demand wallets, address copy, balance, and receive UI. It is pinned to Stellar testnet and requests user confirmation for signing.

To enable it:

1. Create a Pollar app on testnet and configure at least one login provider and wallet creation mode in the Pollar dashboard.
2. Allow the local Starlog origin (for example, `http://127.0.0.1:3000`) in the app settings.
3. Copy the app's **publishable** testnet key into `apps/web/.env.local` as `NEXT_PUBLIC_POLLAR_API_KEY`.
4. Restart `pnpm dev`.

The key is public by design. Do not place a Pollar server secret or MCP personal access token in this frontend environment variable. Without the publishable key, Starlog shows the setup steps instead of pretending that a wallet is connected.
