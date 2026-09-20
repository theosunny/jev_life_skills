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

## 使用

将 Skill 目录复制(或软链)到 `~/.claude/skills/`:

```bash
git clone https://github.com/theosunny/jev_life_skills.git
cp -r jev_life_skills/travel/flight-deals ~/.claude/skills/
```

`flight-deals` 需要的前置:`browser-use`(携程抓取)与 `TYPESAFE_API_KEY`(Jev 判断),详见 [SKILL.md](travel/flight-deals/SKILL.md)。
