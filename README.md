# Salesforce Flow Tests with Claude

A Claude skill and three prompts for writing, running and reviewing Salesforce Flow Tests.

A Flow Test is a saved check on a flow: a starting record, and a list of things that must be true when the flow finishes. This repository holds what one team learned from writing 175 of them with AI agents: the method, the platform facts that broke real tests (with the exact error messages), and prompts that were tried on real flows and scored.

## Who this is for

- Salesforce admins and developers who build record-triggered or autolaunched flows and want them tested.
- Anyone using Claude to help with Salesforce work.
- AI agents: the skill file is written to be read by one.

The prompts were tried with an agent that can read and write files (Claude Code). In a plain Claude chat the same steps apply, pasting the flow's XML in, but that path has not been tried.

## What is in here

| Path | What it is |
|---|---|
| [skills/salesforce-flow-tests/SKILL.md](skills/salesforce-flow-tests/SKILL.md) | The skill: the method, what a Flow Test cannot prove, the platform facts, the file shape, the commands |
| [prompts/01-understand-the-flow.md](prompts/01-understand-the-flow.md) | Prompt 1: a plain-words summary and a table of every path through a flow |
| [prompts/02-write-the-tests.md](prompts/02-write-the-tests.md) | Prompt 2: one Flow Test file per path |
| [prompts/03-review-the-tests.md](prompts/03-review-the-tests.md) | Prompt 3: an independent review of a set of Flow Tests |
| [examples/](examples/README.md) | Four example test files showing the shapes that work |
| [docs/how-the-prompts-were-tested.md](docs/how-the-prompts-were-tested.md) | The trials, the scores, and what is still unproven |

## Quick start

### With the prompts

1. Open [Prompt 1](prompts/01-understand-the-flow.md). Fill in the flow's API name, give it the flow's XML, and send it.
2. Read the path table it gives you against your requirement. Answer its questions.
3. Send [Prompt 2](prompts/02-write-the-tests.md) with your answers. It writes one test and stops.
4. Deploy that one test and run it. When it passes, say so, and Claude writes the rest.
5. Open a new conversation and send [Prompt 3](prompts/03-review-the-tests.md) with the flow and the tests.

### With the skill, in Claude Code

Copy the skill folder into your Salesforce project, or into your home folder to use it everywhere.

Mac or Linux:

```bash
# this project only
mkdir -p .claude/skills
cp -R skills/salesforce-flow-tests .claude/skills/

# every project on this machine
mkdir -p ~/.claude/skills
cp -R skills/salesforce-flow-tests ~/.claude/skills/
```

Windows (PowerShell):

```powershell
# this project only
New-Item -ItemType Directory -Force .claude\skills | Out-Null
Copy-Item -Recurse skills\salesforce-flow-tests .claude\skills\

# every project on this machine
New-Item -ItemType Directory -Force $HOME\.claude\skills | Out-Null
Copy-Item -Recurse skills\salesforce-flow-tests $HOME\.claude\skills\
```

Then ask Claude to "write Flow Tests for" a flow in your project. The Windows commands have not been run by the author; if one fails, please open an issue.

## The method in eight steps

1. Get the flow's API name from the person or the file name. It is not inside the XML.
2. Write a path table before any test: every way into the flow, every Decision outcome including defaults, every list value, every Get Records found and not found.
3. Ask about what the XML does not show (custom metadata, formulas, picklist values). Do not guess.
4. Have a person read the table against the requirement.
5. Write one test, deploy it, run it. Then write the rest.
6. Add a control test that must fail, run it once, delete it.
7. Run the tests, then count how many ran.
8. Review independently, from the flow XML and the tests alone.

## Running the tests

Flow Tests are metadata files. In a Salesforce DX project they live in `force-app/main/default/flowtests/` and end in `.flowtest-meta.xml` (lower-case t).

```bash
# deploy the test files
sf project deploy start --target-org ORG_ALIAS --source-dir force-app/main/default/flowtests

# run one flow's tests and wait for the result
sf flow run test --target-org ORG_ALIAS --class-names FLOW_API_NAME --synchronous --result-format json --output-dir test-results
```

Replace `ORG_ALIAS` with your sandbox alias and `FLOW_API_NAME` with the flow's API name. Always name the org. Use a sandbox.

## What a Flow Test cannot prove

- Screen flows and scheduled flows
- Paths that run asynchronously
- A flow that runs only on delete
- Anything that depends on who is running the flow (a permission, a bypass, a profile)
- A flow that should not start (a record that misses the start condition)
- A subflow with a collection input, in the orgs where this was tried
- Fault paths, unless an input can cause the failure
- Bulk behaviour
- What the person sees on the screen

The skill file says what to use instead for each. Flow Tests show the logic does what the test says. They do not replace a person using the feature as the real user.

## How well do the prompts work?

They were tried six times by a fresh AI agent (Claude Sonnet) that was given only the prompts and a real flow, and graded against tests a team already runs for that flow. Ten yes-or-no checks each time.

| Round | Record-triggered flow | Subflow | What happened |
|---|---|---|---|
| 1 | 8 of 10 | | The files would probably not have run: wrong flow name, wrong form for outcomes |
| 2 | 8 of 10 | | Round 1 faults fixed; too few fields checked |
| 3 | 9 of 10 | | Round 2 faults fixed |
| 4 | | 9 of 10 | Same prompts as round 3, on a different kind of flow |
| 5 | 8 of 10 | 8 of 10 | A new sentence made the model drop tests it should have written |
| 6 | 9 of 10 | 8 of 10 | That sentence reworded; the subflow still lost its error-path test |

The scores moved between 8 and 9. They did not climb steadily: a change that fixed one thing sometimes broke another.

The trial files were traced by hand and compared with accepted tests. They were not deployed to an org. Details are in [docs/how-the-prompts-were-tested.md](docs/how-the-prompts-were-tested.md).

## Limits of this repository

- The platform facts were observed in Salesforce sandboxes on API version 68 in late 2026. They are observations, not Salesforce documentation. Where a fact here disagrees with your org, trust your org.
- The Winter '27 release introduced Test Mode, which calls a test a "test scenario". Your screens may not match older tutorials.
- The example test files show working shapes. They belong to made-up flows and will not deploy as they are.
- This is a personal project. It is not a Salesforce product and is not affiliated with Salesforce, Anthropic or the author's employer.

## Official documentation

- [Automated Flow Testing (Salesforce Help)](https://help.salesforce.com/s/articleView?id=platform.automate_flow_test_record_data_cloud_triggered.htm&type=5)
- [Automated Flow Testing Limits and Limitations (Salesforce Help)](https://help.salesforce.com/s/articleView?id=platform.flow_considerations_feature_testing.htm&type=5)
- [FlowTest metadata type (Metadata API guide)](https://developer.salesforce.com/docs/atlas.en-us.api_meta.meta/api_meta/meta_flowtest.htm)
- [sf flow run test (Salesforce CLI reference)](https://developer.salesforce.com/docs/platform/salesforce-cli-reference/guide/cli_reference_flow_run_test.html)

## Found something wrong?

Open an issue with the fact, what you saw instead, your API version, and the exact error message if there was one. Corrections from other orgs are the most useful contribution.

## License

MIT. See [LICENSE](LICENSE).
