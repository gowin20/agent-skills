# Optimizing and testing a skill description

The `description` is the main thing an agent uses to decide whether to load a skill. Use this guide to check that a description triggers when it should and stays quiet when it should not.

## Writing rules

- Use imperative phrasing ("Use this skill when...").
- Describe the user's **intent**, not how the skill works inside.
- List the contexts where the skill applies, including ones where the user does not name the domain.
- Keep it to a few sentences or a short paragraph. The hard limit is **1024 characters**.
- Agents usually use skills only for tasks they cannot easily do alone. A trivial request such as "read this PDF" may not trigger a PDF skill even when the description matches. Write test queries that need the skill.

## Build an eval set (~20 queries)

Store the queries as JSON:

```json
[
  { "query": "I've got sales_q3.csv with columns region, rev, cost — can you show margin by region in a chart?", "should_trigger": true },
  { "query": "I need to update the formulas in my Excel budget spreadsheet", "should_trigger": false }
]
```

- **8-10 should-trigger queries.** Vary the phrasing, how explicit the request is, how much detail it gives, and how complex it is. The most useful ones are those where the link to the skill is **not obvious**.
- **8-10 should-not-trigger queries.** Use **near misses** that share keywords with the skill but need something different. A query like "Write a fibonacci function" tests nothing.
- Make the queries realistic: include file paths, personal context, column names, casual wording, and typos.

## Measure

1. Run each query **about 3 times**, because model behaviour varies between runs.
2. Compute the trigger rate for each query.
   - A should-trigger query **passes** if its rate is above 0.5.
   - A should-not-trigger query **passes** if its rate is below 0.5.
3. To detect activation, check whether the agent called its skill tool or read the skill's `SKILL.md`. With Claude Code, for example, run `claude -p "<query>" --output-format stream-json` and look for a `Skill` tool call using `jq`. Other clients need their own detection logic.

## Avoid overfitting

- Split the queries **about 60% train and 40% validation**, keeping the same mix of positives and negatives in each. Keep the split fixed across iterations.
- Revise the description using **train failures only**.
- Do not paste keywords from the failed queries into the description. Find the general concept they share.
  - Should-trigger failures mean the description is **too narrow**. Broaden the stated intent.
  - Should-not-trigger failures mean it is **too broad**. Add what the skill does **not** do.
- Run about **5 iterations**. Choose the version with the best **validation** pass rate, which is not always the last one.
- Descriptions tend to grow while you optimize, so recheck the 1024-character limit.

## Apply

1. Update `description` in `SKILL.md`.
2. Confirm it is 1024 characters or fewer and that the YAML still parses.
3. Check it with 5-10 **fresh** queries that were not in the eval set.

The [`skill-creator` skill](https://github.com/anthropics/skills) can automate this loop.
