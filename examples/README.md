# Example Flow Test files

Four files that show the shapes that work. They belong to two made-up flows, so they will not deploy as they are. Use them to see the structure, then let the prompts write tests for your own flow.

The made-up record-triggered flow, `Case_Set_Review_Flag`, runs before save on every create and update of a case. If it ran only when the status changed, test T04 would fail, because in our runs a record that misses the start condition fails its test.

| File | What it shows |
|---|---|
| [Case_Set_Review_Flag_T03_Update_Escalated_Ticks](flowtests/Case_Set_Review_Flag_T03_Update_Escalated_Ticks.flowtest-meta.xml) | An update test: the record before and after, a field check, an outcome taken, an outcome not taken |
| [Case_Set_Review_Flag_T04_Update_Still_Escalated_Leaves](flowtests/Case_Set_Review_Flag_T04_Update_Still_Escalated_Leaves.flowtest-meta.xml) | Its twin: a "nothing changes" test, which only means something next to T03 |
| [Sub_Create_Reminder_Task_T02_Owner_Inactive_No_Task](flowtests/Sub_Create_Reminder_Task_T02_Owner_Inactive_No_Task.flowtest-meta.xml) | A subflow test: input variables, a marked Id placeholder, and a check that an element did not run |
| [Case_Set_Review_Flag_ZZ_Control_Must_Fail](flowtests/Case_Set_Review_Flag_ZZ_Control_Must_Fail.flowtest-meta.xml) | A control test: a copy of T03 with its last assertion reversed. It must fail |

Things to notice:

- The file name ends `.flowtest-meta.xml` with a lower-case t.
- No `Id`, no formula field and no relationship path appears in any record.
- An outcome is referenced by its own API name, with no Decision name in front.
- An outcome that must not be taken is checked with `NotEqualTo` `true`, not `EqualTo` `false`.
- A field left blank is left out of the record, not set to `null`.
- The files use `testType`, which needs API version 66 (Spring '26) or later.
