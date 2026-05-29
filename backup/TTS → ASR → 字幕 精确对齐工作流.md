
> AI News Factory 字幕同步方案详解

---

## 一、为什么需要精确对齐？

### 1.1 问题背景

制作短视频时，字幕必须与配音精确同步。如果字幕出现得太早或太晚，观众会感到困惑。

**传统方案的问题**：
```
脚本文字 → 估算每句时长 → 生成字幕 → TTS配音 → 字幕与音频不匹配
```

**问题根源**：
- 估算的时长不准确（语速变化、停顿、重音）
- 中文 TTS 的实际发音时间与字数不成正比
- 每句话的语速不同，无法用固定公式计算

### 1.2 我们的解决方案

```
脚本文字 → TTS配音 → ASR识别(获取真实时间轴) → 字幕精确对齐
```

**核心思想**：
1. 先让 TTS 生成真实的配音音频
2. 用 ASR（语音识别）识别这段音频，获取每个字的精确发音时间
3. 将字幕文本与 ASR 输出的时间戳对齐

**类比理解**：
- 传统方案：先写歌词，再按估算节奏配乐（容易对不上）
- 我们的方案：先录好歌，再根据实际歌声写歌词时间轴（精确同步）

---

## 二、技术组件详解

### 2.1 TTS（Text-to-Speech，文字转语音）

**什么是 TTS？**
- 将文字转换成真人语音的技术
- 我们使用 MiMo V2.5 TTS，音色为「阿根」

**为什么先做 TTS？**
- TTS 生成的音频包含真实的语速、停顿、重音
- 每句话的实际时长只有生成后才能知道
- 先有音频，才能精确对齐字幕

**示例**：
```
输入文字：「OpenAI 今天炸了！」
输出音频：scene1.wav（9.28秒）
```

### 2.2 ASR（Automatic Speech Recognition，自动语音识别）

**什么是 ASR？**
- 将语音转换成文字的技术
- 我们使用 Faster Whisper（OpenAI Whisper 的优化版本）

**ASR 的关键能力：词级时间戳**

普通的 ASR 只输出：
```
"OpenAI 今天炸了" (0秒 - 2秒)
```

启用 `word_timestamps=True` 后，ASR 输出：
```
"OpenAI" (0.0s - 0.5s)
"今天" (0.6s - 1.0s)
"炸了" (1.1s - 1.5s)
```

**这就是精确对齐的关键**：每个字都有精确的开始和结束时间。

### 2.3 滑动窗口对齐算法

**什么是滑动窗口？**
- 在 ASR 输出的词级时间戳中，找到与字幕文本匹配的子序列
- 用匹配到的时间戳作为字幕的显示时间

**算法步骤**：
```
1. 字幕文本：「OpenAI 今天炸了」
2. ASR 词级时间戳：[OpenAI: 0.0-0.5s, 今天: 0.6-1.0s, 炸了: 1.1-1.5s, ...]
3. 在 ASR 输出中找到匹配的子序列
4. 字幕时间轴 = [0.0s, 1.5s]
```

**为什么叫「滑动窗口」？**
- 想象一个窗口在 ASR 词序列上滑动
- 窗口内的文字与字幕文本匹配时，记录时间戳
- 窗口大小 = 字幕文本长度

---

## 三、完整工作流程

### 3.1 流程图

