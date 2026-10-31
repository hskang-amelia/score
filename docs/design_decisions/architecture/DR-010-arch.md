<!--
Copyright (c) 2026 Contributors to the Eclipse Foundation

See the NOTICE file(s) distributed with this work for additional
information regarding copyright ownership.

This program and the accompanying materials are made available under the
terms of the Apache License Version 2.0 which is available at
https://www.apache.org/licenses/LICENSE-2.0

SPDX-License-Identifier: Apache-2.0
-->

(DR-010-Arch)=
# DR-010-arch: Implement the receiving side of communication tracing

- **Date:** 2026-10-31

```{dec_rec} Implement the receiving side of communication tracing
:id: dec_rec__arch__tracing_receiving_side
:status: proposed
:version: 1
:affects: feat__tracing
:context: S-CORE can emit communication trace points through the Generic Trace API, but nothing receives, records or outputs them, so the Tracing feature has no implementation and no feature requirements.
:decision: Implement the Generic Trace Library behind the existing Generic Trace API and a separate trace daemon that reads traced data read-only from shared memory and writes it to configurable outputs, specified by the Tracing feature requirements.
```

---

## Context / Problem

The stakeholder requirements ask for tracing of communication events
({need}`stkh_req__dev_experience__tracing_of_comm`, ASIL_B, valid from v1.0.0),
and later of execution events ({need}`stkh_req__dev_experience__tracing_of_exec`)
and memory events ({need}`stkh_req__dev_experience__tracing_of_memory`).
The communication feature derives one requirement from them,
{need}`feat_req__com__tracing`: binding-agnostic, zero-copy, read-only tracing of
communication.

Today only one side of this exists:

- **Producer side, implemented.** `score/mw/com/impl/tracing` in the communication
  module calls trace points for skeleton and proxy events, fields and methods. It is
  configured through the `tracing` section of the mw::com configuration (global
  enable, application instance id, trace filter configuration file, per service
  element enable).
- **Interface, implemented.** `score/analysis/tracing/generic_trace_library` in
  baselibs defines the Generic Trace API: client registration per binding type,
  registration of shared-memory objects, trace calls for shared-memory and
  local-memory data with meta information (`AraComMetaInfo`, `DltMetaInfo`), and a
  completion callback per client. Its documentation and error codes already assume
  a trace daemon and a shared-memory ring buffer (for example `kDaemonNotConnectedFatal`,
  `kRingBufferFullRecoverable`).
- **Receiving side, missing.** The only implementation of the Generic Trace API is
  `stub_implementation`, which returns success and records nothing. There is no
  daemon, no output and no tool to read traces.

The documentation mirrors this: {need}`feat__tracing`, the logical interface
`logic_arc_int__tracing__tracing`, module `mod__tracing` and component
`comp__tracing` exist, but the Tracing feature has no feature requirements and no
component requirements.

As a result, an integrator who enables tracing in the mw::com configuration gets
no trace data, and the zero-copy guarantees in {need}`feat_req__com__tracing` are
not exercised by any implementation.

## Decision

Implement the receiving side of tracing behind the existing Generic Trace API,
without changing that API, and specify it with feature requirements under
`docs/features/log_and_trace/tracing/requirements`. The proposed requirements are
part of this proposal.

### Logical architecture

```
 producer process (ASIL_B or QM)                trace daemon process (QM)
+-----------------------------------+          +------------------------------+
| mw::com (LoLa binding)            |          |                              |
|   trace points                    |          |  ring buffer consumer        |
|        |                          |          |        |                     |
|        v                          |          |        v                     |
| Generic Trace Library  -----------+--------->|  read-only mapping of        |
|  - client / shm registration      | meta info|  registered shm objects      |
|  - ring buffer producer           | via shm  |        |                     |
|  - completion callbacks <---------+----------|  output backends             |
+-----------------------------------+  done    |  (file, DLT, ...)            |
                                               +------------------------------+
```

1. **Generic Trace Library** (replaces `stub_implementation` in baselibs).
   Runs in the producer process. Registers clients and shared-memory objects with
   the daemon, writes one ring-buffer entry per trace call holding the meta
   information and the list of data chunks, and invokes the client's completion
   callback when the daemon reports that it has finished with an entry. Data in
   local memory is copied into library-owned shared memory; data already in
   shared memory is referenced, not copied.
