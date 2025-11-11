## ADDED Requirements

Requirement: Upload datasets to Weaviate

#### Scenario: CSV ingest
- Given a user sends `POST /collections/{user_id}/upload` with a CSV file and `collection_name`
- Then the server parses rows, infers schema, creates the collection if missing, and inserts rows in batches
- And responds with `{ upload_id, accepted: true }` while streaming progress over WS

#### Scenario: JSONL ingest
- Given a JSONL file with one JSON object per line
- Then the server handles it identically to CSV (parse → infer → insert)

#### Scenario: Upsert by id
- Given `upsert=true` and `id_property` is present in each row
- Then existing rows are updated and new rows inserted

#### Scenario: Progress streaming
- Given `WS /ws/upload_progress` with `{ user_id, upload_id }`
- Then the server sends periodic messages including `progress` ∈ [0,1], `inserted`, `updated`, and a summary on completion

Requirement: Finalize with preprocess (optional)

#### Scenario: Trigger preprocess
- Given a successful upload
- When the client calls `POST /collections/{user_id}/finalize/{collection_name}?preprocess=true`
- Then the server triggers the existing preprocess pipeline and returns a 202/200 with a confirmation

Requirement: Security

#### Scenario: Auth required
- All new endpoints and WS require a valid Bearer token; missing/invalid → 401/1008

Requirement: Limits and errors

#### Scenario: File validation
- The server rejects unsupported content types or files larger than the configured limit with 400 and a clear error

## MODIFIED Requirements

Requirement: Docs update

#### Scenario: Ingestion docs
- The API docs describe upload endpoints, options, WS progress, limits, and preprocessing linkage

## REMOVED Requirements

- None.
