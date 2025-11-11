## ADDED Requirements

Requirement: Provide a universal Embedding Manager

#### Scenario: Single shared instance
- Given multiple components (ingestion, preprocessing, tools) request embeddings
- Then a single Embedding Manager instance serves batch requests without spawning new embedders per call

#### Scenario: Provider-agnostic API
- Given provider and model are configured in Settings
- Then `embed(texts, model?)` returns vectors using the configured provider with batch support

#### Scenario: Caching
- Given repeated calls for identical (model, text)
- Then vectors are served from an in-memory LRU cache until eviction

#### Scenario: Thread/async safety
- Concurrent `embed()` calls are safe and efficient; underlying client is reused

#### Scenario: Fallback when Weaviate vectorizes
- If a collection is configured for server-side vectorization
- Then the Embedding Manager is not invoked for that flow

## MODIFIED Requirements

Requirement: Settings

#### Scenario: Config knobs
- Settings include provider, model, API base, credentials, batch size, and cache size; reasonable defaults provided

## REMOVED Requirements

- None.