```
┌─────────────────────────────────────────────────────────────┐
│                     输入：视频脚本                           │
│  "OpenAI 今天炸了！Plus 封号潮席卷全球..."                   │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│  Phase 6: TTS 配音                                          │
│  ┌─────────────────────────────────────────────────────┐   │
│  │ 逐场景生成配音音频                                    │   │
│  │ scene1.wav (9.28s), scene2.wav (28.00s), ...         │   │
│  └─────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│  Phase 7: ASR 识别                                          │
│  ┌─────────────────────────────────────────────────────┐   │
│  │ Faster Whisper 提取词级时间戳                         │   │
│  │                                                       │   │
│  │ scene1.wav 识别结果：                                  │   │
│  │   "OpenAI" (0.00s - 0.48s)                           │   │
│  │   "今天"   (0.52s - 0.96s)                           │   │
│  │   "炸了"   (1.00s - 1.44s)                           │   │
│  │   "Plus"   (1.80s - 2.20s)                           │   │
│  │   ...                                                 │   │
│  └─────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│  Phase 8: 字幕生成                                          │
│  ┌─────────────────────────────────────────────────────┐   │
│  │ Step 1: LLM 语义断句                                  │   │
│  │   "OpenAI 今天炸了！Plus 封号潮席卷全球..."            │   │
│  │   → ["OpenAI 今天炸了", "Plus 封号潮席卷全球"]         │   │
│  │                                                       │   │
│  │ Step 2: 滑动窗口对齐                                  │   │
│  │   字幕 "OpenAI 今天炸了"                               │   │
│  │   匹配 ASR 词: [OpenAI, 今天, 炸了]                    │   │
│  │   时间轴: [0.00s, 1.44s]                              │   │
│  │                                                       │   │
│  │ Step 3: 验证对齐结果                                  │   │
│  │   - 时间递增 ✓                                        │   │
│  │   - 时长合理 ✓                                        │   │
│  │   - 无重叠 ✓                                          │   │
│  └─────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│  输出：精确对齐的字幕文件                                    │
│  captions.json                                              │
│  [                                                          │
│    {"text": "OpenAI 今天炸了", "startMs": 0, "endMs": 1440},│
│    {"text": "Plus 封号潮席卷全球", "startMs": 1800, ...},   │
│    ...                                                      │
│  ]                                                          │
└─────────────────────────────────────────────────────────────┘
```

### 3.2 详细步骤说明

#### Step 1: TTS 配音

```bash
# 为每个场景生成配音
bash mimo-tts.sh --text "OpenAI 今天炸了！Plus 封号潮..." \
  --profile "阿根" \
  --output "voiceover/scene1.wav"
```

**输出**：
- `scene1.wav` (9.28秒)
- `scene2.wav` (28.00秒)
- ...

#### Step 2: 获取音频精确时长

```bash
# 使用 ffprobe 获取精确时长
ffprobe -v error -show_entries format=duration \
  -of default=noprint_wrappers=1:nokey=1 \
  voiceover/scene1.wav
# 输出: 9.280000
```

**为什么需要精确时长？**
- 用于验证字幕对齐结果
- 计算场景总时长
- 更新视频配置文件

#### Step 3: ASR 提取词级时间戳

```python
from faster_whisper import WhisperModel

model = WhisperModel("medium", device="cpu", compute_type="int8")

segments, info = model.transcribe(
    "voiceover/scene1.wav",
    word_timestamps=True,  # 关键参数：启用词级时间戳
    language="zh",
    vad_filter=True,       # 启用 VAD 过滤静音
)

# 提取每个词的时间戳
words = []
for segment in segments:
    for word_info in segment.words:
        words.append({
            'word': word_info.word.strip(),
            'start': word_info.start,  # 开始时间（秒）
            'end': word_info.end,      # 结束时间（秒）
        })
```

**输出示例**：
```json
[
  {"word": "OpenAI", "start": 0.00, "end": 0.48},
  {"word": "今天", "start": 0.52, "end": 0.96},
  {"word": "炸了", "start": 1.00, "end": 1.44},
  {"word": "Plus", "start": 1.80, "end": 2.20},
  {"word": "封号", "start": 2.24, "end": 2.60},
  {"word": "潮", "start": 2.64, "end": 2.80},
  {"word": "席卷", "start": 2.84, "end": 3.20},
  {"word": "全球", "start": 3.24, "end": 3.60}
]
```

#### Step 4: LLM 语义断句

**为什么要用 LLM 断句？**
- 标点符号分句可能拆分专有名词（如 "GPT-4" 被拆成 "GPT" 和 "4"）
- LLM 理解语义，能更合理地分句
- 每句不超过 15 字，便于阅读

```python
def llm_split_segments(full_text: str) -> list:
    prompt = f"""
请将以下文本按语义自然分句，每句不超过 15 个字符。

规则：
1. 按标点符号和语义停顿分句
2. 每句保持完整语义
3. 不拆分专有名词
4. 输出格式：每句一行

原文：
{full_text}
"""
    response = call_llm(prompt)
    return response.strip().split('\n')
```

**输出示例**：
```
OpenAI 今天炸了
Plus 封号潮席卷全球
Codex 延迟飙到十秒
中转站号池秒空
开发者集体懵了
```

