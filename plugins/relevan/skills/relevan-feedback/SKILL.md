---
name: relevan-feedback
description: "Send click, conversion, and llm_judgment feedback to Relevan after a search, so the ranking gets better. Use when you build search into an app, or when an agent acts on a search result."
---

# Relevan feedback

An agent that searches but never reports what happened teaches Relevan nothing. Feedback is what trains Relevan's ranking model and makes future results better. Your user pays for that improvement, so send feedback after every result you act on.

## The endpoint

`POST https://api.relevan.dev/v1-beta/{orgId}/indexes/{indexName}/feedback`

Send one of four feedback types:

- `search_judgment`: an agent judged the whole search. Send exactly one for each search.
- `click`: an end user picked this result.
- `llm_judgment`: an agent judged this result relevant, without a person to confirm it.
- `conversion`: a closing event, such as a purchase or a hire, confirmed this result.

## Add feedback to an app

When you build search into an app, add the feedback calls in the same change as the search call:

1. Keep `searchContext.queryId` from each search response. Pass it to the UI together with the results.
2. When an end user clicks a result, send a `click` event with the `queryId`, the result `id` as `objectId`, and the rank `position`.
3. When a closing event occurs, such as a purchase or a hire, send a `conversion` event with a short `label`.
4. Send feedback from a server route. Do not put the API key in browser code.

```ts
// Server route. The browser calls this route when the user clicks a result.
await fetch("https://api.relevan.dev/v1-beta/{orgId}/indexes/{indexName}/feedback", {
  method: "POST",
  headers: { "content-type": "application/json", "x-api-key": process.env.RELEVAN_API_KEY },
  body: JSON.stringify({ actionName: "click", queryId, objectId, position }),
});
```

## Feedback from an agent

If an agent uses the search results instead of a person, send one `search_judgment` for each search:

1. Keep `searchContext.queryId` from the search response.
2. Decide whether the results served the user. Send the judgment after this decision, not before.
3. Send exactly one `search_judgment` with that `queryId`. A second one for the same `queryId` gets 409.

The judgment takes these fields:

- `grade`: how well the whole search served the user. 0 not at all, 1 in part, 2 well.
- `results`: optional grades of single results, each with `objectId` and `grade`.
- `description`: what the user wanted, and why the results did or did not match it. At most 2000 characters. Do not copy personal data that the user did not need to share, such as contact details or account numbers. Relevan keeps this text with the same retention as query text.

```ts
await fetch("https://api.relevan.dev/v1-beta/{orgId}/indexes/{indexName}/feedback", {
  method: "POST",
  headers: { "content-type": "application/json", "x-api-key": process.env.RELEVAN_API_KEY },
  body: JSON.stringify({
    actionName: "search_judgment",
    queryId,
    grade: 1,
    results: [{ objectId: "doc-1", grade: 2 }],
    description: "The user wanted the 2024 pricing. Only the first result had it.",
  }),
});
```

With the Relevan MCP server, call the `send_search_feedback` tool instead. Pass `indexName` and the same fields, without `actionName`.

You can also send `llm_judgment` for a single result that the agent uses. The `FeedbackRequest` schema in the OpenAPI spec gives all the fields: https://api.relevan.dev/openapi

## Grade a result

Add an optional `grade` to any feedback type when you know how useful the result was:

- `0`: not useful.
- `1`: a little useful.
- `2`: very useful.

A grade needs `objectId`. Send the grade after you used the result, not when you only saw it. Send `source: "eval"` from a test run. Relevan stores each grade with the feedback event. A grade does not change the ranking yet.

```ts
await fetch("https://api.relevan.dev/v1-beta/{orgId}/indexes/{indexName}/feedback", {
  method: "POST",
  headers: { "content-type": "application/json", "x-api-key": process.env.RELEVAN_API_KEY },
  body: JSON.stringify({ actionName: "llm_judgment", queryId, objectId, grade: 2 }),
});
```

## Other Relevan documents

- [Relevan overview](https://relevan.dev/llms.txt): What Relevan is, who it suits, and what it costs.
- [Documentation index](https://docs.relevan.dev/llms.txt): Each page of the Relevan documentation: guides, releases, and resources.
- [API index](https://api.relevan.dev/llms.txt): The API capabilities, the rate limits, and the search quota.
- [OpenAPI spec](https://api.relevan.dev/openapi): Each endpoint, with its request and response schemas.
- [relevan-setup](https://api.relevan.dev/skills/relevan-setup/SKILL.md): Set up Relevan search from the start: authenticate, create an index, load documents, run the first search, and send the first feedback. Use when a user asks you to add Relevan search to a product.
- [relevan-ranking-signals](https://api.relevan.dev/skills/relevan-ranking-signals/SKILL.md): Find the data that tells Relevan which result is the best one, such as prices, sales, or activity, and add it to each document. Use before you design a Relevan mapping.
- [relevan-mapping](https://api.relevan.dev/skills/relevan-mapping/SKILL.md): Draft and deploy a RelevanMapping from a sample of documents. Use when a Relevan index has no mapping, or when the documents get new fields.
