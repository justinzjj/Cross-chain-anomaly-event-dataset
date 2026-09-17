# Cross-chain Anomaly Event Dataset

这是跨链异常事件与交易数据集的首版可维护目录。当前版本基于 `2026-09-17-V4-dataset` 整理，保留事件、交易及其原有不确定性，不修改原始数据。

## 目录

```text
1-Event/
├── events.csv
└── event_description/
    └── EVT-*.json                  # 161 个事件描述文件
2-Transaction/
└── transactions.csv
```

`events.csv` 保存事件级索引。`event_description/` 保存独立事件的详细材料，已迁入 161 张按旧事件编号明确匹配的事件卡片。`transactions.csv` 保存逐笔交易索引。

## 数据数量

- 事件：203 条。
- 交易：803 条。
- 事件—交易关联：805 条。关联关系由 `transactions.csv` 的 `event_ids` 汇总表示，其中一笔交易可以关联多个事件。
- 交易标签：696 条 `candidate_anomaly`、84 条 `associated`、23 条 `excluded`，本版没有把候选直接标为 `confirmed_anomaly`。
- 事件记录类型：170 条 `independent_event`、33 条 `pending_split_group`。

## 事件字段

`events.csv` 的固定字段为：

`event_id`、`event_name`、`event_date`、`date_precision`、`record_type`、`project_name`、`protocols`、`chains`、`crosschain_scope`、`description`、`review_status`。

`event_id` 使用 `EVT-000001` 形式，按 V4 事件表原始顺序首次分配，后续不得因排序、日期或名称变化重新编号。`record_type` 区分独立事件和待拆分来源分组。`protocols` 与 `chains` 是合法 JSON 数组字符串，未知或无法确定的值不填入数组。`description` 只保存简短索引概述，详细经过、根因、影响和引用链接应放入事件 JSON。现已迁入 161 张已有事件卡片；其余 42 条事件没有对应卡片，暂不创建空文件。

`review_status` 只使用 `pending`、`verified`、`disputed`。由于本版没有重新完成事件核实，事件行初始均为 `pending`，不能由旧表的 `confirmed` 等状态自动推导为 `verified`。

## 交易字段

`transactions.csv` 的固定字段为：

`transaction_id`、`event_ids`、`chain`、`tx_hash`、`block_number`、`timestamp`、`from_address`、`to_address`、`transaction_role`、`anomaly_type`、`description`、`label_status`、`review_status`、`asset_movements`、`amount_description`、`counterpart_transactions`、`pairing_status`、`reference_urls`、`notes`。

`transaction_id` 使用 `TXN-000001` 形式，按 V4 交易表原始顺序首次分配。交易身份仍应使用 `chain + tx_hash` 检查。链的哈希按来源保留大小写，不统一转小写。源表没有可靠区块号、发送地址或接收地址的记录，本版留空；源表中的一个全零哈希占位按缺失哈希留空，但对应交易记录和关联关系保留。

`event_ids` 是引用新 `event_id` 的 JSON 数组字符串，由 V4 事件—交易对应表聚合得到。`label_status` 按原记录性质映射：攻击/异常交易候选为 `candidate_anomaly`，事件关联交易为 `associated`，已拒绝候选为 `excluded`。`transaction_role` 使用 JSON 数组保存来源角色，按分号拆分、去除占位符并按大小写不敏感去重；`anomaly_type` 使用 JSON 数组保存来源的链上行为标签，二者不互相替代。交易行的 `review_status` 初始均为 `pending`。

`asset_movements` 是形如 `[{"asset":"...","amount":"..."}]` 的 JSON 对象数组。只有来源中资产和金额能够可靠成对解析时才结构化，金额始终为字符串；其余原始金额描述保存在 `amount_description`。`counterpart_transactions` 是包含 `chain` 和 `tx_hash` 的 JSON 对象数组。已有报告明确哈希关系但没有本轮链上核验的记录使用 `report_supported`，不能标为 `onchain_verified`；无法可靠配对的记录使用 `unpaired`。

`timestamp` 统一为 UTC。本次仅凭 V4 的 `target_times` 无法确认其为实际交易时间，因此全部留空；即使格式符合 ISO 8601，也不据此认定为实际交易时间。事件日期不替代交易时间。`reference_urls` 是合法 JSON 数组字符串，直接保留交易记录中的现有链接。

## 使用约定

- 多值字段使用 JSON 数组或 JSON 对象数组字符串，空集合写 `[]`。未知标量留空，不把说明文字写入日期、哈希或数值字段。
- 不把待拆分分组、候选异常、事件关联和已拒绝候选混为确认异常。
- 新增事件或交易应追加稳定编号，并通过 `event_ids` 建立关联，不因展示排序重编号。
- 事件 JSON 的 `event_id`、`event_name` 必须与 `events.csv` 一致；JSON 中的 `transaction_ids` 只能引用明确关联、链与非占位哈希构成可检查身份的交易；不得引用 TXN-000023。
- 本版包含 161 个事件描述 JSON。迁移保留历史材料，不代表本次重新核实事件或交易。


## 本次修正

重新读取 V4 三张 CSV，事件和交易编号严格按各自主表原始行顺序分配；关联完全从 `03-事件交易对应表.csv` 重建，并逐项比较关系集合及数组元素总数。

事件名称优先使用明确旧编号匹配的卡片 H1（去掉标题中的旧编号前缀），无匹配时保留 V4 名称。已识别并迁入 161 张明确编号卡片，按新 EVT 编号保存为独立 JSON。事件概述取卡片中简短、完整的原句；无法取得可靠概述时留空。项目字段清除长描述，协议只保留来源明确协议字段，链数组排除钱包、资金池、泛称和 unknown。`crosschain_scope` 按 V4 `direct_crosschain` 映射为 direct、partial、non_crosschain 或 unclear。

保留全部 803 条源记录，其中 8 条无法形成可检查的交易身份：TXN-000023 的全零哈希置空；TXN-000797 至 TXN-000803 的未知链置空，保留原始非占位哈希。上述记录统一 excluded / pending，并清空角色、行为、描述、资产结构和对端配对；关系全部保留。23 条 excluded 包含原有 15 条已拒绝候选。身份格式可检查不等于已重新链上核实。

全部事件和交易 review_status 均为 pending。本次没有重新联网核实事件或链上交易，交易 description 无可靠直接概述时留空，不用标签模板凑写。

## 事件描述卡片

`1-Event/event_description/<event_id>.json` 与事件表一一关联。161 张卡片均以卡片正文明确的旧事件编号匹配 V4，再对应到既定 EVT 编号，不按名称或日期猜测匹配。已有 THORChain 样例保留并补入完整卡片内容。两张 CSV 不因卡片迁移改变。

每份 JSON 保留 `summary`、`background`、`timeline`、`root_cause`、`impact`、`transaction_ids`、`references`、`uncertainties` 等字段；无法可靠提取的标量为 null，列表为 []。`sections` 保存原卡片各章节全文，包括基本信息、原因经过、处置和证据边界；`source_card_markdown` 无损保留完整原卡片，`source_event_id` 记录编号映射。原文中的旧账本相对链接仅作为历史文本保留；对外引用可通过 `references` 中的绝对 URL 访问。

`transaction_ids` 只引用交易表中与该事件明确关联且具备可检查链与非占位哈希的记录。卡片内原有历史核验措辞不代表本轮新增核实，JSON 的 `review_status` 均为 pending。源卡片目录保持不变。
