---
sidebar_position: 2
---

# Product Architecture

### Overview of the product architecture

Datome functional architecture is composed by:

### Users

Users are digital identities used to access the Datome webservices. They are associated with a single purposed object, as IOT devices, applications or persons.
A user owns a digital certificate, signed by the internal Certificate Authority (CA), which is used to sign every blockchain operation and identifies univocally the user. It cannot be used by others.

The ownership of a digital certificate does not automatically grant permission for all blockchain operations, as users must also be a member of a group (e.g. admin, model admin, or controller) to be permitted to carry out specific actions.

### Groups

Groups are clusters of users. A user is automatically a member of every sub-groups and can belong to multiple groups.
The members of a group acquire the permissions to access the data associated with the group.

### Model

A Model is the digital blue-print of the Asset we want to manage. It describes the Asset’s properties (e.g. weight, materials), the relationships with other Models (e.g. a blister can be part of a packaging), the actions that can be performed on the Asset data, and the specific user groups that have permission to execute those actions.
Models can set powerful data flow control, defining a Finite State Machine (FSM). It consists of a set of “states” or statuses along a process (e.g. “in progress”, “completed”, “shipped”), a set of “input” or actions, and a set of “transitions” that describe how the Asset moves from one state to another. Each state represents a distinct mode of behavior of the Asset and is characterized by a specific set of permissions and actions . The transition from one state to another is governed by a set of rules expressed in the Model, and it determines which state the machine will enter after any action.
The FSM implementation is optional but every Model must include a default state.
Administrators can create or update models using any text editor, according to a custom JSON-schema syntax, enhanced with special-keywords dedicated to Datome specification.
A Versioning Strategy can be created to track any update.

### Asset

An Asset is the representation in digital format of a document, a physical Asset, a dataset or any object. It is created, updated or managed according to the specification, rules and constraints expressed by its Model and can be linked to other Assets to describe provenance or composition. It may have multiple blockchain registrations, describing its history, and all the asset registrations are cryptographically linked to the author’s digital certificate.
As an example, an Asset could be: a car, the car’s ownership information, the details of the maintenance operation, the insurance document of the car.

### Model services

Datome provides webservices for managing the Models.
A Datome OpenAPI reference document (swagger) describes the endpoints, parameters, responses, and functions offered by the services.

### Asset Services

Datome provides webservices for managing the Assets.
A Datome OpenAPI reference document (swagger) describes the services and how to use them.

### Document Services

Files can be linked to an Asset at any point in its lifecycle using Document Services. Each document is transferred to a repository, while the description and a hash value of the file are stored on the blockchain. When the document is read back, its content is compared to the hash information kept on the blockchain. The comparison between those two hash keys certifies the document authenticity.
All the certification phases can be tracked by the QA team, together with the certification document.

### Blockchain Engine

Datome blockchain engine performs all the operation for maintaining the blockchain and executing all the user requests.
Datome uses the Hyperledger Foundation’s Fabric product for its internal blockchain.

### User/Group Services

Datome integrates a user management software which services are available via webservices or web administration UI.
Groups, users and group memberships(roles) are managed by Admin users.
Group Admins are users that can perform administrative tasks inside a group boundary.

### Administrative Web Services

Datome offers a complete administration web environment for executing administrative tasks. These include all the tasks available via webservices.

### User Web Services

User web services is a web based environment used by users for managing and browsing models and assets information, according to their privileges

### System Components

Datome platform is internally composed by:

- A hyperledger Foundation Fabric custom installation, with one or more permissioned blockchains (channels).
- Internal DBMS for services
- High performance web server
- TLS certificates engine
- Datome application
- Datome fabric chaincodes (smart contracts)
- Datome stored procedures
