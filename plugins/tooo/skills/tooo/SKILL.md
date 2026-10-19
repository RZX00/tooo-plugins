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

2.8 专注：一次计时。模式为 pomodoro（番茄，默认 25 分钟工作加 5 分钟休息）或 stopwatch（正计时）。同一时间可以有几件事在计时，其中一件在前台，其余在后台；番茄的工作和休息时长只是目标，到点只提醒，阶段只在用户（或你按用户的话）调用 start_break、end_break 时改变。

2.9 用时记录：一段有起止时刻的时间，来源为 timer（计时产生）、manual（补记）或 rest（番茄休息）。background=true 的一段是和前台那件同时进行的时间：算进这件事自己的用时，不算进当天、当周的总时长。

2.10 视图 view：list_tasks 的筛选方式。

## 3 工具定义

| 用途 | 工具 | 参数（* 必填） | 说明 |
| - | - | - | - |
| 任务 | `list_tasks` | view、date、q、limit、cursor | view 取 today（默认）、week、month、inbox、action、waiting、reference、note、all、undated、overdue。today、week、month 以 date 为锚（默认今天），含已完成的。undated 是自己和步骤都没排日期、且未完成的顶层任务，overdue 是日期早于 date 且未完成的任务（重复任务生成的不算）。q 只保留标题含该文字的。limit 为一页的条数（1 到 200，默认 50）。每条带步骤树、各步用时和上级路径 parent_path。返回 view、date、total（总条数）、tasks 和 next_cursor。 |
| 任务 | `get_task` | id* | 一条任务的全部信息：完整正文和正文版本 note_rev、步骤及各步状态和用时、重复规则（repeat_rule_id 是规则的 id）、时间、提醒 reminder（形式、响的时刻 fires_at、状态 status）、改到以后的次数 postponed（终身计数）。 |
| 任务 | `create_task` | title*、id、type、date、time、reminder、steps、repeat、priority、pomodoros、category、note | 只给 title 进收件箱，给 date 排进那天。reminder 是提醒，见 4.12。steps 的每一项是字符串或 {title, steps}，可嵌套。repeat 为 {freq: daily 或 weekly, weekdays}。pomodoros 为预计番茄数（1 到 8）。category 如 工作、学习、健康、生活、睡眠，也可自定义。note 为纯文本正文。id 为可选的 UUID，见 4.15。remind、remind_before 是老写法，等于 reminder 的 {before}。 |
| 任务 | `update_task` | id*、title、type、status、date、time、reminder、important_today、priority、pomodoros、category、note、note_rev、note_append、cue | 只传要改的字段，含义同 create_task。不能改步骤列表和重复规则，前者用 add_step，后者用 set_repeat、stop_repeat。步骤也用它改状态，步骤进入 in_progress 会自动开始计时。cue 是触发情境（做这件事的具体时机）。note 要同时带 note_rev，见 4.3。 |
| 任务 | `complete_task` | id* | 任务和步骤都可以。 |
| 任务 | `delete_task` | id* | 删除一条任务，连同它的步骤。 |
| 任务 | `restore_task` | id* | 撤销删除：恢复删掉的任务（连同当时一起删的步骤），或停掉的重复（规则和当时一起删的那几次），按 id 自己分辨。没被删的原样返回。 |
| 任务 | `reorder_tasks` | scope*、ids*、date、parent_id | 手动排顺序。scope=day 时 ids 是 date 那一天从上到下的任务，不在这一天的会被挪到这一天；scope=steps 时 ids 是 parent_id 下面步骤的新顺序。返回排好的顺序 order。 |
| 步骤与重复 | `add_step` | task_id*、title*、id | 加在步骤列表末尾。task_id 可以是任何一层的任务或步骤。 |
| 步骤与重复 | `set_repeat` | id*、repeat* | repeat 为 {freq: daily 或 weekly, weekdays}，设或改重复规则，不收 null。weekdays 用 0 到 6，0 为周日。 |
| 步骤与重复 | `stop_repeat` | id* | 停止重复：删掉规则和以后还没做的那几次。id 是规则或其中一次，对某一次调用时那一次留下变成普通任务。返回 rule、removed。 |
| 步骤与重复 | `list_repeats` | 无 | 所有重复规则：id、标题、freq、weekdays、time，以及本周七天各自那一次的状态（那天不重复为 null）。 |
| 专注计时 | `start_focus` | task_id*、mode、parallel | 默认开始会暂停前台那件；parallel=true 时前台那件转到后台接着走，这件放到前台。对已在后台走的事调用，就是把它拉到前台。mode 不给就用设置里的开始方式。给的是带步骤的任务，就接着做进行中的那一步，没有就从第一个未完成的步骤开始（步骤用正计时）；给的是某个步骤，就开始那个步骤。 |
| 专注计时 | `focus_status` | 无 | front 是前台那件，background 是同时在走的其余几件（最近放到后台的在前），next 是刚完成一步后的「接下来」；每件带标题、累计分钟（含后台）、番茄阶段、超出目标的分钟 overrun_minutes、能否撤销。 |
| 专注计时 | `pause_focus` | task_id | task_id 不给是前台那件。事项和已计时间保留。前台那件停下后，后台最近放下的那件顶到前台。 |
| 专注计时 | `resume_focus` | task_id、parallel | 接着暂停的计时。不给 task_id：前台有事在走时什么都不做，否则接最近暂停的那件。给了 task_id：接着那件，parallel 同 start_focus。 |
| 专注计时 | `stop_focus` | task_id、done | 结束一件事的计时，task_id 不给是前台那件。done=true 同时把这件事标记完成，默认只停计时。 |
| 专注计时 | `cancel_focus` | task_id | 撤销开始：删掉这次开始以后记的时间，把这件事放回开始前的样子，被它暂停的那件接着走。只对正在走的事有效，task_id 不给是前台那件。 |
| 专注计时 | `start_break` | task_id | 番茄从工作进入休息：工作在这一刻结束（超出目标的部分照样记上），休息开始计时。不在工作阶段时什么都不做。 |
| 专注计时 | `end_break` | task_id | 结束休息。只结束休息，不开始下一颗；要开始下一颗时调 resume_focus 或 start_focus。 |
| 专注计时 | `get_focus_settings` | 无 | 返回 work_minutes、break_minutes、mode。 |
| 专注计时 | `set_focus_settings` | work_minutes、break_minutes、mode | 至少给一项，没给的不变。work_minutes 为 1 到 180，break_minutes 为 1 到 60。以后新开的计时用新设置。返回改完后的全部设置。 |
| 用时记录 | `add_time_entry` | date*、start*、end*、task_id、label | 补记一段已经过去的时间，整段都记下：和别的事已有记录重叠的部分记成同时进行（后台），只跳过和这件事自己已有记录重叠的部分。返回实际记下的各段 segments 和一句现成的说明 note。task_id 记到某件事上，没有对应事项时写 label。 |
| 用时记录 | `day_timeline` | date | 某天按钟点的记录：起止、事项、大类、来源、是否同时进行（background）。还在计的一条 end 为 null。total_minutes 只算前台。 |
| 复盘 | `day_summary` | date | 某天做完的、没做完的、今日最重要、每件的分钟数、主动计时总分钟数 focus_minutes_total（番茄休息的 rest_minutes_total 不算在内）。computer 是电脑自动记录的用时，没有数据时为 null。 |
| 复盘 | `get_weekly_review` | week_start | 一周（周一到周日）的复盘数据。week_start 为这一周里任意一天，不给是本周。返回 week（from、to、ended）、completed、focus_minutes、rest_minutes、pomodoros、by_day（七天各一行）、by_category、by_task（按任务的分钟和番茄，最多 20 条）、done_tasks（做完的，最多 100 条，completed 完成那天、planned 原来排的那天）、open_tasks（排在这一周没做完的，最多 50 条）、screen（电脑和手机的使用分钟，没有数据为 null）、inbox_count、last_plan（上一周写的下周三件）、review（这一周存过的最新一版周报，没有为 null）。 |
| 复盘 | `list_weekly_reviews` | limit、before | 往期周报，每周最新一版，新的在前：week_start、big、wins、reflection、plan、saved_at、author（写的设备或 Agent 的名字）、versions（这一周存过几版）。limit 为 1 到 12（默认 4）；before 只取这一天之前开始的周。 |
| 复盘 | `save_weekly_review` | week_start、big、wins、reflection、plan、id | 存一周的周报：big 本周大事、wins 本周成就、reflection 本周反思（纯文本，可多行），plan 为下周三件（字符串数组，最多 3 条）。至少给一项，没给的沿用这一周上一版。返回 id、week_start、version。 |
| 提醒 | `remind` | text*、at*、task_id、id | 一句话的提醒。at 为 HH:MM（今天）或 YYYY-MM-DD HH:MM。到点只出现在用户正在用的那一台设备，5 分钟没响应换到手机，再换到微信（接了微信转发时）。可关联一条任务。 |
| 提醒 | `list_reminders` | status、task_id、from、to、limit | 排着和出过的提醒：id、kind、task_id、fire_at（本地时刻）、title、status（upcoming 还没到、sent 出过了、cancelled 取消了）、outcome、place。status 默认 upcoming。 |
| 提醒 | `cancel_reminder` | id* | 取消一条还没到的提醒。任务自己的到点提醒等同 update_task 的 reminder=null。已经出过的取消不了。 |
| 主动提醒 | `prompts` | 无 | tooo 最近 2 小时发给用户、用户还没回应的提醒（没在计时、计时没停、到点、过点没开始、一天收尾、周报 weekly），每条带 title、body 和按钮 actions；另有 untimed_minutes、gaps_today。 |
| 主动提醒 | `act_prompt` | id*、action*、task_id | 替用户按下 prompts 里某条提醒的一个按钮，和用户在通知上按一样，返回一行结果 result。按钮带 needs: "task_id" 时要给 task_id。到点提醒的 done 把那件事标成完成；周报的 snooze 是 1 小时后再提醒，skip_week 是这周不写，open_review 只返回复盘周 weekStart。 |
| 主动提醒 | `cut_prompt_time` | id*、action* | 「计时没停」提醒上会去掉已记时间的两个按钮：cut（不算）、stop_at（停在离开的那一刻）。act_prompt 不执行这两个。 |
| 主动提醒 | `get_prompt_settings` | 无 | 提醒设置（语气、勿扰时段、间隔、每天上限、退避、每种提醒的开关和参数，含周报 weekly：on、weekday、at、where）和近 7 天每种出现、响应的次数。 |
| 主动提醒 | `set_prompt_settings` | 同 get_prompt_settings 的 settings | 只给要改的项，嵌套的也只给要改的键。 |
| 设备用量 | `device_usage` | date、form、top | 手机和电脑上各 App 的使用时长，来自各设备自己上传的记录。form 取 phone 或 computer，top 为每台设备列几个 App（1 到 50，默认 10）。iPhone 无法提供数据。没有数据时 devices 为空。总时长是各设备相加，两台设备同时开着的时间算两份。 |

