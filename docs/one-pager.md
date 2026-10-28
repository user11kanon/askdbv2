# AskDB: one-pager

## What it is
A web app where non-technical business staff ask questions of a database in plain English and get a trustworthy answer. It only looks data up and never changes it.

## Who it's for
Sales, ops, finance and support staff who have never written a query.

## What they can ask
- **Lookups:** "Show me all orders for Acme."
- **Totals and counts:** "How many customers signed up last month?"
- **Trends:** "How did revenue change quarter over quarter?"
- **Rankings and breakdowns:** "Top 10 products by region."

## What they see
1. **Answer:** one or two plain sentences.
2. **The data:** the rows behind the answer, with CSV download.
3. **How I got this:** a plain-language explanation of what was looked up, with the query available behind a "Show query" toggle.

## What must never happen
- **No data changes.** Enforced in three layers: a read-only database connection, a check that every query is a single SELECT, and a read-only transaction.
- **No confident wrong answers.** If the question is ambiguous or can't be answered from the data, the app says so and suggests a rephrase. Numbers in the written answer must appear in the returned rows. Any interpretation, such as a date range, is stated.

## Out of scope for v1
Editing data, dashboards and saved reports, multiple databases, follow-up questions (each question stands alone), charts.