# json-schema-to-typescript [![Build Status][build]](https://circleci.com/gh/bcherny/json-schema-to-typescript) [![npm]](https://www.npmjs.com/package/json-schema-to-typescript) [![mit]](https://opensource.org/licenses/MIT)

[build]: https://img.shields.io/circleci/project/github/bcherny/json-schema-to-typescript/master.svg?style=flat-square
[npm]: https://img.shields.io/npm/v/json-schema-to-typescript.svg?style=flat-square
[mit]: https://img.shields.io/npm/l/json-schema-to-typescript.svg?style=flat-square

> Compile json schema to typescript typings

## Example

Input:
```json
{
  "title": "Example Schema",
  "type": "object",
  "properties": {
    "firstName": {
      "type": "string"
    },
    "lastName": {
      "type": "string"
    },
    "age": {
      "description": "Age in years",
      "type": "integer",
      "minimum": 0
    },
    "hairColor": {
      "enum": ["black", "brown", "blue"],
      "type": "string"
    }
  },
  "additionalProperties": false,
  "required": ["firstName", "lastName"]
}
```

Output:
```ts
export interface ExampleSchema {
  firstName: string;
  lastName: string;
  /**
   * Age in years
   */
  age?: number;
  hairColor?: "black" | "brown" | "blue";
}
```

## Installation

```sh
# Using Yarn:
yarn add json-schema-to-typescript

# Or, using NPM:
npm install json-schema-to-typescript --save
```

## Usage

```js
import { compile, compileFromFile } from 'json-schema-to-typescript'

// compile from file
compileFromFile('foo.json')
  .then(ts => fs.writeFileSync('foo.d.ts', ts))

// or, compile a JS object
let mySchema = {
  properties: [...]
}
compile(mySchema, 'MySchema')
  .then(ts => ...)
```

