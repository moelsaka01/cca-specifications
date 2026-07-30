---
id: API-002-HPP
title: Process Public C++ API
version: "1.0"
status: Draft
derived_from:
  - SP-003
  - CCA-PROC-1.0 requirements
  - API-002
---

# API-002-HPP Process Public C++ API

This document is the sole normative C++ declaration contract for IM-005 and
has precedence over `api.md`. All public declarations MUST match these
interfaces. Private members, private constructors, friends, and implementation
helpers remain implementation-defined.

The declarations MAY be partitioned into focused headers below
`<cca/process/>`. The implementation MUST also provide the umbrella header
`<cca/process/process.hpp>` containing the complete public surface.

```cpp
#pragma once

#include <cca/representation/representation.hpp>

#include <cstddef>
#include <memory>
#include <string>
#include <string_view>
#include <vector>

namespace cca::process
{

class ProcessDefinition;
class ExecutionContext;
class ExecutionResult;
class ProcessEngine;

enum class ExecutionState
{
    Ready,
    Running,
    Completed,
    Failed
};

class ProcessDefinition
{
public:

    explicit ProcessDefinition(
        const cca::representation::RepresentationDocument& document);

    ~ProcessDefinition();

    ProcessDefinition(
        const ProcessDefinition& other);

    ProcessDefinition&
    operator=(
        const ProcessDefinition& other);

    ProcessDefinition(
        ProcessDefinition&& other) noexcept;

    ProcessDefinition&
    operator=(
        ProcessDefinition&& other) noexcept;

    const std::vector<cca::representation::RepresentationId>&
    executionOrder() const noexcept;

    std::size_t
    entityCount() const noexcept;

    std::size_t
    relationshipCount() const noexcept;

    std::size_t
    propertyCount() const noexcept;

private:

    class Impl;

    std::unique_ptr<Impl> impl_;

    friend class ProcessEngine;

};

class ExecutionContext
{
public:

    explicit ExecutionContext(
        ProcessDefinition definition);

    ~ExecutionContext();

    ExecutionContext(
        const ExecutionContext&) = delete;

    ExecutionContext&
    operator=(
        const ExecutionContext&) = delete;

    ExecutionContext(
        ExecutionContext&&) = delete;

    ExecutionContext&
    operator=(
        ExecutionContext&&) = delete;

    const ProcessDefinition&
    definition() const noexcept;

    ExecutionState
    state() const noexcept;

    const std::vector<cca::representation::RepresentationId>&
    trace() const noexcept;

private:

    class Impl;

    std::unique_ptr<Impl> impl_;

    friend class ProcessEngine;

};

class ExecutionResult
{
public:

    enum class Code
    {
        Success,
        InvalidRepresentation
    };

    ExecutionResult(
        const ExecutionResult&) = default;

    ExecutionResult&
    operator=(
        const ExecutionResult&) = default;

    ExecutionResult(
        ExecutionResult&&) noexcept = default;

    ExecutionResult&
    operator=(
        ExecutionResult&&) noexcept = default;

    ~ExecutionResult() = default;

    bool
    succeeded() const noexcept;

    ExecutionState
    state() const noexcept;

    Code
    code() const noexcept;

    std::string_view
    message() const noexcept;

    const std::vector<cca::representation::RepresentationId>&
    trace() const noexcept;

    const std::vector<cca::representation::Diagnostic>&
    diagnostics() const noexcept;

private:

    ExecutionResult(
        ExecutionState state,
        Code code,
        std::string message,
        std::vector<cca::representation::RepresentationId> trace,
        std::vector<cca::representation::Diagnostic> diagnostics);

    ExecutionState state_;
    Code code_;
    std::string message_;
    std::vector<cca::representation::RepresentationId> trace_;
    std::vector<cca::representation::Diagnostic> diagnostics_;

    friend class ProcessEngine;

};

class ProcessEngine
{
public:

    ProcessEngine() noexcept = default;

    ExecutionResult
    execute(
        const cca::representation::RepresentationDocument& document) const;

    ExecutionResult
    execute(
        const ProcessDefinition& definition) const;

    ExecutionResult
    execute(
        ExecutionContext& context) const;

};

} // namespace cca::process
```

`ProcessDefinition` is the only public construction path from a document to an
owned definition. It has no default constructor.

`ExecutionContext` has no default constructor and cannot be copied or moved.
Its identity remains bound to one execution attempt.

`ExecutionResult` has no public default constructor. `ProcessEngine` creates
terminal result values.

`ProcessEngine` is concrete and default-constructible so the required direct
example is valid. A Runtime-hosted implementation exposes this same public
surface through a typed ExactlyOne Service Contract while keeping its Provider
type private.

The behavioral contract for these declarations is defined by `api.md`.
