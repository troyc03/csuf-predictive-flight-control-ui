Universal Telemetry Ingestion Node & Polymorphic Overlay

MVP Specification

1. Product Overview

The Universal Telemetry Ingestion Node & Polymorphic Overlay is a network-based telemetry visualization system that accepts standardized JSON telemetry packets over WebSockets and automatically renders the incoming data as customizable, hardware-accelerated broadcast graphics.

The MVP is designed around a universal pipeline rather than a single telemetry source.

For the initial client use case:

LAX Flight Telemetry
        │
        ▼
External Adapter Script
        │
        │ Universal JSON
        ▼
WebSocket Ingestion Node
        │
        ▼
Telemetry State Engine
        │
        ▼
Polymorphic Overlay Renderer
        │
        ▼
Broadcast Card / Output

The same pipeline should support substantially different telemetry sources without modifying the core ingestion or rendering engine.

Example:

Aircraft telemetry ─┐
Car OBD-II adapter ─┼──► Universal JSON ──► Ingestion Node ──► Overlay
Animal GPS tracker ─┘

The source-specific extraction and protocol handling occurs outside this system.

---

2. MVP Goal

Build a working telemetry-to-graphics pipeline that can:

1. Accept telemetry data over WebSockets.
2. Validate incoming packets against a universal JSON schema.
3. Normalize telemetry into a common internal representation.
4. Maintain the latest state for each telemetry entity/stream.
5. Handle telemetry sources operating at substantially different update rates.
6. Dynamically map telemetry fields to a customizable broadcast card.
7. Render the card using hardware-accelerated graphics where supported.
8. Update the rendered output without requiring manual intervention.
9. Support multiple simultaneous telemetry streams.
10. Remain stable during an extended multi-rate stress test.

The MVP succeeds when a new telemetry source can be connected primarily by writing an external adapter that converts its data into the universal schema.

---

3. Initial Use Case

LAX Flight Telemetry

The first demonstration will use an external script that obtains flight telemetry associated with LAX and converts it into the project's universal JSON format.

The external script is responsible for:

- Obtaining flight data.
- Handling the source's API/protocol.
- Parsing source-specific fields.
- Converting values into the universal schema.
- Establishing the WebSocket connection.
- Sending telemetry packets to the ingestion node.

The MVP is responsible for everything after the standardized JSON packet reaches the WebSocket endpoint.

---

4. Non-Goals

The following are explicitly outside the MVP scope.

4.1 Hardware Drivers

The system will not directly interface with:

- Bluetooth OBD-II adapters
- USB OBD-II devices
- GPS collar hardware
- Radio receivers
- Aircraft transponders
- CAN bus hardware
- Serial telemetry hardware
- Proprietary tracking hardware

---

4.2 Proprietary Protocol Decoding

The system will not implement source-specific protocol decoders for:

- OBD-II
- CAN
- ADS-B
- GPS collar protocols
- Radio telemetry
- Vendor-specific APIs

An external adapter is responsible for converting source-specific data into the universal JSON format.

---

4.3 Source Acquisition

The MVP does not need to know where telemetry originated.

For example, the ingestion server should not contain logic such as:

if source == "aircraft":
    parse aircraft protocol

if source == "vehicle":
    parse OBD-II

if source == "animal":
    parse GPS collar

Instead:

External Source Adapter
        ↓
Universal JSON
        ↓
Generic Ingestion Pipeline

---

4.4 Full Broadcast Automation

The MVP does not attempt to become a complete broadcast automation suite.

Features such as:

- Playlist management
- Multi-camera switching
- Production control
- Replay systems
- Audio mixing
- Broadcast scheduling

are outside the initial scope.

---

5. Core Architecture

The MVP should consist of five logical components.

┌─────────────────────────────────────────────────────┐
│                  External Data Source               │
│                                                     │
│  Aircraft / Vehicle / Animal / Other                │
└───────────────────────┬─────────────────────────────┘
                        │
                        ▼
┌─────────────────────────────────────────────────────┐
│                  Source Adapter                      │
│                                                     │
│  Source-specific parsing and normalization           │
└───────────────────────┬─────────────────────────────┘
                        │
                        │ Universal JSON
                        ▼
