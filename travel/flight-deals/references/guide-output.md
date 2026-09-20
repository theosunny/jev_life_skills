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

## 小红书/微信社交卡片组(多图分享形态)

单张 HTML 长图不适合小红书/微信朋友圈 —— 社交场景用**卡片组**(多张 3:4 竖版单图),发布时按序上传即图文。规范(参考 xhs-cover-skill 风格分类与 write-then-publish 尺寸标准,但纯 CSS 自建、零 API 依赖):

**尺寸与结构**:每卡 1080×1440(3:4),一组 6 张:①封面钩子卡 ②机票卡(往返)③Day1 ④Day2 ⑤美食 ⑥预算+贴士尾卡。

**三件套交付(一个 HTML 出两种图)**:
1. **一个 HTML**:卡片紧凑竖排(`.card{margin:0}`,body 背景与卡片同色),双击打开即一图流滚动浏览 —— 它是两种图片的共同源
2. **一图流 PNG**:**必须分段截图(每段 ≤2000px)+ PIL 竖向拼接**,不要用 captureBeyondViewport 一次截超高图 —— Chrome 对 >6000px 的大图合成有渲染 bug(元素错位/遮挡)。Retina 下截图自动 2x,拼接后统一 2160 宽。适合微信单图发送
3. **图文流 PNG×6**:逐卡 `clip` + `scale:2` 高清。适合小红书/朋友圈多图发布(带 n/6 页码引导划完)

**贴纸风设计要求(与正式 HTML 攻略页区分,这版要"有趣")**:
- 奶油底色 + 大 emoji 做视觉主角(图文并茂不依赖照片,纯 CSS 稳定出图)
- 粗描边圆角贴纸卡(`border:4px solid ink` + `border-radius` + 硬阴影 `8px 8px 0`),禁止细线扁平风
- 价格用大号旋转标签(`transform: rotate(±2deg)` + 高对比底色),这是卡片记忆点
- 行程卡 = 时间线步骤:emoji 图标块 + 时刻徽章 + 一句话理由;美食卡 = 2 列贴纸网格
- 封面:超大标题(120-150px)+ 钩子副标 + 大价格贴纸 + 角落 emoji 涂鸦,信息在 3 秒内可读
- 页码角标(n/6)引导划完全组;尾卡放行动号召("收藏这篇说走就走")
- **禁用 absolute 定位的尾注/页码叠在正文上**——遮挡是实测翻过车的:页码角标用 `position:absolute; bottom` 时必须配合 `padding-bottom≥80px` 留位,备注类尾注一律走正常文档流;卡片用 `min-height` 不用固定 `height`,防内容溢出被裁

**返程必补**:去程查询完成后必须扫返程(oneway 反向 + 邻近 2-3 天),往返价才算完整方案;某日返程被限流未取全时,如实标注并推荐已取到的更优日。