2. **Trace daemon** (new component of `mod__tracing`). A separate QM process.
   Maps registered shared-memory objects read-only, consumes ring-buffer entries,
   timestamps and serializes them, and passes them to output backends. It never
   blocks a producer.
3. **Output backends.** A file backend in a documented binary format is the first
   deliverable, together with a converter into a format that existing analysis
   tools read. A DLT backend can follow, since `DltMetaInfo` already exists in the
   API.

### Freedom from interference

The producer side may run in ASIL_B processes, so the library must not let the
QM daemon affect them:

- A trace call never blocks. If no ring-buffer entry is free, the trace is dropped
  and an error is returned to the producer, which continues.
- The daemon accesses producer memory read-only.
- If the daemon is not running, not reachable or fails, producers keep operating
  with tracing disabled.
- The execution time of a trace call has a documented upper bound that does not
  depend on the size of the traced data.

### Feature requirements

This proposal adds 17 feature requirements (`feat_req__tracing__*`), grouped as
communication tracing, configuration and output, freedom from interference,
security, and execution and memory tracing. Requirements derived from the ASIL_B
stakeholder requirement are ASIL_B, because QM requirements cannot be derived
from ASIL requirements. Execution and memory tracing are QM and target v2.0.0.

### Placement

- The Generic Trace Library stays where its API is, in baselibs
  (`score/analysis/tracing/generic_trace_library`).
- The trace daemon and output backends are developed in an incubator repository
  (`inc_tracing`) during Phase 1, and become the implementation repository of
  `mod__tracing` if this proposal is accepted.

### Limitations and out of scope

- No change to the Generic Trace API or to the trace points in mw::com.
- Tracing across ECUs and time synchronization between ECUs.
- A graphical analysis tool; the converter targets existing tools.
- Bindings other than LoLa are supported through the same API but not tested by
  this proposal.

## Backwards Compatibility

None of the existing interfaces change. Builds that do not enable the new
implementation keep linking the stub and behave as today. Enabling tracing in the
mw::com configuration starts producing data only when the daemon runs.

## Alternatives Considered

### Use an existing tracer (LTTng, Perfetto, ftrace) directly from mw::com

Advantages: mature tooling and visualization; no own daemon.

Disadvantages: the producer would copy payload into the tracer's buffers, which
contradicts the zero-copy requirement; the available tracers differ between Linux
and QNX; and the producer side would bind to one tracer. These tracers remain
useful as output formats for the converter.

### Copy traced data inside the producer

Advantages: simpler library; the daemon would not need access to producer memory.

Disadvantages: trace cost grows with payload size, which breaks the bounded-overhead
requirement for safety-related producers, and it contradicts
{need}`feat_req__com__tracing`.

### Record traces in the producer process without a daemon

Advantages: no inter-process protocol.

Disadvantages: output and storage would run inside ASIL_B processes, and every
producer would need its own output configuration.

## Rationale

The API and the producer side already encode the intended design (shared-memory
registration, ring buffer, completion callbacks, a daemon). Implementing the
receiving side behind that API makes the existing trace points useful without
touching safety-related code paths, and keeps the QM recording and output out of
the producer processes.

## Consequences

- The Tracing feature gets requirements and an implementation path; the stub
  remains for builds without tracing.
- `feat__tracing` is declared QM while its logical interface, module and component
  are ASIL_B. With the ASIL_B requirements of this proposal, docs-as-code reports
  that safety requirements are allocated to a QM feature. The safety level of
  `feat__tracing` has to be aligned as part of this proposal.
- `comp__tracing` needs component requirements that refine the feature
  requirements.

## Open Questions

- Can the recording and output requirements be decomposed to QM, keeping only the
  freedom-from-interference requirements at ASIL_B?
- Which open format should the converter target first (for example CTF or the
  Perfetto trace format)?
- Should the trace daemon support QNX in the first release?
- Is an incubator repository wanted, or should the daemon start in an existing
  module?
