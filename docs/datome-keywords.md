---
sidebar_position: 4
---

# Datome Specific Keywords

### asset

When declared, specifies the Model name on which the operation has to be performed.

### authorized_groups

Optional keyword defining the list of URI identifying the roles or groups whose permissions are to be validated in order to execute an action. If declared inside a mutation, it refers to the permission to modify the Asset. If declared outside, hence in the definition of the Model, it refers to the permission to create the asset.

### changes

Mandatory keyword if a mutation block is declared. It must contain at least one type of change. The possible types are static, dynamic, transition, event.
(For more detailed information, refer to the [Mutations page](./model-syntax#mutations)).

### constraints

Optional keyword. Defines the list of restrictions of a relation. The keywords target is mandatory. Equal is the only operation allowed in order to respect the constraints and is expressed using the JSON-schema keywords {“op: “eq”} and value.

```jsx
    "constraints": {
	"target": "#/properties/{{property}}",
	"op": "eq",
	"value": "{{value}}"
    }
```

### external_mutation

Defines the list of possible actions that can affect an Asset different from the one that is calling the external_mutation. The keywords target and asset are mandatory. It may be declared inside a mutation block.
(For more detailed information, refer to the [Mutations page](./model-syntax#mutations)).

### identifier

Optional keyword. Sets a property of the Model as the unique IDentifier. The property has to be expressed in URI format. (`default: #/properties/{{property}}`)

### label

Optional keyword. It is used to mask the value of the Model ID with a property to enhance readability. The property has to be expressed in URI format. (`default: #/properties/{{property}}`)

### mutation

Optional keyword. Defines the list of possible actions that can affect the Asset. If declared, it must contain the keyword changes.
The keywords `authorized_groups` and `external_mutation` are optional.
(For more detailed information, refer to the [Mutations page](./model-syntax#mutations)).

### relation

Defines a one-to-one or one-to-many relationship between two Models. The keyword asset is mandatory, the keyword constraints is optional.

### search

Optional keyword. Defines the list of Model properties addressable in the Asset search functions.

### states

Mandatory keyword. Define the list of conditional states of being of the Model. Each Model definition must contain at least one `default_state` which always represents the initial state of the Finite State Machine. Every other state must be defined using a transition keyword.
(For more detailed information, refer to the [States page](./model-syntax#states)).

### target

When declared, specifies the URI of the properties, states or event object to be used to fulfill the action requirement.

### transition

Defines the list of states in addition to the default_state to build the Finite State Machine. The keywords `required_state` and `target_state` are mandatory.
(For more detailed information, refer to the [States page](./model-syntax#states)).

### "ui:order“

Optional keyword. Sets the displayed order of the Model properties in the UI.
