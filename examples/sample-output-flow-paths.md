# Case_Set_Review_Flag

## 1. What the flow does

1. It is a record-triggered flow on Case that runs before the save, on both create and update, so it edits the case in memory and does not save it a second time.
2. It has no start condition. Every case create and every case update enters it, so no save is skipped.
3. It reads four things: `Locked__c`, `Status`, whether the case is new (Id is blank), and what `Status` was before this save.
4. It ticks `Needs_Review__c` when the case is not locked, `Status` is Escalated, and the case is either new or was not Escalated before this save.
5. If the case is locked, it unticks `Needs_Review__c`. This happens on every save of a locked case, whatever the status and whatever the box held.
6. Any other save leaves `Needs_Review__c` as it is.
7. The only field it ever changes is `Needs_Review__c`. It creates no records and does not change `Status` or `Locked__c`.

No requirement was available, so I compared the flow only against its own descriptions.

## 2. Path table

**Placeholders**
- `<A>` and `<B>`: two different Case statuses that are not Escalated.
- `<look-alike>`: a status whose name resembles Escalated but is not Escalated.

**Elements by outcome**
- Ask for review: Start → Should this case be reviewed? → Tick_Needs_Review.
- Not eligible: Start → Should this case be reviewed? → Untick_Needs_Review.
- Leave Needs Review as it is (the default): Start → Should this case be reviewed? → end.

**Start terms, Get Records, loops, fault paths**
- Start terms: none. There is no new-record term and no "field changed" term. A row where no term is met cannot exist.
- Comparisons against a list of values: none. `Status` is compared to the single value Escalated, and row 10 is its near miss.
- Get Records: none.
- Loops: none.
- Fault paths: none.

| id | elements in order | input (only what the flow reads) | what must be true at the end | twin | Flow Test? |
|---|---|---|---|---|---|
| 1 | Ask for review, decided by Id blank (new case) | **Create**: Locked false, Status Escalated, Needs Review false | Needs Review **true**. Status and Locked unchanged. Nothing created. | none needed | yes |
| 2 | Ask for review, decided by Prior Status ≠ Escalated | **Update**: Locked false, Status `<A>` → Escalated, Needs Review false before | Needs Review **true**. Status and Locked unchanged. | none needed | yes |
| 3 | Not eligible (locked beats the new-case way in) | **Create**: Locked true, Status Escalated, Needs Review true | Needs Review **false** | none needed | yes |
| 4 | Not eligible (locked beats the status-change way in) | **Update**: Locked true, Status `<A>` → Escalated, Needs Review true before | Needs Review **false** | none needed | yes |
| 5 | Not eligible alone | **Update**: Locked true, Status `<A>` → `<A>`, Needs Review true before | Needs Review **false** | none needed | yes |
| 6 | Leave Needs Review as it is | **Create**: Locked false, Status `<A>`, Needs Review false | Needs Review unchanged (false). Nothing else changes. | row 1 | yes |
| 7 | Leave Needs Review as it is | **Update**: Locked false, Status `<A>` → `<A>`, another field edited, Needs Review false | Needs Review unchanged (false) | row 2 | yes |
| 8 | Leave Needs Review as it is | **Update**: Locked false, Status Escalated → Escalated, another field edited, Needs Review false | Needs Review unchanged (false). The case is already Escalated, so it is not re-ticked. | row 2 | yes |
| 9 | Leave Needs Review as it is | **Update**: Locked false, Status Escalated → `<B>`, Needs Review true before | Needs Review unchanged (true). Leaving Escalated does not untick it. | row 2 | yes |
| 10 | Leave Needs Review as it is (near miss) | **Update**: Locked false, Status `<A>` → `<look-alike>`, Needs Review false | Needs Review unchanged (false) | row 2 | yes |

**Notes on the inputs**
- Set Needs Review before the save to the opposite of what the flow should produce. Ticking a box that is already true, or unticking one that is already false, shows nothing.
- Rows 6, 7, 8 and 10 start with Needs Review false so a wrong tick would show. Row 9 starts true so a wrong untick would show.

**Notes on the Flow Test? column**
- Every row is a yes. The flow has no screen, no asynchronous path, no delete trigger, no `$Permission`, no bypass and no profile check.
- Every row would still pass whoever runs it.
- The start has no condition, so no record can miss it.

## 3. Questions

1. **Is Escalated an exact, active Case Status value** in the record type and support process you will test with? This affects rows 1–5 and 8–9. Case Status choices depend on the support process.
2. **Which two statuses should replace `<A>` and `<B>`?** They must be valid for that record type and not Escalated. This affects rows 2, 4–10.
3. **Is there a status that looks like Escalated but is not?** An example would be "Escalated - Tier 2". If none exists, row 10 cannot be built and should be dropped.
4. **Can `$Record__Prior.Status` be read on a create without error?** Row 1 is decided by the Id-blank term, and row 2 by the Prior Status term. I doubt a record exists where one of those terms is true and the other false in the way a clean separation would need. A new case has no prior record, so I cannot say what the Prior Status term returns on create. Rows 1 and 2 are the closest test of each term; please confirm row 1 behaves as expected.
5. **Does "locked" mean the custom checkbox `Locked__c`**, rather than a record lock from an approval process? Also, the flow's own description says it was never deployed and that `Locked__c` and `Needs_Review__c` do not exist in your org. The flow status is Draft. No test can run until those fields exist and the flow is deployed. This affects all rows.
6. **Is it intended that every save of a locked case clears the box?** The flow-level description says it unticks "when the case is locked", which could mean only the moment of locking. The logic unticks on every save while locked, including a tick a supervisor added by hand. This affects rows 3–5.
7. **Is it intended that an Escalated case is not re-ticked after it is unlocked?** Take a case that is Escalated, ticked, then locked, which unticks it. When it is unlocked its status has not changed, so the flow leaves the box unticked (row 8). This affects row 8.
8. **Should a case that leaves Escalated and comes back be re-ticked?** The flow ticks it again, even if a supervisor had unticked it after reviewing. This affects rows 2 and 9.
9. **The flow-level description says it ticks "when the case moves to Escalated".** The decision's description and logic also tick a new case created as Escalated (row 1). Confirm that is wanted.
