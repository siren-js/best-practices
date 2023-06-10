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

