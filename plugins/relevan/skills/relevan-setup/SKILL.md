---
name: relevan-setup
description: "Set up Relevan search from the start: authenticate, create an index, load documents, run the first search, and send the first feedback. Use when a user asks you to add Relevan search to a product."
---

# Relevan setup

Do these steps to set up Relevan search from the start: authenticate, create an index, load documents, search, and send the first feedback.

## 1. Authenticate

First check whether the `relevan` MCP server is connected, for example through the Relevan agent plugin. If it is, you need no API key for steps 2 to 7. Use its tools in place of the REST calls: `create_index`, `create_mapping`, `upsert_documents`, `describe_index`, `search`, and `send_search_feedback`. The create tools need the `write` scope, which only an organization owner can grant. Keep each `upsert_documents` call under 8 KB. For a larger load, use the REST bulk call in step 5 with an API key.

The app code that you write still needs its own API key on its server. To get one, do these steps:

1. Ask the user for a write API key. The user creates the key in the console: https://console.relevan.dev. A read key gets 403 on the calls that create an index or upload documents.
2. Keep the key in a server environment variable, such as `RELEVAN_API_KEY`. Do not put the key in browser code.
3. Send the key in the `x-api-key` header on each request.
4. Call `GET https://api.relevan.dev/v1/organizations`. An org-scoped API key resolves to one organization. Use its `id` or `slug` as `{orgId}` in each route below.

## 2. Create an index

Call `POST https://api.relevan.dev/v1/{orgId}/indexes` with a body such as `{ "name": "products" }`.

Right after you create the index, give the user the link to the index list page in the console: `https://console.relevan.dev/{orgId}/indexes`. Give the link in your next message, before you deploy a mapping or upload documents. Do not wait and give the link only in a final summary. Tell the user to open the link to monitor the progress. The `consoleUrl` field of the response gives the full link.

## 3. Find the ranking signals

Follow this skill before you design the mapping: https://api.relevan.dev/skills/relevan-ranking-signals/SKILL.md

## 4. Deploy the mapping

Follow this skill to draft and deploy the mapping: https://api.relevan.dev/skills/relevan-mapping/SKILL.md

## 5. Upload the documents

Call `POST https://api.relevan.dev/v1/{orgId}/indexes/{indexName}/documents/_bulk` with a maximum of 1000 documents in each request:

```json
{ "documents": [{ "id": "sku-1", "content": { "title": "Trail Runner 2" } }] }
```

If one document does not agree with the mapping, Relevan rejects the full request and writes no documents. The response names each document with a problem.

## 6. Search

Call `GET https://api.relevan.dev/v1/{orgId}/indexes/{indexName}/skill` once. It tells you how to search this index. Then call `POST https://api.relevan.dev/v1/{orgId}/indexes/{indexName}/search` with a body such as `{ "query": "waterproof trail shoe" }`.

Keep `searchContext.queryId` from the response. The feedback call needs it. Read the `X-Search-Quota-Remaining` header, and plan your work so that you do not use all the searches.

## 7. Send the first feedback

Follow this skill to send feedback for each result that you act on: https://api.relevan.dev/skills/relevan-feedback/SKILL.md

## Before you finish: the integration checklist

Before you tell the user that the integration is complete, show the user this checklist. Mark each item as done or not done. If an item is not done, tell the user why. Do not mark an item as done if the user chose to skip it.

- [ ] The mapping is deployed, and the documents are uploaded.
- [ ] You asked the user which facts make a result good, and the answers are in the mapping.
- [ ] Each ranking signal is a field on every document.
- [ ] A job or a write path keeps the ranking signals current.
- [ ] The UI receives the `queryId` from each search response.
- [ ] A click on a result sends a `click` event through a server route.
- [ ] Each closing event sends a `conversion` event, if the product has one.
- [ ] The UI shows filters for the filterable fields that users need.

## Other Relevan documents

- [Relevan overview](https://relevan.dev/llms.txt): What Relevan is, who it suits, and what it costs.
- [Documentation index](https://docs.relevan.dev/llms.txt): Each page of the Relevan documentation: guides, releases, and resources.
- [API index](https://api.relevan.dev/llms.txt): The API capabilities, the rate limits, and the search quota.
- [OpenAPI spec](https://api.relevan.dev/openapi): Each endpoint, with its request and response schemas.
- [relevan-ranking-signals](https://api.relevan.dev/skills/relevan-ranking-signals/SKILL.md): Find the data that tells Relevan which result is the best one, such as prices, sales, or activity, and add it to each document. Use before you design a Relevan mapping.
- [relevan-mapping](https://api.relevan.dev/skills/relevan-mapping/SKILL.md): Draft and deploy a RelevanMapping from a sample of documents. Use when a Relevan index has no mapping, or when the documents get new fields.
- [relevan-feedback](https://api.relevan.dev/skills/relevan-feedback/SKILL.md): Send click, conversion, and llm_judgment feedback to Relevan after a search, so the ranking gets better. Use when you build search into an app, or when an agent acts on a search result.
