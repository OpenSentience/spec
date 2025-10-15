# spec
OpenSentience™ Specification v1.0
The Open Telemetry Standard for AI Behavioral Trust and Cognitive Transparency


1. Abstract

OpenSentience™ is an open, vendor-neutral specification for describing the behavioral state of AI and other cognitive systems in a common format. It provides a standardized schema for emitting, collecting, and interpreting real-time telemetry signals related to an AI's decision-making process, performance, and adherence to operational guardrails.
By defining a common language for inference metadata, performance drift, and policy violations, OpenSentience enables consistent observability, interoperability, and accountability across the global AI ecosystem. It is designed to be the essential telemetry layer for the next generation of AI assurance, governance, and cognitive security platforms.
This specification is stewarded by the OpenSentience organization and is offered to the community under the Apache 2.0 license.
Mission: To establish a universal, open standard for measuring, communicating, and verifying the trustworthiness of machine intelligence in motion.


2. The Problem: A Babel of AI Signals

As autonomous systems proliferate, they emit a torrent of operational data. However, each model, platform, and vendor describes this data differently. This lack of a common format creates a "Babel of AI Signals," forcing developers to write bespoke logic for every integration and preventing the emergence of universal tooling for AI monitoring, security, and governance.
OpenSentience solves this by providing a common specification for describing event data, inspired by the success of industry standards like CloudEvents for event metadata and OpenTelemetry for application observability.1


3. Core Concepts

OpenSentience defines a standard, extensible envelope for AI telemetry events. Each event contains a set of required context attributes and a data payload that adheres to a specific event type.
Concept	Description	Corresponding Event Type
Inference Telemetry	Captures the contextual details of a single AI decision, including inputs, outputs, latency, and resource consumption.	dev.opensentience.inference.v1
Drift Telemetry	Quantifies statistical deviation in model inputs or outputs over time, comparing a production distribution to a baseline.	dev.opensentience.drift.v1
Guardrail Telemetry	A standardized signal indicating a breach of a predefined operational, safety, or ethical policy.	dev.opensentience.guardrail.violation.v1
Trust Envelope	The secure, signed container for transmitting telemetry events. The specification recommends, but does not mandate, a JWS (JSON Web Signature) format for the envelope to ensure integrity and non-repudiation.	N/A (Transport Layer)

4. Specification Files

This repository contains the formal definition of the OpenSentience standard.
* 📄 /spec/OpenSentience.md — This document. The full v1.0 core specification.
* 📄 /spec/schemas/ — Directory containing formal JSON Schema definitions for all event types.
* 📘 /spec/examples/ — Directory containing valid example payloads for each event type.
* 🧩 /spec/extensions/ — Directory containing documented, community-approved extensions to the core specification.


5. Quick Start


Instrumenting an Application (Python Example)

(Coming soon – Python and Go reference SDKs)
Python

# Pseudocode for a future OpenSentience SDK
from opensentience import Emitter, InferenceEvent

# Initialize the emitter to send events to a collector endpoint
emitter = Emitter(endpoint="https://collector.deepsweep.ai/v1/events")

def process_user_request(prompt: str) -> str:
    start_time = time.time()
    # --- AI Model Inference ---
    result = my_generative_model.invoke(prompt)
    # ------------------------
    end_time = time.time()

    # Create an OpenSentience event
    event = InferenceEvent(
        source_uri="my-app/text-generation-module",
        subject="user-request-12345",
        data={
            "latency_ms": int((end_time - start_time) * 1000),
            "input_hash": "sha256:" + hash(prompt),
            "output_hash": "sha256:" + hash(result.text),
            "token_counts": {
                "input": result.input_tokens,
                "output": result.output_tokens
            }
        }
    )

    # Emit the event
    emitter.emit(event)

    return result.text
