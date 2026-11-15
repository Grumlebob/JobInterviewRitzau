# GrumPlates examples for the Ritzau interview

GrumPlates is the stronger primary example for slides 5–7. It is public, and one book-search feature combines filtering, pagination, projection, and caching. The ownership checks also connect directly to task 4.

Links use pinned commits so their highlighted lines remain stable. Most examples use `5e6d600116bf44ac24776d94b4bb8e19d2e53c2b`. The strengthened cache invalidation test uses `530e8d98113985a15eefdb73c538dd43975a9181`, merged through [PR #463](https://github.com/Grumlebob/GrumPlates/pull/463) on 15 November 2026.

## Best starting link

[Search query and pagination](https://github.com/Grumlebob/GrumPlates/blob/5e6d600116bf44ac24776d94b4bb8e19d2e53c2b/GrumPlates/Features/Books/Services/BooksServerService.cs#L94-L135)

This is the closest match to the article-search discussion. It uses `AsNoTracking`, filters by the authenticated user's ID, searches title and author with PostgreSQL `ILike`, counts matches, orders by a unique book ID, applies `Skip`/`Take`, and projects only the response fields. Both database operations are asynchronous and accept cancellation.

Example request for a signed-in user:

```http
GET /api/books/search?searchTerm=atlas&page=2&pageSize=25
```

## Slide links

- **Slide 5 — Minimal API endpoint:** [Request validation and service call](https://github.com/Grumlebob/GrumPlates/blob/5e6d600116bf44ac24776d94b4bb8e19d2e53c2b/GrumPlates/Features/Books/Endpoints/BooksEndpoints.cs#L86-L98). A compact handler with dependency injection and cancellation. The route is registered as `/api/books/search` at lines 18 and 32.
- **Slide 5 — Search implementation:** [Read-only query and title/author filtering](https://github.com/Grumlebob/GrumPlates/blob/5e6d600116bf44ac24776d94b4bb8e19d2e53c2b/GrumPlates/Features/Books/Services/BooksServerService.cs#L101-L112).
- **Slide 6 — Pagination:** [Count, stable ordering, Skip/Take, and projection](https://github.com/Grumlebob/GrumPlates/blob/5e6d600116bf44ac24776d94b4bb8e19d2e53c2b/GrumPlates/Features/Books/Services/BooksServerService.cs#L114-L134).
- **Slide 6 — Pagination limits:** [Defaults and validation](https://github.com/Grumlebob/GrumPlates/blob/5e6d600116bf44ac24776d94b4bb8e19d2e53c2b/GrumPlates.Core/Features/Books/UseCases/GetBooks/SearchBooksRequest.cs#L3-L38). Default page size 25, maximum 100; invalid requests produce validation errors.
- **Slide 7 — Caching:** [HybridCache around the same search](https://github.com/Grumlebob/GrumPlates/blob/5e6d600116bf44ac24776d94b4bb8e19d2e53c2b/GrumPlates/Features/Books/Services/BooksServerService.cs#L43-L62). The loader runs on cache misses.
- **Slide 7 — Cache key:** [User, search term, page, and page size](https://github.com/Grumlebob/GrumPlates/blob/5e6d600116bf44ac24776d94b4bb8e19d2e53c2b/GrumPlates/Features/Books/Caching/BooksCacheKeys.cs#L12-L27). Search text is uppercased and hashed; the key separates users and pages. The list tag groups a user's search pages for invalidation.
- **Slide 7 — TTL:** [Configurable cache defaults](https://github.com/Grumlebob/GrumPlates/blob/5e6d600116bf44ac24776d94b4bb8e19d2e53c2b/GrumPlates/Features/Books/Caching/BooksCacheOptions.cs#L3-L19). Search uses list settings: 5 minutes distributed and 5 seconds local by default.
- **Slide 7 — Invalidation after a write:** [Update, save, and invalidate](https://github.com/Grumlebob/GrumPlates/blob/5e6d600116bf44ac24776d94b4bb8e19d2e53c2b/GrumPlates/Features/Books/Services/BooksServerService.cs#L196-L212). Create and delete also invalidate affected caches. [Invalidation implementation](https://github.com/Grumlebob/GrumPlates/blob/5e6d600116bf44ac24776d94b4bb8e19d2e53c2b/GrumPlates/Features/Books/Services/BooksServerService.cs#L232-L245) logs failures; it does not roll back the saved write.
- **Task 4 and slide 15 — Ownership and pagination test:** [Search_FiltersAndPaginatesOnlyCurrentUsersBooks](https://github.com/Grumlebob/GrumPlates/blob/5e6d600116bf44ac24776d94b4bb8e19d2e53c2b/GrumPlates.Test/Features/Books/Api/SearchBooksTests.cs#L35-L64). Seeds another user's matching book and checks that only the current user's matches appear across two pages.

## Suggested explanation

“I use this pattern in GrumPlates. The query filters by the current user and search text, selects the response fields, and returns one page. HybridCache wraps that search. The cache key includes the user, search text, and pagination parameters. Writes invalidate the affected user's cached lists.”

## Slide 7 follow-up: cache invalidation

Start with the first two links for a short explanation. Use the remaining links for follow-up questions about multiple servers and failures.

1. [Save the update, then invalidate](https://github.com/Grumlebob/GrumPlates/blob/5e6d600116bf44ac24776d94b4bb8e19d2e53c2b/GrumPlates/Features/Books/Services/BooksServerService.cs#L196-L212). Invalidation happens after `SaveChangesAsync`, so an unsuccessful save does not invalidate the cache.
2. [Choose the affected keys and tags](https://github.com/Grumlebob/GrumPlates/blob/5e6d600116bf44ac24776d94b4bb8e19d2e53c2b/GrumPlates/Features/Books/Caching/BooksCacheKeys.cs#L12-L27). The item key targets one user's book. The list tag covers that user's list and all cached search pages, including search terms whose matches could change after an edit. This avoids enumerating every search key. Search entries receive this tag in `BooksServerService.SearchAsync`, lines 55–59.
3. [Apply key and tag invalidation](https://github.com/Grumlebob/GrumPlates/blob/5e6d600116bf44ac24776d94b4bb8e19d2e53c2b/GrumPlates/Infrastructure/Hosting/CacheInvalidation/HybridCacheInvalidationApplier.cs#L9-L29). Calls `HybridCache.RemoveAsync` and `RemoveByTagAsync`. Describe this as invalidating entries; do not imply that tag invalidation physically deletes every stored value immediately.
4. [Apply locally, then publish to other nodes](https://github.com/Grumlebob/GrumPlates/blob/5e6d600116bf44ac24776d94b4bb8e19d2e53c2b/GrumPlates/Infrastructure/Hosting/CacheInvalidation/HybridCacheInvalidator.cs#L16-L70). Local application and publication have separate timeouts and warning handling. [Redis publisher](https://github.com/Grumlebob/GrumPlates/blob/5e6d600116bf44ac24776d94b4bb8e19d2e53c2b/GrumPlates/Infrastructure/Hosting/CacheInvalidation/RedisCacheInvalidationPublisher.cs#L18-L37) broadcasts the keys and tags.
5. [Other nodes apply the message](https://github.com/Grumlebob/GrumPlates/blob/5e6d600116bf44ac24776d94b4bb8e19d2e53c2b/GrumPlates/Infrastructure/Hosting/CacheInvalidation/RedisCacheInvalidationSubscriber.cs#L106-L128). Checks app/environment, ignores its own messages, and applies the same invalidation request.
6. [Test: prove cache hits, then invalidate from server A](https://github.com/Grumlebob/GrumPlates/blob/530e8d98113985a15eefdb73c538dd43975a9181/GrumPlates.Test/Features/Books/Caching/BooksCrossNodeCacheInvalidationTests.cs#L55-L120). Counts B's book queries: the initial item and list load require two queries, repeated reads require none, an update through A causes two new queries returning the updated title, and subsequent reads use the refreshed cache. B's cache clock stays fixed, so local expiry cannot explain the refresh. The test was also verified to fail when caching was temporarily bypassed and when B's invalidation subscriber was temporarily disabled.
7. [Test: missed notification falls back to local expiry](https://github.com/Grumlebob/GrumPlates/blob/5e6d600116bf44ac24776d94b4bb8e19d2e53c2b/GrumPlates.Test/Features/Books/Caching/BooksCrossNodeCacheInvalidationTests.cs#L106-L138). Disables B's subscriber and uses a controlled clock with a one-second local TTL to demonstrate stale data followed by refresh.

Interview wording: “After saving a change, I invalidate the book and the user's cached lists. Search pages share a list tag, so I do not need to track every query and page. Redis Pub/Sub tells the other servers to invalidate their caches too. Expiry provides a fallback if a notification is missed.”

Failure detail: these invalidations are best effort and are not atomic with the database save. Failures are logged; there is no durable retry/outbox in this path. The missed-message test assumes shared-cache invalidation succeeds. If that also fails, a local-cache expiry alone does not guarantee fresh data; distributed expiry matters too.

## Details to describe accurately

- A cache miss executes two SQL queries: one count and one page query. It avoids a database query per returned book, but is not a single database round trip.
- This is user-level ownership isolation. A company-level requirement would need the appropriate company membership and tenant checks.
- The code caches each requested search page. It does not restrict caching to popular pages or the first five pages; ImprovedDb remains the example for that policy.
- The test above covers ownership filtering and pagination. It is not a dedicated proof of cache isolation between users.
- Wildcard escaping handles literal `%`, `_`, and backslash. EF query parameterization is what separates search values from SQL syntax.
