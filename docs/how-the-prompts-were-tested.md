# How the prompts were tested

The prompts in this repository were tried, graded and changed over seven rounds. The scores did not rise steadily, and this page says where they fell and why. The rule that came out of it: no change to a prompt is published until a fresh run has checked it.

## The set-up

- **A trial agent.** A fresh AI agent (Claude Sonnet) was given Prompt 1, then Prompt 2, and one real flow's XML. It was told to look at nothing else: not the existing tests for that flow, not the rest of the project, not the web, not a Salesforce org. It also reported anything in the prompts it found unclear or had to guess.
- **A grader.** A second agent compared the result with an existing, passing test suite for that flow, using ten yes-or-no checks: is the summary right, is every Decision outcome in the table, are unknown values asked about and not guessed, does every "nothing changes" row have a twin, are the files shaped correctly, is the input free of fields that break a test, does every test check enough fields, and is every path the existing suite covers also covered here.
- **A hand trace.** The grader walked five to seven of the tests through the flow XML element by element, to confirm each input takes the path its test claims and each assertion would hold.

The "I deployed it and it passed" reply that Prompt 2 waits for was simulated. Nothing from the trials was deployed.

## The scores

| Round | Record-triggered flow | Subflow |
|---|---|---|
| 1 | 8 of 10 | |
| 2 | 8 of 10 | |
| 3 | 9 of 10 | |
| 4 | | 9 of 10 |
| 5 | 8 of 10 | 8 of 10 |
| 6 | 9 of 10 | 8 of 10 |
| 7 | 10 of 10 | |

Round 7 is one run, and it is the only round where a written requirement was supplied with the flow. Read it as "nothing got worse", not as "the prompts are perfect".

## The rounds

### Round 1: 8 of 10

22 path rows, 15 test files. The files would probably not have run in an org.

- The flow's API name was built from its label, and was wrong in every file.
- Decision outcomes were referenced as `Decision_Name.Outcome_Name`. The existing passing tests use the outcome's API name alone.
- Blank fields were written as `null` in the test record.
- Three paths the existing suite covers had no test.

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

- Every path the existing suite's eight tests cover had a matching test.
- The model added seven paths that suite did not test, a control test, and a count of tests to expect.
- The miss: it used assertion forms (`WasVisited`, `NotEqualTo`) that the existing tests for that subflow do not use, though passing tests for other flows do.

Changes made after rounds 3 and 4: Prompt 2 starts by applying the answers to the path table and adding the rows they call for; the first test carries one assertion of every kind; a row whose expected result is unknown is blocked instead of guessed; a test asserts the stored fact it relies on.

### Round 5, both flows: 8 of 10 and 8 of 10

A step backwards.

- The fix for list values worked: three rows were added after the answers.
- The new "blocked" sentence was applied too widely. On the record-triggered flow the model blocked rows whose results the flow's own logic settles. On the subflow it blocked the error-handling row.
- One placeholder had no matching question.

Changes made: a row is blocked only when neither the flow's logic nor the answers give its result; every placeholder gets its own numbered question; the first round names any assertion form later tests will need.

### Round 6, both flows: 9 of 10 and 8 of 10

- Record-triggered flow: every round 5 miss was fixed. The miss: the extra list values were covered on an update, not on a create. Four assertions were written on a relationship path, a form no existing test uses.
- Subflow: the error-handling path still had no test. The model marked it "no" in the path table, saying a Flow Test cannot make a lookup fail. That is wrong: passing text that is not a record Id does it.

Change made: Prompt 1 says an error path is testable when an input can cause the failure.

### Round 7, record-triggered flow: 10 of 10

Two things were new: Prompt 2 asks each test to name the part of the requirement it proves, and for the first time a short written requirement (four acceptance criteria) was supplied with the flow.

- 33 path rows, 24 tests and a control. All ten checks passed.
- Every test named a criterion, and all eight that were checked named the right one.
- The round 6 misses did not recur: each list value had its own create test, and no assertion used a relationship path.
- Supplying the requirement did no harm. Expected results still came from the flow's logic, no path was dropped, and four places where the requirement and the flow disagree were raised as questions.

## Prompt 3

Prompt 3 was run three times on 17 existing tests for the record-triggered flow.

- **Run 1** (an early, shorter version): it listed paths with no test and paths a Flow Test cannot prove, some already known and some not. It found a test that repeated another because the flow never reads the field that differed. One of its main findings was a false alarm about how the platform handles formulas.
- **Run 2** (with a paragraph of platform behaviours added): the same real findings, plus a second duplicate test. Its biggest finding was a false alarm, and the prompt caused it. The prompt said a test record "has no Id", which is true only for a create test; the reviewer concluded every update test took the wrong route.
- **Run 3** (that sentence corrected): the false alarm was gone. Its biggest finding was true: one rule had no positive test, so two "nothing happens" tests would still pass if the rule were deleted. It listed what it was unsure of instead of asserting it.

One sentence in Prompt 3 was made more precise after run 3 and has not been run since.

## The slash commands

The three commands in `commands/` are built from the three prompts by a script. The only difference is the first lines, where the command's arguments replace the fill-in brackets. The first command was run once on the sample flow in `examples/`, as a new user would run it; its unedited output is in [examples/sample-output-flow-paths.md](../examples/sample-output-flow-paths.md). The second and third commands have not been run in command form.

## What is still unproven

- **Nothing from the trials was deployed to an org.** Files were compared with existing passing tests and traced by hand. This is why Prompt 2 writes one test first and stops until you have run it.
- **The subflow has not been re-run since round 6**, so the fix for its missing error-path test is unconfirmed on that flow.
- **Assertion forms.** `NotEqualTo`, `IsNull` and `WasVisited` appear in passing tests for some flows, but not every flow. Whether leaving a field out of the "after" record means "cleared" is not established.
- **Two flows.** One record-triggered flow that runs before save, and one subflow. The prompts have not been tried on a flow with loops, on an after-save flow that creates records, or on a Data Cloud-triggered flow.
- **One model, with file tools.** The trial agent was Claude Sonnet working with files. A plain chat, or another model, may behave differently.
- **Run-to-run variation.** The same prompts scored differently on different flows, and one round's miss was the model's own wrong assumption, not a prompt sentence. Check the output; do not just trust it.
