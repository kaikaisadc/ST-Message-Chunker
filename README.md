# ST-Message-Chunker（简体中文分支）

> **非官方简体中文 fork。** 上游项目：[Arczium/ST-Message-Chunker](https://github.com/Arczium/ST-Message-Chunker)，作者 **Arczium**，许可证 **MIT**。

本分支只翻译用户界面和文档，扩展逻辑、设置键、拦截器名称和默认行为保持上游原样。原版英文 README 完整保留在 [README.en.md](README.en.md)。

---

## 这是什么

ST-Message-Chunker 是一个面向 [SillyTavern](https://github.com/SillyTavern/SillyTavern) 的上下文长度管理扩展。它不是在每条消息生成时逐条滑动窗口，而是在生成前的最后一步按“块”一次性移除多条旧消息。这样做的目的是减少提示词缓存的频繁失效，从而更有利于缓存命中；实际能提升多少取决于模型提供商、缓存策略和聊天内容，本项目没有做性能基准承诺。

扩展在 `GENERATE_BEFORE_COMBINE_PROMPTS` 阶段对本次生成用的聊天副本做裁剪。它不删除或改写原消息正文，但会在聊天元数据中记录裁剪锚点 `mchunker_anchor_date`；也不依赖 SillyTavern 原生的消息隐藏功能。

---

## 功能

- **按块裁剪上下文**：一次移除一批旧消息，而不是每条消息都触发一次窗口滑动，对提示词缓存更友好。
- **不删除原消息**：只修改本次生成使用的聊天副本，不删除或改写原消息正文；会写入聊天元数据锚点 `mchunker_anchor_date`。
- **不干扰其他上下文工具**：在生成前最后阶段处理，尽量避开向量存储、摘要、世界书等流程。
- **两种模式**：按消息数量或按最大 token 数控制上下文。
- **可作为原生隐藏的替代方案**：如果你主要靠摘要记住旧剧情，只想把旧消息从本次提示中移除，这个扩展更直接。

---

## 运行模式

### 1. 消息数量模式

按原始消息条数维持一个稳定的上下文长度。

- **最少保留消息数**：希望保留在上下文中的基线消息数量。
- **分块大小**：超过上限时，一次移除多少条最旧消息。
- **示例**：最少保留 30 条，分块大小 10。按非 system 且有 `send_date` 的消息计数，聊天增长到 41 条时才会触发裁剪，一次移除最旧的 10 条，留下 31 条；之后大致在 31–41 条之间波动。

### 2. 最大 Token 数模式

根据聊天历史估算的 token 数进行裁剪。

- **最大上下文 Token 数**：希望控制到的 token 目标。
- **分块大小**：每次移除多少条最旧消息，直到估算 token 数回到目标以内。

token 数由 SillyTavern 当前使用的分词器估算，不同模型/分词器的结果可能不同。这个上限不是整个请求的硬上限：当可裁剪的非 system 消息不足一个分块时，扩展会停止裁剪，仍可能超出目标；系统提示、世界书等其他内容也不在此估算范围内。

---

## 安装

1. 打开 SillyTavern，进入 **Extensions（扩展）** 菜单（积木图标）。
2. 点击 **Install Extension（安装扩展）**。
3. 粘贴本 fork 地址：
   `https://github.com/kaikaisadc/ST-Message-Chunker`
4. 点击安装。
5. 刷新 SillyTavern，在扩展菜单中配置运行模式。

手动安装：

```bash
cd SillyTavern/data/default-user/extensions/third-party/
git clone https://github.com/kaikaisadc/ST-Message-Chunker
```

重启 SillyTavern 后启用。

---

## 从原版切换到本 fork

- 本扩展的内部名称是 `ST-Message-Chunker`，与原版相同，安装目录名也通常是 `ST-Message-Chunker`；**不要把原版和本 fork 并排安装后同时启用**，否则两个实例会同时拦截生成并重复裁剪上下文。
- 切换前请先在 SillyTavern 中导出当前聊天和设置（至少完整备份聊天 JSON 与 SillyTavern 数据目录）。扩展会在聊天元数据中保存裁剪锚点（`mchunker_anchor_date`），保留原聊天 JSON 可避免丢失可恢复的状态。
- 推荐步骤：备份并禁用原版 → 在扩展管理器中移除原版插件目录（只移除插件，不删除聊天）→ 安装本 fork → 重新配置。
- 熟悉 Git 的用户也可以在原插件目录中把 `origin` 改为本 fork 后拉取更新；操作前同样要备份，避免覆盖本地修改。

---

## 设置说明

| 设置 | 说明 |
|---|---|
| 运行模式 | 关闭、消息数量模式或最大 Token 数模式 |
| 分块大小 | 一次移除多少条旧消息 |
| 最少保留消息数 | 消息数量模式下希望保留的基线消息数 |
| 最大上下文 Token 数 | Token 模式下的目标上限（不是整个请求的硬上限） |

设置会保存在 SillyTavern 的扩展设置中，并随 SillyTavern 的设置一起持久化。

---

## 注意事项

- 该扩展只影响发送给模型的上下文，不删除聊天记录；被裁掉的消息仍可在聊天界面中查看。
- 裁剪发生在生成前的聊天副本上，不会删除或改写原消息正文；但会写入聊天元数据锚点 `mchunker_anchor_date`。
- “缓存命中率更高”是机制层面的设计目标，不是对任何具体 API 的实测承诺；实际效果取决于提供商。
- 与其他会修改生成上下文的扩展同时使用时，请自行确认它们的执行顺序和相互影响。

---

## 致谢

上游作者在 README 中说明：本扩展的核心思路受到 Omegastick 的 `SillyTavern-CacheChunker` 启发；由于原项目缺少开源许可证，本扩展是按 MIT 许可证重新实现的完整重写版本。详细说明见 [README.en.md](README.en.md)。

---

## 许可证与版权

- 上游作者：**Arczium**
- 上游仓库：<https://github.com/Arczium/ST-Message-Chunker>
- 许可证：**MIT**，完整文本见 [LICENSE](LICENSE)（未修改）。
- 本简体中文 fork：<https://github.com/kaikaisadc/ST-Message-Chunker>

本分支是非官方翻译，与上游作者无隶属关系。原项目版权归原作者所有；本分支的修改同样按 MIT 许可证发布。
