# DLC 5 - 激励 Task

本 DLC 专门为 V3 设计（Class：V，Cat：3），应用中需要如每日签到、活动参与奖励等激励任务来刺激用户更深入地参与到应用中，因此，激励模块应运而生。

```mermaid
classDiagram
    Goods: ... 继承之前字段
    Goods: +is_task 是否激活 task 模块
    Goods: +task_type 任务类型
    Goods: +task_level 任务级别
    Goods: +task_cron_period 任务循环周期
    Goods: +is_cron 是否是循环任务
    Goods: +is_continue 是否是连续任务
    Goods: +is_repeat 是否是可重复完成的任务
    Goods: +max_repeat 最多重复完成的此时
    Goods: +triggers 任务的完成条件列表
    Goods: +rewards 任务的奖励列表
```

一个任务最多有三级，数值越低越偏低层，代表完成后可能成为更高一级任务的一环。

| 任务级别       | task\_level 数值 |
| ---------- | -------------- |
| LevelOne   | 1              |
| LevelTwo   | 2              |
| LevelThree | 3              |

任务的循环周期分为三种：

| 名称            | 数值 | 意义   |
| ------------- | -- | ---- |
| DailyPeriod   | 1  | 每日循环 |
| WeeklyPeriod  | 2  | 每周循环 |
| MonthlyPeriod | 3  | 每月循环 |