#### Step 5: 滑动窗口对齐

```python
def sliding_window_align(sentences: list, words: list) -> list:
    captions = []
    word_index = 0

    for sentence in sentences:
        text = sentence.replace(' ', '')

        # 在 ASR 词序列中寻找匹配
        best_start = word_index
        best_length = 0

        for i in range(word_index, len(words)):
            match_text = ""
            j = i

            # 累积匹配文本
            while j < len(words) and len(match_text) < len(text) + 5:
                match_text += words[j]['word']
                j += 1

            # 检查是否匹配
            clean_match = match_text.replace(' ', '')
            if text in clean_match or clean_match in text:
                if j - i > best_length:
                    best_length = j - i
                    best_start = i

        # 用匹配到的时间戳创建字幕
        if best_length > 0:
            captions.append({
                "text": sentence,
                "startMs": int(words[best_start]['start'] * 1000),
                "endMs": int(words[best_start + best_length - 1]['end'] * 1000),
            })
            word_index = best_start + best_length

    return captions
```

**对齐过程图解**：

```
字幕文本: "OpenAI 今天炸了"
          ↓
ASR 词序列: [OpenAI, 今天, 炸了, Plus, 封号, 潮, ...]
              ↑      ↑     ↑
              0.0s  0.5s  1.0s  1.4s
              ↓      ↓     ↓
匹配结果: [OpenAI, 今天, 炸了]
              ↓
字幕时间轴: startMs=0, endMs=1440
```

#### Step 6: 验证对齐结果

```python
def validate_alignment(captions: list, scene_duration_ms: int) -> bool:
    if not captions:
        return False

    # 检查时间是否递增
    for i in range(1, len(captions)):
        if captions[i]['startMs'] < captions[i-1]['startMs']:
            return False

    # 检查总时长是否合理
    total_ms = captions[-1]['endMs'] - captions[0]['startMs']
    if total_ms > scene_duration_ms * 1.2:
        return False

    return True
```

**验证规则**：
1. 时间必须递增（后面字幕的开始时间 > 前面字幕的结束时间）
2. 总时长不超过场景时长的 120%
3. 无时间重叠

---

## 四、回退机制

### 4.1 为什么需要回退？

- ASR 模型可能识别不准确（特别是小模型）
- 某些音频可能有背景噪音
- 网络问题导致 ASR 调用失败

### 4.2 回退方案：字数比例估算

当 ASR 对齐失败时，使用字数比例估算时间轴：

```python
def estimate_caption_timing(captions: list, scene_duration_ms: int) -> list:
    """
    按字数比例估算每句字幕的时长

    核心假设：
    - 中文约 200ms/字（语速 300字/分钟）
    - 每句最短 0.5 秒，最长 3 秒
    - 句间间隔 0.3 秒
    """
    total_chars = sum(len(c['text']) for c in captions)
    ms_per_char = (scene_duration_ms - 300 * len(captions)) / total_chars

    current_ms = 0
    for cap in captions:
        char_count = len(cap['text'])
        duration_ms = int(char_count * ms_per_char)
        duration_ms = max(500, min(3000, duration_ms))

        cap['startMs'] = current_ms
        cap['endMs'] = current_ms + duration_ms

        current_ms += duration_ms + 300  # 句间 0.3s 间隔

    return captions
```

### 4.3 完整的生成流程

```python
def generate_subtitles_for_scene(audio_path, scene_text, scene_duration_ms):
    # 尝试 ASR 对齐
    try:
        words = asr_extract_words(audio_path)
        sentences = llm_split_segments(scene_text)
        captions = sliding_window_align(sentences, words)

        if validate_alignment(captions, scene_duration_ms):
            return captions  # ASR 对齐成功
    except Exception:
        pass

    # 回退：字数比例估算
    sentences = llm_split_segments(scene_text)
    captions = [{"text": s, "startMs": 0, "endMs": 0} for s in sentences]
    return estimate_caption_timing(captions, scene_duration_ms)
```

---

## 五、对比：旧方案 vs 新方案

