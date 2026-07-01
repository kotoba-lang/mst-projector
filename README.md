# mst-projector

`kotoba.lang.mst-projector` — the LanceDB-indexer + XRPC query-server half of
`etzhayyim/root:50-infra/mst-projector/py`'s Python `mst_projector` package,
ported directly to Clojure (no intermediate physical-move-as-Python stage).

| Original (Python) | Ported to |
|---|---|
| `indexer.py` (LanceDB write/query path) | `kotoba.lang.mst-projector.indexer` (pure core + injected storage protocol) |
| `query_api.py` (aiohttp XRPC server, 4 NSIDs + healthz) | `kotoba.lang.mst-projector.query-api` |
| `subscriber.py` (AT-Proto firehose / CBOR-frame websocket subscriber) | **NOT ported** — see below |
| `main.py` (CLI entrypoint) | `kotoba.lang.mst-projector.cli` |

## Why `subscriber.py` stays Python

`20-actors/etzhayyim-sdk-py/MIGRATION-TODO.md` (a prior, independent py→cljc
migration of the `etzhayyim_sdk` package that `subscriber.py` depends on)
already recorded explicit founder guidance on this exact file: porting the
AT Protocol firehose / CBOR-frame websocket subscriber to babashka/Clojure is
"impractical," and it's deliberately left in place to coexist with the
now-mostly-Clojure `etzhayyim_sdk` package. This repo respects that decision
rather than re-litigating it — only `indexer.py`/`query_api.py`/`main.py`
(the parts *not* covered by that guidance) are ported here.

## Wire compatibility

A Clojure XRPC **client** for this exact query API already exists and is
tested: `etzhayyim_sdk.mst-projector` (in `20-actors/etzhayyim-sdk-py/src/
etzhayyim_sdk/mst_projector.cljc`, 17 tests / 47 assertions green) — same 4
NSIDs (`com.etzhayyim.mstProjector.{queryByCollection,queryByDid,
queryByField,countByCollection}`), same camelCase wire keys. This repo's
`query-api` server is built to be a wire-compatible server for that exact
client.

## Provenance

Ported 2026-07-02 from `etzhayyim/root:50-infra/mst-projector/py/src/
mst_projector/{indexer,query_api,main}.py` per the org-taxonomy
library-placement rule. Design authority remains ADR-2605215500, in
`etzhayyim/root`.

## Development

```bash
clj-kondo --lint src test
clojure -M:test
```

## License

Apache 2.0 + Charter Compliance Rider v3.6 (`/CHARTER-RIDER.md`).
