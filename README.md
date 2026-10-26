# Starlog: travel web and AI guide

Starlog helps travelers discover destinations, activities and flights. This repository contains the Next.js travel interface, the LiveKit guide agent and destination knowledge for Madrid, Valencia and Barcelona. Payments and reservations are testnet/sandbox workflows until the suppliers and production deployment are configured.

## Project structure

- `apps/web`: travel interface, Pollar wallet, Duffel flight search and API routes.
- `apps/agent`: LiveKit voice/text guide and local Estela MCP connection.
- `knowledge`: destination guides used by the experience.

The Estela payment backend, Soroban contracts, testnet setup and the integrated traveler profile are in the [Starlog repository](https://github.com/yedidromero/Starlog). Its `main` branch is the source for those services.

## Local development

Follow [the web setup guide](apps/web/README.md) for environment variables and startup commands. Keep `.env.local` files and wallet secrets outside Git. Duffel offers and Stellar payments here use sandbox/testnet credentials; they do not purchase a real ticket.
