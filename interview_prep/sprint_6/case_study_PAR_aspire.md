# Case Study: Product Activation Requests @ Aspire FT

# Background

Aspire FT is a multi-national scale up which provides business operating systems for business (from startups to large orgs).

As part of Onboarding Team, there was a requirement to create a scalable, multi-purpose system, which would act as a feature flag plus requirement check for different features.

# Problem

The existing system already had mechanism for approving KYB and KYC of the customers using maker-checker flow.

But a more scalable system/framework is required for evaluating the requirements for any feature and later approved by analysts. (using same maker/checker flow that we had for KYC/B).

# Goal

The goal is to build a system which act as source of truth and requirement evaluator for any feature we release as a org. And this will also be used to relieve the requirement overload on the onboarding team to progressively onboard the businesses/users based on their qualifications.

There is also to evaluate requirements on case to case basis based on each feature. So as to not overwhelm the onboarding business to over asking for requirements. Instead of we can ask only required for the feature.

# Constraints

- The Product Activation Request (PAR) needs to support existing maker-checker mechanism.
- The requirements for each features need to configurable. (via API, no - hard coding)

# Solution

## Existing system
- we already had system which exposed APIs and with state machines and requirements evaluator
- we already had system to ask/demand additional details from our users using missing-info flow that should be supported

### State machines
- State machines controls the transition of kybs's `process_state` parameters. It will allow or reject transition between the states.

## Requirements evaluator
- Requirements evaluator evaluates the transitions between the states. 
- Query the requirements from the DB. and evaluate them against the entity (business / user).- return the result per requirement. similar to this
```JSON
{
  "pob_available": true,
  "aml_completed": false,
  ...
}
```

## Proposed System

### Product Types - Configuration
- we needed a table to maintain different types of `PARs`. and their configurations.
- Store the requirements per type.

### Product Activation Requests
- Store the details on each individual product activation request.
- Attach to existing kybs (table wise) by foreign key.
- store additional data

### APIs and events
- Expose new APIs to create and retrieve PARs 
- Expose APIs to create Product Types. (with configuration)

- Dispatch Kafka events for each transition.


# Interesting Engineering Challenges

# Implementation

# Testing & Reliability

# Outcome / Impact