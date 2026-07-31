---
name: Browse IP-NFTs and IP Tokens
description: Query and browse the Molecule DeSci catalog of IP-NFTs and their fractionalized IP Tokens (IPTs), with filtering, sorting, and pagination, via the read-only GraphQL Data API.
api: https://production.graphql.api.molecule.xyz/graphql
operations: [ipnfts, ipts]
auth: x-api-key header
---

# Browse IP-NFTs and IP Tokens

Use the Molecule **Data API** (read-only GraphQL) to browse decentralized-science
intellectual property. All requests are `POST` to the GraphQL endpoint with an
`x-api-key` header.

## Prerequisites
- An API key (request via the Molecule Discord — see docs).
- Endpoint: `https://production.graphql.api.molecule.xyz/graphql` (staging host also available).

## Steps
1. **List IP-NFTs.** Call the `ipnfts` query with `limit`, `skip`, `sortBy`
   (`IPNFTSortBy`), `sortOrder` (`SortOrder`), and `filterBy` (`IPNFTFilterBy`).
   Select fields such as `id`, `name`, `symbol`, `trlValue`, `organization`,
   `topic`, and the nested `owner`, `researchLead`, `agreements`, and `ipt`.
2. **Paginate.** Increase `skip` by your `limit` to page through results;
   pagination is limit/skip offset-based.
3. **Drill into tokens.** Use the `ipt { id symbol totalIssued }` sub-selection,
   or the `ipts` query, to read the fractionalized ERC-20 IP Token for a project.

## Conventions
- Pagination: `limit` / `skip`; sorting via `sortBy` + `sortOrder`; filtering via `filterBy`.
- Errors: GraphQL top-level `errors[]` array alongside `data`.
- See `conventions/molecule-conventions.yml` and `data-model/molecule-data-model.yml`.

## Example
```bash
curl -X POST https://production.graphql.api.molecule.xyz/graphql \
  -H 'Content-Type: application/json' \
  -H 'x-api-key: YOUR_API_KEY' \
  -d '{"query":"query { ipnfts(limit: 5) { id name trlValue ipt { symbol totalIssued } } }"}'
```
