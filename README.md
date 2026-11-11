# ST-Message-Chunker（简体中文分支）

> **非官方简体中文 fork。** 上游项目：[Arczium/ST-Message-Chunker](https://github.com/Arczium/ST-Message-Chunker)，作者 **Arczium**，许可证 **MIT**。

本分支只翻译用户界面和文档，扩展逻辑、设置键、拦截器名称和默认行为保持上游原样。原版英文 README 完整保留在 [README.en.md](README.en.md)。

---

## 这是什么

ST-Message-Chunker 是一个面向 [SillyTavern](https://github.com/SillyTavern/SillyTavern) 的上下文长度管理扩展。它不是在每条消息生成时逐条滑动窗口，而是在生成前的最后一步按“块”一次性移除多条旧消息。这样做的目的是减少提示词缓存的频繁失效，从而更有利于缓存命中；实际能提升多少取决于模型提供商、缓存策略和聊天内容，本项目没有做性能基准承诺。

扩展在 `GENERATE_BEFORE_COMBINE_PROMPTS` 阶段对本次生成用的聊天副本做裁剪，不改写已保存的聊天记录，也不依赖 SillyTavern 原生的消息隐藏功能。

---

## 功能

- **按块裁剪上下文**：一次移除一批旧消息，而不是每条消息都触发一次窗口滑动，对提示词缓存更友好。
- **非破坏性**：只修改本次生成使用的聊天副本，不改动磁盘上的聊天文件。
- **不干扰其他上下文工具**：在生成前最后阶段处理，尽量避开向量存储、摘要、世界书等流程。
- **两种模式**：按消息数量或按最大 token 数控制上下文。
- **可作为原生隐藏的替代方案**：如果你主要靠摘要记住旧剧情，只想把旧消息从本次提示中移除，这个扩展更直接。

---

## 运行模式

### 1. 消息数量模式

按原始消息条数维持一个稳定的上下文长度。

- **最少保留消息数**：希望保留在上下文中的基线消息数量。
- **分块大小**：超过上限时，一次移除多少条最旧消息。
- **示例**：最少保留 30 条，分块大小 10。聊天增长到 40 条时，扩展一次移除最旧的 10 条，使上下文回到约 30 条。

### 2. 最大 Token 数模式

根据聊天历史估算的 token 数进行裁剪。

- **最大上下文 Token 数**：允许的 token 上限。
- **分块大小**：每次移除多少条最旧消息，直到估算 token 数回到上限以内。

token 数由 SillyTavern 当前使用的分词器估算，不同模型/分词器的结果可能不同。

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

- 本扩展的内部名称是 `ST-Message-Chunker`，与原版相同；**不要同时启用原版和本 fork**，否则两个实例会同时拦截生成并重复裁剪上下文。
- 切换前请先在 SillyTavern 中导出当前聊天和设置（至少完整备份聊天 JSON 与 SillyTavern 数据目录）。扩展会在聊天元数据中保存裁剪锚点（`mchunker_anchor_date`），保留原聊天 JSON 可避免丢失可恢复的状态。
- 如果原版目录仍保留，建议先禁用原版再启用本 fork，确认聊天正常后再删除旧目录。

---

## 设置说明

| 设置 | 说明 |
|---|---|
| 运行模式 | 关闭、消息数量模式或最大 Token 数模式 |
| 分块大小 | 一次移除多少条旧消息 |
| 最少保留消息数 | 消息数量模式下希望保留的基线消息数 |
| 最大上下文 Token 数 | Token 模式下允许的最大 token 数 |

设置会保存在 SillyTavern 的扩展设置中，并随 SillyTavern 的设置一起持久化。

---

## 注意事项

- 该扩展只影响发送给模型的上下文，不会删除聊天记录；被裁掉的消息仍可在聊天界面中查看。
- 裁剪发生在生成前的聊天副本上，因此不会改写原始聊天文件。
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
