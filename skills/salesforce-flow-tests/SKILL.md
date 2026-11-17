---
name: salesforce-flow-tests
description: Write, run and review Salesforce Flow Tests (FlowTest metadata, Flow Builder Test Mode or View Tests, sf flow run test) for record-triggered and autolaunched flows. Use when asked to test a flow, write flow tests, cover a flow's paths, or review existing flow tests.
---

# Salesforce Flow Tests

A Flow Test is a saved check that travels with a flow: a starting record (or input variables) and a list of assertions about what must be true when the flow finishes. This skill is a working method plus a list of platform behaviours that break tests.

The behaviours under "Seen in sandbox runs" were observed in Salesforce sandboxes on API 68 in late 2026. They are observations, not Salesforce documentation. Where one disagrees with what you see, trust the org and say so.

## Method

Work in this order. Each step exists because skipping it costs a failed run.

1. **Get the flow's API name from the person or the file name.** It is not inside the XML. Never build it from the label.
2. **Write the path table before any test.** One row per path, from the XML only:
   - each way into the flow (each term of the start condition; for a subflow, its input variables)
   - every Decision outcome including defaults, and each condition of a multi-condition outcome being the one that decides
   - every override (a lock, an exclusion) paired with each way in its Decision reads
   - list comparisons: each value when six or fewer, else inside, outside, blank, plus a near miss
   - Get Records found and not found; loops with 0, 1, many; fault paths
   - columns: id, elements in order, input, end state (every field the flow can set, value or "unchanged"), twin, Flow Test yes or no
3. **Ask, do not guess.** Custom metadata contents, formula definitions, picklist API values and related-record values are outside the XML. Use named placeholders and ask. A guessed value gives a test that passes for the wrong reason.
4. **Have a person read the table against the requirement.** It is in plain words; this is where a wrong understanding shows.
5. **Write one test, deploy it, run it.** Give it one assertion of every kind the suite will use. Only then write the rest. One real run catches a wrong name or reference before it is in every file.
6. **Add a control test**: a copy of a real test with its last assertion reversed. It must fail. Delete it after one run.
7. **Run, then count.** Compare the number of tests that ran with the number saved.
8. **Review independently.** A second pass that sees only the flow XML and the tests, lists the paths itself first, then traces each test by hand.

## When a test goes red

The expected behaviour lives in the requirement. It does not live in the flow, and it does not live in the test.

1. Read the red test's description. It names the part of the requirement it proves.
2. If the flow no longer does what the requirement says, fix the flow.
3. If the flow is right because the requirement changed, change the requirement first, with a name and a date. Then change the test, and put that date in its description.
4. The person who edited the flow does not decide alone that the test is out of date. Someone else confirms it.
5. Never delete a test without naming what now proves that path: another test, or a real save in a sandbox.

Changing a test until it passes proves nothing. If nobody can say what the flow is meant to do, stop and find out before touching either one.

## What a Flow Test cannot prove, and what proves it instead