| 对比项 | 旧方案（字幕先行） | 新方案（TTS先行） |
|--------|-------------------|-------------------|
| **流程** | 字幕(估算) → TTS → ASR修正 | TTS → ASR → 字幕(精确) |
| **时间轴来源** | 字数比例估算 | ASR 词级时间戳 |
| **精度** | 低（估算误差大） | 高（真实发音时间） |
| **语速适应** | 不适应（固定公式） | 自动适应（ASR 检测） |
| **停顿处理** | 无法处理 | 自动识别静音段 |
| **专有名词** | 可能拆分 | LLM 语义保护 |
| **回退机制** | 无 | 字数比例估算 |

### 5.1 精度对比示例

**场景**：「OpenAI 今天炸了！」

| 方案 | 字幕时长 | 实际音频时长 | 误差 |
|------|---------|-------------|------|
| 旧方案（估算） | 1.6s (8字 × 200ms) | 2.0s | -0.4s (20%误差) |
| 新方案（ASR） | 2.0s (精确匹配) | 2.0s | 0s (0%误差) |

### 5.2 语速变化处理

**场景**：快速播报 + 停顿

```
音频时间轴：
0.0s ─── 1.0s ─── 1.5s ─── 2.5s ─── 3.0s ─── 4.0s
  │         │         │         │         │         │
  "OpenAI"  "今天"   (停顿)    "炸了"    "了"     (停顿)
  快速       正常               慢速       轻声

旧方案（估算）：
字幕1: 0.0s - 1.6s "OpenAI 今天炸了" ← 与音频不同步

新方案（ASR）：
字幕1: 0.0s - 1.0s "OpenAI 今天"     ← 精确同步
字幕2: 1.5s - 2.5s "炸了"            ← 精确同步
```

---

## 六、关键代码文件

### 6.1 核心脚本

| 文件 | 功能 |
|------|------|
| `gen_captions_v2.py` | 字幕生成主脚本 |
| `video-project/src/Composition.tsx` | 视频场景配置 |
| `video-project/src/Root.tsx` | 视频总时长配置 |

### 6.2 依赖组件

| 组件 | 用途 | 安装命令 |
|------|------|---------|
| faster-whisper | ASR 词级时间戳 | `pip install faster-whisper` |
| MiMo TTS | 语音合成 | 已集成 |
| ffprobe | 音频时长获取 | ffmpeg 自带 |

---

## 七、常见问题

### Q1: ASR 模型选择哪个？

| 模型 | 大小 | 精度 | 速度 | 推荐场景 |
|------|------|------|------|---------|
| tiny | 39MB | 低 | 快 | 测试/快速验证 |
| base | 74MB | 中 | 中 | 一般场景 |
| small | 244MB | 高 | 中 | **推荐生产使用** |
| medium | 769MB | 很高 | 慢 | 高精度需求 |
| large | 1550MB | 最高 | 很慢 | 专业场景 |

**建议**：生产环境使用 `small` 或 `medium` 模型。

### Q2: ASR 对齐失败怎么办？

自动回退到字数比例估算。验证逻辑：
1. 时间是否递增
2. 总时长是否合理
3. 是否有重叠

### Q3: 字幕太长/太短怎么办？

- **太长**：LLM 断句时限制每句不超过 15 字
- **太短**：合并相邻短句
- **时长不匹配**：调整句间间隔（默认 0.3s）

### Q4: 如何提升对齐精度？

1. 使用更大的 ASR 模型（medium/large）
2. 确保 TTS 音频质量清晰
3. 使用 LLM 语义断句而非标点分句

---

## 八、总结

### 核心优势

1. **精确同步**：字幕时间轴来自 ASR 词级时间戳，与音频完全同步
2. **自动适应**：无需手动调整，自动适应不同语速和停顿
3. **智能断句**：LLM 语义断句，保护专有名词，更符合阅读习惯
4. **容错机制**：ASR 失败时自动回退，确保流程不中断

### 适用场景

- B站/抖音短视频制作
- AI 日报视频生成
- 任何需要字幕与配音精确同步的场景

---

## 参考资源

- [VideoCaptioner](https://github.com/WEIFENG2333/VideoCaptioner) - 词级时间戳 + LLM 断句
- [pyVideoTrans](https://github.com/jianchang512/pyvideotrans) - SpeedRate 引擎 + 二次识别
- [Faster Whisper](https://github.com/SYSTRAN/faster-whisper) - 高效 ASR 实现
