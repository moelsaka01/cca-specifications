# CCA Constitutional Lexicon 1.0

## Purpose

This lexicon defines the canonical meaning of each constitutional concept. Definitions are authoritative and singular.

## Workspace
**Definition:** A Workspace is the constitutional root that contains and gives context to Domains and their relationships.
**Purpose:** Establish the constitutional root and boundary.
**Relationships:** Contains Domains and contextualizes Assets, Services, Policies, Providers, and Contracts.
**Example:** A product architecture represented as one Workspace containing its Domains.
**Forbidden synonyms:** project, system, application.

## Domain
**Definition:** A Domain is an organized area of semantic concern within a Workspace.
**Purpose:** Organize semantic concerns.
**Relationships:** Contained by a Workspace; organizes Assets, Services, Policies, Providers, and Contracts.
**Example:** A billing Domain grouping billing concerns.
**Forbidden synonyms:** module, namespace, subsystem.

## Asset
**Definition:** An Asset is durable information organized within a Domain.
**Purpose:** Represent durable, semantically identifiable information.
**Relationships:** Belongs to a Domain and may be referenced by other concepts without becoming Service-owned persistent state.
**Example:** A customer record maintained in a customer Domain.
**Forbidden synonyms:** data object, record.

## Service
**Definition:** A Service performs behavior exposed through Contracts and constrained by Policies.
**Purpose:** Perform defined behavior for a Domain.
**Relationships:** Organized in a Domain; fulfills Contracts; may rely on Providers; never owns persistent state.
**Example:** An invoicing Service fulfilling an invoicing Contract.
**Forbidden synonyms:** component, controller, worker.

## Policy
**Definition:** A Policy is a rule that constrains the behavior of Services or Providers.
**Purpose:** Constrain behavior and preserve intended semantics.
**Relationships:** Organized in a Domain and governs behavior defined by Contracts.
**Example:** A retention Policy limiting handling of an Asset.
**Forbidden synonyms:** configuration, preference, guideline.

## Provider
**Definition:** A Provider implements a Contract's required behavior and supplies supporting infrastructure.
**Purpose:** Implement infrastructure required by public behavior.
**Relationships:** Organized in a Domain; implements Contracts; operates under Policies; does not define architecture.
**Example:** A Provider implementing durable Asset access.
**Forbidden synonyms:** adapter, plugin, driver.

## Contract
**Definition:** A Contract is the authoritative definition of public behavior.
**Purpose:** Define behavior dependable across boundaries.
**Relationships:** Organized in a Domain; fulfilled by Services or Providers; may be constrained by Policies.
**Example:** A Contract defining Asset retrieval behavior.
**Forbidden synonyms:** interface, API, protocol.