┌─────────────────────────────────────────────────────┐
│              WebSocket Ingestion Node                │
│                                                     │
│  Connection management                               │
│  Validation                                          │
│  Authentication (if enabled)                         │
│  Packet acceptance                                   │
└───────────────────────┬─────────────────────────────┘
                        │
                        ▼
┌─────────────────────────────────────────────────────┐
│                 Telemetry State Engine               │
│                                                     │
│  Entity state                                        │
│  Timestamps                                          │
│  Stream lifecycle                                    │
│  Rate-independent state updates                      │
└───────────────────────┬─────────────────────────────┘
                        │
                        ▼
┌─────────────────────────────────────────────────────┐
│              Polymorphic Overlay Renderer            │
│                                                     │
│  Data → visual components                            │
│  Templates                                           │
│  Animations                                          │
│  Hardware acceleration                               │
└───────────────────────┬─────────────────────────────┘
                        │
                        ▼
                Broadcast Output

---

6. Universal Telemetry Schema

The MVP requires a standardized JSON envelope.

The schema should separate transport metadata from telemetry payload data.

Example:

{
  "schema_version": "1.0",
  "stream_id": "lax-demo",
  "entity_id": "UAL123",
  "entity_type": "aircraft",
  "timestamp": "2026-10-02T20:15:30.125Z",
  "sequence": 18423,
  "data": {
    "latitude": 33.9425,
    "longitude": -118.4081,
    "altitude": 5200,
    "speed": 182,
    "heading": 271,
    "callsign": "UAL123"
  }
}

---

7. Required Packet Fields

Every telemetry packet should contain:

Field| Type| Required| Purpose
"schema_version"| string| Yes| Identifies schema version
"stream_id"| string| Yes| Identifies telemetry stream
"entity_id"| string| Yes| Identifies tracked entity
"entity_type"| string| Yes| Describes entity category
"timestamp"| ISO-8601| Yes| Source/event timestamp
"sequence"| integer| Recommended| Detects ordering/dropped packets
"data"| object| Yes| Telemetry payload

The "data" object remains polymorphic.

For example, an aircraft might send:

{
  "altitude": 5200,
  "speed": 182,
  "heading": 271
}

A vehicle might send:

{
  "rpm": 3200,
  "speed": 67,
  "coolant_temperature": 91
}

An animal tracker might send:

{
  "latitude": 34.123,
  "longitude": -117.321,
  "battery": 82
}

The core ingestion engine does not need to understand the semantic meaning of every field.

---

8. WebSocket Ingestion

The MVP exposes a WebSocket endpoint:

ws://<host>/telemetry

Production deployments should use:

wss://<host>/telemetry

The server should accept multiple concurrent WebSocket connections.

Each connection may publish:

- One stream
- Multiple entities
- Multiple entity types

The ingestion layer must avoid coupling the lifetime of one telemetry source to another.

---

9. Packet Processing Pipeline

Each received packet should follow this process:

WebSocket Message
       │
       ▼
Parse JSON
       │
       ▼
Validate Envelope
       │
       ▼
Validate Required Fields
       │
       ▼
Validate Timestamp / Sequence
       │
       ▼
Update Entity State
       │
       ▼
Notify Renderer

Invalid packets should be rejected without crashing the connection or server.

Example error:

{
  "error": "INVALID_PACKET",
  "message": "Missing required field: entity_id"
}

---

10. State Management

The system should maintain the latest known state for each entity.

Conceptually:

state[stream_id][entity_id]

Example:

state
└── lax-demo
    ├── UAL123
    ├── DAL456
    └── AAL789

Each entity state should include:

- Latest telemetry data
- Last update timestamp
- Last received timestamp
- Sequence number
- Stream identifier
- Entity type
- Connection/source metadata
- Optional stale-state status

The state engine should be rate independent.

A source sending:

30 packets/second

and a source sending:

1 packet/10 minutes

should both be represented using the same state model.

---

11. Handling Different Data Velocities

The renderer must not assume that every telemetry source updates continuously.

High-frequency source

Example:

Vehicle
30 updates/sec

The telemetry state may change 30 times per second, but the renderer should not necessarily perform 30 complete redraw operations per second.

