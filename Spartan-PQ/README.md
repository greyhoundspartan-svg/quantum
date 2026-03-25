# Spartan-PQ or Spartan_Greyhound : Spartan with Greyhound Post-Quantum PCS

Spartan-PQ is a variant of the Spartan proof system that is **specialized for post-quantum security** by instantiating Spartan with the **Greyhound lattice-based polynomial commitment scheme (PCS)**.

This directory documents and packages the post-quantum Spartan work in this repository, tying together:

- **Spartan2** as the sum-check-based zkSNARK / folding framework.
- **Greyhound PCS** as the underlying lattice-based commitment scheme.
- The **Greyhound integration work** described in `GREYHOUND_INTEGRATION.md` and `GREYHOUND_REPLACEMENT_COMPLETE.md`.

The goal of Spartan-PQ is to provide a **high-speed zkSNARK with no trusted setup and post-quantum assumptions**, suitable as a building block for privacy-preserving protocols, verifiable computation, and MPC-friendly systems.

## What Spartan-PQ Builds On

- **Spartan protocol**  
  Spartan is a sum-check-based zkSNARK with an efficient prover and succinct verification. It supports multiple arithmetizations (R1CS, Plonkish, AIR, CCS) and naturally separates **witness-independent** and **witness-dependent** work, enabling offloading and MPC-friendly proving.

- **Greyhound PCS (lattice-based, post-quantum)**  
  In Spartan-PQ, the multilinear polynomial commitment layer is provided by **Greyhound**, a **network-based, lattice-backed PCS**. This replaces curve-based or hash-based PCS choices and targets **post-quantum security** by working over polynomial rings instead of elliptic curves.

- **NeutronNova-style folding (non-recursive)**  
  We retain the non-recursive NeutronNova-style folding to aggregate many R1CS instances into one, and then prove the folded instance using Spartan instantiated with Greyhound.

## Current Status of the Greyhound Integration

The Greyhound PCS integration is tracked in:

- `GREYHOUND_INTEGRATION.md` – architectural summary of plugging Greyhound into Spartan2.
- `GREYHOUND_REPLACEMENT_COMPLETE.md` – detailed status of the Hyrax → Greyhound replacement and remaining production work.

**High-level status:**

- **Structure and compilation**: The core Greyhound FFI bindings and PCS trait implementation are in place and compile.
-  **Engines wired in**: `*HyraxEngine` types have been replaced by `*GreyhoundEngine` in the provider layer (with deprecated aliases kept for compatibility).
-  **Production hardening needed**:
  - Memory management around FFI (RAII wrappers, proper clean-up).
  - Serialization of proof structures.
  - Proper multilinear polynomial evaluation and point handling.
  - Comprehensive tests and benchmarks.

For a step-by-step breakdown of what was done and what is left, see `GREYHOUND_REPLACEMENT_COMPLETE.md`.

## Using Spartan-PQ in Code (Conceptual)

Spartan-PQ is exposed through **Greyhound-backed engines** in the Spartan2 provider layer. A typical usage pattern (simplified) is:

```rust
use spartan2::provider::PallasGreyhoundEngine;
use spartan2::spartan::SpartanSNARK;
use spartan2::traits::{Engine, snark::R1CSSNARKTrait};

type E = PallasGreyhoundEngine;

// Setup (given a circuit / constraint system)
let (pk, vk) = SpartanSNARK::<E>::setup(circuit)?; 

// Prove
let proof = SpartanSNARK::<E>::prove(&pk, circuit, &prep_snark, true)?;

// Verify
proof.verify(&vk)?;
```

Concrete examples and engine wiring live in the `Spartan2-main` crate (e.g., the `sha256.rs` example updated to `T256GreyhoundEngine`). Spartan-PQ reuses these components but focuses on the **post-quantum, Greyhound-backed configuration**.

## Design Highlights

- **Post-quantum focus**: Lattice-based PCS (Greyhound) instead of elliptic-curve assumptions.
- **No trusted setup**: Follows Spartan’s philosophy of avoiding universal trusted setup.
- **MPC-friendly proving**: Prover work naturally splits into delegatable witness-independent and MPC-suitable witness-dependent phases.
- **Folding for batching**: Non-recursive NeutronNova folding to amortize proving cost across many instances.

## Roadmap for This Directory

Planned improvements in the context of this project:

- Add **examples** that demonstrate end-to-end Spartan-PQ flows (setup, prove, verify) using Greyhound-backed engines.
- Add **benchmarks** to compare Greyhound-based Spartan against curve-based variants.
- Harden **FFI safety**, **memory management**, and **serialization** for production readiness (see the “Next Steps for Production” sections in the Greyhound docs).
- Extend documentation with **API-level reference** once the interface stabilizes.

