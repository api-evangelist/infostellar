---
name: Manage satellite TLE orbital data
description: >-
  Set the orbital-data source for a satellite and add/retrieve its two-line
  element (TLE) data on StellarStation.
api: grpc/infostellar-stellarstation.proto
service: StellarStationService
operations: [SetTleSource, AddTle, GetTle]
---

# Manage satellite TLE orbital data

Keep a satellite's orbital elements current so StellarStation can compute passes.

## Auth
Service-account JWT bearer credentials on the gRPC channel (audience
`https://api.stellarstation.com`). See `authentication/infostellar-authentication.yml`.

## Steps
1. `SetTleSource` — choose where orbital data comes from: `NORAD` (auto) or
   `MANUAL` (you supply it) for the `satellite_id`.
2. If `MANUAL`, `AddTle` with the `satellite_id` and a `Tle` (line 1 / line 2).
3. `GetTle` — read back the current `Tle` to confirm what the platform is using
   for pass prediction.

## Rules
- Setting source to `MANUAL` means you are responsible for keeping TLEs fresh;
  stale TLEs degrade pass accuracy.
- Validate TLE format before `AddTle` to avoid `INVALID_ARGUMENT`.
- `NOT_FOUND` means the `satellite_id` is not provisioned to your organization.
- Errors are gRPC status codes; see `errors/infostellar-problem-types.yml`.
