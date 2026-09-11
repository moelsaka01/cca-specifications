# MIP-001 Reference Examples

## Purpose

These fixtures provide canonical-byte and semantic validation vectors for
MIP-001. Valid `.mip` files are compact RFC 8785 JSON and therefore appear on
one line. Pretty-printing a fixture changes its bytes and makes it noncanonical.

## Valid fixtures

| Fixture | Bytes | File SHA-256 | Package digest | Cognition digest |
|---|---:|---|---|---|
| [`minimal/minimal-observation.mip`](minimal/minimal-observation.mip) | 2,518 | `36cdf11919c58727cc3b5ec0293ffbb1716b57c35899cf06e1a92b130b6b9483` | `0333b3e8dd64e14cc1b114546e49f221e95662e695d45e507b7d87ba8b2178ae` | `80d486e661f8eea7cdd6c257fc706ffa729647879f891be13604e6bc177836e3` |
| [`complete/complete-investigation.mip`](complete/complete-investigation.mip) | 19,519 | `176b81ef6caecdd6db19caf5ae4e0cdd89fe1f30b43f79d485eb72c6e3b96f32` | `0b786c02d37435efe1729d002428ca044175cedb83a9e262c24e18b2f73f1b15` | `b434df4e88219aeefe722ddb6e658b7e156ec319e8f1b15b73c40a7820305dab` |
| [`extended/noncritical-extension.mip`](extended/noncritical-extension.mip) | 2,642 | `824fbab80067a0f915cdceb8bc8cfc24e01978301603e1e22b490369e2934712` | `eded105fd02ca5c66eb03a5aecc995f1560449846d673f0c8a6e7aaf7887a655` | `80d486e661f8eea7cdd6c257fc706ffa729647879f891be13604e6bc177836e3` |

The extended fixture has the same cognition digest as the minimal fixture and
a different package digest. This proves the two digest scopes are distinct.

## Semantic story in the complete fixture

```text
Observation A
  evidence ltm-001
    -> semantic concept semantic-001
    -> retrieval candidate retrieval-001
    -> Reflection reflection-001

Observation B
  evidence ltm-001
  evidence ltm-002 (added)
    -> semantic concept semantic-001
    -> retrieval candidate retrieval-001
    -> Reflection reflection-001

Evolution
  + ltm-002
  + evidence relationship from ltm-002 to semantic-001
  ~ semantic-001 provenance revision (removed old + added new)

Comparative Reconstruction
  shared route, B-only evidence and relationship, one modified semantic
  transformation, deterministic reconvergence
```

The story is deliberately small. It verifies reference closure and derived
artifact rules without introducing UI, renderer, Runtime, Provider, or
generated-explanation content.

## Hash verification

For each fixture, verify section digests in the exact order from MIP-001
section 14. Then recompute cognition and package commitments with domain
separation and compare them to `integrity`.

Do not use the historical compact MemoryOS display fingerprint. It is not a
cryptographic checksum and is not part of MIP-001.

## Invalid fixtures

Negative-vector definitions and expected primary diagnostics are in
[`invalid/README.md`](invalid/README.md). Some lexical-invalid cases cannot be
represented by a valid JSON Schema instance and are described as byte-level
mutations of the valid fixtures.