## 4 要求

4.1 日期应写成 YYYY-MM-DD，时刻应写成 24 小时制 HH:MM。两者按这个空间的时区解释，工具里的“今天”也按它算。时区写在连接时服务端返回的 instructions 里，应以那里的为准，不应假定为某个固定时区。

4.2 任务和步骤的 id 应取自 list_tasks、get_task、create_task 等工具的返回值，不应自行编造。

4.3 update_task 只应传需要修改的字段。date、time、priority、pomodoros、category、cue、reminder 传 null 表示清空，不传表示不改。note 覆盖正文，应同时传 get_task 返回的 note_rev；正文在别处改过时 note_rev 对不上，工具拒绝，应重新 get_task 后再改。note_append 追加在正文之后，不需要 note_rev。

4.4 important_today=true 可将一条任务设为今日最重要。设置后原来的那一条不再是今日最重要。

4.5 delete_task 应仅在用户明确要求删除时调用。tooo 把 delete_task、stop_repeat、cancel_focus、cut_prompt_time 标记为破坏性操作，并在工具元数据里要求每次调用都经用户确认，Claude Code 据此每次弹出确认；其他客户端是否确认由其设置决定，因此不应把确认当作已经发生。其余写工具的确认方式由各客户端的设置决定。删除和停止重复可以用 restore_task 撤销。

