---
name: Schedule and stream a satellite pass
description: >-
  Reserve an upcoming ground-station pass for a satellite on StellarStation and
  open a live telemetry/command stream for it.
api: grpc/infostellar-stellarstation.proto
service: StellarStationService
operations: [ListUpcomingAvailablePasses, ReservePass, ListPlans, OpenSatelliteStream, CancelPlan]
---

# Schedule and stream a satellite pass

Use the StellarStation gRPC API (`api.stellarstation.com:443`, TLS) to book a
ground-station pass for your satellite and exchange telemetry/commands during it.

## Auth
Register service-account JWT bearer call credentials (private key JSON from the
StellarStation Console) on the gRPC channel; audience `https://api.stellarstation.com`.
See `authentication/infostellar-authentication.yml`.

## Steps
1. `ListUpcomingAvailablePasses` — pass `satellite_id`; returns schedulable passes
   within the next 14 days, each with a `reservation_token`.
2. Pick a pass and `ReservePass` with its `reservation_token`; the response returns
   a `Plan` (status `RESERVED`).
3. `ListPlans` — confirm the plan appears for the satellite in the chosen time window.
4. When the pass begins, `OpenSatelliteStream` (bidirectional): send
   `SendSatelliteCommandsRequest` frames and receive `ReceiveTelemetryResponse`
   frames; acknowledge with `ReceiveTelemetryAck`.
5. To abort, `CancelPlan` with the plan `id` (only valid while `RESERVED`/before execution —
   otherwise expect `FAILED_PRECONDITION`).

## Rules
- Errors are gRPC status codes (see `errors/infostellar-problem-types.yml`); retry
  `UNAVAILABLE`/`DEADLINE_EXCEEDED` with backoff, fix `INVALID_ARGUMENT`.
- Streams are long-lived; keep the deadline generous and handle reconnects.
- No idempotency key exists — do not blindly retry `ReservePass`; re-`ListPlans` first.
