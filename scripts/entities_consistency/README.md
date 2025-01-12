The entities consistency script analyze the contents of the entities database in orion DBs and
check several consistency rules, reporting violations found.

Ref: [entity document database model]([../../doc/manuals/admin/database_model.md#entities-collection])

## Requirements

This script is designed to work with Python 3. Install the dependencies in the `requirements.txt` file before using it.
Usage of virtual env is recommended.

## Usage

Run `entities_consistency.py -h` for arguments details.

## Rules

* Rules 1x: DB inconsistencies (use to be severe problems)
* Rules 2x: Syntax restrictions
* Rules 9x: Usage of legacy features

### Rule 10: `_id` field consistency

Each entity in DB has an `_id` field with three subfields:

* `id`
* `type`
* `servicePath`

### Rule 11: mandatory fields in entity

The following fields are mandatory:

* `attrNames`
* `creDate`
* `modDate`

It is not an exhaustive check of every field in the database model, but some entities created/updated with old Orion versions may be missing them.

### Rule 12: mandatory fields in attribute

The following subfields are mandatory for every attribute:

* `mdNames`
* `creDate`
* `modDate`

It is not an exhaustive check of every field in the database model, but some entities created/updated with old Orion versions may be missing them.

### Rule 13: `attrNames` field consistency

For each item in `attrNames` array there is a corresponding key in `attrs` object and the other way around.

### Rule 14: `mdNames` field consistency

For every attribute, for each item in `mdNames` array there is a corresponding key in `md` object and the other way around.

### Rule 15: not swapped subkeys in `_id`

In MongoDB JSON objects are stored taking order into account, so DB allows to have a document with
`_id` equal to `{"id": "E", "type": "T", "servicePath": "/"}` and at the same time have another document with `_id`
equal to `{"type": "T", "id": "E", "servicePath": "/"}` without violating `_id` uniqueness constraint.

This rule checks that this is not happening in the entities collection.

### Rule 16: `location` field consistency

Check that location in consistent. In particular:

* As much as one attribute with `geo:point`, `geo:line`, `geo:box`, `geo:polygon` or `geo:json` type (without `ignoreTypes` and without `null` value).
* If one of such attribute is found, check the `location` field is found and its content is consistent
* If none of such attribute is found, check no `location` field is not found 
  * If the location of the entity is defined using deprecated `location` metadata it will be detected as this case. Additional rules in the 9x group can help to diagnose this situation.

This rule is for location inconsistencies. For usage of deprecated geo types, there are additional rules in the 9x group.

### Rule 17: `lastCorrelator` existence

Check if `lastCorrelator` is included.

This field was introduced in [Orion 1.8.0](https://github.com/telefonicaid/fiware-orion/releases/tag/1.8.0) (released in September 11th, 2017).

### Rule 20: entity id syntax

Check [identifiers syntax restrictions](../../doc/manuals/orion-api.md#identifiers-syntax-restrictions) for this case.

### Rule 21: entity type syntax

Check [identifiers syntax restrictions](../../doc/manuals/orion-api.md#identifiers-syntax-restrictions) for this case.

### Rule 22: entity servicePath syntax

Check [servicePath documentation](https://github.com/telefonicaid/fiware-orion/blob/master/doc/manuals/orion-api.md#entity-service-path)

### Rule 23: attribute name syntax

Check [identifiers syntax restrictions](../../doc/manuals/orion-api.md#identifiers-syntax-restrictions) for this case.

### Rule 24: attribute type syntax

Check [identifiers syntax restrictions](../../doc/manuals/orion-api.md#identifiers-syntax-restrictions) for this case.

### Rule 25: metadata name syntax

Check [identifiers syntax restrictions](../../doc/manuals/orion-api.md#identifiers-syntax-restrictions) for this case.

### Rule 26: metadata type syntax

Check [identifiers syntax restrictions](../../doc/manuals/orion-api.md#identifiers-syntax-restrictions) for this case.

### Rule 90: detect usage of `geo:x` attribute type where `x` different from `json`

Check usage of deprecated geo-location types, i.e:

* `geo:point`
* `geo:line`
* `geo:box`
* `geo:polygon`

Suggested action is to:

* Change attribute type to `geo:json`
* Set the attribute value to the same GeoJSON in `location.coords` field

Note this rule doesn't check location consistency for this case (e.g. more than one geo-location attribute in the same entity). That's done by another rule in the 1x group.

### Rule 91: detect usage of more than one legacy `location` metadata

Check usage of `location` in more than one attribute of the same entity.

Note this rule doesn't check location consistency for this case (that's done by another rule in the 1x group).

### Rule 92: detect legacy `location` metadata should be `WGS84` or `WSG84`

The value of the `location` metadata should be `WGS84` or `WSG84`.

Additional consideration:

* Entities attributes may have `location` metadata with values different from `WGS84` or `WSG84` if created using NGSIv2. That would be a false positive in this rule validation 
* This rule doesn't check location consistency for this case (that's done by another rule in the 1x group).

### Rule 93: detect usage of redundant legacy `location`

Checks usage of redundant `location` metadata, i.e. when at the same time a `geo:` type is used in the
same attribute.

Suggested action is to remove the `location` metadata.

Additional, considerations:

 * This rule assumes only one `location` is in the entity (i.e. Rule 91 is not violated). If that doesn't occur, only the first occurrence is taken into account. 
 * This rule doesn't check location consistency for this case (that's done by another rule in the 1x group).

### Rule 94: detect usage of not redundant legacy `location`

Checks usage of not redundant `location` metadata, i.e. when at the same time the type of the attribute is nog `geo:`.
same attribute.

Suggested action is to:

* Change attribute type to `geo:json`
* Set the attribute value to the same GeoJSON in `location.coords` field
* Remove the `location` metadata from the attribute

Additional, considerations:

* This rule assumes only one `location` is in the entity (i.e. Rule 91 is not violated). If that doesn't occur, only the first occurrence is taken into account.
* This rule doesn't check location consistency for this case (that's done by another rule in the 1x group).

## Testing

You can test the `entities_consistency.py` script this qy:

1. Populate `orion-validation` DB with testing document. To do so, copy-paste the content of the `validation_data.js` in `mongosh`
2. Run `test_entities_consistency.py` test and check the log output

You can also run `test_entities_consistenct.py` under coverage to check every rule is covering all the possible validation cases.