注：不再要做的事可用 update_task 改为 type=trash，任务仍保留。

4.6 add_time_entry 只可补记已经过去的时间，起止应在同一天。用户说的整段应原样传入，不应为避开已有记录而自行截短；回复用户时应依据返回的 segments 和 note 说明实际记下的时间。

4.7 remind 的时刻应晚于当前时间，否则工具返回错误。

4.8 调用 set_repeat 时应知道它修改的是整条重复规则，即使传入的 id 是某一天的那一次。停止重复应使用 stop_repeat。

4.9 list_tasks 返回的 note 最多 400 个字符。需要完整正文时应使用 get_task。

4.10 list_tasks 一次最多返回 limit 条。返回的 next_cursor 不为 null 时，说明还有后续，应把它作为 cursor 传回以取下一页，直到 next_cursor 为 null 才算列完；total 是这组条件下的总条数。翻页时不应改 view、date、q，改了要从第一页重新取。

4.11 连接应由账号所有者在自己的设备上批准。tooo 暂不开放注册，Agent 不应代用户注册账号，也不应把命令行密钥（tk_ 开头）写进任务正文或回复。

4.12 每个任务至多一个提醒，reminder 取以下写法之一：{"before": n} 为开始前 n 分钟（0 到 1440，任务应有 date 和 time）；{"at": "YYYY-MM-DD HH:MM"} 为那一刻，应晚于当前时间，任务没有时刻、没有日期也可以用；{"at": "HH:MM"} 为任务那一天（没有日期为今天）的该时刻，可加 "days_before": n 表示提前 n 天（0 到 7）；null 为不提醒。重复规则上只能用 {"at": "HH:MM"} 或 {"before": n}，每次生成时按那一次的日期算出提醒时刻；对重复任务的某一次设置，只改那一次。改任务日期时，还没到的定点提醒跟着挪同样的天数。到点的提醒带「开始、稍后 10 分钟、完成」三个按钮。

