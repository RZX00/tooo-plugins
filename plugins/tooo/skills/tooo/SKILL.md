---
name: tooo
description: "tooo is the user's personal to-do list and focus timer, reached through the tooo MCP connector. Use it when the user mentions tooo, their tasks, today's plan, reminders, the focus timer or pomodoro, time records, device usage or the weekly review, and the tooo tools are available. tooo 是用户的待办和计时：任务与步骤、重复、专注计时、用时记录、复盘、提醒、屏幕使用时间。"
---

# tooo

## 1 范围

1.1 本技能给出 tooo 连接器的事实：tooo 是什么，提供哪些工具，参数的格式与限制。

1.2 本技能不规定工作流程。何时记录、如何安排、是否追问，由用户的指示决定。

## 2 术语和定义

2.1 空间：一个 tooo 账号的全部数据。连接器只读写批准它的那个账号。

2.2 任务：一条待办，带 id。

2.3 步骤：任务下的子项，可以再有步骤。带步骤的任务即流程或项目。步骤与任务一样有 id、状态和用时。

2.4 类型 type：action 行动、reference 参考、note 笔记、trash 垃圾。尚未分类的任务在输出中显示为 inbox。

2.5 状态 status：pending 待开始、in_progress 进行中、done 已完成。输出中还可能见到 inbox 和 dropped。

2.6 优先级 priority：important_urgent、important_not_urgent、urgent_not_important、not_important_not_urgent。

2.7 今日最重要 important_today：同一时刻只有一条，设置时同时排到今天。

2.8 专注：一次计时。模式为 pomodoro（番茄，默认 25 分钟工作加 5 分钟休息）或 stopwatch（正计时）。

2.9 用时记录：一段有起止时刻的时间，来源为 timer（计时产生）或 manual（补记）。

2.10 视图 view：list_tasks 的筛选方式。

## 3 工具定义

| 用途 | 工具 | 参数（* 必填） | 说明 |
| - | - | - | - |
| 任务 | `list_tasks` | view、date、q、limit、cursor | view 取 today（默认）、week、month、inbox、action、waiting、reference、note、all、undated、overdue。today、week、month 以 date 为锚（默认今天），含已完成的。undated 是自己和步骤都没排日期、且未完成的顶层任务，overdue 是日期早于 date 且未完成的任务（重复任务生成的不算）。q 只保留标题含该文字的。limit 为一页的条数（1 到 200，默认 50）。每条带步骤树、各步用时和上级路径 parent_path。返回 view、date、total（总条数）、tasks 和 next_cursor。 |
| 任务 | `get_task` | id* | 一条任务的全部信息：完整正文、步骤及各步状态和用时、重复规则、时间、提醒。 |
| 任务 | `create_task` | title*、type、date、time、remind、remind_before、steps、repeat、priority、pomodoros、category、note | 只给 title 进收件箱，给 date 排进那天。remind=true 到点提醒（需要 date 和 time），remind_before 是提前的分钟数（0 到 1440，默认 0）。steps 的每一项是字符串或 {title, steps}，可嵌套。repeat 为 {freq: daily 或 weekly, weekdays}。pomodoros 为预计番茄数（1 到 8）。category 如 工作、学习、健康、生活、睡眠，也可自定义。note 为纯文本正文。 |
| 任务 | `update_task` | id*、title、type、status、date、time、remind、remind_before、important_today、priority、pomodoros、category、note、note_append、cue | 只传要改的字段，含义同 create_task。不能改步骤列表和重复规则，前者用 add_step，后者用 set_repeat。步骤也用它改状态，步骤进入 in_progress 会自动开始计时。cue 是触发情境（做这件事的具体时机）。 |
| 任务 | `complete_task` | id* | 任务和步骤都可以。 |
| 任务 | `delete_task` | id* | 删除一条任务，连同它的步骤。 |
| 步骤与重复 | `add_step` | task_id*、title* | 加在步骤列表末尾。task_id 可以是任何一层的任务或步骤。 |
| 步骤与重复 | `set_repeat` | id*、repeat* | repeat 为 {freq: daily 或 weekly, weekdays}，null 表示停止重复。weekdays 用 0 到 6，0 为周日。 |
| 现在做什么 | `what_now` | 无 | 返回最该做的一件事 action、理由 why 和候选 candidates。顺序是正在专注的、今日最重要的、再按优先级和日期打分。 |
| 专注计时 | `start_focus` | task_id*、mode | mode 不给就用设置里的开始方式。给的是带步骤的任务，就接着做进行中的那一步，没有就从第一个未完成的步骤开始（步骤用正计时）。给的是某个步骤，就开始那个步骤。 |
| 专注计时 | `focus_status` | 无 | 当前专注的事项、番茄序号、剩余时间、这件事累计用时。 |
| 专注计时 | `pause_focus` | 无 | 暂停正在跑的计时，事项和已计时间保留。 |
| 专注计时 | `resume_focus` | task_id | 接着暂停的计时。同时有几件暂停着时用 task_id 指定，不给就接最近暂停的。 |
| 专注计时 | `stop_focus` | done | 结束当前计时。done=true 同时把这件事标记完成，默认只停计时。 |
| 专注计时 | `get_focus_settings` | 无 | 返回 work_minutes、break_minutes、mode。 |
| 专注计时 | `set_focus_settings` | work_minutes、break_minutes、mode | 至少给一项，没给的不变。work_minutes 为 1 到 180，break_minutes 为 1 到 60。以后新开的计时用新设置。返回改完后的全部设置。 |
| 用时记录 | `add_time_entry` | date*、start*、end*、task_id、label | 补记一段已经过去的时间。task_id 记到某件事上，没有对应事项时写 label。 |
| 用时记录 | `day_timeline` | date | 某天按钟点的记录：起止、事项、大类、来源。还在计的一条 end 为 null。 |
| 复盘 | `day_summary` | date | 某天做完的、没做完的、今日最重要、每件的分钟数、主动计时总分钟数 focus_minutes_total（番茄休息的 rest_minutes_total 不算在内）。computer 是电脑自动记录的用时，没有数据时为 null。 |
| 复盘 | `get_weekly_review` | 无 | 本周（周一到周日）完成数、计时和休息分钟数、按大类的用时，加上最近一次存下的周反思和本周计划。 |
| 复盘 | `save_weekly_review` | reflection、plan | 至少给一项，只给一项时另一项沿用上一次的。纯文本，可多行。 |
| 提醒 | `remind` | text*、at*、task_id | at 为 HH:MM（今天）或 YYYY-MM-DD HH:MM，发到微信、桌面和网页推送。可关联一条任务。 |
| 设备用量 | `device_usage` | date、form、top | 手机和电脑上各 App 的使用时长，来自各设备自己上传的记录。form 取 phone 或 computer，top 为每台设备列几个 App（1 到 50，默认 10）。iPhone 无法提供数据。没有数据时 devices 为空。总时长是各设备相加，两台设备同时开着的时间算两份。 |

