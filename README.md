# Jev Life Skills — 生活类 Skill 合集

基于 [TypeSafe Jev](https://typesafe.ai) + 浏览器自动化的生活场景 Agent Skills 合集。每个 Skill 都经过真实任务实战验证。

## 目录结构

```
jev_life_skills/
└── travel/                  # 旅游与机票预订
    └── flight-deals/        # 机票比价与购买方案生成器
```

## Skill 索引

| 分类 | Skill | 说明 |
|---|---|---|
| 旅游 | [flight-deals](travel/flight-deals/) | 机票比价与购买方案:携程双路比价(往返套票 vs 分开买单程)+ Jev 结构化权衡,输出含航班时刻的详细票价、省钱/均衡/舒适三档推荐与购买步骤,并报告 token 消耗 |

## 安装

```bash
git clone https://github.com/theosunny/jev_life_skills.git
cp -r jev_life_skills/travel/flight-deals ~/.claude/skills/
```

安装后**重启 Claude Code 会话**(或运行 `/reload-plugins`),skill 即出现在可用列表中。

## 使用(flight-deals)

### 第一步:配好两个前置条件

1. **TypeSafe API Key**(用于 Jev 判断):到 [console.typesafe.ai](https://console.typesafe.ai) 申请,然后在任意项目目录写入 `.env`:
   ```
   TYPESAFE_API_KEY=你的key
   ```
2. **browser-use**(用于携程抓取,需本地 Chrome):
   - 安装:`pip install browser-harness`(或参考 [browser-use skill](https://github.com/browser-use/browser-harness))
   - 首次使用:Chrome 会弹出「允许远程调试」授权,点允许即可
   - 自检:`browser-use --doctor`

没有这两样 skill 也能跑一半:能抓价比价,但会跳过 Jev 判断并明确告知。

### 第二步:直接用自然语言提问

在 Claude Code 里正常对话即可自动触发,例如:

- 「帮我查 10 月去大阪最划算的机票,往返,间隔 6 到 8 天」
- 「十一之后错峰,北京或天津出发去福冈或札幌,什么时候走最便宜?」
- 「往返分开买单程会不会更便宜?查一下给我方案」

也可以显式调用:`/flight-deals`。

### 第三步:看懂输出

skill 会给出:方案总览表(含价格口径:起价/推断实价/实测实价)→ 省钱/均衡/舒适三档推荐(航班号、起降时刻、机场、票价)→ 每档的购买步骤 → 下单必读提醒 → Jev token 消耗账单。

### 注意

- 携程列表价是「起价」,订单实付通常上浮 20-30%,skill 会自动标注口径,下单前以订单页为准
- 连续快速查询会触发携程限流,skill 内置了节奏控制与重试;若长时间限流,换个时间再查
- skill 只做查询与推荐,不会代替下单;涉及支付、账号操作一律交还给用户
