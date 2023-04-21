---
sidebar_position: 2
---

# Model example

For the sake of introducing the syntax of a Model, let’s use a simple Model for defining the information about a fabric lot and a garment.

The body of the POST request contains the Model:

```json
curl --location --request POST
'http://localhost/api/models/fabric/' \
--header 'Authorization: Bearer 74a6c88d-62fe-4c13-8b40-c21fabbae819' \
--header 'Content-Type: application/json' \
--data '{
    "$schema": "http://json-schema.org/draft-07/schema",
    "$id": "http://mangrovia.solutions/generic.json",
    "type": "object",
    "title": "fabric",
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
	"id_technician"
    ],
    "identifier": "#/properties/name",
    "additionalProperties": false,
    "mutations": {
        "send_to_processing": {
            "changes": [
                {
                    "type": "transition",
                    "target": "#/states/transitions/send_to_processing"
                }
            ],
            "authorized_groups": [
        	"/root/group/…/admin"
	    ]
        },

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
        },
	"send_to_recalled": {
            "changes": [
                {
                    "target": "#/states/transitions/send_to_recalled",
                    "type": "transition"
                },
                {
                    "target": "#/properties/notes",
                    "type": "dynamic",
                    "required": true
                },
                {
                    "type": "static",
                    "value": true,
                    "target": "#/properties/available"
                }
            ]
        }
    },
    "states": {
        "default_state": "open",
        "transitions": {
            "send_to_processing": {
                "required_state": "open",
                "target_state": "processing"
            },
            "send_to_confirmed": {
                "required_state": "processing",
                "target_state": "confirmed"
            },
	     "send_to_recalled": {
                "required_state": "confirmed",
                "target_state": "recalled"
            }
        }
    },
    "ui:order": [
      "name",
      "available",
      "id_employee",
      "id_technician"
   ]
}
```

The same process can be applied for the Model “garment” which has a one-to-one relationship with the fabric Model, that has been specified in the “properties” block using the “relation” keyword.

```json
{
  "$id": "http://mangrovia.solutions/garment.json",
  "$schema": "http://json-schema.org/draft-07/schema",
  "title": "garment",
  "description": "Garment Model describes the manufacturing process of a garment.",
  "type": "object",
  "properties": {
    "name": {
      "type": "string"
    },
    "serial_number": {
      "type": "string"
    },
    "garment_type": {
      "type": "string"
    },
    "size": {
      "type": "string"
    },
    "available": {
      "type": "boolean"
    },
    "fabric_relationship": {
      "relation": {
        "Asset": "fabric"
      },
      "type": "string",
      "description": "Relation to a fabric Asset previously created"
    }
  },
  "required": [
    "name",
    "serial_number",
    "garment_type",
    "size",
    "fabric_relationship"
  ],
  "additionalProperties": false,
  "identifier": "#/properties/serial_number",
  "label": "#/properties/garment_type",
  "states": {
    "default_state": "created",
    "transitions": {
      "send_to_available": {
        "required_state": "created",
        "target_state": "available"
      },
      "send_to_recalled": {
        "required_state": "available",
        "target_state": "recalled"
      }
    }
  },
  "mutations": {
    "send_to_available": {
      "changes": [
        {
          "type": "transition",
          "target": "#/states/transitions/send_to_available"
        },
        {
          "type": "static",
          "value": true,
          "target": "#/properties/available"
        }
      ],
      "authorized_groups": ["/root/group2/…/controller"]
    },
    "send_to_recalled": {
      "changes": [
        {
          "target": "#/states/transitions/send_to_recalled",
          "type": "transition"
        }
      ],
      "external_mutations": [
        {
          "asset": "fabric",
          "target": "#/mutations/send_to_recalled"
        }
      ]
    }
  }
}
```

Let’s analyze the fabric Model, it is so composed:

- **4 properties**: `name`, `id_technician`, `id_employee`, `active`
- **3 mutations**: `send_to_processing`, `send_to_confirmed`, `send_to_recalled`
- **4 states**: `open`(default), `processing`, `confirmed`, `recalled`.

