# STAC API - Sort Extension Specification

- [STAC API - Sort Extension Specification](#stac-api---sort-extension-specification)
  - [Overview](#overview)
  - [HTTP GET](#http-get)
  - [HTTP POST JSON Entity](#http-post-json-entity)
  - [Sortables](#sortables)

## Overview

- **Title:** Sort
- **OpenAPI specification:** [openapi.yaml](openapi.yaml)
- **Conformance Classes:**
  - **STAC API - Item Search** binding: <https://api.stacspec.org/v1.0.0/item-search#sort>
  - **STAC API - Features** binding: <https://api.stacspec.org/v1.0.0/ogcapi-features#sort>
  - **STAC API - Collection Search** binding: <https://api.stacspec.org/v1.0.0-rc.1/collection-search#sort>
- **Scope:** STAC API - Features, STAC API - Item Search, STAC API - Collection Search
- **[Extension Maturity Classification](https://github.com/radiantearth/stac-api-spec/tree/main/README.md#maturity-classification):** Stable
- **Dependencies:**
  - [STAC API - Item Search](https://github.com/radiantearth/stac-api-spec/tree/v1.0.0/item-search)
  - [STAC API - Features](https://github.com/radiantearth/stac-api-spec/tree/v1.0.0/ogcapi-features)
  - [STAC API - Collection Search](https://github.com/stac-api-extensions/collection-search/tree/v1.0.0-rc.1)
- **Owner**: @philvarner

This specification defines a new parameter, `sortby`, that allows the user to define the fields by which
to sort results.
Only string, numeric, and datetime attributes of Item (`id` and `collection` only) or Item Properties (any attributes)
may be used to sort results.  

It is not required that implementations support sorting over all attributes, but
implementations should either implement Sortables or just return a 400 Bad Request status code
when attempting to sort over a field name that does not support sorting.
Implementers may choose to require fields in Item Properties to be prefixed with `properties.` or not,
or support use of both the prefixed and non-prefixed name, e.g., `properties.datetime` or `datetime`.

Sort behavior may be bound to any of the following endpoints by advertising the relevant conformance class:

- [STAC API - Item Search](https://github.com/radiantearth/stac-api-spec/tree/v1.0.0/item-search)
  (`/search` endpoint),
- [STAC API - Features](https://github.com/radiantearth/stac-api-spec/tree/v1.0.0/ogcapi-features)
  (`/collections/{collectionId}/items` endpoint)
- [STAC API - Collection Search](https://github.com/stac-api-extensions/collection-search/tree/v1.0.0-rc.1)
  (`/collections` endpoint)

Fields may be sorted in ascending or descending order.  The syntax between GET requests and POST requests with a JSON
body vary.  The `sortby` value is an array, so multiple sort fields can be defined which will be used to sort
the data in the order provided (e.g., first by `datetime`, then by `eo:cloud_cover`).

## HTTP GET

When calling a relevant endpoint using GET, a single parameter `sortby` with a comma-separated list of item field names must
be provided. The field names may be prefixed with either "+" for ascending, or "-" for descending.  If no sign is
provided before the field name, it will be assumed to be "+". Note that `+` is used commonly
by URL encoding as a space, so some tools may require escaping this literal `+` with a URL encoding of `%2B`.  

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

## Sortables

Additional endpoints that provide so called "Sortables" support clients that want to discover the list of resource
properties with their types and constraints that may be used to sort resources.

These Sortables endpoints return lists of properties (or aliases) that can be used in the `sortby` parameter.
It returns a JSON Schema that defines the properties allowed in `sortby`.
The precise definition of this can be found in the
[OGC API - Features - Part 5: Schemas](https://portal.ogc.org/files/108199#rc_sortables).

All Sortables endpoints SHALL be referenced with a link with the link relation type
`http://www.opengis.net/def/rel/ogc/1.0/sortables`.

| Sortables Endpoint                                          | Endpoint linking to the Sortables Endpoint | Conformance class                                                  | Applicable `sortby` endpoints           |
| ----------------------------------------------------------- | ------------------------------------------ | ------------------------------------------------------------------ | --------------------------------------- |
| `GET /sortables`                                            | `GET /`                                    | `https://api.stacspec.org/v1.0.0/item-search#sortables`            | `GET /search` and `POST /search`        |
| `GET /collections/{collectionId}/sortables`                 | `GET /collections/{collectionId}`          | `http://www.opengis.net/spec/ogcapi-features-5/1.0/conf/sortables` | `GET /collections/{collectionId}/items` |
| `GET /...` (*Endpoint name to be chosen by implementation*) | `GET /collections`                         | `https://api.stacspec.org/v1.0.0-rc.1/collection-search#sortables` | `GET /collections/{collectionId}/items` |

An example for a link to the sortables endpoint could be:

```json
{
    "href": "https://stac.example/sortables",
    "type": "application/schema+json",
    "rel": "http://www.opengis.net/def/rel/ogc/1.0/sortables",
    "title": "Sortables"
}
```

For an example of a sortables endpoint response, please see the [openapi.yaml](openapi.yaml).
