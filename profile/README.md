<img src="https://raw.githubusercontent.com/quantforge-rs/.github/main/assets/banner.svg" width="880" alt="QuantForge — Rust-native toolkit for deterministic trading research and controlled execution." />

<div align="center">

<a href="https://github.com/quantforge-rs/quantforge"><img src="https://raw.githubusercontent.com/quantforge-rs/.github/main/assets/quantforge-help.svg" width="840" alt="Terminal output of quantforge --help from the released 0.2.0 CLI, listing the data, backtest, trade and monitor commands" /></a>

**A trading framework for Rust. Pull in market data, backtest a strategy, run it live — all from the terminal.**

<p>
  <a href="https://quantforge.rs"><img src="https://img.shields.io/badge/quantforge.rs-AD521D?style=for-the-badge&logo=rust&logoColor=white" alt="quantforge.rs" /></a>
  <a href="https://quantforge.rs/start-here"><img src="https://img.shields.io/badge/Start_Here-30353C?style=for-the-badge" alt="Start Here: run QuantForge in 5 minutes" /></a>
  <a href="https://quantforge.rs/contact"><img src="https://img.shields.io/badge/Contact-30353C?style=for-the-badge" alt="Contact" /></a>
</p>
<p>
  <a href="https://crates.io/crates/quantforge"><img src="https://img.shields.io/crates/v/quantforge?style=for-the-badge&logo=rust&color=AD521D&labelColor=15171A" alt="crates.io version" /></a>
  <a href="https://docs.rs/quantforge"><img src="https://img.shields.io/docsrs/quantforge?style=for-the-badge&color=30353C&labelColor=15171A&logo=docsdotrs&logoColor=white" alt="docs.rs" /></a>
  <a href="https://github.com/quantforge-rs/quantforge/blob/main/LICENSE"><img src="https://img.shields.io/crates/l/quantforge?style=for-the-badge&color=30353C&labelColor=15171A" alt="License" /></a>
  <a href="https://github.com/quantforge-rs/quantforge"><img src="https://img.shields.io/github/stars/quantforge-rs/quantforge?style=for-the-badge&color=30353C&labelColor=15171A&logo=github&logoColor=white" alt="Stars" /></a>
  <a href="https://github.com/quantforge-rs/quantforge/actions/workflows/ci.yml"><img src="https://github.com/quantforge-rs/quantforge/actions/workflows/ci.yml/badge.svg" alt="CI" /></a>
</p>

QuantForge ingests candles into SQLite, replays them through a strategy bar by bar, and — when you explicitly ask it to — sends the same strategy's orders to a real exchange. One Cargo package, MIT.

There is no UI. The CLI is the product.

</div>

- **Dry-run is the default.** Placing a real order takes an explicit `--mode live` and exchange credentials.
- **Backtests are deterministic.** The same stored candles and settings give the same result. An order fills at the next bar's open, and a position still open at the end is closed at the last bar's close.
- **No floating point for money.** Prices and volumes are decimal, because `f64` quietly loses cents.
- **No `unsafe`, no `unwrap()`.** The crate forbids unsafe code, and CI runs on Linux, macOS and Windows.

<div align="center">

