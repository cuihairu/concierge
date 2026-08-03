# Concierge

> Concierge 是赌场中的客户经理：迎接玩家入场、回答问题、指引方向、协助充值、处理诉求。
>
> 本项目是面向游戏公司的**玩家服务 SDK + 轻量 API 网关**。一家游戏公司旗下多款游戏共享一套账号体系，玩家用一个账号畅玩所有游戏，运营团队统一管理公告、客服、充值。

与 Croupier（运营后台的荷官）相对：Croupier 服务运营人员，Concierge 服务玩家。

## 场景

一家游戏公司运营着多款游戏（MMO、卡牌、塔防……），过去每款游戏各自建账号、公告、客服、充值系统，玩家在每款游戏里都要重新注册，运营团队在每个后台之间来回切换。

Concierge 解决这个问题：

```text
玩家视角：
  注册一个账号 → 登录任意一款游戏 → 统一充值 → 统一客服

运营视角：
  一套后台管理所有游戏的公告、客服、充值 → 数据互通 → 统一运营
```

## 定位

Concierge 不是又一套独立后端。它是**玩家端 SDK + 轻量 API 网关**，后端能力尽量编排复用现有生态：

```text
Game Client (Unity / UE / Cocos / 小程序 / Layabox / Godot)
  -> Concierge SDK (C# / C++ / TypeScript / JavaScript / GDScript)
  -> Concierge Gateway (玩家 API 网关：账号、会话、聚合)
  -> 生态后端：
     - 账号/登录     concierge 自建（accounts，本项目核心新建部分）
     - 公告/活动投递  herald（事件驱动通知投递）
     - 客服工单/FAQ   croupier support / ticket / faq
     - 聊天/实时推送  chirp（gateway + session）
     - 数据与风控     oddsmaker（充值风控、行为分析）
```

### 账号模型

```text
一个公司账号（account）
  ├── 游戏A：角色1（player_id_A）
  ├── 游戏B：角色2（player_id_B）
  └── 游戏C：角色3（player_id_C）
```

- **一个账号登录所有游戏**：玩家注册一次，凭同一套凭证进入公司旗下任意游戏。
- **每个游戏角色独立**：同一账号在不同游戏中有独立的角色数据、背包、等级。
- **统一充值**：账号维度的余额/支付记录跨游戏共享，游戏维度的发货独立。
- **统一客服**：工单/FAQ 按 game_id 隔离，但同一账号的历史工单可跨游戏查看。

scope 模型沿用生态统一约定：`game_id + env` 全局隔离，URL 与 payload 不得覆盖。

## 五项能力

| 能力 | 说明 | 后端来源 | 优先级 |
| --- | --- | --- | --- |
| 登录验证授权 | 账号注册/登录、token 签发与校验、设备绑定；一个账号登录多款游戏 | concierge 自建 | M1 |
| 信息公告 | 拉取/订阅游戏公告、活动、维护通知（按游戏隔离） | herald + croupier message | M2 |
| 客服 | 工单提交/查询、FAQ 检索（按游戏隔离，账号维度可跨游戏查看） | croupier support/ticket/faq | M2 |
| 游戏小助手 | FAQ 机器人、游戏内向导（复用客服知识库起步） | concierge 编排 + croupier faq | M3 |
| 充值 | 下单、渠道支付、发货回调、对账、退款（账号维度余额，游戏维度发货） | concierge 自建（渠道抽象）+ oddsmaker 风控 | M4 |

## 仓库结构

```text
gateway/    Concierge API 网关（Go）：accounts、sessions、announcements、support、assistant、payments
sdks/
  unity/    Unity SDK（C#，UPM 包）
  ue/       Unreal Engine SDK（C++ 插件）
  cocos/    Cocos Creator SDK（TypeScript）
  miniprogram/  微信小程序 SDK（JavaScript）
  laybox/    Layabox SDK（TypeScript/JavaScript）
  godot/     Godot SDK（GDScript/C#）
docs/       架构、路线图、API 契约
```

## 设计边界

1. **SDK 只面向玩家**：不包含任何运营/管理能力，运营操作一律走 Croupier。
2. **网关是唯一入口**：客户端不直连 herald/croupier/chirp/oddsmaker；网关做鉴权、限流、聚合和风控前置。
3. **账号体系自建但不重复**：登录会话可对接 chirp gateway 的会话绑定，不另造实时通道。
4. **一个账号多款游戏**：账号是跨游戏的最高层实体；角色（player）是游戏维度的，每个游戏独立创建。
5. **充值账号维度、发货游戏维度**：余额/支付记录属于账号，发货逻辑属于游戏。
6. **支付最后做**：M4 之前任何"伪充值"不进入代码库；渠道抽象先行，第一个真实渠道接入前必须有对账与发货回调骨架。
7. **无 any/unknown 敷衍**：各端 SDK 公共类型统一定义并跨端对齐（与 Croupier 边界 14 同一约定）。

## 路线图

见 [docs/roadmap.md](docs/roadmap.md)。架构见 [docs/architecture.md](docs/architecture.md)。

## License

Apache-2.0
