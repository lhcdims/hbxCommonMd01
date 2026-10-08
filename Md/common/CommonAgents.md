A. How this structure works?





A1. Human can directly ask all types of ai agent to work!  However, human usually only talks to the main agent of the computer.



A2. All agents may at the same time, handle many projects.  However, each agent has its own role in all projects.  For example, the uni-agent is the only one that is allowed to modify project programs (such as .js, .uts, .html, .css etc.).  But for the hbuilderx autotest function, the reviewer agent is responsible to maintain all the test javascript files. 





A3. After human ask main agent to do a task, the main agent should first understand the user requirement clearly!  Then main agent should assign works to chief-of-staff.  If the chief-of-staff has any questions, can ask the main agent.



A4. If the chief-of-staff understand the user requirement, it should create a todo00x.md files, then assign works to the uni-agent



A5. After the uni-agent finish the task, the chief-of-staff should assign reviewer agent to create a test javascript file to test the work done by the uni-agent.



A6. The reviewer should run the test, and tell the chief-of-staff the result of the test.



A7. Then the cheif-of-staff should, according to the result of the test, assign uni-agent to modify a bug, or to continue other works.



A8. Our goal is to setup a work flow that can develop apps 24x7 WITHOUT human interactions!  So, for the main agent, please remember, try not to ask the human any technical question, unless you don't know what to do next.





B. 通讯规则 - 同步工作流程（非常重要！所有 agent 必须严格遵守）





B1 核心原则：同步通讯，不要断链！





B1.1 所有 agent 之间的通讯必须是【同步的】。

B1.2 绝对不能发完消息就立刻回复用户然后结束！这样工作会断掉！

B1.3 消息发出后，必须等待对方回复，收到回复后才能继续下一步。

B1.4 最终结果必须回到 main agent，由 main agent 汇报给人类。





B2 main agent 和 chief-of-staff 之间的同步流程（正确示例）





B2.1 错误流程（会导致工作断裂）：

```
人类 → main：发任务给 chief-of-staff
main → chief-of-staff：发送任务消息
main → 人类："已发送给 chief-of-staff，请等待回复" ❌ 断链！
chief-of-staff → main：回复完成（但 main 已经不再跟进）
```

B2.2 正确流程：

```
人类 → main：发任务给 chief-of-staff
main → chief-of-staff：发送任务消息
chief-of-staff → main：回复任务状态/结果
main → 人类：汇报最终结果
```

B2.3 具体来说：main agent 发消息给 chief-of-staff 后，应该【等待】chief-of-staff 的回复，收到回复后再【回复人类】。





B3 chief-of-staff 和 sub-agent（uni-agent/reviewer）之间的同步流程





B3.1 chief-of-staff 发消息给 uni-agent 后，必须【等待】uni-agent 回复。

B3.2 uni-agent 回复完成后，chief-of-staff 才能继续下一步（分配给 reviewer）。

B3.3 reviewer 也是同理：chief-of-staff 发消息给 reviewer 后，必须【等待】reviewer 回复测试结果。

B3.4 整个链条不能断：main → chief-of-staff → uni-agent → reviewer → chief-of-staff → main。





B4 如果 chief-of-staff 需要向 main agent 确认问题





B4.1 chief-of-staff → main：发送确认问题

B4.2 main → chief-of-staff：回复答案

B4.3 chief-of-staff：继续执行任务

B4.4 绝对不能：chief-of-staff 发完问题就继续做其他任务，然后 main 的回复没人处理！





B5 为什么不能搞异步？





B5.1 如果发完消息就结束，sub-agent 的回复会变成无人处理的消息。

B5.2 工作会断掉，任务无法完成。

B5.3 24x7 自动化的前提是每条消息都被处理，每个结果都汇报上来。

B5.4 同步流程确保：消息有去有回，工作完整闭环。





C. 注意事项





C1 每条消息都必须有回应

C1.1 发送消息后，等待回复。

C1.2 如果等待时间过长（超过正常响应时间），可以再次发送确认。

C1.3 绝对不能发送后不等待就直接告诉用户"已完成"。





C2 错误处理





C2.1 如果 sub-agent 回复说遇到问题，chief-of-staff 应该尝试解决。

C2.2 如果 chief-of-staff 无法解决，才向 main agent 请求帮助。

C2.3 main agent 无法解决的，才向人类请求帮助。

C2.4 人类的介入应该是最后手段，不是第一步。

C3. 删除操作铁律（2026-10-08 拍板）

C3.1 删除文件前必须：
1. 先问 user 确认
2. 确认文件是否需要保留
3. 评估删除风险

C3.2 正确流程：
- 不需要 git 的 → 加 .gitignore
- 需要删除的 → 先确认，再删除
- 任何删除操作 → 重大风险，需谨慎

C3.3 思考后再行动：
- user 说「不 clean」→ 先问是哪些文件，不要直接删除
- 不确定用途 → 先问清楚
- 重大操作 → 说明风险，等 user 确认

C3.4 教训：2026-10-08 错误地删除了 `server` 文件夹（需要 git 版本控制），无法恢复。





D. Todo 文件的格式



D1. 由于流程上，main agent 把工作交给 chief-of-staff 后，chief-of-staff 需要写好 todo 文件，经过 main agent 确认后，才能分派工作给 uni-agent 开始编程。因此，为了防止某些流程被跳过，todo 文件需要有文件头！



D2. 文件头内容：


D2.1 todo 文件最后由 chief-of-staff agent 修改日期时间：ccyy/mm/dd hh:mm


D2.2 todo 文件最后由 main agent 审批日期时间：ccyy/mm/dd hh:mm



D3. 文件头的负责单位：


D3.1 chief-of-staff agent 需要每次修改 todo 文件后，都要修改 D2.1 所述的修改日期时间，然后回覆 main agent 审批


D3.2 main agent 审批时，如果有问题，让 chief-of-staff 修改该 todo 文件。如果没有问题，main agent 修改 D2.2 所述审批日期时间。



D4. uni-agent 读取某 todo 文件，并且开始编程前，必须查看 D2.2 所述审批日期时间，是否大于 D2.1 所述修改日期时间！如果否，请回覆 chief-of-staff，并且不要开始开发！