## 4 要求

4.1 日期应写成 YYYY-MM-DD，时刻应写成 24 小时制 HH:MM。两者按这个空间的时区解释，工具里的“今天”也按它算。时区写在连接时服务端返回的 instructions 里，应以那里的为准，不应假定为某个固定时区。

4.2 任务和步骤的 id 应取自 list_tasks、get_task、create_task 等工具的返回值，不应自行编造。

4.3 update_task 只应传需要修改的字段。date、time、priority、pomodoros、category、cue 传 null 表示清空，不传表示不改。note 覆盖正文，note_append 追加在正文之后。

4.4 important_today=true 可将一条任务设为今日最重要。设置后原来的那一条不再是今日最重要。

4.5 delete_task 应仅在用户明确要求删除时调用。tooo 把它标记为破坏性操作，并在工具元数据里要求每次调用都经用户确认，Claude Code 据此每次弹出确认；其他客户端是否确认由其设置决定，因此不应把确认当作已经发生。其余写工具的确认方式由各客户端的设置决定。

注：不再要做的事可用 update_task 改为 type=trash，任务仍保留。

4.6 add_time_entry 只可补记已经过去的时间，起止应在同一天，且不应与已有记录重叠。

4.7 remind 的时刻应晚于当前时间，否则工具返回错误。

4.8 调用 set_repeat 时应知道它修改的是整条重复规则，即使传入的 id 是某一天的那一次。

4.9 list_tasks 返回的 note 最多 400 个字符。需要完整正文时应使用 get_task。

4.10 list_tasks 一次最多返回 limit 条。返回的 next_cursor 不为 null 时，说明还有后续，应把它作为 cursor 传回以取下一页，直到 next_cursor 为 null 才算列完；total 是这组条件下的总条数。翻页时不应改 view、date、q，改了要从第一页重新取。

4.11 连接应由账号所有者在自己的设备上批准。tooo 暂不开放注册，Agent 不应代用户注册账号，也不应把命令行密钥（tk_ 开头）写进任务正文或回复。

## 5 示例

<examples>
<example>
create_task：
{"title": "写周报", "date": "2026-10-23", "time": "16:00", "remind": true, "remind_before": 15, "steps": ["收集数据", "写初稿"]}
</example>
<example>
update_task，清空日期：
{"id": "<任务 id>", "date": null}
</example>
<example>
add_time_entry，补记下午两点到三点：
{"date": "2026-10-19", "start": "14:00", "end": "15:00", "label": "健身"}
</example>
<example>
set_repeat，每周一三五：
{"id": "<任务 id>", "repeat": {"freq": "weekly", "weekdays": [1, 3, 5]}}
</example>
</examples>
