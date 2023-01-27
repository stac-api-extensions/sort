# STAC API - Sort Extension Specification

- [STAC API - Sort Extension Specification](#stac-api---sort-extension-specification)
  - [Overview](#overview)
  - [HTTP GET (or POST Form)](#http-get-or-post-form)
  - [HTTP POST JSON Entity](#http-post-json-entity)

## Overview

- **Title:** Sort
- **OpenAPI specification:** [openapi.yaml](openapi.yaml)
- **Conformance Classes:**
  - **STAC API - Item Search** binding: <https://api.stacspec.org/v1.0.0-rc.2/item-search#sort>
  - **STAC API - Features** binding: <https://api.stacspec.org/v1.0.0-rc.2/ogcapi-features#sort>
- **Scope:** STAC API - Features, STAC API - Item Search
- **[Extension Maturity Classification](https://github.com/radiantearth/stac-api-spec/tree/main/README.md#maturity-classification):** Candidate
- **Dependencies:**
  - [STAC API - Item Search](https://github.com/radiantearth/stac-api-spec/tree/v1.0.0-rc.2/item-search)
  - [STAC API - Features](https://github.com/radiantearth/stac-api-spec/tree/v1.0.0-rc.2/item-search)
- **Owner**: @philvarner

This specification defines a new parameter, `sortby`, that allows the user to define the fields by which
to sort results.
Only string, numeric, and datetime attributes of Item (`id` and `collection` only) or Item Properties (any attributes)
may be used to sort results.  

It is not required that implementations support sorting over all attributes, but
implementations should return a 400 Bad Request status code when attempting to sort over a field name that does
not support sorting.
This specification does not yet require the implementation of an "-ables" endpoint (like CQL2 does for queryables)
that defines the names of the
fields that can be sorted over, so implementations must provide this out-of-band. Implementers may choose to require
fields in Item Properties to be prefixed with `properties.` or not, or support use of both the prefixed and non-prefixed
name, e.g., `properties.datetime` or `datetime`.

Sort behavior may be bound to either or both of
[STAC API - Item Search](https://github.com/radiantearth/stac-api-spec/tree/v1.0.0-rc.2/item-search) (`/search` endpoint) or
[STAC API - Features](https://github.com/radiantearth/stac-api-spec/tree/v1.0.0-rc.2/ogcapi-features)
(`/collections/{collectionId}/items` endpoint) by advertising the relevant conformance class.

Fields may be sorted in ascending or descending order.  The syntax between GET requests and POST requests with a JSON
body vary.  The `sortby` value is an array, so multiple sort fields can be defined which will be used to sort
the data in the order provided (e.g., first by `datetime`, then by `eo:cloud_cover`).

**NOTE**: *This specification may change, as our goal is to align with OGC API functionality, and sorting is currently being
worked on as part of OGC API - Records, see [this issue](https://github.com/opengeospatial/ogcapi-records/issues/22)
for the latest discussion.*

## HTTP GET (or POST Form)

When calling a relevant endpoint using GET (or POST with `Content-Type: application/x-www-form-urlencoded` or
`Content-Type: multipart/form-data)`, a single parameter `sortby` with a comma-separated list of item field names must
be provided. The field names may be prefixed with either "+" for ascending, or "-" for descending.  If no sign is
provided before the field name, it will be assumed to be "+".

Examples of `sortby` parameter:

1. `GET /search?sortby=properties.created`
2. `GET /search?sortby=+properties.created`
3. `GET /search?sortby=properties.created,-id`
4. `GET /search?sortby=+properties.created,-id`
5. `GET /search?sortby=-properties.eo:cloud_cover`

Note that examples 1 and 2 are symantically equivalent, as well as examples 3 and 4.

## HTTP POST JSON Entity

When calling the relevant endpoint using POST with`Content-Type: application/json`, this adds an attribute `sortby` with
an object value to the core JSON search request body.

The syntax for the `sortby` attribute is:

```json
{
    "sortby": [
        {
            "field": "<property_name>",
            "direction": "<direction>"
        }
    ]
}
```

```json
{
    "sortby": [
        {
            "field": "properties.created",
            "direction": "asc"
        },
        {
            "field": "properties.eo:cloud_cover",
            "direction": "desc"
        },
        {
            "field": "id",
            "direction": "desc"
        },
        {
            "field": "collection",
            "direction": "desc"
        }
    ]
}
```
