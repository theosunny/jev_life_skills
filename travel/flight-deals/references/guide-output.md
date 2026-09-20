# 攻略多形态输出手册

## HTML 攻略页(默认形态)

单文件 `.html`,内联全部 CSS,无外部依赖,双击即可打开、可直接发送分享。

**设计要求(反模板化,每份攻略都要像"为这个目的地做的"):**

1. 先选一个贴合目的地的风格方向( editorial 杂志风 / 地方志 / 暗色 luxury / bento / 水墨留白…),别用"白底蓝按钮"默认款
2. 配色从目的地本身取(滕王阁=赭石墨绿米白, 海岛=浅蓝砂白珊瑚, 城市=霓虹夜色),CSS 变量定义,语义使用
3. 字体有配对策略:中文标题可用衬线(宋体系 `"Noto Serif SC", "Songti SC", serif`),正文无衬线;层级靠字号对比(2-3 级跨度)不靠加粗堆砌
4. 机票卡片是页面锚点:航班号/时刻/价格用大字号数据化呈现,标注价格口径
5. 行程用时间线或编号日卡片;预算用表格;贴士用轻量引用样式
6. 移动友好(max-width ~760px 居中),图片省略(纯排版),加载即完整
7. 动画只允许 CSS 且克制(fade-in 一次即可),遵守 compositor-friendly 属性

结构模板:
```
<header> 目的地大标题 + 行程一句话 + 机票卡(航班/日期/价格)
<section> 每日行程(Day 1..N 时间线)
<section> 美食清单(名字 + 一句话 + 人均)
<section> 预算表(机票[口径]/住宿/餐饮/门票/合计)
<section> 交通与贴士
<footer> 数据来源与生成时间
```

## HTML → PNG 图片

用 browser-use 打开本地文件再整页截图:

```python
new_tab("file:///绝对路径/guide.html")
import time; time.sleep(2)
png_b64 = cdp("Page.captureScreenshot", format="png", captureBeyondViewport=True)["data"]
open("guide.png", "wb").write(__import__("base64").b64decode(png_b64))
```

`captureBeyondViewport=True` 截整页(长图);要分享卡片式短图就截首屏(不传该参数)。输出 `guide-{目的地}-{日期}.png`。

## 飞书文档(图文版)

1. 正文按 lark-doc XML 写:标题层级 + 行程表格 + 预算表格,风格遵循 lark-doc-style(段落叙述为主、数据用表格、callout ≤1)
2. PNG 用 `lark-cli docs +media-insert --file guide.png` 追加到文末(4 步编排,自动回滚)
3. 文档命名:`{目的地}旅行攻略 + 机票方案({日期})`

## 分享建议

- 微信单发:PNG 长图最稳(HTML 在微信内打不开外链样式)
- 群里协作:飞书文档(可编辑可评论)
- 自存离线:HTML(一个文件带走全部)
