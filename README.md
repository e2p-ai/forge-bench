# forge-bench — public cost-per-success leaderboard

Live board: **https://e2p-ai.github.io/forge-bench/**

An open, reproducible benchmark that ranks coding agents by **cost-per-success** —
the dollars it takes to *actually solve* a task, not raw token counts. Raw
`tokens_in` is not comparable across tools (a multi-turn agent sums cache-inclusive
prompt tokens across a whole loop; a single-turn `claude -p` reports only one turn's
fresh input), so cost-per-success is the headline metric.

## What's here

```
index.html          static, dependency-free page that renders data/results.json
data/results.json    machine-generated leaderboard state (DO NOT hand-edit)
data/history.jsonl   one appended row per publish — powers the trend sparkline
```

`index.html` is the only hand-written file. Everything under `data/` is generated
and pushed by `benchmarks/lib/publish-leaderboard.sh` in the **private**
[`e2p-ai/forge`](https://github.com/e2p-ai/forge) repo, which runs the benchmark
suite, sanitizes the results (task id / pass / cost / tokens / wall / model / date
only — never prompts, transcripts, or keys), and commits the JSON here. GitHub
Pages serves it from `main`.

## The suite

27 tasks across **Rust, Python, and Node**, each a frozen broken repo plus a machine
oracle (`cargo test` / `python3 -m unittest` / `node --test` / grep). Tasks 11–27 also
ship a reference solution proven to pass the oracle offline with zero model calls, so
the suite's solvability is provable without spending a cent. Open-ended tasks are graded
by a *different-vendor* model to avoid self-marking.

Every metric on the board is provenance-stamped: the harness commit sha, the model
id(s), and the run date travel with the data and are shown in the page footer.

## Reproduce

```bash
git clone https://github.com/e2p-ai/forge && cd forge
cargo build --release -p forge-voice
cd benchmarks
FORGE_VOICE_BIN=$PWD/../target/release/forge-voice ./score.sh --agents forge
./lib/build-report.sh          # cost-per-success scorecard
./bench-gate.sh --agent forge  # regression gate vs baseline.json
```

The board renders whatever agents are present in the data — additional forge model
configs and third-party CLIs appear automatically as head-to-head runs are published.
