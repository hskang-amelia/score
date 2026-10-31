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

Tracing Assumption of Use Requirements
######################################

.. document:: Tracing Assumption of Use Requirements
   :id: doc__feature_tracing_aou_reqs
   :status: draft
   :version: 1
   :security: YES
   :safety: ASIL_B
   :realizes: wp__requirements_feat_aou[version==1]

The tracing framework protects safety-related producers by design (see the "Freedom from Interference"
requirements in :ref:`tracing_requirements`). Some of that protection depends on how the system integrator
deploys the trace daemon and configures the operating system. These assumptions state what the integrator has to
provide.

.. aou_req:: Execution Priority of the Trace Daemon
   :id: aou_req__feature_tracing__daemon_priority
   :reqtype: Non-Functional
   :security: NO
   :safety: ASIL_B
   :status: valid
   :version: 1

   The system integrator shall run the trace daemon with a scheduling priority and CPU budget such that it cannot
   delay safety-related processes beyond their timing requirements.

   Note: This assumption supports :need:`feat_req__tracing__non_blocking` and
   :need:`feat_req__tracing__bounded_overhead`, which bound only the trace call inside the producer.

.. aou_req:: Read-Only Access Rights to Producer Shared Memory
   :id: aou_req__feature_tracing__shm_read_only
   :reqtype: Non-Functional
   :security: YES
   :safety: ASIL_B
   :status: valid
   :version: 1

   The system integrator shall configure the operating system access rights so that the trace daemon can open the
   shared-memory objects of producers for reading only.

   Note: This assumption enforces :need:`feat_req__tracing__read_only` at operating system level.

.. aou_req:: Protection of Recorded Trace Data
   :id: aou_req__feature_tracing__data_protection
   :reqtype: Non-Functional
   :security: YES
   :safety: QM
   :status: valid
   :version: 1

   The system integrator shall restrict read access to the configured trace outputs, including stored trace files,
   to users and tools that are authorized to see the communication payload.

   Note: This assumption supports :need:`feat_req__tracing__access_control`.

.. aou_req:: Dimensioning of Trace Buffers
   :id: aou_req__feature_tracing__buffer_dimensioning
   :reqtype: Non-Functional
   :security: NO
   :safety: QM
   :status: valid
   :version: 1

   The system integrator shall dimension the trace buffers for the expected trace rate of the enabled trace points.

   Note: If the buffers are too small, traces are dropped and reported as lost, as required by
   :need:`feat_req__tracing__non_blocking` and :need:`feat_req__tracing__data_loss`. Producers are not affected.

.. aou_req:: Trace Configuration on a Safe Filesystem
   :id: aou_req__feature_tracing__config_safe_fs
   :reqtype: Non-Functional
   :security: NO
   :safety: ASIL_B
   :status: valid
   :version: 1

   The system integrator shall store any tracing configuration that safety-related producers read at runtime, such
   as the trace filter configuration, on a filesystem that is certified for the highest safety level of those
   producers.
