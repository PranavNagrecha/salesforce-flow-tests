# Prompt 3: review the tests

An independent second look at a set of Flow Tests, whoever wrote them. It lists the flow's paths itself before reading the tests, then traces each test by hand.

Use a fresh conversation that has not seen the first two prompts. Give it only the flow XML and the test files.

```text
Review someone else's Flow Tests for a Salesforce flow as an independent second pair of eyes. You get only the flow XML and the tests, on purpose. Before reading the tests, list the flow's paths from the XML yourself (every way into the flow; every Decision outcome including defaults; in, out and blank for each list comparison; found and not found; loops with 0, 1 and many; fault paths), so the author's view does not shape yours.

Flow XML: [PASTE OR GIVE THE PATH]
Flow Tests: [PASTE ALL THE .flowtest-meta.xml FILES OR GIVE THE FOLDER]

Then trace each test by hand: walk its input through the flow element by element and note the path it really takes and the values at the end. Compare that with its assertions and its description.

Platform facts to judge against, from real runs: Salesforce calculates formula fields itself from the fields in the test record, on the before record too. The test record is never saved. In a create test it has no Id; in an update test the flow sees an existing record, so a check such as "$Record.Id is null" takes the update route. Every test runs as the person who starts it. A record that misses the start condition makes the test fail, not skip. An outcome the flow did not evaluate, because an earlier outcome in the same Decision matched, is not false.

Report, quoting the file:
1. A path on your list with no test.
2. A test whose input does not reach the path it claims.
3. A test that would pass on a different route through the flow: for each test, try to name another route where every assertion still holds. If you can, the test proves nothing about its own route.
4. A "nothing changes" test with no twin where something must change.
5. A field asserted that no path ever sets, so the assertion only repeats the input. (A field other paths set, asserted unchanged, is fine; say so.)
6. An Id or a formula field in an input record (either ends the test in Error with no message), or a roll-up, system or relationship field (these should be left out too).
7. A hard-coded record Id, which will not exist in another org.
8. An outcome asserted EqualTo false that the flow may never reach (NotEqualTo true is the safe form); an update test missing the before or after record; a test of what a Flow Test cannot prove (who is running, a record outside the start condition, an asynchronous path).
9. Two tests that take the same route and differ only in a field the flow never reads.

Give me: your path list with the tests covering each; a findings table (file | problem number | what is wrong | the fix, with corrected XML where it helps); the missing tests, one line each; the tests that are fine; and, as questions, what you could not judge without the org (custom metadata records, formula definitions, picklist values, who holds which permission).
```
