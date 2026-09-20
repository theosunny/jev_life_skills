# TypeSafe Jev 判断手册

## 端点与认证

```
POST https://api.typesafe.ai/v1/systemone
Authorization: Bearer $TYPESAFE_API_KEY
Content-Type: application/json
model: "jev-latest"   (响应会回显实际版本,如 jev-1.13.0)
```

key 从环境变量或项目 `.env` 读;**打印时只显示字符数,绝不回显 key 本身**。

## 请求结构

```json
{
  "state": { "需求": "...", "背景": "...", "候选组合": {"c1": "...", "c2": "..."} },
  "model": "jev-latest",
  "questions": {
    "best_value": {
      "type": "choice",
      "instructions": "综合总价、飞行体验、抵达时间对首日影响、目的地价值、余票风险,哪个对普通休闲旅客最划算?",
      "criteria": { "c1": "完整描述", "c2": "完整描述" }
    },
    "key_tradeoff": {
      "type": "noul",
      "instructions": "对比 `候选组合.cX`(…)与 `候选组合.cY`(…):为省 ¥N 选择前者是否值得?",
      "criteria": { "true": "值得的描述", "false": "不值得的描述" }
    }
  }
}
```

## 设计要点

- **state 放结构化事实**(JSON 对象,中文字段名即可):需求约束、价格口径说明、节假日背景、每个候选的完整描述(价格+口径、航班号、起降时刻、直飞/中转、红眼、余票、目的地一句话)。
- **choice 的 criteria 必须罗列全部选项**(≤255 个),模型不能选未列出的;描述要自包含。
- **noul 用于该次决策最纠结的单个权衡**:直飞溢价、红眼容忍、分开买 vs 套票。一问一个,是/否描述要具体。
- 用 backtick 路径(`` `候选组合.c1` ``)引用 state 字段。
- 独立问题并行问(一次请求多个 questions),不串行追问。
- **价格口径必须写进 state**("起价"/"推断实价(起价+26%)"/"实测实价"),否则 Jev 会在错误口径上权衡。

## 响应解读

```json
{"answers": {"best_value": {"choice": "c3", "confidence": 0.37, "probabilities": {...}},
             "key_tradeoff": {"noul": 0.61}},
 "usage": {"input_tokens": 2458, "output_tokens": 90}}
```

- `choice` + `probabilities`:选中的概率高且分布集中 → 结论可靠;置信度 <0.5 或概率分散 → 如实说"偏好敏感",输出分档建议而非唯一答案
- `noul` >0.5 偏 yes、<0.5 偏 no;0.4-0.6 区间同样视为"接近五五开"
- 置信度随数据完整度上升:给 Jev 的信息越全(实价、航班时刻、余票),结论越稳

## Token 账单(必须做)

每次响应的 `usage` 累计进报告:

```
| # | 内容 | input | output |
累计: total_in + total_out
```

浏览器抓取走本地 CDP,不耗 token,无需计入。用户没问也要报——这是流程的一部分。
