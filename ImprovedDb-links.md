# ImprovedDb examples for the Ritzau interview

These links match the topics in `Ritzau.odp`. They are pinned to commit `aa03d0692192f542f4781920dedb210d1755eddb`, verified against GitHub `main` on 15 November 2026, so the highlighted lines remain stable. ImprovedDb is private: open the links while signed into a GitHub account with repository access.

## Slides 5–7: search, pagination, and caching

The closest combined example is `GET /api/media-catalog/`. It accepts a search query, filters, sorting, `page`, and `pageSize`. The separate `/api/media-catalog/search` route provides limited search suggestions; it does not implement page-based pagination.

- [Endpoint and request parameters](https://github.com/Grumlebob/ImprovedDb/blob/aa03d0692192f542f4781920dedb210d1755eddb/BlazorAutoApp/Features/MediaCatalog/Endpoints/MediaCatalogEndpoints.cs#L30-L86): minimal API handler, dependency injection, cancellation, and request construction.
- [Search query with EF Core](https://github.com/Grumlebob/ImprovedDb/blob/aa03d0692192f542f4781920dedb210d1755eddb/BlazorAutoApp/Features/MediaCatalog/Services/MediaCatalogServerService.cs#L354-L411): compact example of `AsNoTracking`, PostgreSQL `ILike`, ordering, `Take`, DTO projection, and asynchronous execution. This is the suggestion search, without pagination.
- [Pagination: CountAsync, Skip, and Take](https://github.com/Grumlebob/ImprovedDb/blob/aa03d0692192f542f4781920dedb210d1755eddb/BlazorAutoApp/Features/MediaCatalog/Services/MediaCatalogServerService.cs#L191-L219): best link for slide 6. Highlight lines 195–200 for the database pagination path.
- [Pagination response metadata](https://github.com/Grumlebob/ImprovedDb/blob/aa03d0692192f542f4781920dedb210d1755eddb/BlazorAutoApp/Features/MediaCatalog/Services/MediaCatalogServerService.cs#L80-L96): `Page`, `PageSize`, `TotalCount`, and `TotalPages`.
- [Caching around the existing service](https://github.com/Grumlebob/ImprovedDb/blob/aa03d0692192f542f4781920dedb210d1755eddb/BlazorAutoApp/Features/MediaCatalog/Services/CachedMediaCatalogApi.cs#L31-L83): best link for slide 7. Browse caching is at lines 31–58; suggestion-search caching is at lines 61–82. Cache misses call the underlying service.
- [Cache eligibility rules](https://github.com/Grumlebob/ImprovedDb/blob/aa03d0692192f542f4781920dedb210d1755eddb/BlazorAutoApp/Infrastructure/Hosting/PublicDataCacheKeys.cs#L145-L154): decides which browse requests qualify for caching.
- [Cache defaults: pages and TTL](https://github.com/Grumlebob/ImprovedDb/blob/aa03d0692192f542f4781920dedb210d1755eddb/BlazorAutoApp/Infrastructure/Hosting/PublicDataCacheOptions.cs#L41-L65): first five browse pages, maximum page size 60, local browse TTL 600 seconds, distributed browse TTL 1,440 minutes. These are configurable defaults.
- [HybridCache GetOrCreateAsync](https://github.com/Grumlebob/ImprovedDb/blob/aa03d0692192f542f4781920dedb210d1755eddb/BlazorAutoApp/Infrastructure/Hosting/PublicDataCacheFacade.cs#L74-L92): the actual cache lookup/load operation, including generation in the key.
- [Cache invalidation](https://github.com/Grumlebob/ImprovedDb/blob/aa03d0692192f542f4781920dedb210d1755eddb/BlazorAutoApp/Infrastructure/Hosting/PublicDataCacheInvalidator.cs#L44-L71): follow-up link for “Spørg mig om invalidation.” Bumps the cache generation, then dispatches tag invalidation.

Presentation wording: “I already use this pattern in ImprovedDb: build an EF Core query, return one page, and cache eligible public results around the service.”

Limits to describe accurately:

- Authenticated browse requests bypass this shared browse cache. Browse requests containing search text also bypass it by default (`BrowseCacheRequestsWithQuery = false`). The separate suggestion search has its own caching path.
- The browse operation is not a single database round trip: it counts results and loads the page, with additional work for response metadata. Use the suggestion search when demonstrating a compact query with DTO projection.
- Genre-filtered browsing currently loads scalar matches and filters/paginates in memory. Use the branch without genre filters to demonstrate database pagination, and acknowledge the other branch if asked.
- “First five pages” is a configured policy, not automatic detection of popular pages.

## Slide 4: dictionary lookup

- [Build a dictionary once, then use TryGetValue inside the loop](https://github.com/Grumlebob/ImprovedDb/blob/aa03d0692192f542f4781920dedb210d1755eddb/BlazorAutoApp.DataImport/MediaCatalogImporter.cs#L283-L305): direct match for the slide’s dictionary placeholder. Existing media is loaded by IMDb ID; each imported item uses an average O(1) lookup. This illustrates the lookup technique, not a complete word-replacement algorithm.

## Slide 15: integration testing

- [WebApplicationFactory and PostgreSQL test setup](https://github.com/Grumlebob/ImprovedDb/blob/aa03d0692192f542f4781920dedb210d1755eddb/BlazorAutoApp.Test/TestSupport/Integration/WebAppFactory.cs#L1-L60): the existing `webappfactory` hyperlink placeholder.
- [Start the PostgreSQL test container](https://github.com/Grumlebob/ImprovedDb/blob/aa03d0692192f542f4781920dedb210d1755eddb/BlazorAutoApp.Test/TestSupport/Integration/WebAppFactory.cs#L178-L188): real database infrastructure for integration tests.
- [Reset database state with Respawn](https://github.com/Grumlebob/ImprovedDb/blob/aa03d0692192f542f4781920dedb210d1755eddb/BlazorAutoApp.Test/TestSupport/Integration/WebAppFactory.cs#L243-L254): PostgreSQL reset configuration that preserves EF migration history.
- [HTTP integration tests for search](https://github.com/Grumlebob/ImprovedDb/blob/aa03d0692192f542f4781920dedb210d1755eddb/BlazorAutoApp.Test/Features/MediaCatalog/Api/SearchMediaTests.cs#L15-L56): verifies returned titles/order and literal handling of SQL LIKE wildcards `%` and `_`.

The deck’s external placeholders for Bobby Tables, Mircea’s presentation, and Keep are separate from these ImprovedDb examples.
