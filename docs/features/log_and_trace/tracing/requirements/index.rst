..
   # *******************************************************************************
   # Copyright (c) 2026 Contributors to the Eclipse Foundation
   #
   # See the NOTICE file(s) distributed with this work for additional
   # information regarding copyright ownership.
   #
   # This program and the accompanying materials are made available under the
   # terms of the Apache License Version 2.0 which is available at
   # https://www.apache.org/licenses/LICENSE-2.0
   #
   # SPDX-License-Identifier: Apache-2.0
   # *******************************************************************************

.. _tracing_requirements:

Tracing Requirements
####################

.. document:: Tracing Requirements
   :id: doc__tracing_requirements
   :status: draft
   :version: 1
   :safety: ASIL_B
   :security: YES
   :realizes: wp__requirements_feat[version==1]

Tracing records runtime events of the platform, together with the data they carry, for timing analysis,
debugging and the reconstruction of communication flows. The producer side already exists: the communication
framework calls trace points through the Generic Trace API (``score/analysis/tracing/generic_trace_library`` in
baselibs). These requirements describe the tracing framework that receives, records and outputs those traces.

All requirements derived from the ASIL_B stakeholder requirement for communication tracing are ASIL_B, as QM
requirements cannot be derived from ASIL requirements. The ones under "Freedom from Interference" protect the
safety-related producers; whether the recording and output functions can be decomposed to QM is open.

Communication Tracing
=====================

.. feat_req:: Tracing of Communication Events
   :id: feat_req__tracing__comm_events
   :reqtype: Functional
   :security: NO
   :safety: ASIL_B
   :derived_from: stkh_req__dev_experience__tracing_of_comm[version==1]
   :satisfied_by: feat__tracing[version==1]
   :status: valid
   :version: 1
   :valid_from: v2.0.0

   The tracing framework shall record the communication events reported by the communication framework, at least
   event send and receive, field update, get and set, method call and result, and subscribe and unsubscribe, on the
   skeleton and the proxy side.

.. feat_req:: Binding-Agnostic Tracing
   :id: feat_req__tracing__binding_agnostic
   :reqtype: Functional
   :security: NO
   :safety: ASIL_B
   :derived_from: stkh_req__dev_experience__tracing_of_comm[version==1]
   :satisfied_by: feat__tracing[version==1]
   :status: valid
   :version: 1
   :valid_from: v2.0.0

   The tracing framework shall accept traces from any communication binding through one common interface and shall
   record the binding type with each trace record.

.. feat_req:: Zero-Copy Tracing of Shared-Memory Data
   :id: feat_req__tracing__zero_copy
   :reqtype: Functional
   :security: NO
   :safety: ASIL_B
   :derived_from: stkh_req__dev_experience__tracing_of_comm[version==1]
   :satisfied_by: feat__tracing[version==1]
   :status: valid
   :version: 1
   :valid_from: v2.0.0

   The tracing framework shall record data located in a registered shared-memory region without the producer copying
   that data; only the trace meta information shall be copied.

.. feat_req:: Tracing of Local-Memory Data
   :id: feat_req__tracing__local_data
   :reqtype: Functional
   :security: NO
   :safety: ASIL_B
   :derived_from: stkh_req__dev_experience__tracing_of_comm[version==1]
   :satisfied_by: feat__tracing[version==1]
   :status: valid
   :version: 1
   :valid_from: v2.0.0

   The tracing framework shall record data located in the producer's local memory by copying it into memory owned by
   the tracing framework.

.. feat_req:: Trace Completion Notification
   :id: feat_req__tracing__done_notification
   :reqtype: Functional
   :security: NO
   :safety: ASIL_B
   :derived_from: stkh_req__dev_experience__tracing_of_comm[version==1]
   :satisfied_by: feat__tracing[version==1]
   :status: valid
   :version: 1
   :valid_from: v2.0.0

   The tracing framework shall notify the producer, identified by the context id the producer supplied, once it no
   longer accesses the shared-memory data of a trace.

.. feat_req:: Trace Record Meta Information
   :id: feat_req__tracing__meta_info
   :reqtype: Functional
   :security: NO
   :safety: ASIL_B
   :derived_from: stkh_req__dev_experience__tracing_of_comm[version==1]
   :satisfied_by: feat__tracing[version==1]
   :status: valid
   :version: 1
   :valid_from: v2.0.0

   Each trace record shall contain the trace point type, the identifiers of the service, instance and service
   element, the identifier of the producing application instance and a timestamp.

.. feat_req:: Trace Timestamps
   :id: feat_req__tracing__timestamp
   :reqtype: Functional
   :security: NO
   :safety: ASIL_B
   :derived_from: stkh_req__dev_experience__tracing_of_comm[version==1]
   :satisfied_by: feat__tracing[version==1]
   :status: valid
   :version: 1
   :valid_from: v2.0.0

   The tracing framework shall timestamp trace records from a monotonic clock that is common to all processes on a
   computation unit, so that records of different processes can be ordered.

Configuration and Output
========================

