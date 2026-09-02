# Sokuji Tsuyaku API — 调研分析(2026-09-02)

> 仓库: `kizuna-ai-lab/sokuji-tsuyaku-api` · Fork: `o1laabs/sokuji-tsuyaku-api`
> 即 Sokuji(实时双向语音翻译)官方推出的「可自部署的同传 API 服务器」。
> 本分析来自对上游仓库 README / CHANGELOG / 目录结构 / GitHub 元数据的实查核对。

## 一、一句话定位

把 Sokuji 桌面同传引擎的同传能力拆成**可独立部署的 API 服务端**:STT → 翻译 → TTS 全链路打包,
走 **WebSocket / WebRTC**,并**兼容 OpenAI 的 API 格式**(base_url 指向它 + OpenAI SDK/客户端即可调)。
MIT 协议。

## 二、出身与血缘

- 从 **speaches-ai/speaches**(TTS/STT 服务器)fork 改造,砍掉通用闲聊,专注**低延迟实时同传翻译**流水线。
- 主分支 `main`,Python 3.12,MIT License。
- 进度特征:★2 / 1 fork;实质内容提交集中在 2025-11 ~ 2026-01(2026-01-10 后基本趋缓)。
  属**官方早期 dev 项目**,用作生产核心需自行评估。

## 三、引擎与技术栈

| 环节 | 实现 |
|---|---|
| STT 语音识别 | Whisper / faster-whisper(SYSTRAN) |
| 翻译 | Helsinki-NLP **MarianMT**,1000+ 语对(opus-mt-en-zh 等) |
| TTS 语音合成 | Kokoro 82M / Piper |
| VAD 端点检测 | Silero VAD v5 |
| 推理加速 | CTranslate2(int8 / float16 / float32) |

## 四、性能口径(官方)

- 文本翻译:<100ms(模型加载后)
- 模型加载:2–5s(首次,含下载/转换),之后带 **TTL 缓存**(默认 300s)
- 实时音视频同传:<400ms 端到端(STT + 翻译 + TTS)
- 模型体积:约 200MB/模型(int8)

## 五、部署与启动(三选一)

Docker(仓库已备 compose 变体):
```bash
git clone https://github.com/kizuna-ai-lab/sokuji-tsuyaku-api.git && cd sokuji-tsuyaku-api
docker compose -f compose.cpu.yaml up   # 或 compose.cuda.yaml / compose.cuda-cdi.yaml
```
pip / uv 直接装:
```bash
pip install -e .  # 或 uv sync
speaches serve --host 0.0.0.0 --port 8000
# 等价: uvicorn speaches.main:create_app --factory --host 0.0.0.0 --port 8000
```

## 六、API 端点

| Endpoint | 方法 | 用途 |
|---|---|---|
| `/v1/translations` | POST | 纯文本翻译 |
| `/v1/audio/transcriptions` | POST | 语音转文字(ASR) |
| `/v1/audio/speech` | POST | 文字转语音(TTS) |
| `/v1/realtime` | WebSocket | 实时音视频同传 |
| `/v1/realtime` | POST | 实时音视频同传(WebRTC) |
| `/v1/models` | GET | 列出可用模型 |
| `/api/ps` | GET | 列出当前已加载模型 |
| `/health` | GET | 健康检查 |

实时同传模型参数(竖杠拼接 STT|TTS|翻译):
```
ws://localhost:8000/v1/realtime?model=Systran/faster-distil-whisper-small.en|speaches-ai/Kokoro-82M-v1.0-ONNX|Helsinki-NLP/opus-mt-en-zh
```

## 七、OpenAI 兼容用法示例

Python(OpenAI SDK,key 填 `not-needed`):
```python
from openai import OpenAI
client = OpenAI(base_url="http://localhost:8000/v1", api_key="not-needed")
tr = client.audio.translations.create(
    file=("text.txt", "Hello, how are you?".encode()),
    model="Helsinki-NLP/opus-mt-en-zh")
print(tr.text)  # 你好，你好吗？
```
curl:
```bash
curl -X POST "http://localhost:8000/v1/translations" \
  -F "text=Hello, how are you?" -F "model=Helsinki-NLP/opus-mt-en-zh"
```

## 八、关键配置(环境变量)

```bash
export MARIAN__MODEL_TTL=300           # 模型缓存秒数,默认 300
export MARIAN__INFERENCE_DEVICE=auto   # auto / cpu / cuda
export MARIAN__COMPUTE_TYPE=int8       # int8 / float16 / float32
export MARIAN__DEVICE_INDEX=0          # GPU 索引
```

## 九、与主仓库 sokuji 的关系对照

| 维度 | `whatever/sokuji` | `sokuji-tsuyaku-api` (本文) |
|---|---|---|
| 形态 | Electron 桌面 App + 浏览器扩展(人操作) | **API 服务端**(部署后代码/系统调用) |
| 协议 | AGPL-3.0 | **MIT**(更宽松) |
| 典型场景 | 装到 Win/Mac/Linux,虚拟麦克风插进 Zoom/Meet/Teams 直接同传 | 自建同传服务、写进自己的程序/会议系统后端 |
| 与桌面版的打通 | 桌面版内置"OpenAI Compatible"供应商(可指任意 OpenAI Realtime 兼容端点)→ 理论上可用它指向本服务器 | 提供给需要以 API 方式接入的服务 |

## 十、结论与建议

- **是**:Sokuji 家族确实提供可自部署的 API 版,重点在 WebSocket/WebRTC 实时同传 + OpenAI 兼容,自托管成本可控、数据不出本地。
- **但要清醒**:该项目 ★2、迭代趋缓、属早期 dev,直接扛生产单点服务有风险;实时链路对语对与模型配置敏感(<400ms 是理想负载口径)。
- 落地路径建议:
  1. 先在本地把 `compose.cpu.yaml` 拉起来,用 `curl /v1/translations` 和 WebSocket `/v1/realtime` 实测延迟;
  2. 语对够用优先用 Helsinki-NLP opus-mt(1000+ 语对,轻);
  3. 生产上若嫌维护成本,可只复用其"A 方案"设计,把小模型同传逻辑并入现有后端管道,而非直接托管完整服务器。
- 运行硬件:开发/评估机(MacBook Pro, M5 Pro, 24GB)用 CPU 跑小模型够看效果;要压低延迟再考虑 bigger Whisper 或 GPU。
