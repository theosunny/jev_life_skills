# 携程抓取实战手册(2026-09 实测有效)

## 城市代码(常用中日航线)

北京 bjs、上海 sha、广州 can、深圳 szx、成都 ctu、杭州 hgh、南京 nkg、西安 sia、厦门 xmn、天津 tsn
东京 tyo(城市级,含羽田HND/成田NRT)、大阪 osa(含关西KIX/伊丹ITM)、福冈 fuk、札幌 spk(新千岁,北海道门户)、名古屋 ngo、冲绳 oka

规则:目的地给"全域/地区"时用门户城市代码(北海道→spk,关西→osa)。

## URL 模板

```
往返: /online/list/round-{dep}-{arr}?depdate={YYYY-MM-DD}_{YYYY-MM-DD}&cabin=y_s&adult=1&child=0&infant=0
单程: /online/list/oneway-{dep}-{arr}?depdate={YYYY-MM-DD}&cabin=y_s&adult=1&child=0&infant=0
首页: /online/channel
```

域名 `flights.ctrip.com`。注意往返路径是 **round-** 不是 roundtrip-,日期参数是 `D1_D2` 下划线拼接。

## 限流(最重要的章节)

症状(按严重程度):
1. 列表价格还在但"自由搭配"日历价格消失,只剩"查看"链接
2. 航班列表整个为空(body.innerText 只有页头页脚 ~800-1900 字符)
3. 任意查询 2 次重试仍空

对策(按顺序):
1. **新开 tab**(`new_tab(url)`)复用已登录 session —— 轻中度限流常能绕过
2. 冷却 3-5 分钟再试;连续失败别硬刚,如实报告数据缺口
3. 放慢节奏:查询间隔 ≥3s;批量脚本 ≤8 次查询;后台跑 + 行缓冲输出
4. 回首页走 UI 搜索(点表单→搜索按钮)有时能重建正常会话

预防:绝不做无 sleep 的连发循环;福冈等冷门线更容易被限。

## 页面抓取

**必须在已建立 session 的 tab 内 goto_url 直跳**(新 tab 首次直跳会返回空列表;从首页 UI 点搜索最稳)。

等加载:`wait_for_load()` 常超时,包 try/except;价格出现前轮询 `document.body.innerText` 中的 `¥\d{4}`,最多 ~40s。

**航班行**(去重后取前 5):
```js
[...document.querySelectorAll('div,li')].filter(e =>
  /\b[A-Z]{2}\d{3,4}\b/.test(e.textContent) &&
  (e.textContent.includes('选为去程') || e.textContent.includes('订票')) &&
  (e.innerText||'').length < 450 && e.querySelectorAll('*').length < 220
)
```
往返列表行含"选为去程",单程含"订票"。innerText 即完整信息:航司+航班号+机型+时刻+机场+时长+价格+余票。中转行含"转1次 转XX Xh Xm"和"+1天"。

**低价日历**:往返页"自由搭配往返组合"块,正则 `(\d{2}-\d{2})\s*\S{0,3}\s*去\s*(\d{2}-\d{2})\s*\S{0,3}\s*返\s*¥(\d+)(低)?`。**日历组合的间隔跟随你查询的间隔**,查间隔 6 就显示间隔 6 的邻近组合。单程页日历:`(\d{2}-\d{2})[^\n]{0,6}\n¥(\d{4})`。

## UI 兜底(URL 直跳失效时)

1. 回首页 `/online/channel`,表单默认保留上次查询
2. 改城市:点击到达城市输入框(值如"南京(NKG)")→ React 受控组件需 native setter 触发:
```js
const setter = Object.getOwnPropertyDescriptor(window.HTMLInputElement.prototype,'value').set;
setter.call(input,'东京'); input.dispatchEvent(new Event('input',{bubbles:true}));
```
3. 联想下拉出现后,从 `span.highlight` 向上找 `div.address` 条目点击
4. 改日期:点日期输入框弹日历,格子有 **`data-testid="date-day-YYYY-MM-DD"`**,直接 querySelector 点
5. 搜索按钮:`button.search-btn`(AX 树里 role=button name=搜索)

## 已知坑

- 列表初渲染价是缓存价,几分钟后刷新会变(实测 ¥3,732→¥6,382);重要价格隔几分钟复验
- CDP 连接偶发 "no close frame" 断连:重试即可;Chrome 可能重弹远程调试授权
- 往返列表显示的是**该组合的往返总价**;单程页是单程含税价 —— 两页口径不同,别混算
- 日期格子:周六周日会带"班/休/节假日名"前缀,别用纯文本匹配,用 data-testid
