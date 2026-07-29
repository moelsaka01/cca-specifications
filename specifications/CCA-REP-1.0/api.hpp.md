---
id: API-001-HPP
title: Representation Public C++ API
version: 1.0.0
status: Draft
derived_from:
  - SP-002
  - RR-001
  - API-001
---

# Representation Public C++ API

> This document is the normative C++ API contract for IM-004.
>
> All public declarations shall match these interfaces.
> Internal implementation remains implementation-defined.

```cpp
#pragma once

#include <cstdint>
#include <functional>
#include <memory>
#include <optional>
#include <string>
#include <string_view>
#include <vector>

namespace cca::representation
{

//----------------------------------------------------------
// Forward declarations
//----------------------------------------------------------

class RepresentationDocument;
class RepresentationEntity;
class RepresentationRelationship;
class RepresentationProperty;
class RepresentationType;
class RepresentationValue;
class RepresentationMetadata;
class RepresentationId;

class ValidationService;
class QueryService;
class TransactionService;
class FreezeService;

class Transaction;

//----------------------------------------------------------
// Collections
//----------------------------------------------------------

using EntityCollection =
    std::vector<std::reference_wrapper<const RepresentationEntity>>;

using RelationshipCollection =
    std::vector<std::reference_wrapper<const RepresentationRelationship>>;

using PropertyCollection =
    std::vector<std::reference_wrapper<const RepresentationProperty>>;

//----------------------------------------------------------
// RepresentationId
//----------------------------------------------------------

class RepresentationId
{
public:

    explicit RepresentationId(std::string value);

    std::string_view value() const noexcept;

    bool operator==(const RepresentationId&) const noexcept;

    bool operator!=(const RepresentationId&) const noexcept;

    bool operator<(const RepresentationId&) const noexcept;

private:

    std::string value_;

};

//----------------------------------------------------------
// RepresentationType
//----------------------------------------------------------

class RepresentationType
{
public:

    explicit RepresentationType(std::string name);

    std::string_view name() const noexcept;

private:

    std::string name_;

};

//----------------------------------------------------------
// Value
//----------------------------------------------------------

enum class ValueKind
{
    Boolean,
    Integer,
    FloatingPoint,
    String,
    Enumeration,
    Identifier,
    Collection
};

//----------------------------------------------------------
// Enumeration
//----------------------------------------------------------

struct Enumeration
{
    std::string value;
};

//----------------------------------------------------------
// Value
//----------------------------------------------------------

enum class ValueKind
{
    Boolean,
    Integer,
    FloatingPoint,
    String,
    Enumeration,
    Identifier,
    Collection
};

class RepresentationValue
{
public:

    RepresentationValue(bool);

    RepresentationValue(int64_t);

    RepresentationValue(double);

    RepresentationValue(std::string);

    RepresentationValue(Enumeration);

    RepresentationValue(RepresentationId);

    RepresentationValue(
        std::vector<RepresentationValue>);

    ValueKind
    kind() const noexcept;

    bool
    asBoolean() const;

    int64_t
    asInteger() const;

    double
    asFloatingPoint() const;

    std::string_view
    asString() const;

    const Enumeration&
    asEnumeration() const;

    const RepresentationId&
    asIdentifier() const;

    const std::vector<RepresentationValue>&
    asCollection() const;
};

//----------------------------------------------------------
// Property
//----------------------------------------------------------

class RepresentationProperty
{
public:

    RepresentationProperty(
        std::string name,
        RepresentationType type,
        RepresentationValue value);

    const RepresentationId&
    id() const noexcept;

    std::string_view
    name() const noexcept;

    const RepresentationType&
    type() const noexcept;

    const RepresentationValue&
    value() const noexcept;

};

//----------------------------------------------------------
// Entity
//----------------------------------------------------------

class RepresentationEntity
{
public:

    const RepresentationId&
    id() const noexcept;

    const RepresentationType&
    type() const noexcept;

    const PropertyCollection&
    properties() const noexcept;

    RepresentationProperty&
    addProperty(
        RepresentationProperty property);

    const RepresentationProperty*
    property(
        std::string_view name) const;

};

//----------------------------------------------------------
// Relationship
//----------------------------------------------------------

class RepresentationRelationship
{
public:

    const RepresentationId&
    id() const noexcept;

    const RepresentationType&
    type() const noexcept;

    const RepresentationEntity&
    source() const noexcept;

    const RepresentationEntity&
    target() const noexcept;

    RepresentationProperty&
    addProperty(
        RepresentationProperty property);

    const PropertyCollection&
    properties() const noexcept;

};

//----------------------------------------------------------
// Metadata
//----------------------------------------------------------

class RepresentationMetadata
{
public:

    std::string_view author() const noexcept;

    std::string_view version() const noexcept;

    std::string_view provenance() const noexcept;

};

//----------------------------------------------------------
// Document
//----------------------------------------------------------

class RepresentationDocument
{
public:

    RepresentationEntity&
    createEntity(
        std::string typeName);

    RepresentationRelationship&
    createRelationship(
        RepresentationEntity& source,
        RepresentationEntity& target,
        std::string relationshipType);

    void
    removeEntity(
        const RepresentationId&);

    void
    removeRelationship(
        const RepresentationId&);

    const EntityCollection&
    entities() const noexcept;

    const RelationshipCollection&
    relationships() const noexcept;

    RepresentationMetadata&
    metadata() noexcept;

    const RepresentationMetadata&
    metadata() const noexcept;

    bool
    isFrozen() const noexcept;

};

//----------------------------------------------------------
// Diagnostics
//----------------------------------------------------------

enum class DiagnosticSeverity
{
    Information,
    Warning,
    Error
};

enum class DiagnosticCode
{
    None,

    DuplicateIdentifier,
    MissingType,
    DuplicateProperty,
    InvalidRelationship,
    FrozenDocument,
    ValidationError,
    TransactionError
};

struct Diagnostic
{
    DiagnosticCode code;
    DiagnosticSeverity severity;
    std::string message;
};

struct ValidationResult
{
    bool valid;
    std::vector<Diagnostic> diagnostics;
};

//----------------------------------------------------------
// Validation
//----------------------------------------------------------

class ValidationService
{
public:

    ValidationResult
    validate(
        const RepresentationDocument&) const;
};

//----------------------------------------------------------
// Transactions
//----------------------------------------------------------

class Transaction
{
public:

    bool active() const noexcept;

    ValidationResult commit();

    void rollback();

};

class TransactionService
{
public:

    Transaction
    begin(
        RepresentationDocument&);
};

//----------------------------------------------------------
// Freeze
//----------------------------------------------------------

class FreezeService
{
public:

    ValidationResult
    freeze(
        RepresentationDocument&);
};

//----------------------------------------------------------
// Queries
//----------------------------------------------------------

class QueryService
{
public:

    const RepresentationEntity*
    findEntity(
        const RepresentationDocument&,
        const RepresentationId&) const;

    const RepresentationRelationship*
    findRelationship(
        const RepresentationDocument&,
        const RepresentationId&) const;

    EntityCollection
    entitiesByType(
        const RepresentationDocument&,
        const RepresentationType&) const;
};

} // namespace cca::representation
```