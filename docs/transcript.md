# 逐字稿：我指哪，它改哪

长视频改稿最费时间的是「说清楚改哪里」。逐字稿要解决的就是这个：每句话、每个画面都有一个编号，说「EP1 #12」或者给句子 id，就能定位到脚本里那一句和负责画面的组件文件。

## 本地版：`pnpm transcript`

先渲染成片（`out/<ep>.mp4`），再运行：

```sh
cd video
pnpm transcript          # 全部
pnpm transcript ep1      # 只刷新某一集
pnpm transcript --no-stills
```

输出在仓库根目录的 `transcript/`：

- `README.md`：怎么提修改、各集时长、合集起点、句数；
- `<ep>.md`：按场景分组，每个场景写编号（①②③）、时间段、负责画面的组件文件、场景 id；每句话写序号、时间码、句子 id；念法和字幕不同的句子多一行「配音读作」；
- `<ep>/NN-<scene>.jpg`：每个场景一张拼图，每句一格，格子里标着同样的序号。

截图的取法：每句取念到 80% 的那一帧，这时随口播出现的高亮和卡片都已经到位。一集的截图用 ffmpeg `select` 一次解码截完，再用 ImageMagick `magick montage` 拼图。

下面的台词、编号和时间码仅为格式示例，不对应实际视频：

```markdown
## 设置面板

画面组件：SettingsDemo · 场景：settings-demo

- **#1** `00:00` 打开设置面板，查看当前选项。 · `settings-demo-01`
- **#2** `00:05` 等待 5 秒，再查看预览。 · `settings-demo-02`
  - 配音读作：等待五秒，再查看预览。
```

## 飞书版：`pnpm transcript:lark`

本地 Markdown 不方便在手机上看，也不能评论。飞书版用 [lark-cli](https://github.com/larksuite/cli)（npm 包 `@larksuite/cli`，第一次用先 `lark-cli config init`、`lark-cli auth login`）以你自己的身份读写文档：

```sh
pnpm transcript:lark push          # 按本地数据重建飞书文档，每个场景插一张截图
pnpm transcript:lark pull          # 列出文档里改过的字、所有未解决的评论
pnpm transcript:lark pull --apply  # 把改过的字写回 video/src/script/ep*.ts
```

- 第一次 push 会新建文档，文档 id 记在 `transcript/lark.json`。
- pull 按每行末尾的句子 id 对应到脚本，改过的字逐句列出；评论按引用的文字对应到句子。
- `--apply` 只改没有 `tts` 念法的句子。有念法的句子只报告，因为念法要人一起改。

### 两条规矩

1. **push 会重建整篇文档，评论会丢。每次 push 之前先 pull**，把评论和改字取下来。
2. **pull 报出来的「改字」不一定是人改的。** 本地改了台词、飞书还是旧版时，比对出来也是差异。先看清楚再决定要不要 `--apply`。

push 一次要调几百次接口，网络不稳会在 TLS 握手时超时，脚本按错误类型自动重试。

## 修改记录表

零散意见在逐字稿上评论最方便；成批的问题（断句、读音、口径）用一张表更好管。可以使用这些列：

| 集 | 时间 | 句子编号 | 原文 | 问题类型 | 希望改成 | 备注 | 处理结果 |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 示例段落 | 00:05 | settings-demo-02 | 等待 5 秒，再查看预览。 | 停顿 | 稍延长句间停顿 | 示意 | 更新配音，待复查 |

- 时间使用当前成片时间码，句子 id 不确定时先描述场景，再由 agent 确认。
- agent 改完回填「处理结果」，时间按最新合集重算。
- **写表之前先读一遍**，只替换表格那一节，顶部说明不动，免得冲掉别人新加的行。

## 怎么提修改

直接从逐字稿复制一行，后面写要求：

```text
settings-demo-02：请调整句间停顿，先给音频小样，再更新成片。
```

或者：

- 指定句子 id，写明新的表达。
- 指定场景 id，说明哪个元素遮住了字幕。
- 使用当前时间码定位过场，说明想调整的地方。

让 agent 攒一批再统一重渲，每批改完给你一张表：合集几分几秒、看什么。

## 流程

```text
改脚本 → pnpm tts（只重配改动的句子）→ 出静帧小样 → 你确认
→ 攒够一批 → 重渲受影响的分集 → finalize → delivery-check
→ pnpm transcript → pnpm transcript:lark pull → push → 回填修改记录表
```