The only required property is `id_technician`.
The mutation `send_to_processing` contains only one transition change to shift the Asset from the open state to the processing state.
This mutation can be executed only by a user who is a member of the admin group.
The mutation `send_to_confirmed` contains a collection of 5 different changes:

1. _transition_ type: change the Asset state from processing to confirmed.
2. _dynamic type_: the `id_technician` value will be updated with the one written by the user in the API request. The field must be valued in the body of the request as it is marked as `required: 'true'`.
3. _dynamic type_: the `id_employee` value will be updated with the one written by the user in the API request. The field may be valued in the request as it is marked as `required: 'false'` (_a.k.a. optional field_).
4. _static type_: the value of the field available will be updated automatically by the system with the predetermined value true.
5. one external mutation,which will invoke the `send_to_available` mutation defined in the garment Model.
   The users authorized to call this mutation are the members of the admin or the controller groups. This implies that an Asset outside the fabric Model, the asset garment, will also undergo a change as a consequence of the `send_to_confirmed` mutation.

The mutation send_to_recalled contains a collection of 2 different changes:

1. _transition_ type: change the Asset state from confirmed to recalled.
2. _dynamic_ type: the notes value will be updated with the one written by the user in the API request. The field must be valued in the body of the request as it is marked as `required: 'true'`.
3. _static_ type: the value of the field available will be updated automatically by the system with the predetermined value true.

When the `send_to_confirmed` mutation is called by a user, he will necessarily specify the `id_technician` in the API request and he may specify the `id_employee` too.
When the request is made, Datome will validate the fields valued by the user (according to the specifications expressed in the Model `properties`), "move" the Asset fabric from open to the confirmed state and update the value of the available property with the value true.
Finally the `send_to_available` mutation will be called on the Asset garment.

```jsx
curl --location --request POST
'http://localhost/api/assets/fabric/' \
--header 'Authorization: Bearer 74a6c88d-62fe-4c13-8b40-c21fabbae819' \
--header 'Content-Type: application/json' \
--data '{
    "name": "silk",
    "id_technician": 965,
    "id_employee": 15264,
    "available": false
}
```

After the Asset silk is created, the new Garment can be linked to it directly during the creation:

```jsx
curl --location --request POST
'http://localhost/api/assets/garment/' \
--header 'Authorization: Bearer 74a6c88d-62fe-4c13-8b40-c21fabbae819' \
--header 'Content-Type: application/json' \
--data '{
    "serial_number": "12345",
    "garment_type": "shirt",
    "size": "M",
    "fabric_relationship": "silk",
    "available": false
}
```

To call the mutation `send_to_confirmed`, it is sent a POST request to the correct endpoint containing all the parameters defined in the Model:

```jsx
curl --location --request POST
'http://localhost/api/assets/fabric/silk/mutations/send_to_confirmed' \
--header 'Authorization: Bearer 74a6c88d-62fe-4c13-8b40-c21fabbae819' \
--header 'Content-Type: application/json' \
--data '{
    "id_technician": 987,
    "id_amployee": 14005
}
```

In this last example of API request, in order to call the mutation `send_to_recalled` defined in the garment Model, the body of the request contains the necessary input parameter to call the second mutation `send_to_recalled` which is defined in the _fabric_ Model instead, which, in this example, updates the _notes_ field of the _silk_ Asset.

```jsx
curl --location --request POST
'http://localhost/api/assets/garment/12345/mutations/send_to_recalled' \
--header 'Authorization: Bearer 74a6c88d-62fe-4c13-8b40-c21fabbae819' \
--header 'Content-Type: application/json' \
--data '{
    "params": {},
    "external_mutations": {
        "fabric": {
            "send_to_recalled": {
                "silk": {
                     "params": {
                         "notes": "invalid product"
                     }
                }
            }
        }
    }
}


```
