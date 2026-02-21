# Pentagon Chain - Technical Specification

*For CEX, Node Operators, and Data Aggregators (CoinMarketCap, CoinGecko, CoinRank, etc.)*

---

## Quick Facts

| Item | Current | Max (Elastic) |
|------|---------|---------------|
| **Name** | Pentagon Chain (PC) | - |
| **Type** | zkEVM Validity Rollup | - |
| **Decimals** | 18 (standard EVM) | - |
| **Block time** | ~20 seconds | <1 second (with ZK prover scaling) |
| **TPS** | ~100-500 | 2,000-4,000+ (scalable with parallel provers) |
| **Memo/tag** | Not supported | - |

**Performance Notes:** Pentagon Chain uses elastic scaling architecture. Block time and throughput are adjustable based on demand via ZK prover scaling. Current configuration optimized for cost efficiency. Can scale horizontally with additional prover capacity to meet high-throughput requirements.

---

## Endpoints

| Service | URL |
|---------|-----|
| Website | https://pentagon.games |
| Add RPC (Auto) | https://pentagon.games/addrpc |
| Explorer | https://explorer.pentagon.games |
| RPC | https://rpc.pentagon.games |
| Explorer API | https://explorer.pentagon.games/api |
| GitHub | https://github.com/blockchainsuperheroes |

---

## Node Requirements

| Type | Disk | RAM |
|------|------|-----|
| Full Node | 1 TB SSD min | 16-32 GB |
| Archival | 2 TB+ SSD | 16-32 GB |

---

## Key Points for CEX

1. **No memo/tag** - assign unique deposit address per user
2. **Standard EVM accounts** - create via MetaMask or any EVM wallet
3. **JSON-RPC compatible** - standard eth_* calls work
4. **Explorer API** - Tracehawk format

---

## API Calls

```
# Validate account
eth_getBalance(address)
eth_getCode(address) # check if contract

# Get block height
eth_blockNumber

# Get tx history
/api?module=account&action=txlist&address=0x...

# Get tx details
eth_getTransactionByHash
eth_getTransactionReceipt
```

---

## Node Setup

Follows Polygon CDK docs: https://docs.polygon.technology/cdk/operate/node/

Pentagon-specific configs (bootnodes, genesis, prover URLs, sequencer URL) provided by Zeeve.

Standard flags:
- `--http.port` (RPC)
- `--ws.port` (WebSocket)
- `--port` (P2P)
- `--datadir` (data directory)

---

## Deposit Security (CEX Integration)

To determine accurate deposits and avoid cheating recharge, follow standard EVM deposit policy. Confirm receipt.status == 1. Confirm the "to" address matches your deposit address. For ERC-20 tokens, verify Transfer event logs and that amount matches expected deposit. Wait for 50+ confirmations. Ensure no nonce replacement occurred. CDK validity proofs also guarantee finality once L1 batch is proven.

---

## Offline Signing & Broadcasting

Build raw transaction with chainId = 3344. Sign offline using private key or HSM. Broadcast via eth_sendRawTransaction.

---

## Account Balance

Native PC balance: eth_getBalance(address, "latest")

ERC-20 balance: call balanceOf using eth_call.

---

## Fork Prevention

Pentagon Chain uses Polygon CDK Validity Rollup. Blocks become immutable once ZK proof is posted on L1. Deep reorgs are not possible past proof finality. Exchanges may choose 50-100 confirmations or wait until batch is proven on L1 for maximum safety.

---

## Account Recovery

Standard EVM - backup seed phrase or private key. Pentagon Chain team cannot restore keys. For institutions: MPC, multi-signature, or hardware wallets recommended.

---

## Symbol

PC

---

## Official Wallets

Pentagon Pen Wallet (mobile): https://pentagon.games/penwallet/download

Also compatible with MetaMask and other EVM wallets using chainId 3344.

---

## Cross-Chain Bridge

PC exists as native gas token on Pentagon Chain and as ERC-20 on Ethereum via canonical bridge. Bridge uses Polygon CDK / AggLayer architecture, audited by Polygon Labs. No separate Pentagon-specific audit PDF published yet.

---

## Consensus & Security