| Cannot prove | Use instead |
|---|---|
| Screen flows, scheduled flows | Flow Builder debug runs with the inputs recorded; a person on the screen |
| A path on Run Asynchronously (automated tests with assertions do not support it) | A real save on a throwaway record, then read the debug log |
| A flow that runs only on delete (no assertions allowed) | A real delete in a sandbox |
| Anything that depends on who runs it ($Permission, a bypass, a profile) | Log in as that person and save. A test with assertions runs as the person who starts it |
| A record that misses the start condition | A real save. The test itself reports Fail with `the flow entry condition formula evaluated to false` |
| A subflow with a collection input | Seen: `Collection values are unsupported by test parameters`. Check your org; otherwise a real run, or design the subflow to take one record |
| A fault path, unless an input can cause the failure | A temporary validation rule that refuses the save, removed afterwards. (Salesforce added a Has Error assertion in Summer '25; something still has to cause the failure) |
| Bulk behaviour | A scripted save of many records. A test runs one record |
| What the person sees on the page | A person, once per screen |

## Seen in sandbox runs

Each of these was observed in a failed or surprising run.

- **A capital T in the suffix fails a deploy.** The suffix is `.flowtest`, all lower-case. A Metadata API folder assembled with `.flowTest` failed with `An object '<name>' of type FlowTest was named in package.xml, but was not found in zipped directory`.
- **An `Id` or a formula field in the record JSON ends the test in Error with an empty message.** Leave both out. Salesforce calculates formulas itself, on the before record too: set the fields the formula reads.
- **An outcome the flow did not evaluate is not false.** When an earlier outcome in the same Decision matched, the later ones are never evaluated, and asserting one of them `EqualTo false` fails. `NotEqualTo true` passes in both cases.
- **Asserting a Create Records element's Id** fails when the element stores no output variable: `The test point for elementApiName "Finish" contains a condition with "<element>" and Is null, which isn't compatible.` Assert `$Flow.FaultMessage` is null instead.
- **A create test of a flow that queries its own object** gets the test's own unsaved record back from the Get. Write that path as an update test, or prove it with a real record.
- **In a create test the record has no Id; in an update test the flow sees an existing record.** A Decision that checks `$Record.Id` is null takes the "new" route in a create test and the "update" route in an update test.
- **Tests depend on the org's data.** A lookup Id exists in one org only, and a missing configuration record in the next sandbox can fail most of a suite.
- **Tests read live records.** A green suite goes red when something else edits the records its tests point at. Keep test records nothing else touches.
- **The command line runs the flow's active version.** A test pinned to a draft version came back Skipped with 0 tests run.
- **The runner can report Passed after running only some of the tests.** Count them.
- **An org-wide run can skip whole flows.** Run flow by flow.
- **A failed test often carries no message** on the command line. Open the test in Flow Builder to see which assertion failed and the actual value.
- **A lazy test proves nothing.** If every path sets a field the same way and the test asserts only that field, it passes on every path. Assert every field the flow can set, plus the outcome.

## Working rules

From Salesforce's documentation, or conventions that keep test files consistent.

- **File**: `flowtests/<TestName>.flowtest-meta.xml` in a Salesforce DX project. Metadata type `FlowTest`.
- **Name**: keep the test name within 80 characters.
- **Create or update, never both.** A create gives `InputTriggeringRecordInitial`. An update gives `InputTriggeringRecordInitial` (before) and `InputTriggeringRecordUpdated` (after).
- **Subflows and autolaunched flows** take one `InputVariable` parameter per input variable.
- **Leave roll-up fields, system fields and relationship paths out of the record JSON**, along with `Id` and formulas.
- **Blank means absent.** Leave the field out of the JSON rather than writing `null`.
- **Outcome taken**: the outcome's API name alone, `EqualTo` `true`. No Decision name, no dot. Salesforce documents only "a reference to the flow resource"; this is the form that passes.
- **Default outcomes have no API name.** Prove one by asserting every named outcome of that Decision `NotEqualTo true`.
- **Blank field**: `IsNull` `true`. **Element that must or must not run**: its API name, `WasVisited` `true` or `false`.
- **The triggering record is never saved.** Test Mode runs in rollback mode. Check a sandbox before relying on that for flows that create other records.
- **Every "nothing happens" test needs a twin** where something must happen.
- **Every description names the part of the requirement the test proves**, when a requirement exists.
- **API version**: `testType` needs API 66 (Spring '26) or later.

## File shape

```xml
<?xml version="1.0" encoding="UTF-8"?>
<FlowTest xmlns="http://soap.sforce.com/2006/04/metadata">
    <description>Situation, what should happen, the twin test, and the part of the requirement this proves, in admin words.</description>
    <flowApiName>FLOW_API_NAME</flowApiName>
    <label>T03 Update: status moves to Escalated, Needs Review ticked</label>
    <testPoints>
        <elementApiName>Start</elementApiName>
        <isUseMockOutput>false</isUseMockOutput>
        <parameters>
            <leftValueReference>$Record</leftValueReference>
            <type>InputTriggeringRecordInitial</type>
            <value><sobjectValue>{"Status": "Working", "Locked__c": false, "Needs_Review__c": false}</sobjectValue></value>
        </parameters>
        <parameters>
            <leftValueReference>$Record</leftValueReference>
            <type>InputTriggeringRecordUpdated</type>
            <value><sobjectValue>{"Status": "Escalated", "Locked__c": false, "Needs_Review__c": false}</sobjectValue></value>
        </parameters>
    </testPoints>
    <testPoints>
        <assertions>
            <conditions>
                <leftValueReference>$Record.Needs_Review__c</leftValueReference>
                <operator>EqualTo</operator>
                <rightValue><booleanValue>true</booleanValue></rightValue>
            </conditions>
            <errorMessage>Needs Review should be ticked</errorMessage>
        </assertions>
        <assertions>
            <conditions>
                <leftValueReference>Ask_For_Review</leftValueReference>
                <operator>EqualTo</operator>
                <rightValue><booleanValue>true</booleanValue></rightValue>
            </conditions>
            <errorMessage>The "Ask for review" outcome should be taken</errorMessage>
        </assertions>
        <assertions>
            <conditions>
                <leftValueReference>Not_Eligible</leftValueReference>
                <operator>NotEqualTo</operator>
                <rightValue><booleanValue>true</booleanValue></rightValue>
            </conditions>
            <errorMessage>The "Not eligible" outcome should not be taken</errorMessage>
        </assertions>
        <elementApiName>Finish</elementApiName>
        <isUseMockOutput>false</isUseMockOutput>
    </testPoints>
    <testType>WithAssertion</testType>
</FlowTest>
```

Subflow input, in place of the `$Record` parameters:

```xml
<parameters>
    <leftValueReference>OpportunityId</leftValueReference>
    <type>InputVariable</type>
    <value><stringValue>REPLACE_WITH_ID_open_Opportunity</stringValue></value>
</parameters>
```

## Commands

Always name the org. Never rely on a default org for a deploy. Use a sandbox.

```bash
# check the files without saving them (NoTestRun stops the org's Apex tests from starting)
sf project deploy start --target-org ORG_ALIAS --source-dir force-app/main/default/flowtests --dry-run --test-level NoTestRun

# deploy
sf project deploy start --target-org ORG_ALIAS --source-dir force-app/main/default/flowtests

# run one flow's tests and wait; a synchronous run covers one flow
sf flow run test --target-org ORG_ALIAS --class-names FLOW_API_NAME --synchronous --result-format json --output-dir test-results
```

The user running the tests needs the View All Data permission.

## When to run them

On every deploy of the flow. Before any other testing starts. After anyone edits the flow by hand. Before every promotion. In each higher org before the flow is activated there, after replacing the Id placeholders for that org.

## Reporting

Say how many paths the table has, how many are proven by a Flow Test, how many by something else and what, and how many are not proven. "All tests passed" on its own says nothing about the paths without a test.
