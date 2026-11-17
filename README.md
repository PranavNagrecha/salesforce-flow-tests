# Salesforce Flow Tests with Claude

Let an AI do the dull part of testing your flows, so the testing actually gets done.

Most flows are tested by clicking Debug, looking at the result, and doing it again after every change. Salesforce has a better tool, Flow Tests, and few people use it, because writing a test for every path of a flow is slow, repetitive work.

That kind of work is what an AI assistant is good at. This repository gives Claude the method and the platform knowledge to do it:

1. **Read a flow and list every path through it, in plain words.** You check the list against what the flow is meant to do.
2. **Write a Flow Test for each path.** One first, which you run, then the rest.
3. **Review a set of tests** and point out the paths nobody covered.

You stay in charge of the judgement: what the flow should do, and whether the tests prove it. Claude does the typing.

The full write-up is on Medium: [Salesforce Flow Tests: What Breaks Them, and AI Prompts That Write Them](https://medium.com/@ifxubc/salesforce-flow-tests-what-breaks-them-and-ai-prompts-that-write-them-168a8295951b)

## What is a Flow Test?

A saved check that lives with a flow. You give it a starting record (a Case with Status = Working) and say what must be true after the flow runs (Needs Review is ticked, the "Ask for review" path was taken). Salesforce runs it and reports pass or fail. The record is never saved.

Next month, when someone changes the flow, you run the tests again and know in seconds whether something that used to work is now broken.

Flow Tests work for record-triggered, autolaunched and Data Cloud-triggered flows. In Flow Builder they are under **View Tests**, or **Test Mode** from the Winter '27 release on.

## Install

### Claude Code: one command, the same on Windows and Mac

Inside Claude Code, type:

```
/plugin install salesforce-flow-tests --marketplace PranavNagrecha/salesforce-flow-tests
```

Or from a terminal (PowerShell on Windows, Terminal on Mac):

```
claude plugin install salesforce-flow-tests --marketplace PranavNagrecha/salesforce-flow-tests
```

Then restart Claude Code. Type `/salesforce-flow-tests` and the three commands appear.

This needs Claude Code 2.1.275 or later. On an older version, run these two inside Claude Code instead:

```
/plugin marketplace add PranavNagrecha/salesforce-flow-tests
/plugin install salesforce-flow-tests@salesforce-flow-tests
```

The plugin is text files only: no scripts, nothing to compile, no path that belongs to one operating system. The install was run and checked on a Mac. It has not yet been run on Windows by the author. If it fails there, please open an issue with the message you saw.

### Any Claude chat: copy a prompt

No install. Open the three files in [prompts/](prompts/), fill in the lines in square brackets, and paste.

## Use it

Open your Salesforce project in Claude Code and run the three steps in order.

**Step 1. Understand the flow.**

```
/salesforce-flow-tests:flow-paths force-app/main/default/flows/My_Flow.flow-meta.xml
```

You get a plain-words summary, a table with one row per path, and a list of questions about things the flow file does not show (custom metadata, formulas, picklist values). Read the table against what the flow is meant to do. Add the requirement after the file path if you have one.

**Step 2. Write the tests.**

```
/salesforce-flow-tests:flow-tests <your answers to the questions> <your sandbox alias>
```

Claude writes one test and stops. Deploy it and run it in a sandbox. When it passes, say so, and Claude writes the rest, plus one control test that is meant to fail.

**Step 3. Review.** In a new conversation:

```
/salesforce-flow-tests:flow-review force-app/main/default/flows/My_Flow.flow-meta.xml
```

It lists the flow's paths for itself, traces each test by hand, and reports paths with no test, tests that would pass on a broken flow, and record Ids that will not exist in the next org.

You can also skip the commands and just ask: "write Flow Tests for this flow". The plugin's skill loads on its own.

### Try it in one minute

This repository has a made-up flow you can practise on. Clone it, open the folder in Claude Code with the plugin installed, and run:

```
/salesforce-flow-tests:flow-paths examples/flows/Case_Set_Review_Flag.flow-meta.xml
```

In about a minute you get a plain-words summary, a table of the flow's ten paths, and a list of questions. Three of those questions are about the flow's design, not its tests: for example, whether a case that leaves Escalated and comes back should be flagged again. The unedited output of that run is in [examples/sample-output-flow-paths.md](examples/sample-output-flow-paths.md), and the matching example tests are in [examples/](examples/README.md).

## What is in here

| Path | What it is |
|---|---|
| [commands/](commands/) | The three slash commands |
| [skills/salesforce-flow-tests/SKILL.md](skills/salesforce-flow-tests/SKILL.md) | The skill: the method, what to do when a test goes red, what a Flow Test cannot prove, platform behaviours that break tests, the file shape, the CLI commands |
| [prompts/](prompts/) | The same three steps as copy-and-paste prompts for a chat window |
| [examples/](examples/README.md) | A sample flow and four example test files |
| [docs/how-the-prompts-were-tested.md](docs/how-the-prompts-were-tested.md) | How the prompts were tried and scored, and what is still unproven |

## Running the tests yourself

Flow Tests are metadata files. In a Salesforce DX project they live in `force-app/main/default/flowtests/` and end in `.flowtest-meta.xml` (lower-case t).

```bash
# deploy the test files
sf project deploy start --target-org ORG_ALIAS --source-dir force-app/main/default/flowtests

# run one flow's tests and wait for the result
sf flow run test --target-org ORG_ALIAS --class-names FLOW_API_NAME --synchronous --result-format json --output-dir test-results
```

Replace `ORG_ALIAS` with your sandbox alias and `FLOW_API_NAME` with the flow's API name. Always name the org. Use a sandbox.

## When a test goes red

The expected behaviour lives in the requirement, not in the flow and not in the test.

- If the flow no longer does what the requirement says, fix the flow.
- If the flow is right because the requirement changed, change the requirement first, with a name and a date. Then change the test.
- The person who edited the flow does not decide alone that the test is out of date.
- Never delete a test without naming what now proves that path.

Changing a test until it passes proves nothing.

## What the AI cannot do for you

- **It cannot see outside the flow file.** Custom metadata, formula definitions and picklist values are not in the XML. The prompts make Claude ask instead of guess. Answer the questions.
- **It cannot run anything in your org unless you let it.** Run the first test yourself before trusting the rest.
- **It can be wrong.** In the trials it sometimes left a path without a test or wrote a check in a form that had not been proven. That is what step 3 and the one-test-first rule are for.

## What a Flow Test cannot prove

- Screen flows and scheduled flows
- Paths that run asynchronously
- A flow that runs only on delete
- Anything that depends on who is running the flow (a permission, a bypass, a profile)
- A flow that should not start (a record that misses the start condition)
- A subflow with a collection input, where this was tried
- Fault paths, unless an input can cause the failure
- Bulk behaviour
- What the person sees on the screen

The skill file says what to use instead for each. Flow Tests show the logic does what the test says. They do not replace a person using the feature as the real user.

## How well does it work?

The prompts were tried over several rounds by a fresh AI agent that was given only the prompts and a real flow. Each result was graded with ten yes-or-no checks against an existing, passing test suite for that flow. Scores ran from 8 to 10 out of 10 over seven rounds. They did not climb steadily: a wording change that fixed one thing sometimes broke another, which is why every change is now re-run before it is published.

The rounds, the scores and the known weak spots are in [docs/how-the-prompts-were-tested.md](docs/how-the-prompts-were-tested.md).

## Limits

- The platform behaviours listed in the skill were observed in Salesforce sandboxes on API version 68 in late 2026. They are observations, not Salesforce documentation. Where one disagrees with your org, trust your org.
- The Winter '27 release introduced Test Mode, which calls a test a "test scenario". Your screens may not match older tutorials.
- The sample flow and example tests are made up. They have not been deployed to an org.
- This is a personal project. It is not a Salesforce product and is not affiliated with Salesforce, Anthropic or the author's employer.

## Official documentation

- [Automated Flow Testing (Salesforce Help)](https://help.salesforce.com/s/articleView?id=platform.automate_flow_test_record_data_cloud_triggered.htm&type=5)
- [Automated Flow Testing Limits and Limitations (Salesforce Help)](https://help.salesforce.com/s/articleView?id=platform.flow_considerations_feature_testing.htm&type=5)
- [FlowTest metadata type (Metadata API guide)](https://developer.salesforce.com/docs/atlas.en-us.api_meta.meta/api_meta/meta_flowtest.htm)
- [sf flow run test (Salesforce CLI reference)](https://developer.salesforce.com/docs/platform/salesforce-cli-reference/guide/cli_reference_flow_run_test.html)

## Found something wrong?

Open an issue with what you saw, your API version, and the exact error message if there was one. A behaviour that differs in your org is the most useful thing you can report.

## License

MIT. See [LICENSE](LICENSE).