**[quantforge.rs](https://quantforge.rs)** · **[docs.rs](https://docs.rs/quantforge)** · `cargo install quantforge`

<sub>Engineering software. Not investment advice, and no promise of profit.</sub>

</div>

<hr/>

<h2 align="center">Status</h2>

**Released: [v0.2.0](https://github.com/quantforge-rs/quantforge/releases/tag/v0.2.0).** This is what `cargo install quantforge` gives you today.

- Binance Spot
- SQLite storage
- A built-in SMA crossover, plus a Rust SDK for your own strategies
- Closed-bar polling
- Dry-run by default, live only with an explicit flag and credentials

One limit to know in 0.2.0: when a sync reaches the present, `data sync` stores the candle that is still forming and keeps that snapshot, so a backtest over that data reads bars that had not closed. A sync stores only closed candles, and replaces any snapshots in its range, once the candle containing its `--end` has closed. The quickstart syncs a past month and backtests exactly that month. The fix is on `main`.

**In development: v0.3.** The work lands on `main` in the open. It is not released, there is no date, and nothing on `main` counts as a release.

| Planned for 0.3 | Where it stands on `main` |
|---|---|
| PostgreSQL as a second storage backend, with SQLite staying the default | **Landed.** Every command runs on either store, with a verified SQLite → PostgreSQL migration and CI against PostgreSQL 16, 17 and 18. Container images are still to come. |
| Remote backtests and market data on a QuantForge server you run | **In progress.** The server serves candle history over HTTP and queues backtest jobs in sandboxed workers. Jobs cannot run a strategy yet, and live streaming is still ahead. |
| Strategies written in Python and run by the Rust core | **Groundwork.** The strategy boundary is plain data, and the backtest engine is a library API. The Python package is not built yet. |
| Bybit alongside Binance | **Groundwork.** Exchanges sit behind connector ports with a shared conformance suite. The Bybit adapter is not built yet. |
| Automation from your own TradingView alerts, on perpetual futures | **Designed.** [ADR 0002](https://github.com/quantforge-rs/quantforge/blob/main/docs/adr/0002-automation-architecture.md) fixes the architecture, and the futures types and ports are in. Execution is not built yet. |
| Stricter live runs | **Landed.** Going live asks for a typed `yes`, a resumed run must match its market, strategy and mode, and dry-run applies the same pre-trade checks as live. |

<sub>Checked against <code>main</code> on 2026-11-11. The <a href="https://github.com/quantforge-rs/quantforge/blob/main/CHANGELOG.md">changelog</a> lists everything unreleased, and the <a href="https://github.com/quantforge-rs/quantforge#roadmap">roadmap</a> says what comes after.</sub>

<hr/>

<h2 align="center">Quickstart</h2>

```bash
cargo install quantforge --locked

# pull a month of one-minute candles into SQLite
quantforge data sync --symbol BTCUSDT --interval 1m --start 2026-08-01T00:00:00Z --end 2026-09-01T00:00:00Z

# replay an SMA crossover over that month — same input, same numbers, every time
quantforge backtest --symbol BTCUSDT --interval 1m --fast 20 --slow 50 --cash 10000 \
  --start 2026-08-01T00:00:00Z --end 2026-08-31T23:59:00Z

# run the same strategy against live candles, without sending an order
quantforge trade run --symbol BTCUSDT --interval 1m --fast 20 --slow 50 --mode dry-run
```

<div align="center">

<sub>These commands are for 0.2.0, the current release. Dry-run is the default and sends no orders. Live needs <code>--mode live</code> with <code>QF_BINANCE_API_KEY</code> and <code>QF_BINANCE_API_SECRET</code> — point <code>QF_BINANCE_BASE_URL</code> at <code>https://testnet.binance.vision/</code> first. On <code>main</code> these settings have new names.</sub>

</div>

<hr/>

<h2 align="center">Repositories</h2>

| Repo | Lang | What it does | Links |
|---|---|---|---|
| [`quantforge`](https://github.com/quantforge-rs/quantforge) | Rust | the framework — data ingestion, deterministic backtests and guarded live trading in one CLI | [crates.io](https://crates.io/crates/quantforge) · [docs.rs](https://docs.rs/quantforge) |

<hr/>

<h2 align="center">Built with</h2>

<div align="center">
  <img src="https://go-skill-icons.vercel.app/api/icons?i=rust,sqlite,githubactions,linux&perline=7" alt="Rust, SQLite, GitHub Actions, Linux" />
</div>

<hr/>

<div align="center"><sub>MIT licensed · Rust 1.85+ · maintained by <a href="https://github.com/formaldehid">Marko Sabec</a> · more repos are being opened up as they stabilize.</sub></div>
