---
name: relevan-mapping
description: "Draft and deploy a RelevanMapping from a sample of documents. Use when a Relevan index has no mapping, or when the documents get new fields."
---

# Relevan mapping

A mapping tells Relevan how to read the documents in an index: which fields to search, filter, and sort on. Relevan rejects document ingestion until the index has a mapping.

Before you draft the mapping, find the ranking signals: https://api.relevan.dev/skills/relevan-ranking-signals/SKILL.md

After the mapping is deployed, fetch the skill doc of the index: `GET https://api.relevan.dev/v1/{orgId}/indexes/{indexName}/skill`. It lists the fields and tells you how to search the index.

## How to create a mapping

Do these steps before you ingest a document:

1. Collect a sample of 100 to 1000 documents from different sources, types, and dates. Include documents that have optional fields.
2. Find the search signals that your sample does not show. Follow "Find the signals outside your sample" below.
3. Draft the mapping yourself. For each field in your documents, choose `use` values and, where needed, `kind` and `language`, following "Mapping shape" and "How to choose `use` values" below.
4. Deploy the mapping: `POST /v1/{orgId}/indexes/{indexName}/mappings` with the mapping as the body. If Relevan rejects it, the response names the fields with a problem. Fix them and deploy again. A later try can still find a new problem in another field.

You already know what your users search and filter on, so draft the mapping yourself.

## Find the signals outside your sample

A signal is a value that helps decide which result comes first, or which results a user can narrow to. Many signals are not in the main document. They live in other tables, in a side table of attributes, or in an events log. A signal that is not in the document does not exist to search. Read the source schema, and add each signal below to the document before you ingest it. Names differ between systems, so read the meaning, not the name.

| Signal       | Where it hides                                                                        | Field to send                                                                                                                       |
| ------------ | ------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------- |
| Recency      | A created, published, or updated timestamp                                            | `date`, `filterable` and `sortable`                                                                                                              |
| Availability | A status, active, published, or in-stock flag, an inventory table, a deleted marker   | `boolean`, `filterable`. Or leave out the rows that are not available before you ingest                                        |
| Popularity   | An events table (views, clicks, orders, downloads, applications), or a cached counter | `number`, `filterable` and `sortable`. Count the events in a window, such as `views_30d`. The main table does not hold this count             |
| Quality      | A reviews or ratings table, helpful votes, an editor score                            | `number`, `filterable` and `sortable`. Send the average and the count. An average of one vote is not reliable                                   |
| Business     | A price, a sale price, or a featured, promoted, or sponsored flag                     | `number` or `boolean`, `filterable` and `sortable`                                                                                            |
| Identity     | A SKU, ISBN, slug, external ID, or model number                                       | `id`, `filterable` and `searchable`, `language: none`. A user who pastes a code expects one exact result                        |
| Facets       | A category, brand, type, color, size, tag, author, or department, often in a join     | `id` or `array`, `filterable`. Send the label, not the id. Add `searchable` when a user types it, and set `kind: text` and `language: none` |
| Place        | A city, state, or country column, as a name or a code, often in an address join      | `id`, `filterable`, and `entityType` (`city`, `state`, `country`, or `postal_code`). Send one place in each field, such as `city` and `state` |

Follow these rules when you build the document:

- Join a side table into the document. An attribute table with one row for each key and value becomes one field for each key that a user filters on.
- Count the rows of a related table into a number field. Send the count, not the rows.
- Give each variant its own field when one document holds many. A document with one title for each locale has `title_en` and `title_de`, not one `title`.
- Send a name or a label, not the id of a related row.
- Join a latitude and a longitude from two columns into one `geo_point` field. Leave the field out of a document when a value is missing or out of range. Relevan rejects the full request when one point is invalid.
- Leave out a private value, such as an email address, a cost, or an internal note.
- Do not invent a signal. A `popularity` field with the same value in every document adds nothing. If the source does not count views or clicks, say so in your report to the user.

## Mapping shape

A mapping is a JSON object. Its `fields` object has one entry for each field name in your documents. Each entry has these properties:

- `use` (required): a non-empty array of query capabilities for the field.
- `kind` (optional): the data type. One of `text`, `id`, `number`, `date`, `boolean`, `object`, `array`, `geo_point`. If you omit it, Relevan uses `text` for a field with `searchable`, `autocompletable`, or `highlightable`, and `id` for other fields.
  - A field with `kind: geo_point` holds one point. Send it as `{ "lat": number, "lon": number }`. `lat` must be from -90 to 90 and `lon` from -180 to 180.
