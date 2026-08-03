# Concierge

> Concierge 是赌场中的客户经理：迎接玩家入场、回答问题、指引方向、协助充值、处理诉求。
> 本项目是面向游戏客户端的**玩家服务 SDK**——登录验证授权、游戏小助手、信息公告、充值、客服。

与 Croupier（运营后台的荷官）相对：Croupier 服务运营人员，Concierge 服务玩家。

## 定位

Concierge 不是又一套独立后端。它是**玩家端 SDK + 轻量 API 网关**，后端能力尽量编排复用现有生态：

```text
Game Client (Unity / UE / Cocos)
  -> Concierge SDK (C# / C++ / TypeScript)
  -> Concierge Gateway (玩家 API 网关：账号、会话、聚合)
  -> 生态后端：
     - 账号/登录     concierge 自建（accounts，本项目核心新建部分）
     - 公告/活动投递  herald（事件驱动通知投递）
     - 客服工单/FAQ   croupier support / ticket / faq
     - 聊天/实时推送  chirp（gateway + session）
     - 数据与风控     oddsmaker（充值风控、行为分析）
```

scope 模型沿用生态统一约定：`game_id + env` 全局隔离，URL 与 payload 不得覆盖。

## 五项能力

| 能力 | 说明 | 后端来源 | 优先级 |
| --- | --- | --- | --- |
| 登录验证授权 | 账号注册/登录、token 签发与校验、设备绑定；后续接渠道登录与实名 | concierge 自建 | M1 |
| 信息公告 | 拉取/订阅游戏公告、活动、维护通知 | herald + croupier message | M2 |
| 客服 | 工单提交/查询、FAQ 检索 | croupier support/ticket/faq | M2 |
| 游戏小助手 | FAQ 机器人、游戏内向导（复用客服知识库起步） | concierge 编排 + croupier faq | M3 |
| 充值 | 下单、渠道支付、发货回调、对账、退款 | concierge 自建（渠道抽象）+ oddsmaker 风控 | M4 |

## 仓库结构

```text
gateway/    Concierge API 网关（Go）：accounts、sessions、announcements、support、assistant、payments
sdks/
  unity/    Unity SDK（C#，UPM 包）
  ue/       Unreal Engine SDK（C++ 插件）
  cocos/    Cocos Creator SDK（TypeScript）
docs/       架构、路线图、API 契约
```

## 设计边界

1. **SDK 只面向玩家**：不包含任何运营/管理能力，运营操作一律走 Croupier。
2. **网关是唯一入口**：客户端不直连 herald/croupier/chirp/oddsmaker；网关做鉴权、限流、聚合和风控前置。
3. **账号体系自建但不重复**：登录会话可对接 chirp gateway 的会话绑定，不另造实时通道。
4. **支付最后做**：M4 之前任何"伪充值"不进入代码库；渠道抽象先行，第一个真实渠道接入前必须有对账与发货回调骨架。
5. **无 any/unknown 敷衍**：各端 SDK 公共类型统一定义并跨端对齐（与 Croupier 边界 14 同一约定）。

## 路线图

见 [docs/roadmap.md](docs/roadmap.md)。架构见 [docs/architecture.md](docs/architecture.md)。

## License

Apache-2.0