.. feat_req:: Trace Point Configuration
   :id: feat_req__tracing__configuration
   :reqtype: Functional
   :security: NO
   :safety: ASIL_B
   :derived_from: stkh_req__dev_experience__tracing_of_comm[version==1]
   :satisfied_by: feat__tracing[version==1]
   :status: valid
   :version: 1
   :valid_from: v2.0.0

   The tracing framework shall allow enabling and disabling tracing per service element and per trace point type by
   configuration, without changing application code.

.. feat_req:: Trace Output
   :id: feat_req__tracing__output
   :reqtype: Functional
   :security: NO
   :safety: ASIL_B
   :derived_from: stkh_req__dev_experience__tracing_of_comm[version==1]
   :satisfied_by: feat__tracing[version==1]
   :status: valid
   :version: 1
   :valid_from: v2.0.0

   The tracing framework shall forward trace records to at least one configurable output channel in a documented
   format that offline analysis tools can read.

.. feat_req:: Indication of Trace Data Loss
   :id: feat_req__tracing__data_loss
   :reqtype: Functional
   :security: NO
   :safety: ASIL_B
   :derived_from: stkh_req__dev_experience__tracing_of_comm[version==1]
   :satisfied_by: feat__tracing[version==1]
   :status: valid
   :version: 1
   :valid_from: v2.0.0

   The tracing framework shall indicate in its output when trace records were dropped and how many.

Freedom from Interference
=========================

These requirements bound what the tracing framework itself does inside a producer. Their effect on the whole
system also depends on the deployment, which the system integrator provides through the assumptions of use
:need:`aou_req__feature_tracing__daemon_priority`, :need:`aou_req__feature_tracing__shm_read_only` and
:need:`aou_req__feature_tracing__config_safe_fs`.

.. feat_req:: Non-Blocking Trace Calls
   :id: feat_req__tracing__non_blocking
   :reqtype: Functional
   :security: NO
   :safety: ASIL_B
   :derived_from: stkh_req__dev_experience__tracing_of_comm[version==1], stkh_req__communication__safe[version==1]
   :satisfied_by: feat__tracing[version==1]
   :status: valid
   :version: 1
   :valid_from: v2.0.0

   A trace call shall not block the producer; when no trace buffer is available, the trace shall be dropped and the
   call shall return an error to the producer.

.. feat_req:: Read-Only Access to Producer Memory
   :id: feat_req__tracing__read_only
   :reqtype: Functional
   :security: YES
   :safety: ASIL_B
   :derived_from: stkh_req__dev_experience__tracing_of_comm[version==1], stkh_req__communication__safe[version==1]
   :satisfied_by: feat__tracing[version==1]
   :status: valid
   :version: 1
   :valid_from: v2.0.0

   The tracing framework shall access the shared-memory regions of a producer read-only.

.. feat_req:: Tracing Unavailability
   :id: feat_req__tracing__unavailability
   :reqtype: Functional
   :security: NO
   :safety: ASIL_B
   :derived_from: stkh_req__dev_experience__tracing_of_comm[version==1], stkh_req__communication__safe[version==1]
   :satisfied_by: feat__tracing[version==1]
   :status: valid
   :version: 1
   :valid_from: v2.0.0

   When the tracing framework is not running, not reachable or fails, the producer shall continue to operate with
   tracing disabled, and the failure shall be reported to the producer as an error.

.. feat_req:: Bounded Trace Call Overhead
   :id: feat_req__tracing__bounded_overhead
   :reqtype: Non-Functional
   :security: NO
   :safety: ASIL_B
   :derived_from: stkh_req__dev_experience__tracing_of_comm[version==1], stkh_req__communication__safe[version==1]
   :satisfied_by: feat__tracing[version==1]
   :status: valid
   :version: 1
   :valid_from: v2.0.0

   The execution time of a trace call in the producer shall have a documented upper bound that is independent of the
   size of the traced data.

Security
========

.. feat_req:: Access Control for Trace Data
   :id: feat_req__tracing__access_control
   :reqtype: Functional
   :security: YES
   :safety: ASIL_B
   :derived_from: stkh_req__dev_experience__tracing_of_comm[version==1]
   :satisfied_by: feat__tracing[version==1]
   :status: valid
   :version: 1
   :valid_from: v2.0.0

   The tracing framework shall make recorded trace data, which may contain communication payload, accessible only to
   consumers that are authorized by configuration.

Execution and Memory Tracing
============================

.. feat_req:: Tracing of Execution Events
   :id: feat_req__tracing__exec_events
   :reqtype: Functional
   :security: NO
   :safety: QM
   :derived_from: stkh_req__dev_experience__tracing_of_exec[version==1]
   :satisfied_by: feat__tracing[version==1]
   :status: valid
   :version: 1
   :valid_from: v2.0.0

   The tracing framework shall record start and stop events of executable entities and threads, so that they can be
   correlated with communication trace records on the same time base.

.. feat_req:: Tracing of Memory Events
   :id: feat_req__tracing__memory_events
   :reqtype: Functional
   :security: NO
   :safety: QM
   :derived_from: stkh_req__dev_experience__tracing_of_memory[version==1]
   :satisfied_by: feat__tracing[version==1]
   :status: valid
   :version: 1
   :valid_from: v2.0.0

   The tracing framework shall record memory allocation, copy and de-allocation events reported by the platform.
