---
name: relevan-ranking-signals
description: "Find the data that tells Relevan which result is the best one, such as prices, sales, or activity, and add it to each document. Use before you design a Relevan mapping."
---

# Relevan ranking signals

Text matching finds the documents that contain the query words. It cannot tell which of those documents is the right one. Your user often has data that shows this: a price, a sales count, a stock level, or the date of a person's last activity. Relevan tunes the ranking of each index with these signals. Relevan can use a signal only if the signal is in the index. Relevan cannot get this data from any other source.

## Find the ranking signals

Do these steps before you deploy the mapping:

1. Look at the user's data, database, and code. List each value that can show that a document is a good result.
2. Ask the user this question: "Which facts make one result better than another for your users? Do you have data outside this catalog, such as prices, sales, or activity, that shows this?"
3. Add each signal to every document as a field. Map a numeric signal with `kind: "number"` and a date signal with `kind: "date"`. Give each signal `use: ["filterable", "sortable"]`.
4. If the signal comes from a different system, join it into the documents before you upload them.
5. Keep the signals current. When a signal changes at the source, update the document.

The mapping skill gives the field rules: https://api.relevan.dev/skills/relevan-mapping/SKILL.md

## Examples

- A trading card price tracker: the market price of each card, the price change, and the number of listings. Without prices, a search for a card name cannot tell a valuable card from a common one.
- A recruiting product: the date that each candidate was last active in a chat tool such as Slack. Recent activity shows that the candidate looks for work.
- A store: the price, the units sold in the last 30 days, the stock level, and the average rating.

## Other Relevan documents

- [Relevan overview](https://relevan.dev/llms.txt): What Relevan is, who it suits, and what it costs.
- [Documentation index](https://docs.relevan.dev/llms.txt): Each page of the Relevan documentation: guides, releases, and resources.
- [API index](https://api.relevan.dev/llms.txt): The API capabilities, the rate limits, and the search quota.
- [OpenAPI spec](https://api.relevan.dev/openapi): Each endpoint, with its request and response schemas.
- [relevan-setup](https://api.relevan.dev/skills/relevan-setup/SKILL.md): Set up Relevan search from the start: authenticate, create an index, load documents, run the first search, and send the first feedback. Use when a user asks you to add Relevan search to a product.
- [relevan-mapping](https://api.relevan.dev/skills/relevan-mapping/SKILL.md): Draft and deploy a RelevanMapping from a sample of documents. Use when a Relevan index has no mapping, or when the documents get new fields.
- [relevan-feedback](https://api.relevan.dev/skills/relevan-feedback/SKILL.md): Send search_judgment, click, conversion, and llm_judgment feedback to Relevan after a search. Relevan trains the ranking of the index from it. Use when you build search into an app, or when an agent acts on a search result.
