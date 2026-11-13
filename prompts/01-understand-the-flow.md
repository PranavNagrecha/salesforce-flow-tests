# Prompt 1: understand the flow

Gives you a plain-words summary of the flow and a table of every path through it. Read the table against your requirement before any test is written. A path missing from the table will have no test.

Fill the three lines in square brackets. Use it in Claude Code with the path to the flow file, or paste the flow's XML into a chat.

```text
I'm a Salesforce admin. Help me understand this flow path by path, so I can check it against the requirement and so we can later write one Flow Test per path. Don't write any tests yet.

Flow API name: [THE FILE NAME WITHOUT .flow-meta.xml, e.g. Case_Set_Priority]
Flow XML: [PASTE THE .flow-meta.xml HERE, OR GIVE ITS PATH IF YOU CAN READ FILES]
What it is supposed to do: [THE REQUIREMENT IN A FEW SENTENCES, OR "not available"]

The API name is not inside the XML, so use the one I gave and never build one from the label.

Work only from the XML. When a path depends on something outside the file (custom metadata records, formula field definitions, picklist values, values on a related record), use a named placeholder such as <a status that is in the list> and ask me about it at the end, one numbered question per placeholder. A guessed value gives a test that passes for the wrong reason.

1. In plain words, five to eight sentences I can hold against the requirement: the object or the inputs, when the flow starts, before or after the save, and every field it changes or record it creates.

2. A path table, one row per path. Build the rows in this order so none are missed:
- The start. For a record-triggered flow, list the terms of the start condition (new record, and each "this field changed" term). Each term gets a row where it alone starts the flow, because each is a separate way in. Add one row where no term is met. For an autolaunched flow or subflow, list its input variables instead.
- Every Decision outcome, including each default. A default outcome has no API name, so call it by its label. Where an outcome has several conditions, add a row for each condition being the one that decides, and say in the questions if you doubt such a record can exist.
- Overrides. When one condition beats the others (a lock, an exclusion), pair it with each way in that its Decision reads. If the Decision does not read the way in, one row is enough.
- Every comparison against a list of values: one row per listed value when there are six or fewer, otherwise one inside, one outside and one blank (the blank on a create as well as an update, when the flow runs on both). Add a near miss, a value that looks like a listed one but is not.
- Each Get Records found and not found; each loop with 0, 1 and many items; each fault path.
Write "none" for any of these the flow lacks, so I know you looked. Two rows that differ only in a field the flow never reads are one row.

Columns: id | elements in order, with the outcome at each Decision | input (create: the values; update: before and after the save; subflow: each input variable; only what this path reads) | what must be true at the end: each field the flow can set, with its value or "unchanged", each record created or not, and the outcomes taken | twin | Flow Test?

Every row where nothing changes or nothing is created needs a twin: the nearest input where something must. A "nothing happens" test also passes when the flow is broken. Other rows need no twin.

In the last column write yes, or no with the reason and another way to prove it. A Flow Test cannot cover:
- a screen flow, a Run Asynchronously path or a delete trigger
- anything that depends on who is running the flow (a $Permission check, a bypass, a profile). Every test runs as the one person who starts it, so those rows are proven by logging in as that person. For each yes row, say whether it would still pass if the runner held that permission.
- a record that misses the start condition. In practice that test fails; it does not show a skip.
A fault path is a yes when an input can cause the failure (text that is not a record Id passed to a Get Records, for example). Use no other reason for a no without telling me why.

3. Questions, numbered, each naming the rows it affects: what the XML could not settle, and anything in the flow's labels or descriptions that disagrees with what its logic does. If I said the requirement is not available, don't ask for it.
```
