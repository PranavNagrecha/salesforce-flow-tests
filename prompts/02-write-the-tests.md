# Prompt 2: write the tests

Turns the path table into Flow Test files: one test first, and the rest after that one has run in your org.

Answer the questions from Prompt 1 first. Use this in the same conversation, or paste the path table and your answers into a new one.

```text
First apply my answers to the path table: where an answer settles something the table left open (the values in a list, for example), add the rows the table's own rules call for, under new ids, and show me the added rows. Then write a Salesforce Flow Test for every row marked yes. Where one of my answers is still missing, keep the placeholder and say which test it blocks. Block a row, with no test, only when neither the flow's logic nor my answers give its expected result. If the logic settles the result and only the way to trigger it is unknown (a value that makes a read fail, for example), or only a label or description disagrees with the logic, write the test from the logic, mark it "not yet proven to work", and put the open point in the closing table.

Flow API name: [SAME AS BEFORE]
Path table: [PASTE IT, OR SAY "above" IF THIS IS THE SAME CHAT]
Answers: [YOUR ANSWERS, NUMBERED LIKE THE QUESTIONS]
Org alias: [YOUR SANDBOX ALIAS IN THE SALESFORCE CLI, OR "none" IF YOU WILL USE FLOW BUILDER]
Folder, if you can write files: [e.g. force-app/main/default/flowtests]. If you cannot, give each file in its own code block under its file name.

Work in two rounds. First write one test only: the row with the fewest placeholders, carrying one assertion of every kind the suite will use (a field value, an outcome taken, an outcome not taken, a blank field, an element visited or not). Give me the commands to deploy and run it, name any assertion form later tests will need that this one lacks, and stop. One real run catches a wrong name or a wrong reference before it is copied into every file. When I say it passed, write the rest.

Each of these comes from a failed run or from Salesforce's own guidance:
- File name: <FlowApiName>_<case>.flowtest-meta.xml, lower-case t, in a flowtests folder. <case> is the row id plus the path and result in a few words (T03_Update_MovesIn_Ticks). Keep the part before .flowtest-meta.xml within 80 characters.
- A record-triggered test is a create or an update. A create gives the new record (InputTriggeringRecordInitial). An update gives the record before (InputTriggeringRecordInitial) and after (InputTriggeringRecordUpdated). An autolaunched flow or subflow takes one InputVariable parameter per input variable instead. Treat a row that needs a collection input as a no, unless I tell you my org accepts one in a test.
- In the record JSON never include Id or a formula field. Either one ends the test in Error with no message. Leave out roll-up fields, system fields and paths like Account__r.Name as well. Salesforce calculates formulas itself, on the before record too, so set the fields the formula reads.
- To leave a field blank, leave it out of the JSON. Do not write null. If a row needs the update itself to clear a field, write the test and mark it "not yet proven to work" in the closing table.
- Record Ids belong to one org. Write each as a marked placeholder (REPLACE_WITH_ID_Contact_with_Status_Active) and say what that record must contain. Tests read the real record, so it must not change between runs; where a row relies on one stored fact about it (the owner is active), assert that fact too, so a changed record fails visibly.
- The record under test is never saved. The test runs as me. From the command line it runs the flow's active version. Use picklist API values exactly as my answers give them.

What every test asserts:
- Every field the flow can set, on every test, with its expected value or its unchanged value. On an update, give such a field a value in the before record so "unchanged" can be checked.
- The outcome taken: the outcome's own API name, alone, EqualTo true. No Decision name in front, no dot, no $.
- Each named outcome that must not be taken: NotEqualTo true. EqualTo false works only when the Decision evaluated that outcome and fails when the flow never reached it, so NotEqualTo true is the safe form. A default outcome has no API name; prove it by asserting every named outcome of that Decision NotEqualTo true.
- A field that must be blank: IsNull true. An element that must run or must not run (a Create Records, an action, a subflow): its API name, WasVisited true or false.
A test that checks only what every path sets the same way passes on every path, so never rely on one. Two tests that differ only in a field the flow never reads are one test. Each error message says what should have happened. The description says, in admin words, the situation, what should happen and which row is its twin. The label starts with the row id and stays short.

Use this shape, and only these element names:

<?xml version="1.0" encoding="UTF-8"?>
<FlowTest xmlns="http://soap.sforce.com/2006/04/metadata">
    <description>...</description>
    <flowApiName>FLOW_API_NAME</flowApiName>
    <label>T03 ...</label>
    <testPoints>
        <elementApiName>Start</elementApiName>
        <isUseMockOutput>false</isUseMockOutput>
        <parameters>
            <leftValueReference>$Record</leftValueReference>
            <type>InputTriggeringRecordInitial</type>
            <value><sobjectValue>{"Status__c": "Open", "Locked__c": false}</sobjectValue></value>
        </parameters>
        <!-- update tests only: a second parameters block, type InputTriggeringRecordUpdated -->
        <!-- autolaunched flows instead: one block per input variable:
             <leftValueReference>VariableApiName</leftValueReference>
             <type>InputVariable</type>
             <value><stringValue>...</stringValue></value> -->
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
                <leftValueReference>Outcome_Api_Name</leftValueReference>
                <operator>EqualTo</operator>
                <rightValue><booleanValue>true</booleanValue></rightValue>
            </conditions>
            <errorMessage>The "..." outcome should be taken</errorMessage>
        </assertions>
        <assertions>
            <conditions>
                <leftValueReference>$Record.Closed_Reason__c</leftValueReference>
                <operator>IsNull</operator>
                <rightValue><booleanValue>true</booleanValue></rightValue>
            </conditions>
            <errorMessage>Closed Reason should stay blank</errorMessage>
        </assertions>
        <elementApiName>Finish</elementApiName>
        <isUseMockOutput>false</isUseMockOutput>
    </testPoints>
    <testType>WithAssertion</testType>
</FlowTest>

The commands, with my alias filled in:
sf project deploy start --target-org ORG_ALIAS --source-dir FLOWTESTS_FOLDER
sf flow run test --target-org ORG_ALIAS --class-names FLOW_API_NAME --synchronous --result-format json --output-dir RESULTS_FOLDER
If I have no alias, tell me where to find and run the tests in Flow Builder instead.

Finish with:
1. A table: row id | file | create, update or subflow | what it asserts | placeholders to replace.
2. The rows marked no, and how each will be proven instead.
3. One control test: a copy of the first test with its last assertion reversed, named <FlowApiName>_ZZ_Control_Must_Fail. If it passes, the suite cannot fail and proves nothing. Tell me to delete it after one run.
4. The number of tests that should run, with and without the control. The runner can report "Passed" after running only some of them, so I will compare the count.
```