See [server demo](example) and [browser demo](https://github.com/bcherny/json-schema-to-typescript-browser) for full examples.

## Options

`compileFromFile` and `compile` accept options as their last argument (all keys are optional):

| key | type | default | description |
|-|-|-|-|
| bannerComment | string | `"/* tslint:disable */\n/**\n* This file was automatically generated by json-schema-to-typescript.\n* DO NOT MODIFY IT BY HAND. Instead, modify the source JSONSchema file,\n* and run json-schema-to-typescript to regenerate this file.\n*/"` | Disclaimer comment prepended to the top of each generated file |
| cwd | string | `process.cwd()` | Root directory for resolving [`$ref`](https://tools.ietf.org/id/draft-pbryan-zyp-json-ref-03.html)s |
| declareExternallyReferenced | boolean | `true` | Declare external schemas referenced via `$ref`? |
| enableConstEnums | boolean | `true` | Prepend enums with [`const`](https://www.typescriptlang.org/docs/handbook/enums.html#computed-and-constant-members)? |
| ignoreMinAndMaxItems | boolean | `false` | Ignore maxItems and minItems for `array` types, preventing tuples being generated. |
| style | object | `{ bracketSpacing: false,  printWidth: 120,  semi: true,  singleQuote: false,  tabWidth: 2,  trailingComma: 'none',  useTabs: false }` | A [Prettier](https://prettier.io/docs/en/options.html) configuration |
| unreachableDefinitions | boolean | `false` | Generates code for `definitions` that aren't referenced by the schema. |
| strictIndexSignatures | boolean | `false` | Append all index signatures with `| undefined` so that they are strictly typed. |
| $refOptions | object | `{}` | [$RefParser](https://github.com/BigstickCarpet/json-schema-ref-parser) Options, used when resolving `$ref`s |
## CLI

A CLI utility is provided with this package.

```sh
cat foo.json | json2ts > foo.d.ts
# or
json2ts foo.json > foo.d.ts
# or
json2ts foo.json foo.d.ts
# or
json2ts --input foo.json --output foo.d.ts
# or
json2ts -i foo.json -o foo.d.ts
```

You can pass any of the options described above (including style options) as CLI flags. Boolean values can be set to false using the `no-` prefix.

```sh
# generate code for definitions that aren't referenced
json2ts -i foo.json -o foo.d.ts --unreachableDefinitions
# use single quotes and disable trailing semicolons
json2ts -i foo.json -o foo.d.ts --style.singleQuote --no-style.semi
```

The CLI supports directory of definitions as well. It supports directory paths, glob patterns, and output directories.

Example 1: Directory of type definitions to an output directory

Input Directory
```
schemas /
 | a.json
 | b.json
```

```sh
json2ts -i schemas/ -o types/
```

Output Directory
```
types /
 | a.d.ts
 | b.d.ts
```

Example 2: Directory to pipe out
Input Directory
```
schemas /
 | a.json
 | b.json
```

```sh
json2ts -i schemas/
```

Example 3: Nested input directory mapped to nested output
Input Directory
```
schemas /
  foo /
    | a.json
  bar /
    | b.json
  fuzz /
    c.json
    buzz /
      d.json
```

```sh
json2ts -i schemas/ -o types/
```

Output Directory
```
types /
  foo /
    | a.d.ts
  bar /
    | b.d.ts
  fuzz /
    c.d.ts
    buzz /
      d.d.ts
```

## Tests

`npm test`

## Features

- [x] `title` => `interface`
- [x] Primitive types:
  - [x] array
  - [x] homogeneous array
  - [x] boolean
  - [x] integer
  - [x] number
  - [x] null
  - [x] object
  - [x] string
  - [x] homogeneous enum
  - [x] heterogeneous enum
- [x] Non/extensible interfaces
- [ ] Custom JSON-schema extensions
- [x] Nested properties
- [x] Schema definitions
- [x] [Schema references](http://json-schema.org/latest/json-schema-core.html#rfc.section.7.2.2)
- [x] Local (filesystem) schema references
- [x] External (network) schema references
- [x] Add support for running in browser
- [x] default interface name
- [x] infer unnamed interface name from filename
- [x] `allOf` ("intersection")
- [x] `anyOf` ("union")
- [x] `oneOf` (treated like `anyOf`)
- [x] `maxItems` ([eg](https://github.com/tdegrunt/jsonschema/blob/67c0e27ce9542efde0bf43dc1b2a95dd87df43c3/examples/all.js#L166))
- [x] `minItems` ([eg](https://github.com/tdegrunt/jsonschema/blob/67c0e27ce9542efde0bf43dc1b2a95dd87df43c3/examples/all.js#L165))
- [x] `additionalProperties` of type
- [x] `patternProperties` (partial support)
- [x] [`extends`](https://github.com/json-schema/json-schema/wiki/Extends)
- [x] `required` properties on objects ([eg](https://github.com/tdegrunt/jsonschema/blob/67c0e27ce9542efde0bf43dc1b2a95dd87df43c3/examples/all.js#L130))
- [ ] `validateRequired` ([eg](https://github.com/tdegrunt/jsonschema/blob/67c0e27ce9542efde0bf43dc1b2a95dd87df43c3/examples/all.js#L124))
- [x] literal objects in enum ([eg](https://github.com/tdegrunt/jsonschema/blob/67c0e27ce9542efde0bf43dc1b2a95dd87df43c3/examples/all.js#L236))
- [x] referencing schema by id ([eg](https://github.com/tdegrunt/jsonschema/blob/67c0e27ce9542efde0bf43dc1b2a95dd87df43c3/examples/all.js#L331))
- [x] custom typescript types via `tsType`

## Not expressible in TypeScript:

- `dependencies` ([single](https://github.com/tdegrunt/jsonschema/blob/67c0e27ce9542efde0bf43dc1b2a95dd87df43c3/examples/all.js#L261), [multiple](https://github.com/tdegrunt/jsonschema/blob/67c0e27ce9542efde0bf43dc1b2a95dd87df43c3/examples/all.js#L282))
- `divisibleBy` ([eg](https://github.com/tdegrunt/jsonschema/blob/67c0e27ce9542efde0bf43dc1b2a95dd87df43c3/examples/all.js#L185))
- [`format`](https://github.com/json-schema/json-schema/wiki/Format) ([eg](https://github.com/tdegrunt/jsonschema/blob/67c0e27ce9542efde0bf43dc1b2a95dd87df43c3/examples/all.js#L209))
- `multipleOf` ([eg](https://github.com/tdegrunt/jsonschema/blob/67c0e27ce9542efde0bf43dc1b2a95dd87df43c3/examples/all.js#L186))
- `maximum` ([eg](https://github.com/tdegrunt/jsonschema/blob/67c0e27ce9542efde0bf43dc1b2a95dd87df43c3/examples/all.js#L183))
- `minimum` ([eg](https://github.com/tdegrunt/jsonschema/blob/67c0e27ce9542efde0bf43dc1b2a95dd87df43c3/examples/all.js#L182))
- `maxProperties` ([eg](https://github.com/tdegrunt/jsonschema/blob/67c0e27ce9542efde0bf43dc1b2a95dd87df43c3/examples/all.js#L113))
- `minProperties` ([eg](https://github.com/tdegrunt/jsonschema/blob/67c0e27ce9542efde0bf43dc1b2a95dd87df43c3/examples/all.js#L112))
- `not`/`disallow`
- `oneOf` ("xor", use `anyOf` instead)
- `pattern` ([string](https://github.com/tdegrunt/jsonschema/blob/67c0e27ce9542efde0bf43dc1b2a95dd87df43c3/examples/all.js#L203), [regex](https://github.com/tdegrunt/jsonschema/blob/67c0e27ce9542efde0bf43dc1b2a95dd87df43c3/examples/all.js#L207))
- `uniqueItems` ([eg](https://github.com/tdegrunt/jsonschema/blob/67c0e27ce9542efde0bf43dc1b2a95dd87df43c3/examples/all.js#L172))

## Further Reading

- JSON-schema spec: https://tools.ietf.org/html/draft-zyp-json-schema-04
- JSON-schema wiki: https://github.com/json-schema/json-schema/wiki
- JSON-schema test suite: https://github.com/json-schema/JSON-Schema-Test-Suite/blob/node
- TypeScript spec: https://github.com/Microsoft/TypeScript/blob/master/doc/spec.md

## Projects That Use JSON-Schema-to-TypeScript

- [RAML-to-TypeScript](https://github.com/ducin/raml-to-typescript)
- See more: https://www.npmjs.com/browse/depended/json-schema-to-typescript
