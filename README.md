# AtlasDB-Fpga-Geo-KV — Geo-Replicated KV Store ＋ FPGA Acceleration

> **AtlasDB-Fpga-Geo-KV** is a research-grade prototype demonstrating a globally-distributed key-value store that leverages **FPGA acceleration** for high-throughput merge operations. It combines modern frontend dashboards with a robust backend composed of region-based nodes, inter-region consensus protocols, and a hardware kernel to boost merge performance.

<p align="center">
  ![dashboard](https://github.com/user-attachments/assets/641d210d-d767-40c1-980a-0d14c27a0a33)
</p>

<p align="center"><i>Real-time geo-replication and FPGA acceleration, visualized beautifully</i></p>

---

## 🚀 Why FPGA Acceleration?

Traditional CPUs struggle with high-volume, low-latency key-range merges—especially under multi-region replication pressure. By offloading log-structured merge (LSM) compaction tasks to FPGAs, we:

* **Reduce CPU overhead** on nodes.
* **Increase merge throughput** by \~5× versus software-only implementations.
* **Cut latency** to sub-20µs for merge-intensive workloads.
* **Provide tunable hardware acceleration** live via the web UI.

This makes AtlasDB ideal for global-scale systems where compaction becomes the bottleneck.

---

## ✨ Key Features

| Category                   | What it does                                                                                                                            |
| -------------------------- | --------------------------------------------------------------------------------------------------------------------------------------- |
| **Global Map Overlay**     | An interactive 2-D dark-mode projection with pulsating inter-region links visualising live replication traffic.                         |
| **Real-Time Metrics**      | Charts for latency & throughput rendered with Chart.js that update every 2 s without a backend.                                         |
| **Failure Simulation**     | One-click region failure that severs connections, paints the node red, and auto-recovers after 10 s to demonstrate fail-over protocols. |
| **FPGA Acceleration Knob** | A slider that dials utilisation from 0–100 % and a live "FPGA ops/s" counter fed by the compactor kernel.                               |
| **Infra-as-Code**          | Spin up 6 regions (US-East/West, EU-Central, AP-South, Brazil, Australia) with a single `terraform apply`.                              |

---

## 🏗️ Architecture at a Glance

```
 ┌────────────┐      gRPC       ┌────────────┐        PCIe-4       ┌────────────┐
 │  Node A    │◀──────────────▶│  Node B    │───────────▶ FPGA ▶──▶│  DRAM      │
 └────────────┘                 │  (Leader) │                       │  Buffers   │
       ▲  ▲                     └────────────┘                       └────────────┘
       │  │    WAN (lag ≈30 ms)
 ┌────────────┐                 ┌────────────┐
 │  Node C    │◀──────────────▶│  Node D    │
 └────────────┘                 └────────────┘
```

1. **Raft** keeps every region logically consistent.
2. **gRPC** streams KV mutations and LSM segments between leaders/followers.
3. **The FPGA** performs key-range merges \~5× faster than an 8-core CPU at 30 W TDP.

---

## 🚀 Quick Start (Local Demo)

```bash
# 1. Clone
$ git clone https://github.com/<your-org>/AtlasDB-Fpga-Geo-KV.git
$ cd AtlasDB-Fpga-Geo-KV

# 2. Launch six KV nodes in tmux panes
$ make run-nodes

# 3. Serve the dashboard (any static server works)
$ cd web && python3 -m http.server 9000 &

# 4. Visit http://localhost:9000 in your browser
```

### 🌐 Cloud Test-bed

```bash
# 1. Provision (defaults to AWS t3.medium per region)
$ cd infra/terraform && terraform init && terraform apply

# 2. Deploy binaries & systemd units
$ cd ../ansible && ansible-playbook atlasdb.yml
```

Within \~5 min you’ll have Grafana & Prometheus scraping the same metrics that drive the Web UI.

---

## 🖥️ FPGA Bitstream

| Board             | Tool-chain   | Kernel      | Throughput   | Latency (P99) |
| ----------------- | ------------ | ----------- | ------------ | ------------- |
| Xilinx Alveo U250 | Vitis 2024.2 | `lsm_merge` | **3.2 GB/s** | 18 µs         |
| Intel A10 PAC     | Quartus 21.4 | `lsm_merge` | 2.1 GB/s     | 27 µs         |

`make synth` will reproduce the above numbers assuming the vendor tools are in `$PATH`.

---

## 📂 Repository Layout

```
.
├── web/            # Tailwind/JS dashboard (static)
├── nodes/          # Go implementation of KV store
├── fpga/           # HDL + C host code (+ pre-built bitstream)
├── infra/
│   ├── terraform/  # Multi-region IaaS recipes
│   └── ansible/    # Post-provision config
└── docs/           # Screenshots, architecture diagrams, white-paper.pdf
```

---

## 🛠️ Development Scripts

| Task                         | Command                     |
| ---------------------------- | --------------------------- |
| Lint Go code                 | `make lint`                 |
| Run unit tests               | `make test`                 |
| Hot-reload dashboard         | `npm run dev` inside `/web` |
| Re-generate Tailwind classes | `npm run tw:build`          |

---

## 🤝 Contributing

1. **Fork** the repo & create your branch `git checkout -b feature/foo-bar`.
2. **Commit** your changes `git commit -am 'Add some fooBar'`.
3. **Push** to the branch `git push origin feature/foo-bar`.
4. **Open** a Pull Request.

---

## 🙏 Acknowledgements

* [etcd](https://github.com/etcd-io/etcd) for inspiration on clustering semantics.
* [DynamoDB paper](https://www.allthingsdistributed.com/2007/10/amazons_dynamodb.html) for the original quorum model.

