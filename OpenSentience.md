OpenSentience Specification v1.0


Table of Contents

1. Introduction
   1.1. Overview
   1.2. Conformance
    1.3. Relation to Other Standards
2. Context Attributes
    2.1. Required Attributes
   2.2. Optional Attributes
3. Event Data
   3.1. Data Attribute
   3.2. Event Types
4. Core Event Types
    4.1. Inference Event (dev.opensentience.inference.v1)
   4.2. Drift Event (dev.opensentience.drift.v1)
    4.3. Guardrail Violation Event (dev.opensentience.guardrail.violation.v1)
5. Extensibility
   5.1. Extension Context Attributes
6. Protocol Bindings
    6.1. HTTP Binding
8. Security Considerations
   7.1. The Trust Envelope


1. Introduction


1.1. Overview

This specification defines the format of OpenSentience events. An "event" is a data record expressing an occurrence within a cognitive system and its context. The OpenSentience format contains two types of information: Context Attributes providing metadata about the occurrence, and Event Data representing the occurrence itself.

1.2. Conformance

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in(https://www.rfc-editor.org/rfc/rfc2119.html).

1.3. Relation to Other Standards

* CloudEvents: The OpenSentience envelope is heavily inspired by and designed for compatibility with the CNCF CloudEvents v1.0 specification.3 An OpenSentience event can be transported as the data payload of a CloudEvent.
* OpenTelemetry: OpenTelemetry excels at describing the operational performance of application code (traces, metrics, logs).1 OpenSentience is complementary, focusing on the behavioral performance of the AI model itself. An OpenSentience event can be attached as an attribute or event to an OpenTelemetry span to enrich a trace with cognitive context.

2. Context Attributes


This section defines the metadata attributes that provide context for an event.

2.1. Required Attributes

The following attributes MUST be present in every OpenSentience event.
* specversion
    * Type: String
    * Description: The version of the OpenSentience specification which the event uses. For this version, it MUST be "1.0".
* id
    * Type: String
    * Description: A unique identifier for the event. The combination of id and source MUST be unique for each distinct event.
* source
    * Type: URI-reference
    * Description: Identifies the context in which an event happened. This is often the service, application, or model that produced the event.
* type
    * Type: String
    * Description: A string describing the type of event related to the originating occurrence. This specification defines three core event types. The format is reverse-DNS notation.

2.2. Optional Attributes

The following attributes MAY be present in an OpenSentience event.
* subject
    * Type: String
    * Description: Describes the subject of the event in the context of the event producer (identified by source). This is often a user ID, session ID, or trace ID that the event pertains to.
* time
    * Type: Timestamp
    * Description: Timestamp of when the occurrence happened. MUST adhere to the format specified in(https://www.rfc-editor.org/rfc/rfc3339.html).
* dataschema
    * Type: URI
    * Description: A URI that identifies the schema that the data attribute adheres to.

3. Event Data


3.1. Data Attribute

An OpenSentience event MAY include domain-specific information about the occurrence. If present, this information MUST be encapsulated within a data attribute. The structure of the data object is determined by the event's type.

3.2. Event Types

This specification defines the following core event types:
* dev.opensentience.inference.v1
* dev.opensentience.drift.v1
* dev.opensentience.guardrail.violation.v1

4. Core Event Types


4.1. Inference Event (dev.opensentience.inference.v1)

This event represents a single inference or decision-making cycle of an AI model. It is used to monitor performance, usage, and basic operational health.
data Schema:
YAML

{
  "latency_ms": Integer,            # REQUIRED. Total time for the inference in milliseconds.
  "input_hash": String,             # OPTIONAL. A cryptographic hash (e.g., "sha256:...") of the input payload.
  "output_hash": String,            # OPTIONAL. A cryptographic hash of the output payload.
  "token_counts": {                 # OPTIONAL. For language models.
    "input": Integer,
    "output": Integer
  },
  "confidence_score": Float,        # OPTIONAL. A score between 0.0 and 1.0 representing the model's confidence.
  "custom_metrics": {               # OPTIONAL. A key-value map for any other model-specific metrics.
    "metric_name": Number | String
  }
}

4.2. Drift Event (dev.opensentience.drift.v1)

This event represents the detection of a statistically significant drift in the distribution of a feature or model output, compared to a baseline.
data Schema:
YAML

{
  "feature_name": String,           # REQUIRED. The name of the feature or output being monitored (e.g., "user_age", "model_output_confidence").
  "baseline_id": String,            # REQUIRED. An identifier for the baseline dataset used for comparison.
  "drift_type": String,             # REQUIRED. Enum: "feature" or "prediction".
  "statistical_test": {             # REQUIRED. Details of the statistical test performed.
    "name": String,                 # REQUIRED. Enum: "jensen_shannon_divergence", "kolmogorov_smirnov", "population_stability_index". [5, 6]
    "value": Float,                 # REQUIRED. The calculated value from the statistical test.
    "threshold": Float,             # REQUIRED. The threshold at which drift was declared.
    "p_value": Float                # OPTIONAL. The p-value from the statistical test, if applicable.
  }
}

4.3. Guardrail Violation Event (dev.opensentience.guardrail.violation.v1)

This event represents the detection of a model behavior that violates a predefined policy or guardrail.
data Schema:
YAML

{
  "policy_id": String,              # REQUIRED. A unique identifier for the policy that was violated.
  "violation_type": String,         # REQUIRED. A high-level category for the violation. Enum: "toxicity", "pii_leakage", "bias", "factual_inconsistency", "custom".
  "severity": String,               # REQUIRED. The severity of the violation. Enum: "info", "low", "medium", "high", "critical".
  "details": String,                # OPTIONAL. A human-readable description of the violation.
  "violated_text": String           # OPTIONAL. The specific text or output that triggered the violation.
}

5. Extensibility


5.1. Extension Context Attributes

OpenSentience events MAY include additional context attributes not defined in this specification. These "extension attributes" MUST consist of lowercase alphanumeric characters and MUST NOT exceed 20 characters in length. To prevent naming collisions, it is RECOMMENDED that extensions use a prefix that is unique to the producer.

6. Protocol Bindings


6.1. HTTP Binding

This section defines how OpenSentience events are mapped to HTTP messages.
* Structured Content Mode: The entire event is encoded in the HTTP request body using a media type like application/opensentience+json.
* Binary Content Mode: The data attribute is placed in the HTTP request body, and all other Context Attributes are mapped to HTTP headers, prefixed with os-. For example, the id attribute becomes the os-id HTTP header.

7. Security Considerations


7.1. The Trust Envelope

While the transport protocol (e.g., TLS) provides channel security, it does not guarantee the integrity or authenticity of the event itself once it reaches the consumer. It is RECOMMENDED that producers sign OpenSentience events using a standard like JSON Web Signature (JWS). This creates a "Trust Envelope," allowing consumers to verify that the event was emitted by a trusted source and has not been tampered with in transit.
