# GIL repair for taos-ws-py 0.7.0

Base: upstream tag `taos-ws-py-v0.7.0`, commit
`315f180979469897852deb8d18c72bc343f1175d`.

Local package version: `0.7.0+fastbull.1`.

## Scope

- Release the CPython GIL while establishing connections and cursors.
- Release it during connection and DB-API cursor query/execute operations.
- Release it while fetching network blocks through cursors and result iterators.
- Construct Python values with the GIL held. Yield between batches of 1024 rows
  to avoid monopolizing the interpreter during large result conversion.
- Keep Python method arguments, SQL semantics, row types and ordering unchanged.
- Preserve the upstream Cargo dependency lockfile, except for the local version.

This patch targets the connection/query/fetch paths. It does not claim that all
statement, subscription or consumer methods in the library are GIL-free.

## Build and install

The `Build GIL-fixed wheels` GitHub Actions workflow builds Windows x64 and
manylinux2014 x86_64 wheels using Python 3.10 and the existing abi3 configuration.
The runtime does not require Rust. Download the matching workflow artifact,
extract its wheel, and install that exact file with `python -m pip install`.
Do not rename the wheel or mix operating-system builds.

Pin the wheel file or its immutable URL in the application dependencies. Keep
the original official 0.7.0 wheel available for rollback. This is a fork build,
not an official upstream release, and no package is published to PyPI.

## Validation

Both platforms run an isolated, delayed WebSocket-handshake regression test.
Linux additionally creates a disposable local TDengine container and exercises
connection/cursor query, cross-block fetchall, fetchmany, fetchone, dictionary
results, and the direct result iterator through a delayed TCP proxy.
The tests do not use production credentials or upload production data.

Before production rollout, compare the patched wheel with the official wheel
using identical queries and rows, including large historical result sets.
Measure both end-to-end latency and the longest pause of another Python thread.
Removing the client's GIL blocking does not guarantee lower database execution
time, nor does it prove that every production latency spike has the same cause.
