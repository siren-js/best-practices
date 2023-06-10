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

