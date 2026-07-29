---
id: API-001
title: Representation Public API
version: 2.0.0
status: Draft
derived_from:
  - SP-002
  - RR-001
---

# API-001 Representation Public API

## Purpose

This document defines the normative public C++ API for the Representation Foundation.

It specifies the public types, operations, ownership model, diagnostics, and behavioral contracts required by all conforming implementations.

Internal implementation is intentionally unspecified.

---

# Namespace

```cpp
namespace cca::representation
```

---

# Common Types

```cpp
using EntityCollection =
    std::vector<std::reference_wrapper<const RepresentationEntity>>;

using RelationshipCollection =
    std::vector<std::reference_wrapper<const RepresentationRelationship>>;

using PropertyCollection =
    std::vector<std::reference_wrapper<const RepresentationProperty>>;
```

Collections shall preserve insertion order.

---

# RepresentationId

```cpp
class RepresentationId
{
public:

    static RepresentationId generate();

    std::string toString() const;

    bool operator==(const RepresentationId&) const;

    bool operator!=(const RepresentationId&) const;

    bool operator<(const RepresentationId&) const;

private:

    /* implementation-defined */

};
```

Identifiers are immutable.

Identifiers are never reused.

Rolled-back identifiers remain retired.

---

# RepresentationType

```cpp
class RepresentationType
{
public:

    std::string name() const;

    bool operator==(const RepresentationType&) const;
};
```

---

# RepresentationValue

Supported value kinds:

```cpp
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
```

Public API:

```cpp
class RepresentationValue
{
public:

    ValueKind kind() const;

    bool asBoolean() const;

    int64_t asInteger() const;

    double asFloatingPoint() const;

    std::string asString() const;

    RepresentationId asIdentifier() const;

    std::vector<RepresentationValue> asCollection() const;
};
```

Calling an incompatible accessor shall throw
`std::bad_variant_access`.

---

# RepresentationProperty

```cpp
class RepresentationProperty
{
public:

    const RepresentationId& id() const;

    std::string_view name() const;

    const RepresentationType& type() const;

    const RepresentationValue& value() const;
};
```

---

# RepresentationEntity

```cpp
class RepresentationEntity
{
public:

    const RepresentationId& id() const;

    const RepresentationType& type() const;

    const PropertyCollection& properties() const;

    const RepresentationProperty&
    property(std::string_view name) const;
};
```

---

# RepresentationRelationship

```cpp
class RepresentationRelationship
{
public:

    const RepresentationId& id() const;

    const RepresentationType& type() const;

    const RepresentationEntity& source() const;

    const RepresentationEntity& target() const;

    const PropertyCollection& properties() const;
};
```

---

# RepresentationMetadata

```cpp
class RepresentationMetadata
{
public:

    std::string author() const;

    std::string version() const;

    std::string provenance() const;
};
```

---

# RepresentationDocument

```cpp
class RepresentationDocument
{
public:

    RepresentationEntity&
    createEntity(
        const RepresentationType&);

    RepresentationRelationship&
    createRelationship(
        RepresentationEntity& source,
        RepresentationEntity& target,
        const RepresentationType&);

    void
    removeEntity(
        const RepresentationId&);

    void
    removeRelationship(
        const RepresentationId&);

    const EntityCollection&
    entities() const;

    const RelationshipCollection&
    relationships() const;

    const RepresentationMetadata&
    metadata() const;

    bool
    isFrozen() const;
};
```

The document owns all semantic objects.

Returned references remain valid until:

- removal
- rollback
- document destruction

---

# Diagnostics

```cpp
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

    TransactionError,

    ValidationError
};

struct Diagnostic
{
    DiagnosticCode code;

    DiagnosticSeverity severity;

    std::string message;
};
```

Diagnostics shall be returned in deterministic order.

---

# Validation

```cpp
struct ValidationResult
{
    bool valid;

    std::vector<Diagnostic> diagnostics;
};
```

```cpp
class ValidationService
{
public:

    ValidationResult
    validate(
        const RepresentationDocument&) const;
};
```

Validation never modifies semantic state.

A successful validation does not change the document lifecycle.

---

# Transactions

```cpp
class Transaction
{
public:

    void commit();

    void rollback();

    bool active() const;
};
```

```cpp
class TransactionService
{
public:

    Transaction
    begin(
        RepresentationDocument&);
};
```

Rules:

- only one active transaction per document
- nested transactions are not supported
- failed commit leaves transaction active
- rollback restores previous state
- rolled-back identifiers remain retired

---

# Freeze

```cpp
class FreezeService
{
public:

    ValidationResult
    freeze(
        RepresentationDocument&);
};
```

Freeze performs:

1. validation

2. lifecycle transition

If validation fails:

the document remains mutable.

If validation succeeds:

the document becomes frozen.

---

# Query Service

```cpp
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
```

Rules:

- nullptr indicates "not found"
- results preserve insertion order
- queries never invalidate references
- queries never modify semantic state

---

# Ownership Rules

- RepresentationDocument owns all semantic objects.
- Services own no semantic state.
- Returned references remain valid until object removal, rollback, or document destruction.
- Callers never own semantic objects.

---

# Thread Safety

No thread-safety guarantees are provided.

Concurrent access is outside the scope of IM-004.

---

# Exceptions

Public API shall throw only standard C++ exceptions.

No implementation-specific exception hierarchy shall be exposed.

---

# ABI

Binary compatibility is not required for IM-004.

Source compatibility is required within version 1.x.
