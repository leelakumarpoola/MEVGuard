# MEVGuard: Builder and Relay Decentralization Analysis

## 1. Overview and Objectives

In modern Proof-of-Stake (PoS) Ethereum, **Proposer-Builder Separation (PBS)** via MEV-Boost separates block construction from block proposal. Specialized entities known as **block builders** aggregate transactions and searcher MEV bundles to build candidate execution payloads, while **relays** act as trusted, mutually authenticated data-availability escrows between builders and consensus validators (proposers).

While PBS prevents validators from needing complex MEV-searching infrastructure, it has introduced centralization bottlenecks at both the builder and relay layers. High concentration creates systemic risks:
- **Transaction Censorship & Filtering**: Dominant builders and relays can enforce arbitrary transaction filtering (e.g., OFAC compliance) or delay inclusions.
- **Liveness and Resilience Risks**: Validator missed slots due to relay latency or outages.
- **Oligopoly Rent Extraction**: Concentration among a few builders dampens competitive execution for end-users.

The **Builder and Relay Decentralization Module** in MEVGuard provides analytical frameworks, statistical concentration metrics, and reproducible simulations to:
1. Measure structural concentration across block builders and relays independently.
2. Calculate the Herfindahl–Hirschman Index (HHI), largest participant share, and top-3 share.
3. Simulate multi-round competitive PBS block auctions with deterministic tie-breaking.
4. Model builder and relay outage resilience, quantifying re-normalization and single-point-of-failure vulnerabilities.
5. Benchmark architectural evolutions (Proto-PBS vs. Enshrined PBS EIP-7732 vs. Forward Inclusion Lists vs. SUAVE).

---

## 2. Builder and Relay Market Share Concepts

### Block Attribution
- **Builder Market Share**: Measures the proportion of proposed blocks successfully constructed by each distinct block builder over a defined observation window:
  $$\text{Market Share}_i = \left( \frac{\text{Block Count}_i}{\text{Total Observed Blocks}} \right) \times 100\%$$
- **Relay Market Share**: Measures the proportion of blocks delivered to proposers through each relay intermediary. Because relays do not build blocks, relay metrics are calculated **independently** from builder metrics without cross-contamination.

### Key Metrics
- **Largest Participant Share**: Percentage of total blocks constructed or delivered by the leading entity.
- **Top-3 Share Ratio**: Cumulative percentage share of the top three participants ($\sum_{i=1}^{\min(3, n)} \text{Share}_i$).
- **Diversity Count**: Count of active, distinct participants observed in the measurement window.

---

## 3. Herfindahl–Hirschman Index (HHI)

The Herfindahl–Hirschman Index (HHI) is a standard economic indicator used to measure market concentration. It is defined as the sum of squared market shares expressed in whole percentage points.

### Formula
$$\text{HHI} = \sum_{i=1}^{n} s_i^2$$

where $s_i$ is the market share percentage of participant $i$ (e.g., $50$ for $50\%$).

The scale ranges from near $0$ (infinitely decentralized market) to $10{,}000$ (pure monopoly, $100^2$).

### Worked Example
Consider three participants with market shares of $50\%$, $30\%$, and $20\%$:

$$\text{HHI} = 50^2 + 30^2 + 20^2 = 2500 + 900 + 400 = 3800$$

### Analytical Categories
Adapted from antitrust and market-structure guidelines:
- **Low Concentration ($\text{HHI} < 1{,}500$)**: Block construction is broadly distributed among competitive participants.
- **Moderate Concentration ($1{,}500 \le \text{HHI} \le 2{,}500$)**: Noticeable reliance on leading participants; moderate resilience risks.
- **High Concentration ($\text{HHI} > 2{,}500$)**: Severe concentration; an oligopoly controls block construction or relaying.

> [!NOTE]
> Concentration metrics measure structural distribution only. High concentration indicates architectural risk and reliance bottlenecks, but does not constitute proof of active collusion or malicious behavior.

---

## 4. PBS Auction Simulation

The PBS auction simulator models the competitive dynamics of proposers selecting blocks from competing builders.

### Process
1. **Builder Bidding**: For each round/slot, each participating builder generates a candidate bid in ETH. Bids are drawn from a Gaussian distribution $(\mu, \sigma)$ representing the dynamic value of extractable MEV and transaction fees in that block.
2. **Proposer Selection**: The proposer evaluates all candidate bids and chooses the block with the highest bid value (first-price sealed auction).
3. **Deterministic Tie-Breaking**: If two or more builders submit identical bids within tolerance ($10^{-6}$), the tie is resolved lexicographically by builder identifier.
4. **Seed Reproducibility**: When a numeric `seed` is supplied, pseudorandom bid generation is 100% deterministic, enabling exact reproduction for testing and academic demonstrations.

