---
sidebar_position: 1
---

# Model syntax

### An example regarding a fabric.

Models are written according to the json-schema syntax.

A basic Model defines the blue-print (data structure) of the Asset we need to manage with Datome and it is written according to the json-schema Type-specific keyword [documentation](https://json-schema.org/understanding-json-schema/reference/type.html).

This is the beginning of our Model:

- `$id` and `$schema` are fixed values required by JSON-schema.
- `title` and `description` are optional parameters that helps with the definition of the model.
- `type` is used for backward compatibility.
- `properties` contains the list of the Asset's characteristics.
- `required` is a JSON-schema keyword used to define which of the properties are mandatory during the creation of the Asset. It always refers to the `properties` block on the same level.
- `identifier`: optional keyword. Sets a property of the Model as the unique IDentifier.
- `additionalProperties` is a JSON-schema keyword used to enhance the Asset with additional characteristics only during its creation.
  There are three possible values:
  1. `true` new properties can be declared directly in the API request without any kind of validation because they are defined "on-the-fly".
  2. `Object` the additional properties are defined inside a json Object written using JSON-schema keywords. Hence, each new attribute can be optional or "required" (since the "object" is defined within a "properties" block)
  3. `false` Default, no additional properties have to be evaluated during the creation of the Asset.

```json
{
    "$id": "https://mangrovia.solutions/generic.json",
    "$schema": "http://json-schema.org/draft-07/schema",
    "title": "fabric",
    "description": "Fabric Model describes a fabric lot",
    "type": "object",
    "properties": {
        "name": {
            "type": "string"
        },
        "id_technician": {
            "type": "number"
        },
        "id_employee": {
            "type": "number"
        },
        "available": {
            "type": "boolean"
        },
        "notes": {
            "type": "string"
        }
    },
    "required": [
    	"name",
    	"id_technician",
    ],
    "identifier": "#/properties/name",
    "additionalProperties": false,
    ...
```

### search

`search` is an optional parameter used for listing the values addressable in the search endpoint

```json
"search": [
    "name",
    "type",
    "meters",
    "composition",
    "manufacturer",
    "invoice_number"
],
```

### authorized_groups

`authorized_groups` indicates the privileges needed to create the Asset.
When declared it specifies which groups have the authority to create (only create) the Asset.
If the clause is not specified, this gives to everyone the authority to create the Asset.

```json
"authorized_groups": [
	“/root”,
	“/root/group1”,
	“/root/group2/sub-group”
],
```

### states

`states` defines the entry point of the Finite State Machine and every other state. A Model does not need a FSM but the declaration of a default_state is mandatory for each model.

```json
...
"states": {
        "default_state": "created",
    }
}
...
```

### transitions

`transitions` is a Datome specific keyword used to compose the transition map, nested inside the “states” block, defining every other state besides the `default_state`.

**If specified**, it must contain at least one state in addition to the `default_state`.

Transitions are defined by a label that identify the new state (e.g. `send_to_quality_control`), a `required_state`, the original state in which the Asset must be when the transition mutation is called, and a `target_state` which refers to the destination state.

```json
"transitions": {
    "send_to_quality_control": {
      "required_state": "manufacturing",
      "target_state": "quality_control"
    },
    "send_to_warehouse": {
      "required_state": "quality_control",
      "target_state": "warehouse"
    },
    "send_to_distribution": {
      "required_state": "warehouse",
      "target_state": "distribution"
    }
  }
```

### mutations

`mutations` is a Datome specific keyword used to define a set of operations that modify the Asset’s properties, as well as the actions and privileges required to perform those changes on data (updates) or states (transitions).
The declaration of `mutations` on an Asset is **optional**.

Only if the `mutations` key is included in the Model definition, compliance checks will be performed, hence the possibility of uncontrolled updates is avoided.

A `mutation` block is defined as follows:

- a custom name used to identify the mutation (e.g. `send_to_confirmed`)
- `authorized_groups` indicates the privileges needed to modify the Asset, hence to call the mutation. If the clause is not specified, everyone has the authority to modify the Asset.
- `changes` identifies the list of operations applied by the mutation; each one must declare a `target`. It is mandatory and must contain at least one operation, the order of which is irrelevant. Each change behavior is influenced by his “type” and there are four possible values:

  1. `static`: the target is always a property of the Asset. The new value is prefixed and defined within the definition of the StaticChange. Therefore, each time the mutation is executed, that specific property will always be updated with the same value.
  2. `dynamic`: the target is always a property of the Asset. The new value is passed in the body of the API re quest, hence is defined by the authorized user who is calling the mutation. It contains a label “required” of type boolean. If set to true, the new value of the field must be expressed in the API request, otherwise it could be omitted (it answers the question, “Is there the necessity to check that in the body of the call to the Datome API there is a field (the target) valorized?”).
  3. `transition`: is the type used to change the state of an Asset. The target is always a state, identified by URI, defined within the transitions block.
  4. `events`: the target is always a json object defined in the Model. Its behavior is equal to a DynamicChange, so the fields of the object needed to be updated have to be specified in the body of the API request.

```json
"mutations": {
    "send_to_confirmed": {
        "changes": [
            {
                "type": "transition",
                "target": "#/states/transitions/send_to_confirmed"
            },
            {
                "type": "dynamic",
                "required": true,
                "target": "#/properties/id_technician"
            },
            {
                "type": "dynamic",
                "required": false,
                "target": "#/properties/id_employee"
            },
            {
                "type": "static",
                "value": true,
                "target": "#/properties/available"
            }
        ],
        "authorized_groups": [
            "/root/group/…/admin",
            "/root/group2/…/controller"
        ],

        "external_mutations": [
            {
                "Asset": "garment",
                "target": "#/mutations/send_to_available"
            }
        ]
    }
}

```

- `external_mutations` is an optional Datome specific keyword that allows the user to execute a mutation that affects an Asset different from the one where the mutation is called. The permissions to execute such mutations are validated by referring to the clause `authorized_groups` in the definition of the Model which is calling the external_mutation. In case of Dynamic and Event changes, the new values need to be written in the API request, inside a `params` block.

```jsx
curl --request POST 'http://localhost/api/Assets/{{model_name}}/{{asset_id}}/mutations/{{mutation_name}}/' \
--header 'Authorization: Bearer ....' \
--header 'Content-Type: application/json' \
--data-raw '
  {
    "params": {
      ...
    },
    "external_mutations": {
      "{{target_model_name}}": {
        "{{mutation_name}}": {
          "{{asset_id}}": {
            "params": {
              ...
            },
            "external_mutations": {
              ...
            }
          }
        }
      }
    }
  }
'
```

In addition, this keyword can be used to call a `cascade mutation`: it is possible to build a list of mutations, nested one inside the other to be executed in one single sequence of actions, that modifies one or more different Assets.
The combination of the keywords “params - external_mutation” may be used recursively to fulfill the API request with the necessary values.

A mutation can be executed a potentially infinite number of times. Nevertheless, mutations that contain the execution of a state “transition” may fail if executed again, as the `required_state` of the Asset has changed.
In order to execute a mutation, it is necessary to call the correct endpoint providing the necessary input parameters through the URL and inside the body of the API request if needed.

```jsx
curl --location --request POST 'http://localhost/api/assets//fabric/silk/mutations/send_to_confirmed//' \
--header 'Authorization: Bearer ....' \
--header 'Content-Type: application/json' \
--data-raw '{
    "params": {
       "id_technician": 987,
       "id_employee": 14005
    },
}'
```

### relationships

Model relationships define links to existing Models.
Relationships are used for defining ownership, provenance, composition:
`fabric` is the label of the relationship to the Asset `fabric`. `Description` is a readable explanation of the link.
Relationships can be of types one-to-one or one-to-many. They create complex data infrastructure, introducing controls and verification.
The label `relation` is a Datome specific keyword used inside the `properties` block to declare the type of relationship and the Assets involved. It is identified by a custom `relation_name` and has a `type` that refers to the type of the field that receives the relationship and it must be consistent with the type of the Asset’s ID to which it refers. To specify a one-to-many relationship, it is mandatory to declare a list (_array_) of `type` and the Model of the Assets to link.

```json
one-to-one
    "{{relation_name}}": {
      "relation": {
        "Asset": "{{model_name}}"
      },
      "type": "string",
      "description": "Relation to a {{model_name}} Asset previously created"
    }
one-to-many
    "{{relation_name}}": {
	“type”: “array”,
	“items”: {
	    "type": "string"
	}
       "relation": {
           "Asset": "{{model_name}}"
       },
    }

```

The existence of the Asset father is controlled **automatically** by Datome.
