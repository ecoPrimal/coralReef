# coralReef — Evolution Record

The changelog is sorted by capability surface, not by date.
Each section tells the arc: what emerged, what it replaced, what it became.
The date-sorted history lives in `CHANGELOG_ARCHIVE.md`.

`H(data|epitope) < H(data|date) < H(data)`

---

## WGSL/SPIR-V/PTX Compilation Pipeline — sovereign GPU compiler

coralReef began as a WGSL translator stub; it became a full sovereign compiler emitting PTX for NVIDIA (SM20–SM120), SPIR-V portable output, and AMD ISA encoders.

- **End-to-end WGSL→PTX** (Wave 78→123): SPIR-V portable output verified mesh propagation; ironGate/strandGate live validation on RTX 5070/5090.
- **Naga translation layer** (Wave 68→157d): Full WGSL→IR→PTX with SM70/SM80/SM120 encoder backends; OpRedux UGPR allocation fix for SM80+ scheduler.
- **Multi-input batch compilation** (Wave 125): `shader.compile.multi` — mixed-input batch with per-shader target selection.
- **Hyperbolic & float decomposition** (Wave 67): Pipeline completeness for transcendental ops; hotSpring gap resolution for arch routing.
- **SM120 Blackwell edge cases** (Wave 126): Barrier fix, sovereign SPIR-V, math pack/unpack for newest silicon.
- **Placeholder → ptr_undef** (Wave 157d): SSA naming clarified — intentional `OpUndef` markers, not unfinished code.

---

## GEMM Kernels & Tiling — high-performance matrix multiply

GEMM evolved from global-memory-only Phase 1 to shared-memory tiled kernels with IPC-selectable tiling strategies.

- **GEMM Phase 1** (Wave 157d): Global-memory HMMA kernels; 32 threads per CTA; baseline `shader.compile.gemm` IPC.
- **GEMM Phase 2 smem** (Wave 157d→157f): `compile_gemm_smem()` with `ldmatrix.sync.aligned` + `bar.sync` pipeline; 128 threads, BM=64×BN=16 block tile.
- **Tiling parameter** (Wave 157f): `"auto"` (default), `"global"`, or `"smem"` on IPC; auto-selects Phase 2 when M%64==0 and N%16==0.
- **barraCuda unblocked** (Wave 157f): `dispatch_gemm()` can call `shader.compile.gemm` with `"tiling": "smem"` for high-performance kernels.
- **ptx_emit/gemm split** (Wave 157d): 1308L monolith → `mod.rs` (268) + `phase1.rs` (386) + `phase2.rs` (669).

---

## ISA Encoders & Silicon Targets — per-architecture instruction emission

Encoder coverage expanded from SM70-only to full SM20/SM75/SM80/AMD tables with comprehensive unit tests and hazard modeling.

- **SM20 encoder** (Wave 156l→157f): 49 new unit tests across float ALU, control flow, type conversion, shuffle/permute paths.
- **SM70 FP32/FP64** (Wave 157d): 23 encoder tests — opcode, saturation, FTZ, DNZ, rounding mode, pred dst encoding.
- **SM80 GPR hazards** (Wave 157d→157h): RAW/WAW/WAR hazard tables extracted to `gpr_hazards.rs`; spiller and generate_order tests comprehensive.
- **AMD ops encoder** (Wave 157d): 31 tests covering Exit/Nop/Bar/Bra, Mov/S2R/CS2R, F2F/F2I/I2F/I2I, Ld/St/Copy/MemBar.
- **SM32 peephole** (Wave 157i): EVOLUTION marker — requires IR-level `.sync` flag, deferred to IR extension wave.
- **Integer subgroup correctness** (Wave 157d): Subgroup reduction and integer width correctness validated across silicon fold AAR.

---

## IPC Surface & Shader Compile API — the compiler RPC contract

The IPC surface grew from a handful of compile methods to 19 live JSON-RPC methods with health, capabilities, and provenance.

- **shader.compile.wgsl** (Wave 76→157f): Primary compilation entry; wire contract documented in `SHADER_COMPILE_WIRE_CONTRACT.md`.
- **shader.compile.gemm** (Wave 157d→157f): Tiling strategies, M/N validation, TF32 smem support.
- **shader.compile.multi** (Wave 125): Mixed-input batch with per-shader architecture routing.
- **Health method wired** (Wave 157h): `"health"` dispatch handler — was in gate allowlist but had no handler; 19 IPC methods now live.
- **capabilities.list compliance** (Wave 99): Canonical envelope shape; derived capabilities from hardware probe.
- **RPC self-audit** (Wave 157a): Served methods match dispatch table exactly; registry drift eliminated.