- `language` (optional): `english` or `none`. The default is `english`. Use `none` for codes, names, or a language that is not in this list.
- `entityType` (optional): the type of place that the field holds. One of `city`, `state`, `country`, `postal_code`. The field must be `filterable`, with kind `text`, `id`, or `array`. Many fields can have the same value, such as `state` on a home state and a work state.
  - A query that names a place, such as "nurse jobs in Virginia", then ranks or filters the documents of that place, whether the field holds `Virginia` or `VA`.
  - Set it only on a field that holds a place and nothing else. Do not set it on a free-text field that mentions places, such as a description.

Relevan does not map the contents of an object. A field with `kind: object` accepts an object, but search v1 cannot search or filter its contents. To search them, flatten the object into top-level fields before ingest. Relevan does not accept an array of objects. Remove it or flatten it before ingest.

Use each field's name exactly as it appears in your documents. Do not rename a field to a clearer name, such as changing `order_dt` to `order_date` — Relevan rejects a document that sends a field not declared in the mapping. If a field's name is unclear, add a second field with a clearer name next to the original, such as `order_date` next to `order_dt`. Keep the original field as the primary field; do not remove it.

## How to choose `use` values

- `searchable`: the full-text query matches words in this field.
- `filterable`: a request can filter on an exact value of this field. A `number` or `date` field also accepts a range filter.
- `sortable`: a request can order results by this field. Use it on a `number` or `date` field.
- `aggregatable`: search v1 has no facets. This value has no effect until facets ship.
- `autocompletable`: the field gets an autocomplete subfield and becomes the result title. Put it on one short label, such as a title or a name.
- `highlightable`: search v1 requests no highlights. The field can be the result title when no field is autocompletable.

A field must have one or more of `searchable`, `filterable`, `sortable`, `aggregatable`, or `autocompletable`. Relevan rejects a field that has only `highlightable`.

## Declare every field that a document can contain

Relevan rejects a document that contains a field that is not in `fields`. This rule also applies to a field that nobody searches or filters on. Add every field from your sample to `fields`, except a field with an unsupported shape from "Mapping shape" above. If your other documents have more fields, add them before you deploy.

## Worked example

This example is not a template. Do not copy its field names or its document shape.

A product document:

```json
{
  "title": "Trail Runner 2 Waterproof Shoe",
  "description": "A light trail shoe with a waterproof upper and a grippy outsole.",
  "brand": "Northpeak",
  "category": "footwear",
  "price": 129.99,
  "in_stock": true,
  "release_date": "2026-03-01",
  "tags": [
    "running",
    "waterproof",
    "trail"
  ],
  "sku": "NP-TR2-WP-42",
  "weight_kg": 0.62,
  "material": "nylon"
}
```

A mapping for this document:

```json
{
  "fields": {
    "title": {
      "use": [
        "searchable",
        "autocompletable",
        "highlightable"
      ],
      "kind": "text"
    },
    "description": {
      "use": [
        "searchable",
        "highlightable"
      ],
      "kind": "text"
    },
    "brand": {
      "use": [
        "filterable",
        "aggregatable"
      ]
    },
    "category": {
      "use": [
        "filterable",
        "aggregatable"
      ]
    },
    "price": {
      "use": [
        "filterable",
        "sortable"
      ],
      "kind": "number"
    },
    "in_stock": {
      "use": [
        "filterable"
      ],
      "kind": "boolean"
    },
    "release_date": {
      "use": [
        "sortable"
      ],
      "kind": "date"
    },
    "tags": {
      "use": [
        "filterable",
        "aggregatable"
      ],
      "kind": "array"
    },
    "sku": {
      "use": [
        "filterable"
      ]
    },
    "weight_kg": {
      "use": [
        "filterable",
        "sortable"
      ],
      "kind": "number"
    },
    "material": {
      "use": [
        "filterable"
      ]
    }
  }
}
```

## Other Relevan documents

- [Relevan overview](https://relevan.dev/llms.txt): What Relevan is, who it suits, and what it costs.
- [Documentation index](https://docs.relevan.dev/llms.txt): Each page of the Relevan documentation: guides, releases, and resources.
- [API index](https://api.relevan.dev/llms.txt): The API capabilities, the rate limits, and the search quota.
- [OpenAPI spec](https://api.relevan.dev/openapi): Each endpoint, with its request and response schemas.
- [relevan-setup](https://api.relevan.dev/skills/relevan-setup/SKILL.md): Set up Relevan search from the start: authenticate, create an index, load documents, run the first search, and send the first feedback. Use when a user asks you to add Relevan search to a product.
- [relevan-ranking-signals](https://api.relevan.dev/skills/relevan-ranking-signals/SKILL.md): Find the data that tells Relevan which result is the best one, such as prices, sales, or activity, and add it to each document. Use before you design a Relevan mapping.
- [relevan-feedback](https://api.relevan.dev/skills/relevan-feedback/SKILL.md): Send click, conversion, and llm_judgment feedback to Relevan after a search, so the ranking gets better. Use when you build search into an app, or when an agent acts on a search result.
