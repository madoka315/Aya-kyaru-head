# Aya-kyaru-head

HoshinoBot **接头霸王**插件（猫猫头接头）：发一张图，bot 把猫猫头贴到检测到的脸上。

本仓库在[原版](https://github.com/pcrbot/plugins-for-Hoshino/tree/master/shebot/concatHead)基础上做了 NapCat 适配与检测优化。

## 指令

| 指令 | 说明 |
|---|---|
| `接头` + 图片 | opencv 接头（默认模式） |
| `接头1` + 图片 | 百度 api 接头（需在 `config.py` 填入百度密钥） |
| `接头2` + 图片 | opencv 接头 |

每人每天 5 次，15 秒冷却。

## 与原版的差异

1. **兼容 NapCat**
   - 原版从 CQ 码字符串用 `http.*?term=\d` 正则抽图片 URL，该格式只存在于 go-cqhttp 时代；NapCat 的图片 URL（`multimedia.nt.qq.com.cn`，rkey 格式）没有 `term=` 参数，导致发图必报「请附带图片！」。现改为遍历消息段直接取图片段的 `url`（缺省兜底 `file`），CQ 字符串仅作兜底。
   - QQ 图床现在常发 PNG（RGBA/P 带透明），原版直接存 `temp.jpg` 会报 `cannot write mode RGBA as JPEG`。现落盘前先把非 RGB 模式转成 RGB。
   - 图片下载/落盘失败时给群内反馈并记日志，不再静默无响应。

2. **opencv 检测优化（先严后松重试阶梯）**
   - 原版写死 `scaleFactor=1.1, minNeighbors=5`，对高分辨率大图、侧脸偏严，经常漏检清晰正脸而报「接头失败」。
   - 现按 `(1.1,5) → (1.05,4) → (1.02,3) → (1.02,2) → (1.02,1)` 五档逐级放松重试：原参数能检出的图行为完全不变（无回归），原参数漏检的图逐档放宽救回。经多张样图回归验证。

## 安装

1. 整个目录放到 HoshinoBot 的 `hoshino/modules/` 下（目录名 `conhead`），在 `hoshino/config/__bot__.py` 的 `MODULES_ON` 加一行 `'conhead'`。
2. 依赖：`opencv-python`、`pillow`、`aiohttp`（HoshinoBot 基础环境一般已具备）。
3. 不使用 api 模式可跳过百度密钥配置；`DEFAULT_MODE` 控制不带编号的 `接头` 走哪个模式。

原版说明（api 模式加头、百度 api 与 opencv 模式优缺点）见 [readme.txt](./readme.txt)。

## License

[GPL-3.0](./LICENSE)