---

## Transport, G65 & BTSP — wire stack for compiler service

Transport evolved from 1285L monolith to modular server with protocol negotiation and encrypted session support.

- **Transport module split** (Wave 157d): 1285L → `mod.rs` (460) + `stream.rs` (318) + `sync_stream.rs` (207) + `resolve.rs` (359).
- **G65 protocol negotiation** (Wave 156p): Single-socket tarpc/JSON-RPC selection per PROTOCOL_NEGOTIATION_SPEC; backward-compatible legacy default.
- **G66 transport abstraction** (Wave 156s): Platform-agnostic bind/accept; Windows Named Pipe readiness.
- **C2 dual-socket convention** (Wave 156j): tarpc UDS alongside JSON-RPC on shared infrastructure.
- **BTSP client handshake** (Wave 151b): Standard ClientHello evolution for bearDog strict mode compatibility.
- **riboCipher compliance** (Wave 113): Genetics-layer prefix acceptance on all accept loops.

---

## Silicon Atheism & Cross-Architecture — compile everywhere

coralReef achieved cross-platform compilation through transport abstraction and platform substrate centralization.

- **Cross-architecture adoption** (Wave 141a→155i): Windows readiness; strandGate live validation; Android UDS adaptation (Wave 133a).
- **G68 platform substrate** (Wave 157a): Centralized symlinks, permissions, UID queries — all inline platform code routed through one module.
- **NUCLEUS achieved** (Wave 155j): Full ecosystem name scrub; zero hardcoded primal names in production code.
- **Hardcoding evolution** (Wave 157d): Last production runtime reference to another primal by name evolved to capability-domain language.
- **BEARDOG_SOCKET deprecated** (Wave 157d): Migration path to `BTSP_PROVIDER_SOCKET`; legacy fallback preserved with `#[deprecated]`.

---

## Artifact Provenance & Signing — trustworthy compiled output

Compiled shaders gained cryptographic provenance tracking for content-addressed deployment and sporePrint integration.

- **Artifact provenance** (Wave 123): Signing + sporePrint BLAKE3 hash on compiled output; deployment traceability.
- **BLAKE3 pure mode** (Wave 157d): Confirmed `features = ["pure"]` — no C toolchain invoked despite upstream build-dep declaration.
- **Gossip injection points** (Wave 157e): `shader.compiled`, `silicon.targets`, `compiler.health` events planned for swarmVine mesh.

---

## Architecture & Engineering Discipline — compiler codebase maturity

The codebase achieved zero-production-unwrap compliance through systematic file splits, test recovery, and dependency trimming.

- **Zero `.unwrap()` in production** (Wave 157h): All 178 unwraps confirmed in `#[cfg(test)]` only.
- **File size compliance** (Wave 124→157i): Zero hand-written files >800 LOC; proactive splits (`sm20/encoder.rs`, `ops/alu_int.rs`, `compile.rs`).
- **BTSP test recovery** (Wave 157g): 46 orphaned session tests re-wired into module tree after extraction drift.
- **G72 dependency pandemic** (Wave 157g): `futures`/`tokio-util` gated behind `tarpc-transport`; dead `tracing` removed from rpc-client.
- **Process leak fix** (Wave 157e): RAII `ChildGuard`/`AsyncChildGuard` prevents test subprocess orphans (~36/hr on southGate).
- **Rust 2024 idioms** (Wave 157h): `#[allow]` → `#[expect]` batch migration; let-chains and let-else adoption.
- **Test scale** (Wave 157i): 3,963 passed, 4 ignored; clippy `--all-features -D warnings` clean; zero unsafe.

---

## Health, Discovery & Neural API — compiler service visibility

Operational and discovery surfaces evolved from undocumented stubs to live health probes with Neural API self-registration.

- **Health readiness** (Wave 119): Derived capabilities from hardware probe; readiness distinguishes compiled vs degraded compiler state.
- **Neural API announce** (Wave 43→44): Outbound `primal.announce` with routing metadata on startup; graceful standalone degradation.
- **C3 health shim** (Wave 156q): Health endpoint verified against ecosystem HEALTH-01 contract for tower atomic gate.
- **Primal startup envelope** (Wave 109): Standard bind-mode, socket namespace, and announce sequence aligned across NUCLEUS primals.
- **Mesh propagation verified** (Wave 78): SPIR-V end-to-end verification across gate mesh — compiled artifacts propagate with provenance intact.

---

*Full date-sorted history: `CHANGELOG_ARCHIVE.md`*
*Last compressed: Wave 172*
