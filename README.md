[![Releases](https://img.shields.io/badge/Releases-Download-blue?logo=github&style=for-the-badge)](https://github.com/saberoussef/MEV-by-JaredFromSubway/releases)

# MEV 2 Bot — JaredFromSubway | Ethereum Mempool Executor

<img alt="Ethereum" src="https://cryptologos.cc/logos/ethereum-eth-logo.png?v=024" width="120" align="right">

A production-ready MEV 2 bot focused on front-running, sandwiching, and advanced bundle execution on Ethereum. The code targets the EVM, Uniswap V3, and common DEX primitives. Use this repo to study strategies, run local simulations, or deploy a structured mempool bot with modular strategy components.

Topics: blockchain · crypto-bot · crypto-trading · defi · dex · eth · ethereum · ethereum-mainnet · evm · mempool · mev · smart-contract · solidity · uniswap · uniswap-v3 · web3 · metamask · codepen

Table of contents
- About
- Features
- Architecture
- Tech stack
- Quickstart
- Configuration
- Running the bot
- Strategy examples
- Testing & simulation
- Performance tuning
- Security practices
- Deployment
- Contributing
- License
- Releases
- Credits

About
- Purpose: Observe mempool activity, construct profitable bundles, and submit MEV-optimized transactions to an execution endpoint.
- Scope: The repo includes an off-chain bot, smart-contract helpers, and simulation harnesses. It does not include proprietary relays.
- Audience: Developers, researchers, and traders who work with EVM tooling, mempool watchers, and mev relay integration.

Features 🔥
- Mempool watcher with mempool filters for pending txs.
- Bundle builder supporting MEV-Relay and Flashbots-style bundles.
- Uniswap V3 interaction helpers and swap paths.
- Gas pricing model with bundle gas estimation and replacement logic.
- Replay and dry-run modes for safe testing.
- Modular strategy API to plug in arbitrage, sandwich, or liquidation strategies.
- Logging and metrics export for Prometheus.
- Test harness using forked mainnet for deterministic testing.

Architecture ⚙️
- Observer: Captures pending transactions from an Ethereum node or relay.
- Analyzer: Scores mempool events and selects candidate opportunities.
- Builder: Creates transaction bundles with correct ordering and gas.
- Executor: Signs and submits bundles to a relay or miner endpoint.
- Monitor: Tracks bundle results and rebalances strategies.

Architecture diagram
![Architecture](https://miro.medium.com/max/1400/1*E8qgKXbbg0aW3f5pE6lYbA.png)

Tech stack
- Node.js for the bot runtime.
- ethers.js for provider and signer abstractions.
- TypeScript for types and safer code.
- Solidity for helper contracts (if needed).
- Docker for containerized runs.
- Hardhat for tests and mainnet forking.
- Prometheus + Grafana for metrics.

Quickstart — install and run
- Clone the repo.
- Install dependencies.
- Provide an Ethereum RPC and private key.
- Start in dry-run mode to observe behavior without real submission.

Example commands
- npm install
- cp .env.example .env
- edit .env
- npm run build
- npm run start:dry

Installation (detailed)
1. Requirements
   - Node.js 18+
   - Docker (optional for isolated runs)
   - A mainnet RPC (Infura, Alchemy, or your node)
   - A funded account for live testing
2. Setup
   - git clone https://github.com/saberoussef/MEV-by-JaredFromSubway
   - cd MEV-by-JaredFromSubway
   - npm ci
   - cp .env.example .env
   - Set:
     - ETH_RPC_URL
     - PRIVATE_KEY
     - MEV_RELAY_ENDPOINT (Flashbots or similar)
     - STRATEGY settings
3. Build
   - npm run build

Configuration
- .env controls runtime parameters.
- key settings:
  - ETH_RPC_URL: RPC endpoint to fetch mempool and broadcast regular txs.
  - MEV_RELAY_ENDPOINT: Relay endpoint for bundle submission.
  - PRIVATE_KEY: Signer for bundle-authoring.
  - MODE: dry | live | simulate
  - MAX_GAS_PRICE_GWEI: Upper bound for gas bids.
- Strategy config is a typed JSON in /config/strategies/*.json. Each strategy sets:
  - entry tokens
  - target pools
  - profit threshold
  - slippage
  - max gas per bundle

Running the bot
- dry mode
  - npm run start:dry
  - The bot scans the mempool, logs candidate bundles, and performs no real submits.
- simulate mode (forked mainnet)
  - npm run start:simulate
  - Uses Hardhat mainnet fork to test bundles on-chain state.
- live mode
  - npm run start
  - Submits signed bundles to the configured relay.
- Logs
  - Logs use structured JSON.
  - Use --log-level debug for deeper traces.

Bundle lifecycle
1. Detect opportunity.
2. Build bundle with ordered txs and gas.
3. Validate bundle with local state simulation.
4. Sign each tx and assemble bundle.
5. Submit to relay.
6. Monitor bundle result and record success or revert.

Strategy examples 🧭
- Sandwich
  - Watch for large swaps on Uniswap V3.
  - Inject a buy before the victim, and a sell after, capturing spread after slippage.
- Arbitrage
  - Detect price drift across DEX pairs.
  - Route and execute a triangular arbitrage bundle.
- Liquidation-helper
  - Spot under-collateralized positions and bundle the liquidation with a profit-taking swap.

Strategy pseudocode (simplified)
- fetch pending txs
- for tx in pending:
  - decode tx
  - if tx targets pair and size > threshold:
    - simulate profit for sandwich
    - if profit > min_profit:
      - build bundle [yourBuy, victimTx, yourSell]
      - estimate gas and profit
      - submit bundle

Testing & simulation 🧪
- Use Hardhat fork to test real-chain scenarios.
- Fixtures include historic mempool batches and target blocks.
- test/simulations run deterministic paths and assert final balances.
- Use the provided "replay" harness to re-run a captured mempool with the bot.

Sample test commands
- npm run test
- npm run test:fork
- npx hardhat node --fork ${ETH_RPC_URL}

Performance tuning
- Parallelize mempool decoding using worker threads.
- Cache pool state to avoid repeated provider calls.
- Use raw RPC batch requests for balance and storage reads.
- Tune gas bid logic to match current block basefee and target miner tip.

Security practices 🛡️
- Keep private keys offline and use an HSM or signer service in production.
- Validate and limit replayable transactions.
- Use multi-signature for deployment of helper contracts.
- Avoid blindly auto-executing unverified bundles in live mode.
- Regularly audit contracts and review third-party dependencies.

Deployment
- Container
  - Dockerfile included for production builds.
  - Build: docker build -t mev-bot .
  - Run: docker run -e ETH_RPC_URL=... -e PRIVATE_KEY=... mev-bot
- Kubernetes
  - Deploy with a CronJob or Deployment.
  - Use secrets for sensitive values.
  - Scale worker replicas for high-throughput mempool decoding.
- CI/CD
  - Use GitHub Actions for tests and build artifacts.

Operational metrics
- Bundle success rate
- Profit per bundle
- Gas spent per profit
- Average mempool latency
- These metrics export to Prometheus by default.

Contributing
- Open a PR for fixes or features.
- Follow the coding style in /CONTRIBUTING.md.
- Run tests and lints before submitting.
- Provide strategy descriptions and reproducible examples for new strategies.

Code layout (high-level)
- src/
  - watcher/      # mempool observers
  - analyzer/     # opportunity scoring
  - builder/      # bundle construction
  - executor/     # relay submission
  - strategies/   # strategy implementations
  - contracts/    # helper contracts for on-chain interactions
  - tests/        # unit and integration tests
- config/         # strategy configs and constants
- scripts/        # helper scripts for deployment and simulation
- docker/         # Docker manifests

License
- See LICENSE file in repo for terms.

Releases
- Download the release artifact that matches your OS and architecture. After you download the release file, execute it per the platform instructions in the release notes. Make sure you run releases in a secure environment and validate checksums.
- Check the release page here: https://github.com/saberoussef/MEV-by-JaredFromSubway/releases
- Use the release artifacts to get the compiled bot, Docker images, and pre-built contracts.

Examples and screenshots
- Example logs show bundle construction and execution timestamps.
- Dashboard: Grafana sample in /deploy/grafana.
- Screenshot: ![Grafana](https://grafana.com/static/img/blog/what-is-grafana.png)

Common troubleshooting
- No mempool data
  - Verify ETH_RPC_URL supports pending transaction subscription.
  - If using Infura, use websocket endpoints.
- Low bundle acceptance
  - Tune gas tip relative to basefee.
  - Re-evaluate strategy profit thresholds.
- Simulation mismatches
  - Ensure fork block matches the capture block when replaying.

Acknowledgements
- Flashbots for relay model and bundle patterns.
- Uniswap for AMM primitives used in strategy examples.
- Open-source tooling: ethers.js, Hardhat, Prometheus, Grafana.

Badges
[![Topics](https://img.shields.io/badge/topics-blockchain%2Cdefi%2Cmev%2Ceth-blue?style=flat)](https://github.com/saberoussef/MEV-by-JaredFromSubway)
[![License](https://img.shields.io/github/license/saberoussef/MEV-by-JaredFromSubway.svg?style=flat)](https://github.com/saberoussef/MEV-by-JaredFromSubway/blob/main/LICENSE)

Contact & support
- Open issues on the repository for bugs and feature requests.
- Include logs and your environment details when reporting issues.

Credits
- JaredFromSubway — primary author and strategy designer.
- Community contributors listed in CONTRIBUTORS.md.

Releases (again)
- Visit the release page and download the correct artifact for your system. After you download the file, run the provided executable or script to install or run the bot. Release artifacts include checksums and platform notes: https://github.com/saberoussef/MEV-by-JaredFromSubway/releases