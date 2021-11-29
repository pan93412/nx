# Customizing generator options

## Adding a TypeScript schema

To create a TypeScript schema to use in your generator function, define a TypeScript file next to your schema.json named `schema.ts`. Inside the `schema.ts`, define an interface to match the properties in your schema.json file, and whether they are required.

```typescript
export interface GeneratorOptions {
  name: string;
  type?: string;
}
```

Import the TypeScript schema into your generator file and replace the any in your generator function with the interface.

```typescript
import { Tree, formatFiles, installPackagesTask } from '@nrwl/devkit';
import { libraryGenerator } from '@nrwl/workspace';

export default async function (tree: Tree, schema: GeneratorOptions) {
  await libraryGenerator(tree, { name: `${schema.name}-${schema.type || ''}` });
  await formatFiles(tree);
  return () => {
    installPackagesTask(tree);
  };
}
```

## Adding static options

Static options for a generator don't prompt the user for input. To add a static option, define a key in the schema.json file with the option name, and define an object with its type, description, and optional default value.

```json
{
  "$schema": "http://json-schema.org/schema",
  "id": "my-generator",
  "type": "object",
  "properties": {
    "name": {
      "type": "string",
      "description": "Library name",
      "$default": {
        "$source": "argv",
        "index": 0
      }
    },
    "type": {
      "type": "string",
      "description": "Provide the library type, such as 'data-access' or 'state'"
    }
  },
  "required": ["name"]
}
```

If you run the generator without providing a value for the type, it is not included in the generated name of the library.

## Adding dynamic prompts

Dynamic options can prompt the user to select from a list of options. To define a prompt, add an `x-prompt` property to the option object, set the type to list, and define an items array for the choices.

```json
{
  "$schema": "http://json-schema.org/schema",
  "id": "my-generator",
  "type": "object",
  "properties": {
    "name": {
      "type": "string",
      "description": "Library name",
      "$default": {
        "$source": "argv",
        "index": 0
      }
    },
    "type": {
      "type": "string",
      "description": "Provide the library type",
      "x-prompt": {
        "message": "Which type of library would you like to generate?",
        "type": "list",
        "items": [
          {
            "value": "data-access",
            "label": "Data Access"
          },
          {
            "value": "feature",
            "label": "Feature"
          },
          {
            "value": "state",
            "label": "State Management"
          }
        ]
      }
    }
  },
  "required": ["name"]
}
```

Running the generator without providing a value for the type will prompt the user to make a selection.

## All configurable schema options

### Schema

```json
{
  "properties": {
    "name": {} // see Properties
  },
  "required": [],
  "description": "",
  "definitions": {}, // same as "properties"
  "additionalProperties": false,
}
```

#### `properties`

The properties of a generator. It is formed in:

```json
{
  "properties_name": {
    // properties configuration
  }
}
```

The available options of the properties configuration can be
seen at [Properties](#properties) section.

#### `required`

The properties that is required. Example:

```json
{
  "properties": {
    "name": {
      "type": "string"
    },
    "type": {
      "type": "string"
    }
  },
  "required": ["name"]
}
```

In this example, the property `name` is required, while
the property `type` is optional. You can define your schema like:

```ts
interface Schema {
  name: string; // required
  type?: string; // optional
}
```

#### `description`

The description of your schema for user to understand
what he can do with the generator.

Example: `A exception class generator.`

#### `definitions`

<!-- wip -->

Not pretty sure what it is. Its structure is pretty similar
to `properties`.

#### `additionalProperties`

<!-- wip -->

Not pretty sure what it is, either.

### Properties

```json
{
  "type": "",
  "required": [],
  "enum": [],
  "properties": {
    "name": {
      "type": "string",
      "description": "Library name",
      "$default": {
        "$source": "argv",
        "index": 0
      }
    },
  },
  "oneOf": [],
  "anyOf": [],
  "allOf": [],
  "items": [],
  "alias": "",
  "aliases": [],
  "description": "",
  "format": "",
  "visible": false,
  "default": "",
  "$ref": "",
  "$default": {
    "$source": "argv",
    "index": 0
  },
  "additionalProperties": false,
  "x-prompt": {
    "message": "",
    "type": "",
    "items": []
  },
  "x-deprecated": false,
}
```

Number-only options:

```json
{
  "multipleOf": 5,
  "minimum": 5,
  "exclusiveMinimum": 5,
  "maximum": 200,
  "exclusiveMaximum": 200,
}
```

String-only options:

```json
  "pattern": "\\d+",
  "minLength": 10,
  "maxLength": 100,
```

[The current configurable options (and its parse method) can be found here](https://github.com/nrwl/nx/blob/master/packages/tao/src/shared/params.ts). You would need a basic knowledge of TypeScript to read this.
