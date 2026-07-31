---
name: Write to a Lab via the x402 pay-per-call gateway
description: Perform Molecule Labs write mutations (create a lab, publish an announcement, upload files) as an autonomous agent without a long-lived service token, by settling a per-request USDC payment through the HTTP 402 x402 gateway.
api: https://docs.molecule.xyz/api-reference/x402-gateway.md
operations: [createLab, createAnnouncement, initiateCreateOrUpdateFile, finishCreateOrUpdateFile, generateDataEncryptionKey, decryptDataKey]
auth: x402 payment (Payment-Signature header), USDC on Base
---

# Write to a Lab via the x402 gateway

The x402 gateway fronts a small allow-list of Labs write mutations with the
HTTP 402 Payment Required protocol. Each call settles a USDC payment on Base and
the gateway mints a short-lived scoped service token, so an agent needs **no**
long-lived service token.

## Allow-listed endpoints
`POST /x402/labs/{mutation}` where `{mutation}` is one of:
`initiateCreateOrUpdateFile`, `finishCreateOrUpdateFile`, `createAnnouncement`,
`createLab`, `generateDataEncryptionKey`, `decryptDataKey`. The path `{mutation}`
must equal the top-level GraphQL mutation field in the body, or the gateway
returns `400`.

## Steps (verify → serve → settle)
1. **Challenge.** `POST` the mutation with a JSON body `{ query, variables }` and
   no payment header. The gateway returns `402` with x402 payment requirements
   (network, asset, price, payTo).
2. **Sign.** Sign an EIP-3009 `transferWithAuthorization` (or Permit2) for the
   quoted USDC amount to the `payTo` address on the quoted network (`base`).
   Always read the price from the 402 challenge — never hardcode it.
3. **Retry with payment.** `POST` again with the base64 payment payload in the
   `Payment-Signature` (or `X-Payment` / `Payment`) header. On success the
   gateway verifies, mints a scoped token (`ttl` default 300s), forwards the
   GraphQL mutation to AppSync, settles, and returns `200` with the result.

## Idempotency (important)
Requests are **not idempotent**: minted tokens have a unique `jti`, replays may
be rejected by replay protection, and a retry after a settlement failure can
succeed twice. Treat settlement failures as "payment not charged yet" and re-sign.
See `conventions/molecule-conventions.yml`.

## Errors
`400` path/query mismatch or unresolvable payer; `402` payment required/failed;
`4xx/5xx` upstream AppSync error (settlement skipped). See `errors/molecule-problem-types.yml`.
