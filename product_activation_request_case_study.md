

# Product Activation Request
A product activation request is a concept, in which a product feature is activated to a customer / business when the request is approved.

## Background
At "Aspire FT" we build many features and products for our users.
Since some features or abilities require a level of compliance from our customers. We created a mechanism to enforce/enable product activation by a request and approve flow.

## Why is it needed
- We needed a common framework for our analyst - customer interactions for non-KYC/KYB activities.
- We knew the existing KYC / KYB flow are inadequate to handle such requests. But the underlying concepts like maker/checker flow (state machine), missing info flows and history tracking are inherited from KYB flow.

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
user -> app: Completes {action}
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
dash -> api: PUT v1/product-activation-request/{uuid}
api -> service: PUT v1/product-activation-request/{uuid}

service -> api: Submit `UpdateProductActivationRequest` event
note right api
    UpdateProductActivationRequest {
        type: success,
        ...other_details
    }
end note
api -> events: Publish `ActivationRequestSuccessful`

@enduml
```

### APIs and Events

01. Create Activation Request 

POST /v1/product-activation-request

Request Body:
```JSON
{
  "type": "{type}",
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
    ...kyb_properties,
    "additional_data": {
      ...additional_properties_associated
    }
  },
  ...
]
```

03. Update Activation Request

PUT v1/product-activation-request/{uuid}

Request Body:
```JSON
{
  "state": "{new_target_state}",
  "process_state": "{new_target_process_state}",
  "analyst_uuid": "{uuid of analyst to be assigned}"
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
```