---

## 5. Builder and Relay Outage Simulation

The outage resilience simulator models the impact when a key participant suddenly fails, goes offline, or is partitioned from the network.

### Methodology
1. **Target Identification**: The user specifies the participant type (`builder` or `relay`) and the participant ID undergoing an outage.
2. **Impact Assessment**: The module records the baseline blocks and market share percentage that would be lost.
3. **Surviving Share Re-normalization**: The target participant is removed from the dataset, and shares are recalculated among the surviving participants:
   $$\text{New Share}_j = \left( \frac{\text{Block Count}_j}{\sum_{k \neq \text{target}} \text{Block Count}_k} \right) \times 100\%$$
4. **Boundary Safety**: If all participants fail (zero survivors), the simulation reports total workflow collapse safely without division-by-zero errors.

---

## 6. Architectural Evolution: Overcoming PBS Limitations

MEVGuard evaluates four paradigms of block building architecture:

| Paradigm | Relay Dependency | Dominant Relays | Missed Slot Risk | Censorship Rate | Builder HHI | Trust Model |
| :--- | :---: | :---: | :---: | :---: | :---: | :--- |
| **Proto-PBS (MEV-Boost)** | Yes | Top 3 (~90%) | Off-chain timeout (~5%) | High (OFAC filtering ~45%) | ~3,108 (High) | Out-of-protocol trusted relays |
| **Enshrined PBS (ePBS / EIP-7732)** | No | None (0) | 0% (Protocol PTC) | High (~45%) | ~3,108 (High) | In-protocol consensus committees |
| **ePBS + Inclusion Lists (FOCIL)** | No | None (0) | 0% (Protocol PTC) | **0% (Enforced inclusion)** | ~2,850 (Moderate) | Consensus-forced inclusion |
| **SUAVE / TEE Builders** | No | None (0) | 0% | **0%** | **~820 (Decentralized)** | Confidential compute enclaves |

---

## 7. API Endpoints Reference

### 1. `POST /api/decentralization/analyze`
- **Description**: Computes builder and relay market shares, top-3 concentration, HHI, and diagnostic warnings.
- **Request Body**:
  ```json
  {
    "source_type": "user_supplied",
    "measurement_window": "Observation Window",
    "builder_observations": [{"builder_id": "b1", "block_count": 50}],
    "relay_observations": [{"relay_id": "r1", "block_count": 50}],
    "save_experiment": true
  }
  ```
- **Response**: `DecentralizationAnalysisResponse` with `builder_metrics`, `relay_metrics`, and `warnings`.

### 2. `POST /api/decentralization/pbs-simulate`
- **Description**: Runs multi-round PBS block auction simulation.
- **Request Body**:
  ```json
  {
    "builder_ids": ["builder_a", "builder_b", "builder_c"],
    "num_rounds": 100,
    "seed": 42,
    "save_experiment": true
  }
  ```
- **Response**: `PBSSimulationResponse` with win distributions and round-by-round logs.

### 3. `POST /api/decentralization/outage-simulate`
- **Description**: Simulates the resilience impact of participant failure and share re-normalization.
- **Request Body**:
  ```json
  {
    "participant_type": "builder",
    "outage_participant_id": "builder_alpha",
    "baseline_observations": [...],
    "save_experiment": true
  }
  ```
- **Response**: `OutageSimulationResponse` with baseline vs. surviving metrics comparison.

### 4. `GET /api/decentralization/sample-data`
- **Description**: Returns the pre-calibrated 1,000-block academic dataset with hand-calculated benchmark metrics.

### 5. `POST /api/decentralization/epbs-compare`
- **Description**: Benchmarks Proto-PBS, Enshrined PBS, Inclusion Lists (FOCIL), and SUAVE architectures.

---

## 8. Limitations: Synthetic vs. Live Mainnet Data

- **Provenance**: Sample datasets in MEVGuard are labeled `illustrative_sample`. They are synthetically generated for deterministic educational analysis and unit testing.
- **Live Latency & Network Dynamics**: In production Ethereum mainnet, builder bids are dynamic real-time streams delivered via p2p gossip and WebSocket relays within milliseconds of slot boundaries. The simulation abstracts latency variance and payload validation overhead.
- **Relay Fallbacks**: In live PBS, relay timeouts trigger local validator block building fallbacks, which change the resulting block distribution compared to a static re-normalization model.
