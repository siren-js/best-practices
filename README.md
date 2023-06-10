# Siren Best Practices

This document outlines best practices for Siren API servers and clients. It does not intent to alter or usurp [the original specification](https://github.com/kevinswiber/siren).

## Notational Conventions

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [[RFC2119](https://datatracker.ietf.org/doc/html/rfc2119)].

[link]: https://github.com/kevinswiber/siren#links-1
[embedded-link]: https://github.com/kevinswiber/siren#embedded-link

## Specify Entity's Type

The server SHOULD include one or more type identifiers in every entity's `class` array. A type identifier SHOULD indicate which set of `properties`, action `name`s, and link and sub-entity relation types might be present in the entity.

Type identifiers MAY be anything from a simple string to a fully-qualified class name (e.g., `order`, `com.example.Order`). The server SHOULD be consistent in its type identifier naming conventions. The server SHOULD [document type identifiers](#appendix-a-document-your-api).

```json
{
  "class": ["Person"],
  "links": [
    {
      "rel": ["profile"],
      "href": "https://schema.org/Person"
    },
    {
      "rel": ["https://schema.org/knows"],
      "href": "https://example.com/hermione",
      "class": ["Person"]
    }
  ],
  "properties": {
    "givenName": "Neville",
    "familyName": "Longbottom",
    "birthDate": "1980-07-30"
  }
}
```

## Prefer Primitive Properties

The values in an entity's `properties` object SHOULD be primitives (string, number, boolean, or null) or arrays of primitives. An object or object array property value is typically a sign that another entity is more appropriate and should be linked to or embedded in the context.

## Follow Relation Type Standards

[Links][link]' and [embedded links][embedded-link]' relation types SHOULD conform to [Section 3.3 of RFC 8288](https://www.rfc-editor.org/rfc/rfc8288#section-3.3), which obsoletes [the RFC mentioned in the spec](https://www.rfc-editor.org/rfc/rfc5988). Thus `rel` values SHOULD either be a name from [the IANA link relations registry](https://www.iana.org/assignments/link-relations/link-relations.xhtml), a name in [your API documentation](#appendix-a-document-your-api), or an absolute URI. When using the latter, the server SHOULD use a URI that points to documentation describing the link relation type.

```json
{
  "links": [
    {
      "rel": ["self"],
      "href": "https://api.example.com/orders/69"
    },
    {
      "rel": ["https://schema.org/customer"],
      "href": "https://api.example.com/people/42"
    }
  ],
  "entities": [
    {
      "rel": ["https://api.example.com/about#order-items"],
      "href": "https://api.example.com/orders/69/items"
    }
  ]
}
```

## Provide Type Hint to Entity Links

When a [link][link] or [embedded link][embedded-link] reference another Siren entity, the `class` property SHOULD match the target entity's `class` property.

```json
{
  "rel": ["author"],
  "href": "https://api.example.com/people/42",
  "class": ["Person"]
}
```

## Distinguish Links to Non-Siren Resources

When a [link][link] points to a non-Siren resource, the server SHOULD specify the target's default media type in the `type` property.

```json
{
  "rel": ["icon"],
  "href": "https://api.example.com/images/icon",
  "type": "image/png"
}
```

> Note that `type` is only a hint; for example, it does not override the `Content-Type` header field of a HTTP response obtained by actually following the link. [[RFC 8288, Section 3.4.1](https://www.rfc-editor.org/rfc/rfc8288#section-3.4.1)]

## Include `self` Links

The server SHOULD include a `self` [link][link] for every entity it serves.

```json
{
  "rel": ["self"],
  "href": "https://api.example.com/orders/21"
}
```

## Link to API Documentation

The server SHOULD include a `profile` link [[RFC6906](https://www.rfc-editor.org/rfc/rfc6906.html)] for every entity it serves. The [link][link] SHOULD point to relevant [documentation describing the semantics of the resource](#appendix-a-document-your-api).

```json
{
  "rel": ["profile"],
  "href": "https://api.example.com/about",
  "type": "application/xhtml+xml"
}
```

## Resolve Relative URIs

When a [link][link]'s or [embedded link][embedded-link]'s `href` is a [relative URI](https://www.rfc-editor.org/rfc/rfc3986#section-4.2), clients SHOULD [resolve the reference](https://www.rfc-editor.org/rfc/rfc3986#section-5) by [establishing a base URI](https://www.rfc-editor.org/rfc/rfc3986#section-5.1) as follows:

1. [Base URI Embedded in Content](https://www.rfc-editor.org/rfc/rfc3986#section-5.1.1). This is the context entity's `self` link's `href`.
2. [Base URI from the Encapsulating Entity](https://www.rfc-editor.org/rfc/rfc3986#section-5.1.2). If the context entity has no `self` link with an absolute URI and it is a [sub-entity](https://github.com/kevinswiber/siren#sub-entities), then traverse the entity graph up from the context, searching for a `self` link with an absolute URI at each level, and stopping when one is found or the graph is exhausted. The `href` of that link, if found, is the base URI.
3. [Base URI from the Retrieval URI](https://www.rfc-editor.org/rfc/rfc3986#section-5.1.3). If no `self` link is found in the entity graph, then the URI used to retrieve the entity is the base URI.

Let's work through an example. Consider the following `Order` entity:

```json
{
  "class": ["Order"],
  "links": [
    {
      "rel": ["self"],
      "href": "https://api.example.com/orders/69"
    }
  ],
  "entities": [
    {
      "rel": ["https://schema.org/orderedItem"],
      "class": ["OrderItem"],
      "links": [
        {
          "rel": ["self"],
          "href": "https://api.eg.io/orders/69/items/1"
        }
      ],
      "entities": [
        {
          "rel": ["https://schema.org/seller"],
          "class": ["Organization"],
          "links": [
            {
              "rel": ["self"],
              "href": "https://example.org"
            }
          ],
          "entities": [
            {
              "rel": ["https://schema.org/member"],
              "class": ["Person"],
              "href": "/people/42"
            }
          ]
        }
      ]
    }
  ]
}
```

In order to resolve the `https://schema.org/member` `Person` sub-entity, we need to first resolve its `href`. We start by checking the context `Organization` entity for a `self` link. Since one is present, we use it's `href` as the base URI, making the `Person` sub-entity's URI `https://example.org/people/42`.

If however the `Organization` entity had no `self` link or it was itself relative, we would move up the entity graph once and check again for a `self` link. In that case, we would use the `OrderItem` entity's `self` link's `href` as the base URI, and the `Person` sub-entity's URI then becomes `https://api.eg.io/people/42`.

Similarly, had the `OrderItem` entity's `self` link been missing, we would once again traverse and search the entity graph. This time we fine the `Order` entity's `self` link and thus our `Person` sub-entity's URI is `https://api.example.com/people/42`.

Finally, if no `self` links had been present, the URI used to retrieve the entity would be the base URI since we have exhausted the entity graph.