---

Standard-frequency source

Example:

Aircraft
1–10 updates/sec

The renderer should reflect incoming changes while maintaining smooth visual output.

---

Slow source

Example:

Animal tracker
1 update / 10 minutes

The most recently received state should remain visible until:

1. A newer packet arrives, or
2. The configured stale-data threshold is reached.

The UI should distinguish between:

LIVE

and:

STALE

where appropriate.

---

12. Render Loop

Telemetry reception and rendering should be decoupled.

Do not make the WebSocket receive handler directly responsible for rendering.

Preferred architecture:

                  ┌──────────────────┐
                  │ WebSocket Server │
                  └────────┬─────────┘
                           │
                           ▼
                  ┌──────────────────┐
                  │ State Store      │
                  └────────┬─────────┘
                           │
                           ▼
                  ┌──────────────────┐
                  │ Render Loop      │
                  └────────┬─────────┘
                           │
                           ▼
                  ┌──────────────────┐
                  │ GPU Renderer     │
                  └──────────────────┘

This separation is critical to the MVP's variable-rate telemetry assumption.

A burst of incoming telemetry should not cause an equivalent burst of rendering work.

---

13. Frame Coalescing

If multiple telemetry packets arrive between render frames, the renderer should generally consume the latest state rather than rendering every intermediate state.

For example:

Packet 1
Packet 2
Packet 3
Packet 4
Packet 5
      │
      ▼
Latest State
      │
      ▼
Next Render Frame

This prevents high-frequency telemetry from creating an unnecessary render backlog.

The system should prioritize:

«Current state over historical render events.»

The MVP does not require perfect visual representation of every intermediate telemetry packet.

---

14. Polymorphic Overlay System

The renderer should use a generic component/template model.

A broadcast card should be constructed from reusable visual components.

Example:

┌───────────────────────────────────────────┐
│ UAL123                         LIVE ●      │
│                                           │
│ ALT       SPEED       HDG                 │
│ 5,200     182 MPH     271°                │
│                                           │
│ LAT 33.9425     LON -118.4081             │
└───────────────────────────────────────────┘

The data source should not determine the layout.

Instead, a template defines which fields are displayed.

Example conceptual configuration:

{
  "template": "telemetry-card",
  "fields": [
    {
      "label": "ALT",
      "path": "data.altitude",
      "format": "number"
    },
    {
      "label": "SPEED",
      "path": "data.speed",
      "format": "number"
    },
    {
      "label": "HDG",
      "path": "data.heading",
      "format": "degrees"
    }
  ]
}

The same mechanism could render:

AIRCRAFT
ALT / SPEED / HEADING

or:

VEHICLE
RPM / SPEED / COOLANT

without changing the underlying ingestion engine.

---

15. Hardware-Accelerated Rendering

The rendering layer should use a GPU-accelerated graphics API or framework appropriate to the selected implementation platform.

The MVP should establish:

- A stable render loop.
- GPU-backed drawing.
- Texture-based assets where appropriate.
- Efficient text rendering.
- Minimal CPU/GPU synchronization.
- Decoupling between telemetry ingestion and frame rendering.

The specific graphics technology is an implementation decision and should not affect the universal telemetry schema.

---

16. MVP Configuration

The MVP should support a configuration defining:

server:
  websocket_port: 8080

render:
  width: 1920
  height: 1080
  target_fps: 60

telemetry:
  stale_after_ms: 30000

overlay:
  template: telemetry-card

Configuration values should not be hard-coded into the telemetry processing logic.

---

17. Stream Management

The system should be capable of handling multiple simultaneous streams.

Example test configuration:

Stream A
Aircraft
10 updates/sec

Stream B
Vehicle
30 updates/sec

Stream C
Animal
1 update/10 minutes

Stream D
Synthetic
100 updates/sec

Each stream should maintain independent state.

A problem with one stream must not stop rendering for other streams.

---

18. Error Handling

The MVP should gracefully handle:

Invalid JSON

Receive → Parse failure → Reject packet

Invalid schema

Receive → Validation failure → Reject packet

Out-of-order packet

The system should use timestamps and/or sequence numbers to determine whether an update should replace current state.

