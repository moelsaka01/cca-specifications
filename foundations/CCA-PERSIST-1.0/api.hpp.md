---
id: API-003-HPP
title: Persistence Public C++ API
version: "1.0"
status: Draft
derived_from:
  - SP-004
  - CCA-PERSIST-1.0 requirements
---

# API-003-HPP Persistence Public C++ API

This document is the sole authority for public C++ declarations in
CCA-PERSIST-1.0. Private members and helpers are implementation-defined.

```cpp
#pragma once

#include <cca/process/process.hpp>
#include <cca/representation/representation.hpp>

#include <cstddef>
#include <memory>
#include <string>
#include <string_view>
#include <vector>

namespace cca::persistence
{

enum class PersistenceFormat
{
    Canonical
};

class PersistenceMetadata
{
public:
    PersistenceMetadata() noexcept;
    PersistenceMetadata(std::string workspaceName, std::string description);
    ~PersistenceMetadata();
    PersistenceMetadata(const PersistenceMetadata&);
    PersistenceMetadata& operator=(const PersistenceMetadata&);
    PersistenceMetadata(PersistenceMetadata&&) noexcept;
    PersistenceMetadata& operator=(PersistenceMetadata&&) noexcept;

    const std::string& workspaceName() const noexcept;
    const std::string& description() const noexcept;
};

class ExecutionContextSnapshot
{
public:
    ExecutionContextSnapshot(
        std::size_t processDefinitionIndex,
        cca::process::ExecutionState state,
        std::vector<cca::representation::RepresentationId> trace);
    ~ExecutionContextSnapshot();
    ExecutionContextSnapshot(const ExecutionContextSnapshot&);
    ExecutionContextSnapshot& operator=(const ExecutionContextSnapshot&);
    ExecutionContextSnapshot(ExecutionContextSnapshot&&) noexcept;
    ExecutionContextSnapshot& operator=(ExecutionContextSnapshot&&) noexcept;

    std::size_t processDefinitionIndex() const noexcept;
    cca::process::ExecutionState state() const noexcept;
    const std::vector<cca::representation::RepresentationId>& trace() const noexcept;
};

class PersistencePackage
{
public:
    PersistencePackage(
        const cca::representation::RepresentationDocument& workspace,
        const PersistenceMetadata& metadata,
        std::vector<std::string> policies = {},
        std::vector<cca::process::ProcessDefinition> processDefinitions = {},
        std::vector<ExecutionContextSnapshot> executionContexts = {},
        PersistenceFormat format = PersistenceFormat::Canonical);
    ~PersistencePackage();
    PersistencePackage(const PersistencePackage&);
    PersistencePackage& operator=(const PersistencePackage&);
    PersistencePackage(PersistencePackage&&) noexcept;
    PersistencePackage& operator=(PersistencePackage&&) noexcept;

    const cca::representation::RepresentationDocument& workspace() const noexcept;
    const PersistenceMetadata& metadata() const noexcept;
    const std::vector<std::string>& policies() const noexcept;
    const std::vector<cca::process::ProcessDefinition>& processDefinitions() const noexcept;
    PersistenceFormat format() const noexcept;
    const std::vector<ExecutionContextSnapshot>& executionContexts() const noexcept;

private:
    friend class PersistenceEngine;
    class Impl;
    std::unique_ptr<Impl> impl_;
};

class PersistenceResult
{
public:
    PersistenceResult(PersistenceResult&&) noexcept;
    PersistenceResult& operator=(PersistenceResult&&) noexcept;
    PersistenceResult(const PersistenceResult&) = delete;
    PersistenceResult& operator=(const PersistenceResult&) = delete;
    ~PersistenceResult();

    bool succeeded() const noexcept;
    const std::string& code() const noexcept;
    const std::string& message() const noexcept;
    const PersistencePackage* package() const noexcept;

private:
    PersistenceResult();
    friend class PersistenceEngine;
    class Impl;
    std::unique_ptr<Impl> impl_;
};

class PersistenceEngine
{
public:
    PersistenceEngine() noexcept = default;

    PersistenceResult save(
        const cca::representation::RepresentationDocument& workspace,
        const PersistenceMetadata& metadata,
        const std::vector<std::string>& policies = {},
        const std::vector<cca::process::ProcessDefinition>& processDefinitions = {},
        const std::vector<ExecutionContextSnapshot>& executionContexts = {},
        PersistenceFormat format = PersistenceFormat::Canonical) const;

    PersistenceResult load(const PersistencePackage& package) const;

    PersistenceResult validate(const PersistencePackage& package) const;
};

} // namespace cca::persistence
```
