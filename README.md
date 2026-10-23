# 雪之下雪乃 GPT-SoVITS v2 日语声线模型

基于 [GPT-SoVITS](https://github.com/RVC-Boss/GPT-SoVITS) **v2** 微调的单说话人日语语音模型。

- 音色：**雪之下雪乃**（《我的青春恋爱物语果然有问题》，CV：早见沙织）
- 训练素材：游戏内日语语音（公开数据集 [233lol/Ai-YukinoshitaYukino](https://github.com/233lol/Ai-YukinoshitaYukino)），422 条 / 约 27 分钟，已做切分、去静音、日语转写
- 用途：同人二创 / 学习交流，**禁止商用**（详见 [LICENSE.txt](LICENSE.txt)）

> ⚠️ 模型文件较大（约 230MB），不在仓库里，请到 **[Releases](../../releases)** 下载 `yukino_v2_GPT-SoVITS.zip`。

## 试听

以下音频均由本模型 + `ref/yukino_v2_ref.wav` 参考音频生成（top_k=5、temperature=1.0、语速 1.0）：

| 文件 | 台词 | 时长 |
| --- | --- | --- |
| [demo_01_tsundere.wav](demo/demo_01_tsundere.wav) | ちょっと、何見てるのよ。別にアンタのことなんて興味ないんだからね。 | 6.4s |
| [demo_02_short.wav](demo/demo_02_short.wav) | え、なにそれ？ | 1.9s |
| [demo_03_pout.wav](demo/demo_03_pout.wav) | べ、別にアンタのためにやったんじゃないんだからね！ | 4.6s |
| [demo_04_cold.wav](demo/demo_04_cold.wav) | ふん、アンタに構ってる暇なんてないのよ。 | 3.7s |

## 文件说明

| 文件 | 说明 |
| --- | --- |
| `GPT_weights_v2/yukino_v2-e15.ckpt` | GPT 模型（15 epoch 最终档） |
| `SoVITS_weights_v2/yukino_v2_e10_s530.pth` | SoVITS 模型（10 epoch 最终档） |
| `ref/yukino_v2_ref.wav` | 推荐参考音频（约 5.2 秒） |
| `ref/ref_text.txt` | 参考音频对应文本 |

参考音频文本（`ja`）：

```
そ、そうね。今のうちに少しでも慣れておかないと。
```

## 使用方法一：GPT-SoVITS WebUI（推荐）

1. 使用 **v2 版本** 的 GPT-SoVITS（本项目在 `v2-240821` 上训练/验证）
2. 把 `GPT_weights_v2/*.ckpt` 放进 GPT-SoVITS 的 `GPT_weights_v2/` 目录，`SoVITS_weights_v2/*.pth` 放进 `SoVITS_weights_v2/` 目录
3. 启动 WebUI → 「1C-推理」→ 点「刷新模型路径」→ 选择本模型的 GPT / SoVITS 两项
4. 参考音频选择 `ref/yukino_v2_ref.wav`，参考音频文本填上面对应文本，**参考音频语种选 日文**
5. 推理文本语种选 **日文**，参数建议：

| 参数 | 推荐值 |
| --- | --- |
| top_k | 5 |
| top_p | 1 |
| temperature | 1.0 |
| 语速 speed | 1.0 |
| 切分方式 | 不分句（cut0） |

6. 点「开始推断」即可。合成结果每次随机，不满意可多点几次。

## 使用方法二：api_v2（本地 API 服务）

编辑 `GPT_SoVITS/configs/tts_infer.yaml` 的 `custom` 段：

```yaml
custom:
  bert_base_path: GPT_SoVITS/pretrained_models/chinese-roberta-wwm-ext-large
  cnhuhbert_base_path: GPT_SoVITS/pretrained_models/chinese-hubert-base
  device: cuda
  is_half: true
  t2s_weights_path: GPT_weights_v2/yukino_v2-e15.ckpt
  version: v2
  vits_weights_path: SoVITS_weights_v2/yukino_v2_e10_s530.pth
version: v2
```

启动：

```bash
python api_v2.py -a 127.0.0.1 -p 9880 -c GPT_SoVITS/configs/tts_infer.yaml
```

调用示例（GET `/tts`）：

```
http://127.0.0.1:9880/tts?text=ちょっと、何見てるのよ。&text_lang=ja&ref_audio_path=ref/yukino_v2_ref.wav&prompt_text=そ、そうね。今のうちに少しでも慣れておかないと。&prompt_lang=ja&text_split_method=cut0&speed_factor=1.0&top_k=5&top_p=1.0&temperature=1.0
```

## 训练信息

| 项目 | 内容 |
| --- | --- |
| 框架 | GPT-SoVITS v2（`v2-240821`） |
| 底模 | `gsv-v2final-pretrained`（s1bert25hz-5kh / s2G2333k.pth） |
| 数据 | 422 条 / 约 27 分钟 / 单说话人 / 日语；32kHz，1A 格式化自动重采样 |
| 有效训练样本 | 412 条（过滤 phoneme/sec 异常 10 条） |
| SoVITS | batch 8，10 epoch，text_low_lr_rate 0.4，save_every 2 → 采用 e10 |
| GPT | batch 8，15 epoch，save_every 3 → 采用 e15 |
| 硬件 | RTX 4080 SUPER 32G（AutoDL） |

## 已知限制

- **仅适合日语**：实测合成中文会有明显日式口音、咬字不准，不建议用于中文
- 长句偶有断句/语气不自然，建议每次多生成几条挑选
- 仅适用于 **GPT-SoVITS v2**；v1/v3/v4 权重不通用
- 参考音频质量会明显影响输出音色，建议使用自备的合法参考音频（3~10 秒、干净、单人）

## 免责声明（摘要）

- 本项目为**非官方同人二创**，仅供学习、研究与个人交流，**禁止任何商业用途**
- 角色与语音版权归版权方所有（《我的青春恋爱物语果然有问题》相关权利方，CV：早见沙织）
- 禁止用于冒充他人、诈骗、传播虚假信息或任何违法违规用途
- 原始训练音频**未包含**在本仓库中
- 如有侵权，请联系删除

完整条款见 [LICENSE.txt](LICENSE.txt)。

## 致谢

- [GPT-SoVITS](https://github.com/RVC-Boss/GPT-SoVITS)，作者 RVC-Boss（MIT License）
- 训练语音数据集：[233lol/Ai-YukinoshitaYukino](https://github.com/233lol/Ai-YukinoshitaYukino)
