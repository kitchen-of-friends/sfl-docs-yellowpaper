# DLC 1 - 可售卖票据 Ticket

第一个 DLC 是“可售卖票据”，它赋予了 Goods 以出售、核销票据的能力。从数据结构上看，Goods 会额外存在以下字段以实现 Ticket 模块：

```mermaid
classDiagram
    Goods: ... 继承之前字段
    Goods: +is_ticket 是否激活 ticket 模块
```

对于 Own 数据表，会出现以下字段：

```mermaid
classDiagram
    Own: ... 继承之前字段
    Own: +ticket_step 票据目前的开展步骤（在下文会有各状态详细介绍）
    Own: +burner_addr 核销人的地址
    Own: +burnt_num 核销了多少张票（与之前的 goods_num 搭配使用）
    Own: +frozen_num 冻结了多少张票（如因退款，与之前的 goods_num 搭配使用）
```

此时，一个 Own 可以理解为“一张票据”、“一张门票”。一个 Own 会根据状态有以下的票组步骤：

<table><thead><tr><th width="226">票组步骤值（ticket_step）</th><th>缩略意义</th><th>描述</th></tr></thead><tbody><tr><td>90</td><td>PendingPay</td><td>当前票正在等待用户付费</td></tr><tr><td>100</td><td>WaitingList</td><td>等待列表</td></tr><tr><td>150</td><td>PendingApproval</td><td>已支付，等待管理员通过审核</td></tr><tr><td>200</td><td>Approved</td><td>（常用）已通过，待使用状态</td></tr><tr><td>230</td><td>Burnt</td><td>验票完成，已销毁</td></tr><tr><td>300</td><td>Rejected</td><td>被管理员拒绝</td></tr><tr><td>400</td><td>Withdrawn</td><td>未通过审核前，被发起人主动撤回</td></tr><tr><td>410</td><td>Pigeon</td><td>已通过的票组，但是持票人主动取消</td></tr><tr><td>500</td><td>Timeout</td><td>超过审核时限/活动结束，仍未有一个审核结果</td></tr><tr><td>510</td><td>EventCanceled</td><td>活动取消</td></tr></tbody></table>

各个状态的转换流程图如下：

```mermaid
flowchart TD
    StartAct[管理员创建好活动] --> |用户报名活动| PriceCheck{{活动是否收费}}
    PriceCheck --> |活动收费| PendingPayStep[PendingPay 90]
    PriceCheck --> |活动不收费| FullCheck{{是否有人数限制}}
    PendingPayStep --> |付费后| FullCheck
    FullCheck --> |没有限制或队伍未满| ApprovalCheck{{是否需要活动主批准}}
    FullCheck --> |队伍已满| WaitingStep[Waiting 100]
    WaitingStep --> |候补成功|ApprovalCheck
    ApprovalCheck --> |不需要批准| ApprovedStep[Approved 200]
    ApprovalCheck --> |需要批准| PendingApStep[PendingApproval 150]
    PendingApStep --> |管理员批准| ApprovedStep
    ApprovedStep --> |核销票据| BurntStep[Burnt 230]
    ApprovedStep --> |参与者得到批准后咕咕| PegionStep[Pegion 230]
    PendingApStep --> |管理员拒绝|RejectStep[Rejected 300]
    WPStep[在 90 等待支付 或 100 候补或 150 等批准时] --> |参与者得到批准前撤回/一直未有回复/十分钟内未付款| WithdrawnStep[Withdrawn 400/Timeout 500]
    CancelAct[任意状态管理员取消或人数不足自动取消聚会]-.->|有钱退钱|CancelStep[EventCanceled 510]
```



您也可以通过在线链接预览该图：

{% embed url="https://www.mermaidchart.com/raw/c649d2ae-eb8e-4e13-8588-80c6ef92c3d8?format=svg&theme=light&version=v0.1" %}

