# docs/ 目录索引

> 本目录是 **Sokuji Tsuyaku API**(即時通訳 API サーバー)的文档区。
> 仓库 `kizuna-ai-lab/sokuji-tsuyaku-api` 是 [speaches-ai/speaches](https://github.com/speaches-ai/speaches)
> 的 fork 改造版,因此大部分 mkdocs 页面沿用了上游 Speaches 的结构与说明,
> 再叠加本仓库特有的「实时翻译」扩展。

## 本仓库新增/核心文档

| 文件 | 说明 |
|---|---|
| [`sokuji-tsuyaku-api-analysis.md`](./sokuji-tsuyaku-api-analysis.md) | ★ 本 fork 增加的调研分析(o1laabs 分析,2026-09-02)。**这个才是讲「本 API 服务器本身是什么、怎么部署、怎么调」的中文地图**,和上游英文文档互补。 |
| [`UPSTREAM-SYNC.md`](./UPSTREAM-SYNC.md) | ★ 本 fork 增加的说明:如何把本 fork 与上游 `kizuna-ai-lab/sokuji-tsuyaku-api`/`speaches-ai/speaches` 保持同步。 |

## 上游沿用的文档(Speaches / mkdocs 站点)

> 注意:命名主要面向开源交付/开发者直接读原始 speaches.ioshed,未在本 fork 彻底 re-brand(见 mkdocs.yml 仍指向 speaches.ai / speaches-ai/speaches)。

| 文件 | 内容 |
|---|---|
| [`index.md`](./index.md) | Speaches 主页说明(上游模型,音频模型"学 Ollama 之于 TTS/STT")。 |
| [`installation.md`](./installation.md) | 安装指南(Docker / pip / uv)。 |
| [`api.md`](./api.md) | OpenAI 兼容 API 参考与端点说明。 |
| [`configuration.md`](./configuration.md) | 服务配置选项。 |
| [`ctranslate2-cudnn-fix.md`](./ctranslate2-cudnn-fix.md) | CUDA/CTranslate2 cuDNN 相关的已知问题与修复。 |
| [`troubleshooting.md`](./troubleshooting.md) | 常见故障排查。 |
| [`openapi.json`](./openapi.json) | OpenAPI schema 文件。 |

### usage/ 使用能力细览

| 文件 | 内容 |
|---|---|
| [`usage/model-discovery.md`](./usage/model-discovery.md) | 模型如何被发现/按需加载。 |
| [`usage/speech-to-text.md`](./usage/speech-to-text.md) | ASR(语音→文本)能力。 |
| [`usage/text-to-speech.md`](./usage/text-to-speech.md) | TTS(文本→语音)能力。 |
| [`usage/voice-chat.md`](./usage/voice-chat.md) | 语音对话(chat)用法。 |
| [`usage/realtime-api.md`](./usage/realtime-api.md) | WebRTC/WebSocket 实时音频接口。 |
| [`usage/vad.md`](./usage/vad.md) | 语音端点检测(Silero VAD v5)。 |
| [`usage/open-webui-integration.md`](./usage/open-webui-integration.md) | 与 Open WebUI 集成。 |
| [`usage/speech-embedding.md`](./usage/speech-embedding.md) | 语音嵌入。 |
| [`usage/dynamic-loading.md`](./usage/dynamic-loading.md) | 动态加载/TTL 卸载模型。 |

## 本 fork 源码里与「翻译」相关的核心部分

- 源码主目录 `src/speaches/`。翻译服务(端点 `/v1/translations`、实时 pipeline)
  在 fork 中做了大幅改造,与上游的"纯 TTS/STT"实现不同。改动脉络见仓库根
  [`CHANGELOG.md`](../CHANGELOG.md)([0.2.0] 迁移到 MarianMT 翻译为正线的 break)与
  [`MIGRATION_SUMMARY.md`](../MIGRATION_SUMMARY.md)。

---

> 快速上手看这份 ⇨ [`sokuji-tsuyaku-api-analysis.md`](./sokuji-tsuyaku-api-analysis.md)