Not PoW. Pentagon Chain is a zkEVM Validity Rollup. No miners. No 51% attack vector. Sequencer + ZK proofs guarantee chain integrity.

---

## Common Transfer Types

PC native transfer, ERC-20 transfer/transferFrom, ERC-721 safeTransferFrom, ERC-1155 safeTransferFrom. No chain-specific transfer opcodes.

---

## Rollback Policy

CDK supports minimal short reorgs (a few blocks) before proofs. After ZK proof finality, blocks are immutable. Exchanges should treat deposits as final after 50+ confirmations or after proof.

---

## Transaction Timeout

Ethereum mempool rules apply. No protocol-level TTL. Nodes may drop stale transactions. Replacement possible via higher gas with same nonce.

---

## Rent / Reserved Balance

None. Pentagon Chain has no account rent and no minimum balance requirement.

---

## UTXO

N/A. Pentagon Chain is account-based. To simulate failure, use eth_estimateGas(tx) - if reverted, the tx will fail in execution.

---

## Mainnet Status

Pentagon Chain mainnet and PC transfers launched in 2024. Transfers are fully live.

---

## Node Ports & NAT

P2P port should be reachable via NAT with port forwarding. RPC port MUST remain private and not exposed publicly.

---

## Address Format

0x + 40 hex characters (42 chars total). EIP-55 checksum optional.

---

## Node Sync Whitelist

No whitelist required. Pentagon Chain is permissionless. Nodes sync via standard CDK P2P.

---

---

## Pentagon Chain Architecture

Pentagon Chain is a zkEVM validity rollup with customized deployment and bridge logic.

**Permissionless:**
- Contract deployment
- Deposits via canonical bridge
- Node synchronization

**Permissioned:**
- Withdrawals (requires verification through our bridge contracts)

---

## Node Architecture

Pentagon Chain nodes consist of multiple components working together:

**Sequencer** - Orders transactions and produces L2 blocks. Pentagon Chain runs a centralized sequencer for performance, with decentralization roadmap planned.

**Executor** - Processes transactions and updates state. Handles EVM execution with full compatibility.

**State DB** - Stores current chain state. Uses PostgreSQL for state management and Merkle tree storage.

**Prover** - Generates zero-knowledge proofs for batches of transactions. Proofs are submitted to L1 for finality.

**Synchronizer** - Keeps nodes in sync with the network. Handles reorgs and state reconciliation.

**JSON-RPC Server** - Exposes standard Ethereum JSON-RPC interface for wallet and dApp connectivity.

---

## Node Deployment

Pentagon-specific configs including bootnodes, genesis, prover URLs, sequencer URL, and rollup config are provided by our infrastructure partner.

**Minimum Hardware:**
- CPU: 8 cores
- RAM: 32 GB
- Storage: 1 TB NVMe SSD (2 TB+ for archival)
- Network: 100 Mbps symmetric

**Software Requirements:**
- Linux (Ubuntu 22.04 recommended)
- Docker and Docker Compose
- PostgreSQL 15+

**Configuration Files:**
- genesis.json - Chain genesis configuration
- node-config.toml - Node runtime configuration  
- prover-config.json - ZK prover settings

---

## Batch & Proof Flow

1. Sequencer collects transactions and creates L2 blocks
2. Blocks are grouped into batches
3. Executor processes batch and generates witness
4. Prover creates ZK proof from witness
5. Proof is verified and posted to L1
6. State becomes final and immutable

Average batch time: 10-30 minutes depending on network load.

---

## Bridge Architecture

Pentagon Chain uses a native bridge for asset transfers between L1 (Ethereum) and L2 (Pentagon Chain).

**Deposits (L1 → L2):**
- Permissionless
- Lock assets on L1 bridge contract
- Claim on L2 after batch inclusion
- ~10-30 minute finality

**Withdrawals (L2 → L1):**
- Permissioned verification
- Initiate on L2
- Wait for ZK proof finality
- Claim on L1 after proof verification
- ~30-60 minute finality

---

## Reference Docs

JSON-RPC: https://ethereum.org/developers/docs/apis/json-rpc/

Explorer (Tracehawk): https://tracehawk.io/

---

*Last updated: 2026-02-21*