Duplicate packet

Duplicate packets should not cause duplicate entities or unnecessary state growth.

Disconnected client

The server should clean up connection-specific resources.

Reconnecting client

A reconnecting source should be able to resume publishing without requiring a server restart.

---

19. Observability

The MVP should expose enough information to diagnose performance problems.

At minimum, track:

Active WebSocket connections
Active telemetry streams
Active entities
Packets received
Packets accepted
Packets rejected
Packets/sec
Render FPS
Dropped frames
Renderer errors
Memory usage
CPU usage
GPU usage (where available)

A basic development diagnostics overlay or console output is sufficient for the MVP.

---

20. Performance Requirements

The MVP should target:

Rendering

Target: 60 FPS

with a stable frame rate under the expected test load.

Telemetry

The system must support simultaneous:

High-frequency streams
Standard-frequency streams
Slow streams

without one rate causing another to stall.

Memory

Memory usage should remain bounded during continuous operation.

The system must not create an unbounded history of telemetry packets unless historical storage is explicitly enabled.

The default behavior should be:

Latest state
    +
minimal metadata

rather than:

Entire telemetry history in memory

---

21. Killer Assumption

Assumption

A single universal telemetry schema and state-management system can handle telemetry sources with dramatically different update velocities without causing:

- Rendering jitter
- State synchronization lockups
- Stream lag
- Memory leaks
- Unbounded queues
- Dropped frames
- Cross-stream interference

This assumption is central to the MVP.

---

22. Validation Experiment

The primary MVP validation will be an 8-hour multi-stream simulation.

The simulation will generate multiple synthetic telemetry sources.

Example workload

Stream| Type| Frequency
Stream A| Vehicle| 30 Hz
Stream B| Aircraft| 10 Hz
Stream C| Aircraft| 1 Hz
Stream D| Animal| 1 / 10 min
Stream E| Burst test| Variable

The exact stream counts can be adjusted as implementation capacity allows.

---

23. Eight-Hour Stability Test

The test should run continuously for:

8 hours

During the test, monitor:

- Process memory
- CPU utilization
- GPU utilization
- Render FPS
- Frame-time variance
- Packets received
- Packets rejected
- Queue depth
- Active streams
- Active entities
- WebSocket connection state
- Renderer errors

---

24. Stress Events

The simulator should intentionally introduce:

- High-frequency bursts
- Slow streams
- Temporary disconnections
- Reconnection
- Duplicate packets
- Out-of-order packets
- Invalid packets
- Sudden increases in stream count
- Sudden decreases in stream count

The purpose is to test whether the architecture remains stable when telemetry conditions change.

---

25. MVP Acceptance Criteria

The MVP is considered functional when all of the following are true:

Ingestion

- [ ] WebSocket endpoint accepts telemetry packets.
- [ ] Multiple clients can connect simultaneously.
- [ ] JSON packets are parsed successfully.
- [ ] Invalid packets are rejected safely.
- [ ] Schema version is validated.
- [ ] Required fields are validated.

State

- [ ] Multiple streams can coexist.
- [ ] Multiple entities can coexist within a stream.
- [ ] Latest state is maintained correctly.
- [ ] Sequence/timestamp handling prevents stale updates from overwriting newer state.
- [ ] Slow telemetry sources remain represented after their last update.
- [ ] Stale telemetry can be identified.

Rendering

- [ ] Telemetry state is rendered into a broadcast card.
- [ ] Overlay fields can be mapped to arbitrary telemetry paths.
- [ ] Rendering is independent of source-specific protocols.
- [ ] Rendering remains responsive during high-frequency ingestion.
- [ ] Target render rate is maintained under the defined test workload.

Performance

- [ ] High-frequency telemetry does not create an unbounded render queue.
- [ ] Memory usage remains bounded during the 8-hour test.
- [ ] No sustained frame-rate degradation occurs during the test.
- [ ] Slow streams do not block high-frequency streams.
- [ ] High-frequency streams do not block slow streams.

Extensibility

- [ ] A new telemetry source can be integrated through an external adapter.
- [ ] Adding a new source does not require changes to the core ingestion pipeline.
- [ ] At least two substantially different telemetry schemas can be demonstrated using the same rendering engine.

