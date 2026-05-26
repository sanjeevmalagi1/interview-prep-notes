

# Product Activation Request
A product activation request is a concept, in which a product feature is activated to a customer / business when the request is approved.

## Background
At "Aspire FT" we build many features and products for our users.
Since some features or abilities require a level of compliance from our customers. We created a mechanism to enforce/enable product activation by a request and approve flow.

## Why is it needed
- We needed a common framework for our analyst - customer interactions for non-KYC/KYB activities.
- We knew the existing KYC / KYB flow are inadequate to handle such requests. But the underlying concepts like maker/checker flow (with state machine), missing info flows and history tracking are inherited from KYC / KYB flow are required.

## Requirements
### Functional Requirements
- Create a common framework for our analyst - customer interactions for non-KYC/KYB activities.
- Should be able to support different types of activation requests dynamically

### Non Functional Requirements
- Should support existing features like maker/checker flow (inheriting from KYB/KYC flows), missing info flow, trackable history
- Should use existing `approvables` 
- Should use existing `key-value` store to store additional details

## High Level Design

```UML
@startuml
Actor user as "User"
Actor analyst as "Analyst"

Boundary app as "Customer\nFrontend"
Boundary dash as "Dashboard"

Control api as "Main API"
Control service as "Micro-Service"

autonumber
user -> app: Completes an {action}
app -> api: Register action


api -> service: Create Product Activation Request
note right service
internal state machine based on
State, ProcessState
(inherited from existing design)
end note

analyst -> dash: View Product Activation Requests
dash -> api: v1/product-activation-request?filter=
api -> service: GET v1/product-activation-request?filter=
service --> api: list
api --> dash: list

analyst -> dash: Approves Product Activation Request
dash -> api: PUT v1/approvables/{approvable_uuid}
api -> service: PUT v1/approvables/{approvable_uuid}

service --> api: Submit `UpdateProductActivationRequest` event
note right api
    UpdateProductActivationRequest {
        state: {state},
        type: {activation_request_type},
        business_uuid: {business_uuid},
        ...other_related_details
    }
end note
api -> events: Publish `ActivationRequestSuccessful`

analyst -> dash: Declines Product Activation Request
dash -> api: PUT v1/approvables/{approvable_uuid}
api -> service: PUT v1/approvables/{approvable_uuid}

service --> api: Submit `UpdateProductActivationRequest` event
note right api
    UpdateProductActivationRequest {
        state: {state},
        type: {activation_request_type},
        business_uuid: {business_uuid},
        ...other_related_details
    }
end note
api -> events: Publish `ActivationRequestFailed`

@enduml
```

### APIs and Events

01. Create Activation Request 

POST /v1/product-activation-request

Request Body:
```JSON
{
  "product_slug": "{product-slug}",
  "business_uuid": "{business_uuid}",
  "business_group_uuid": "{business_group_uuid}",
  "application_uuid": "{application_uuid}"
}
```

Response:
- 200

02. Get list of Activation Requests

GET /v1/product-activation-request

Request Query:
```
```

Response:
- 200
```JSON
"data": [
  {
    "uuid": "uuid",
    "approvable_uuid": "kyb-uuid",
    "business_uuid": "{business_uuid}",
    ...approvables_properties,
    "additional_data": {
      ...additional_properties_associated
    }
  },
  ...
]
```

03. Update Activation Request (only for properties)

PUT v1/product-activation-request/{uuid}

Request Body:
```JSON
{
  "property_key": "{property_value}"
}
```

04. Support existing all operations on approveables

PUT v1/approvables/{uuid}

Request Body:
```JSON
{
  "assignee_uuid": "uuid", // Used to assign analysts to 
  "state_code": "state_code", // Existing state code to support maker-checker-flow
  "process_state_code": "process_state_code", // Existing process state code to support maker-checker-flow
}
```

#### Events
```
UpdateProductActivationRequest {
  state: {state},
  type: {activation_request_type},
  business_uuid: {business_uuid},
  ...other_related_details
}

ActivationRequestSuccess {
  type: {activation_request_type},
  business_uuid: {business_uuid},
  ...other_related_details
}
```

## Low Level Design

### Database Design

```
table product_activation_types {
  id: int: auto incrementing,
  uuid: unique generated uuid,
  slug: unique slug
}

table product_activation_requests {
  id: int: auto incrementing,
  uuid: unique generated uuid,
  type_id: product_activation_types.id
  approveable_id: approvables.id,
}

table approvables {
  id: int: auto incrementing,
  uuid: unique generated uuid,
  type: enum(product_activation_request, kyb, kyc),
  model_type: model_type,
  model_uuid: uuid of model,
  analyst_uuid: uuid,
  state_code: string,
  process_state_code: string,
}

// this is a existing feature with generics to track property and values across this project
table properties {
  id: int: auto incrementing,
  uuid: unique generated uuid,
  property_key: string, // unique slug
}

table property_values { 
  id: int: auto incrementing,
  uuid: unique generated uuid,
  property_id: properties.id,
  model_uuid: uuid, // uuid of model, `activation_request_uuid`
  model_type: string, // type of model, `activation_request`
  value: json
}
```

## 