4.13 tooo 不调用大模型。提醒文案、act_prompt 的 result 等返回的文字都是固定模板；总结、理由、建议应由调用方生成，不应把这些文字当作 tooo 的判断转述给用户。

4.14 周报：tooo 按 get_prompt_settings 里 weekly 设的星期和时刻（默认周日 20:00）发一条周报提醒（prompts 里 kind=weekly），带那一周做完的件数、计时和番茄数；提醒落在周五到周日时说的是这一周，落在周一到周四时说的是上一周。这一周已存过周报、或这一周没有任何记录时不发。周报内容由调用方读 get_weekly_review、list_weekly_reviews 后起草，用 save_weekly_review 写回；每次写入存成那一周的新一版，旧版保留；提醒出现后 24 小时内存了那一周的周报，记为已响应。MCP 是无状态的 HTTP，tooo 不会主动唤起 Agent；按时起草需要调用方自己安排定时任务，或由用户在对话中提出。

4.15 create_task、add_step、remind、save_weekly_review 接受可选的 id（UUID）。重试同一请求时应传同一个 id，tooo 不会建出第二条，返回已建的那一条。

## 5 示例

<examples>
<example>
create_task：
{"title": "写周报", "date": "2026-10-23", "time": "16:00", "reminder": {"before": 15}, "steps": ["收集数据", "写初稿"]}
</example>
<example>
update_task，给没有时刻的任务定一个提醒时刻：
{"id": "<任务 id>", "reminder": {"at": "2026-10-20 18:30"}}
</example>
<example>
save_weekly_review，写一周的周报：
{"week_start": "2026-10-19", "big": "发了 0.6", "wins": "主动提醒上线", "reflection": "周三下午被会议打断两次", "plan": ["周报模板", "安卓倒计时", "复盘页周切换"]}
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
<example>
set_prompt_settings，周报改到周五 17:00：
{"weekly": {"weekday": 5, "at": "17:00"}}
</example>
</examples>