---

26. MVP Demo

The final MVP demonstration should show three conceptual sources feeding the same system.

                ┌──────────────────────┐
                │  Aircraft Adapter    │
                └──────────┬───────────┘
                           │
                ┌──────────▼───────────┐
                │                      │
                │ Universal Telemetry  │
                │      WebSocket       │
                │       Server         │
                │                      │
                └──────────┬───────────┘
                           │
            ┌──────────────┼──────────────┐
            │              │              │
            ▼              ▼              ▼
       Aircraft         Vehicle         Animal
         State            State           State
            │              │              │
            └──────────────┼──────────────┘
                           ▼
                  Polymorphic Renderer
                           │
                           ▼
                    Broadcast Output

The key demonstration is that the renderer and ingestion engine do not care which telemetry source produced the data.

---

27. Suggested Repository Structure

/
├── README.md
├── docs/
│   └── MVP.md
│
├── server/
│   ├── websocket/
│   ├── validation/
│   ├── state/
│   └── telemetry/
│
├── renderer/
│   ├── components/
│   ├── templates/
│   ├── assets/
│   └── render-loop/
│
├── schema/
│   └── telemetry.schema.json
│
├── simulator/
│   ├── vehicle/
│   ├── aircraft/
│   ├── animal/
│   └── stress/
│
├── adapters/
│   └── lax/
│
├── config/
│   └── development.yaml
│
└── tests/
    ├── ingestion/
    ├── state/
    ├── rendering/
    └── stress/

---

28. Implementation Order

The MVP should be built in the following order.

Phase 1 — Schema

Create and validate the universal telemetry schema.

Deliverable:

telemetry.schema.json

---

Phase 2 — WebSocket Ingestion

Implement:

- WebSocket server
- JSON parsing
- Schema validation
- Error handling

Deliverable:

A client can send a valid telemetry packet and receive confirmation that it was accepted.

---

Phase 3 — State Engine

Implement:

- Stream state
- Entity state
- Timestamp handling
- Sequence handling
- Stale-state detection

Deliverable:

The server can maintain multiple telemetry entities at different update rates.

---

Phase 4 — Renderer

Implement:

- Render loop
- GPU-accelerated drawing
- Basic telemetry card
- Dynamic field mapping

Deliverable:

A telemetry packet produces a visible broadcast card.

---

Phase 5 — Integration

Connect the LAX external adapter.

Deliverable:

LAX telemetry
      ↓
External adapter
      ↓
WebSocket
      ↓
Universal telemetry engine
      ↓
Broadcast graphic

---

Phase 6 — Simulator

Implement synthetic sources representing:

- 30 Hz telemetry
- 10 Hz telemetry
- 1 Hz telemetry
- Slow telemetry
- Burst traffic

Deliverable:

A repeatable load-test environment.

---

Phase 7 — Stability Test

Run the complete system for eight hours.

Record:

- Memory
- CPU
- GPU
- FPS
- Frame timing
- Packet rates
- Errors
- Connection state

Deliverable:

An 8-hour stability-test report demonstrating whether the killer assumption holds under the defined workload.

---

29. Definition of Done

The MVP is complete when:

«A standardized telemetry JSON packet can enter the system over WebSockets, update an entity's state, and automatically produce a hardware-accelerated broadcast graphic without the core system knowing whether the underlying data originated from an aircraft, vehicle, animal tracker, or another telemetry source.»

The final proof is an extended multi-stream test demonstrating that high-frequency, normal-frequency, and slow telemetry can coexist without causing uncontrolled memory growth, state synchronization failures, or sustained rendering degradation.

---

30. Future Expansion

These features may be considered after MVP validation:

- Authentication
- TLS configuration
- Persistent telemetry history
- Replay
- Multiple output resolutions
- NDI/SDI integration
- Browser-based overlay editor
- Remote template management
- Alert/trigger system
- Telemetry interpolation
- Multiple simultaneous overlays
- Recording/export
- Distributed ingestion nodes
- Horizontal scaling
- Source adapter SDK
- Plugin architecture

These should remain separate from the core MVP unless required to validate the central architecture.
