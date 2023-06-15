# Siren Best Practices

This document outlines best practices for Siren API servers and clients. It does not intend to alter or usurp [the original specification](https://github.com/kevinswiber/siren).

## Table of Contents <!-- omit in toc -->

- [Specify Entity's Type](#specify-entitys-type)
- [Prefer Primitive Properties](#prefer-primitive-properties)
- [Follow Relation Type Standards](#follow-relation-type-standards)
- [Provide Type Hint to Entity Links](#provide-type-hint-to-entity-links)
- [Distinguish Links to Non-Siren Resources](#distinguish-links-to-non-siren-resources)
- [Include `self` Links](#include-self-links)
- [Link to API Documentation](#link-to-api-documentation)
- [Resolve Relative URIs](#resolve-relative-uris)
- [Appendix A: API Documentation](#appendix-a-api-documentation)

[embedded-link]: https://github.com/kevinswiber/siren#embedded-link
[link]: https://github.com/kevinswiber/siren#links-1
[sub-entity]: https://github.com/kevinswiber/siren#sub-entities

## Specify Entity's Type

As the server, include one or more type identifiers in the entity `class` array. A type identifier indicates which set of `properties`, action `name`s, and [link][link] and [sub-entity][sub-entity] relation types (`rel`) might be present in the entity.

Type identifiers can be anything from a simple string to a fully-qualified class name (e.g., `order`, `com.example.Order`). Be consistent in type identifier naming conventions, and [document them](#appendix-a-document-your-api).

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

Prefer primitives (string, number, boolean, or null) or arrays of primitives as the values of an entity's `properties` object. An object or object array property value is typically a sign that another entity/resource is more appropriate and should be hyperlinked in the context entity.

## Follow Relation Type Standards

[Links][link]' and [embedded links][embedded-link]' relation types conform to [Section 3.3 of RFC 8288](https://www.rfc-editor.org/rfc/rfc8288#section-3.3), which obsoletes [the RFC mentioned in the spec](https://www.rfc-editor.org/rfc/rfc5988). Thus `rel` values are either a name from [the IANA link relations registry](https://www.iana.org/assignments/link-relations/link-relations.xhtml), a name in [your API documentation](#appendix-a-document-your-api), or an absolute URI. When using the latter, use a URL that points to documentation describing the link relation type.

```json
{
  "links": [
    {
      "rel": ["self"],
      "href": "/orders/69"
    },
    {
      "rel": ["https://schema.org/customer"],
      "href": "/people/42"
    }
  ],
  "entities": [
    {
      "rel": ["https://api.example.com/about#order-items"],
      "href": "/orders/69/items"
    }
  ]
}
```

## Provide Type Hint to Entity Links

When a [link][link] or [embedded link][embedded-link] reference another Siren entity, match the link's `class` property with the target entity's `class` property.

```json
{
  "rel": ["author"],
  "href": "/people/42",
  "class": ["Person"]
}
```

```http
GET /people/42 HTTP/1.1
Host: api.example.com
```

```http
HTTP/1.1 200 OK
Content-Type: application/vnd.siren+json

{
  "class": ["Person"],
  "links": [
    { "rel": ["self"], "href": "/people/42" }
  ]
}
```

## Distinguish Links to Non-Siren Resources

When a [link][link] points to a non-Siren resource, specify the target's default media type in the `type` property.

```json
{
  "rel": ["icon"],
  "href": "https://api.example.com/images/icon",
  "type": "image/png"
}
```

> Note that `type` is only a hint; for example, it does not override the `Content-Type` header field of a HTTP response obtained by actually following the link. [[RFC 8288, Section 3.4.1](https://www.rfc-editor.org/rfc/rfc8288#section-3.4.1)]

## Include `self` Links

Include a `self` [link][link] for every entity. An entity represents a resource, which is identified by a URL, and therefore has a `self` link.

```json
{
  "rel": ["self"],
  "href": "/orders/21"
}
```

## Link to API Documentation

Include a `profile` link [[RFC6906](https://www.rfc-editor.org/rfc/rfc6906.html)] for every entity. The [link][link] points to relevant [documentation describing the semantics of the resource](#appendix-a-document-your-api).

```json
{
  "rel": ["profile"],
  "href": "https://api.example.com/about#Person",
  "type": "application/xhtml+xml"
}
```

## Resolve Relative URIs

When a [link][link]'s or [embedded link][embedded-link]'s `href` is a [relative URI](https://www.rfc-editor.org/rfc/rfc3986#section-4.2), [resolve the reference](https://www.rfc-editor.org/rfc/rfc3986#section-5) by [establishing a base URI](https://www.rfc-editor.org/rfc/rfc3986#section-5.1) as follows:

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

## Appendix A: API Documentation

At minimum, Siren API documentation describes the following:

- `class` values
- Names of `properties`
- `name`s of `actions` and `fields`
- Custom `rel` values

The exact format of Siren API documentation is beyond the scope of this document. Consider using a format designed for documenting APIs like [ALPS](http://alps.io/) or [XMDP](https://gmpg.org/xmdp/).
