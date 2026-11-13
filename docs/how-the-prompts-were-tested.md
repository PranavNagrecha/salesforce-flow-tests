# How the prompts were tested

The prompts in this repository were tried, graded and changed over six rounds. The scores did not rise steadily, and this page says where they fell and why.

## The set-up

- **A trial agent.** A fresh AI agent (Claude Sonnet) was given Prompt 1, then Prompt 2, and one real flow's XML. It was told to look at nothing else: not the existing tests for that flow, not the rest of the project, not the web, not a Salesforce org. Its logged tool calls stayed in the folder that held only those files. It also reported anything in the prompts it found unclear or had to guess.
- **A grader.** A second agent compared the result with the tests a team already runs for that flow, using ten yes-or-no checks: is the summary right, is every Decision outcome in the table, are unknown values asked about and not guessed, does every "nothing changes" row have a twin, are the files shaped correctly, is the input free of fields that break a test, does every test check enough fields, and is every path the accepted tests cover also covered here.
- **A hand trace.** The grader walked five to seven of the tests through the flow XML element by element, to confirm each input takes the path its test claims and each assertion would hold.

The "I deployed it and it passed" reply that Prompt 2 waits for was simulated. Nothing was deployed.

## The scores

| Round | Record-triggered flow | Subflow |
|---|---|---|
| 1 | 8 of 10 | |
| 2 | 8 of 10 | |
| 3 | 9 of 10 | |
| 4 | | 9 of 10 |
| 5 | 8 of 10 | 8 of 10 |
| 6 | 9 of 10 | 8 of 10 |

## The rounds

### Round 1: 8 of 10

22 path rows, 15 test files. We do not think any of the files would have run in an org.

- The flow's API name was built from its label, and was wrong in every file.
- Decision outcomes were referenced as `Decision_Name.Outcome_Name`. Our accepted tests use the outcome's API name alone.
- Blank fields were written as `null` in the test record.
- Three paths the accepted tests cover had no test.

Changes made: the prompt asks for the API name; the example file shows an outcome assertion and a blank-field assertion; one test is written and run before the rest; the path table is built in a fixed order, with each part of the start condition and each override as its own row.

### Round 2: 8 of 10

28 path rows, 30 test files. Every round 1 fault was gone.

- Rows that depend on the runner's permissions were marked as testable. A suite run by one person cannot pass both the "holds the permission" and the "lacks it" rows.
- Most update tests checked only one field and the outcomes.
- A few tests repeated others, differing only in a field the flow never reads.
- The run command was a guess, and the agent said it was unsure of it.

Changes made: permission-dependent rows are a "no" with a by-hand proof; every test asserts every field the flow can set; duplicate rows are ruled out; the commands are in the prompt.

### Round 3: 9 of 10

27 path rows (20 testable, 7 not), 20 test files and a control test. Both round 2 misses were fixed.

- When the answers revealed that a list holds four values, the model did not go back and add a row for each.

### Round 4, a subflow: 9 of 10

The round 3 prompts, unchanged, on a different kind of flow, with no answers given to the model's questions.

- Every path the team's eight accepted tests cover had a matching test.
- The model added seven paths the accepted set did not test, a control test, and a count of tests to expect.
- The miss: it used assertion forms (`WasVisited`, `NotEqualTo`) that the accepted tests for that subflow do not use. Accepted tests elsewhere in the same repository, in two other projects, do use them.

Changes made after rounds 3 and 4: Prompt 2 starts by applying the answers to the path table and adding the rows they call for; the first test carries one assertion of every kind; a row whose expected result is unknown is blocked instead of guessed; a test asserts the stored fact it relies on.

### Round 5, both flows: 8 of 10 and 8 of 10

A step backwards.

- The fix for list values worked: three rows were added after the answers.
- The new "blocked" sentence was applied too widely. On the record-triggered flow the model blocked the blank-status rows and a hand-tick row, although the flow's own logic settles their results. On the subflow it blocked the error-handling row.
- One placeholder had no matching question.

Changes made: a row is blocked only when neither the flow's logic nor the answers give its result; every placeholder gets its own numbered question; the first round names any assertion form later tests will need.

### Round 6, both flows: 9 of 10 and 8 of 10

- Record-triggered flow: every round 5 miss was fixed. The blank-status and hand-tick tests came back, marked "not yet proven to work" where the form is uncertain. The miss: the three extra list values were covered on an update, not on a create.
- Subflow: the error-handling path still had no test. The model marked it "no" in the path table, saying a Flow Test cannot make a lookup fail. That is wrong: passing text that is not a record Id does it.

Change made, not yet tried: Prompt 1 now says an error path is testable when an input can cause the failure.

## Prompt 3

Prompt 3 was run three times on 17 accepted tests for the record-triggered flow.

- **Run 1** (an early, shorter version): it listed five paths with no test and four a Flow Test cannot prove. The team had two of those gaps on record. It found a test that repeated another because the flow never reads the field that differed, which the team had not noticed. One of its main findings was a false alarm about how the platform handles formulas.
- **Run 2** (with a paragraph of platform facts added): the same real findings, plus a second duplicate test. Its biggest finding was a false alarm, and the prompt caused it. The prompt said a test record "has no Id", which is true only for a create test; the reviewer concluded every update test took the wrong route.
- **Run 3** (that sentence corrected): the false alarm was gone. Its biggest finding was true and already known: one rule has no positive test, so two "nothing happens" tests would still pass if the rule were deleted. It listed what it was unsure of instead of asserting it.

One sentence in Prompt 3 was made more precise after run 3 and has not been run since.

## What is still unproven

- **Nothing from the trials was deployed to an org.** Files were compared with accepted tests and traced by hand. This is why Prompt 2 writes one test first and stops until you have run it.
- **One sentence in Prompt 1 and one in Prompt 3 were changed after the last run.**
- **Known weak spots in the last round:** a create test per list value may be missing; an assertion on a relationship path (`$Record.Account__r.Name`) may be written, which no accepted test uses; an error path may be marked untestable when it is not.
- **Assertion forms.** `NotEqualTo`, `IsNull` and `WasVisited` pass in some accepted tests, but not in every flow's set. Whether leaving a field out of the "after" record means "cleared" is not established.
- **Two flows.** One record-triggered flow that runs before save, and one subflow. The prompts have not been tried on a flow with loops, on an after-save flow that creates records, or on a Data Cloud-triggered flow.
- **One model, with file tools.** The trial agent was Claude Sonnet working with files. A plain chat, or another model, may behave differently.
- **Run-to-run variation.** The same prompts scored 9 on one flow and 8 on another, and one round's miss was the model's own wrong assumption, not a prompt sentence. Expect to check the output, not to trust it.
