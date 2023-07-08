# JSON processing

## JSON Operators

```json title="Demo data"
{
  "firstName": "Gonzo",
  "lastName": "Puppet",
  "age": 81,
  "streetAddress": "100 Internet Dr",
  "city": "MuppetTown",
  "state": "MP",
  "postalCode": "12345",
  "phoneNumbers": [
    {
      "mobile": "111-111-1111"
    },
    {
      "home": "222-222-2222"
    }
  ]
}
```

=== "Accessing JSON Values"

    | Description                  | Operator      |                                                          Example |
    |------------------------------|---------------|-----------------------------------------------------------------:|
    | field in JSON object         | `-> text`     |                 `demo -> 'streetAddress'`<br />"100 Internet Dr" |
    | element in JSON array        | `-> integer`  | `demo -> 'phoneNumbers' -> 1`<br /> '{ "home": "222-222-2222" }' |
    | deep access by path segments | `#> text[]`   |         `demo #> '{phoneNumbers, 1, home}'`<br /> "222-222-2222" |

    !!! note "Retrieving text values instead of JSON values"
        Extending the operator with a second `>` character, i.e. `->>, #>>`,
        returns the JSON value as text rather than as JSONB.

=== "Querying containment"

    | Description                   | Operator     |                                                Example |
    |-------------------------------|--------------|-------------------------------------------------------:|
    | contains JSON ?               | `@> jsonb`   |   `demo @> '{"firstName": "Gonzo"}'::jsonb`<br /> true |
    | is JSON contained ?           | `<@ jsonb`   |   `'{"firstName": "Gonzo"}'::jsonb <@ demo`<br /> true |
    | contains top-level key ?      | `? text`     | `demo ? 'fullName'`<br /> false                        |
    | contains any top-level key ?  | `?| text[]` | `demo ?| '{fullName, firstName, lastName}'`<br /> true  |
    | contains all top-level keys ? | `?& text[]`  | `demo ?& '{fullName, firstName, lastName}'`<br /> false |

=== "Querying containment (JSONPath)"

    | Operator      |                                Example |
    |---------------|---------------------------------------:|
    | `@? jsonpath` | `demo @? '$.age ? (@ > 80)'`<br />true |
    | `@@ jsonpath` | `demo @@ '$.age > 80'`<br />true       |

## JSON Functions
