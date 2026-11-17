# Examples

A made-up flow to practise on, and four test files that show the shapes that work.

Nothing here has been deployed to an org. The flow uses two custom fields on Case (`Locked__c` and `Needs_Review__c`) that your org does not have. Use these files to see the structure and to try the commands, then run the commands on your own flow.

## The sample flow

[flows/Case_Set_Review_Flag.flow-meta.xml](flows/Case_Set_Review_Flag.flow-meta.xml) runs before save on every create and update of a case.

- It ticks Needs Review when an unlocked case is created as Escalated, or moves to Escalated.
- It unticks Needs Review when the case is locked.
- Otherwise it leaves the box alone.

Try the first command on it:

```
/salesforce-flow-tests:flow-paths examples/flows/Case_Set_Review_Flag.flow-meta.xml
```

## The example tests

| File | What it shows |
|---|---|
| [Case_Set_Review_Flag_T03_Update_Escalated_Ticks](flowtests/Case_Set_Review_Flag_T03_Update_Escalated_Ticks.flowtest-meta.xml) | An update test for the sample flow: the record before and after, a field check, an outcome taken, an outcome not taken |
| [Case_Set_Review_Flag_T04_Update_Still_Escalated_Leaves](flowtests/Case_Set_Review_Flag_T04_Update_Still_Escalated_Leaves.flowtest-meta.xml) | Its twin: a "nothing changes" test, which only means something next to T03 |
| [Case_Set_Review_Flag_ZZ_Control_Must_Fail](flowtests/Case_Set_Review_Flag_ZZ_Control_Must_Fail.flowtest-meta.xml) | A control test: a copy of T03 with its last assertion reversed. It must fail |
| [Sub_Create_Reminder_Task_T02_Owner_Inactive_No_Task](flowtests/Sub_Create_Reminder_Task_T02_Owner_Inactive_No_Task.flowtest-meta.xml) | A test for a subflow (not included here): input variables, a marked Id placeholder, and a check that an element did not run |

Things to notice:

- The file name ends `.flowtest-meta.xml` with a lower-case t.
- No `Id`, no formula field and no relationship path appears in any record.
- An outcome is referenced by its own API name, with no Decision name in front.
- An outcome that must not be taken is checked with `NotEqualTo` `true`, not `EqualTo` `false`.
- A field left blank is left out of the record, not set to `null`.
- The sample flow has no start condition. If it ran only when the status changed, test T04 would fail, because a record that misses the start condition fails its test.
- The files use `testType`, which needs API version 66 (Spring '26) or later.
