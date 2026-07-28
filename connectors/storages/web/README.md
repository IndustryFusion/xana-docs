# Web / storage connector

> Part of [the documentation](/docs/README.md) → [Architecture: web / storage connector](/docs/architecture/connectors-web.md).

A configurable connector for authorized websites, wikis, and document portals — not a general internet crawler. It signs in to a configured site, reads its folder and document structure into a consistent shape, and streams document downloads through the hub rather than exposing the source system directly.

## Security posture

- Only sites explicitly configured as allowed can be reached — nothing else.
- Requests toward internal or non-public network addresses are blocked by default.
- Credentials are never stored or logged in plain text.
- Session tokens are kept in memory only, and scrubbed from anything logged.
- Downloads are bounded in size and depth, so a misconfigured or unusual site can't be used to pull an unbounded amount of data.

## Configuration

Which sites this connector can reach, and its credentials, are set at deployment time rather than being something changed through the product itself day to day — the same way any integration's access is provisioned once and then used.

## Where to go next

- How it fits with the rest of XANA: [Architecture: web / storage connector](/docs/architecture/connectors-web.md)
- How an admin registers one: [4. Connecting data sources](/docs/guides/04-connectors.md)
