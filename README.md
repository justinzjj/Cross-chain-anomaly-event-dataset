# Cross-chain Anomaly Event Dataset

我们收集并整理了一套面向跨链安全研究的异常事件与交易数据集，包含跨链桥、跨链消息系统及相关生态中的攻击事件、漏洞披露和运行异常。数据集中已知事件日期从 **2020 年 5 月 18 日至 2026 年 9 月 11 日**，交易记录覆盖 Ethereum、BNB Chain、Bitcoin、Solana、XRP Ledger 等 **17 条已知区块链**。

当前数据集共包含 **203 条事件记录、803 条交易记录、805 条事件—交易关联，以及 161 份事件详细描述**。我们将事件背景、相关协议与链、交易信息和参考来源整理为 CSV 与 JSON 文件，作为公开数据集供研究者和开发者查询、下载与分析。

本数据集已收录 **[XChainWatcher](https://github.com/AndreAugusto11/XChainWatcher) 和 [Xscope](https://github.com/Xscope-Tool/Results)** 两项代表性跨链安全研究中报告的异常案例，包括 XChainWatcher 分析的 Ronin、Nomad 攻击，以及 Xscope 报告的 THORChain 攻击案例。我们将这些案例与其他公开异常事件统一组织，提供事件描述、相关交易及证据来源，便于查询与交叉分析。这里的覆盖指异常案例的收录，不表示完整复刻两项工作的全部交易数据或标签。

## 数据概览

| 内容 | 数量 / 范围 |
| --- | --- |
| 事件记录 | 203 条，其中独立事件 170 条、待拆分来源分组 33 条 |
| 交易记录 | 803 条 |
| 事件—交易关联 | 805 条，一笔交易可关联多个事件 |
| 事件详细描述 | 161 份 |
| 已知事件日期 | 2020-05-18 至 2026-09-11，另有 28 条记录日期未知 |
| 交易覆盖链 | 17 条已知链，另有 7 条交易记录缺少链信息 |

事件记录中，115 条为直接跨链相关，35 条为部分跨链相关，43 条的跨链相关性尚不明确，10 条为非跨链记录。交易记录中，696 条标记为异常候选，84 条为事件关联交易，23 条为已排除候选。

## 查询与下载

- [事件索引：events.csv](1-Event/events.csv) — 查询事件名称、日期、项目、协议及涉及的链。
- [交易索引：transactions.csv](2-Transaction/transactions.csv) — 查询交易哈希、所在链、关联事件、交易角色及参考来源。
- [事件详细描述：event_description/](1-Event/event_description) — 按事件编号查阅事件经过、原因、影响与证据材料。

CSV 文件可使用表格软件或 Python 等工具读取；JSON 文件可直接查看或按字段解析。事件与交易通过稳定编号关联，便于检索和联合分析。

## 数据格式

```text
1-Event/
├── events.csv
└── event_description/
    └── EVT-*.json
2-Transaction/
└── transactions.csv
```

### 事件索引

`events.csv` 每行对应一条事件记录。

| 字段 | 信息 |
| --- | --- |
| `event_id`、`event_name` | 事件编号与名称 |
| `event_date`、`date_precision` | 事件日期与日期精度 |
| `record_type` | 独立事件或待拆分来源分组 |
| `project_name`、`protocols`、`chains` | 相关项目、协议与链 |
| `crosschain_scope` | 跨链相关性 |
| `description` | 事件概述 |
| `review_status` | 审核状态 |

### 交易索引

`transactions.csv` 每行对应一条交易记录，通过 `event_ids` 关联一条或多条事件。链上交易身份由 `chain` 与 `tx_hash` 共同确定。

| 字段 | 信息 |
| --- | --- |
| `transaction_id`、`event_ids` | 交易记录编号与关联事件编号 |
| `chain`、`tx_hash` | 所在链与交易哈希 |
| `block_number`、`timestamp` | 区块高度与 UTC 交易时间 |
| `from_address`、`to_address` | 发送与接收地址 |
| `transaction_role`、`anomaly_type` | 交易角色与异常行为标签 |
| `description` | 交易概述 |
| `label_status`、`review_status` | 交易标签与审核状态 |
| `asset_movements`、`amount_description` | 资产变动与金额描述 |
| `counterpart_transactions`、`pairing_status` | 对端交易与配对状态 |
| `reference_urls`、`notes` | 参考来源与补充说明 |

### 事件详细描述

`event_description/<event_id>.json` 以事件编号命名，与事件索引对应。主要字段包括事件摘要（`summary`）、背景（`background`）、时间线（`timeline`）、根因（`root_cause`）、影响（`impact`）、关联交易（`transaction_ids`）、参考来源（`references`）和不确定性（`uncertainties`）。详细章节及来源原文保存在 `sections` 和 `source_card_markdown` 中。

CSV 中的多值字段使用 JSON 数组或对象数组字符串，空集合为 `[]`，未知标量留空。事件 JSON 中的缺失标量为 `null`，空列表为 `[]`。资产金额以字符串保存，交易哈希保留原始大小写。

## 数据说明

本数据集保留来源信息及其不确定性。事件记录数不等同于已确认的独立攻击数量；交易标签 `candidate_anomaly`、`associated` 和 `excluded` 分别表示异常候选、事件关联交易和已排除候选，不能直接作为已核验的异常或正常标签。

当前事件和交易的 `review_status` 均为 `pending`，交易时间全部留空，部分记录缺少区块高度、地址或完整交易身份。对端交易配对中的 `report_supported` 表示有报告支持，尚不代表链上核验。使用时可结合标签、审核状态及参考来源筛选所需数据。

欢迎通过 Issue 或 Pull Request 补充记录、提供证据或反馈问题。

## 许可

本仓库采用 [MIT License](LICENSE)。外部链接及引用材料的权利归其各自权利人所有。
