---
description: API TESTING (STACK LEARNER)
---

# 🥂 Workshop on REST API Design (Day 02)

**Partial Responses**

* **Partial response** allows API consumers to request only specific fields of a resource, reducing unnecessary data transfer.
* Clients specify desired fields via query parameters e.g., **fields=name,price,photo**.
* Benefits:
  * **Reduces bandwidth usage** by sending only necessary data.
  * **Improves response time and performance**, especially for clients with limited network.
  * Avoids over-fetching large datasets.
* Example: A product resource may have many fields; mobile apps may only need **name** and **photo**.
* Without partial responses, full data is sent, causing latency and bandwidth wastage.

**Query Parameters**

Query parameters are a fundamental aspect of HTTP requests used to specify information for a resource retrieval or manipulation operation.

* Used for **filtering**, **sorting**, **searching**, and **pagination**.
* Examples:
  * Filtering products by category or price range.
  * Sorting by price ascending/descending.
  * Pagination using **limit** and **page** parameters to retrieve manageable chunks of data.
* Query parameters enable flexible, client-driven data retrieval.

**Pagination**

* Essential when dealing with large datasets (e.g., 1 million products).
* Prevents overloading the client and network by limiting data per request.
* Typical parameters: **limit** (items per page), **page** (page number).
* Response includes metadata such as current page, total pages, total items, and links for navigation.
* Ensures scalable API consumption.

**Error Handling and Consistency**

* Proper HTTP status codes: 200 (OK), 400 (Bad Request), 401 (Unauthorized), 403 (Forbidden), 404 (Not Found), 422 (Unprocessable Entity), 500 (Internal Server Error).
* Error responses should be:
  * **Consistent in format** across the API.
  * Include descriptive messages, error codes, hints for troubleshooting, and a **trace ID**.
* Trace IDs help developers quickly identify root causes using tools like **OpenTelemetry** or **Sentry**.
* Avoid exposing sensitive internal information (e.g., database queries) in error responses.

#### Cacheability <a href="#cacheability" id="cacheability"></a>

Responses from the server should be explicitly labeled as cacheable or non-cacheable.

**HTTP Caching (Cache-Control Headers)**

* Critical for **scalability, performance, and reducing latency**.
* Controls what data is cached, by whom, and for how long.
* Common directives:
  * **public** — response can be cached by any cache (browser, proxy).
  * **private** — response cached only by the client’s browser.
  * **no-cache** — requires validation before use.
  * **no-store** — no caching allowed (used for sensitive data).
  * **max-age** — duration (in seconds) to cache the response.
  * **S-Max-Age** — Similar to max-age
* Caching can happen at multiple levels: client, proxy, ISP.
* Example: Products list responses can be cached for a minute, reducing unnecessary server hits.

**ETag (Entity Tag) Headers and Conditional Requests**

* ETag is a unique hash representing the current version of a resource.
* Server sends ETag in response headers.
* Client sends ETag back in **If-None-Match** header during subsequent requests.
* Server compares ETag values:
  * If unchanged, responds with **304 Not Modified** (no body), saving bandwidth.
  * If changed, sends updated resource data with new ETag.
* ETag enables **efficient cache validation** and **avoids redundant data transfer**.
* Implementation involves generating a hash (e.g., SHA1) of resource content or metadata.
* Helpful for large payloads and frequently accessed resources.

**API Versioning**

* API versioning manages **changes and backward compatibility** as APIs evolve.
* Prevents breaking existing clients when introducing new features or breaking changes.
* Two main categories of changes:
  * **Breaking Changes**: Modify existing functionality, requiring a new API version.
  * **Non-breaking Changes**: Add new features or optimize without affecting existing clients.
* Versioning strategies:
  *   **URL-based** (e.g.,

      ```
      /api/v1/products
      ```

      ).
  * **Header-based** (passing version info in HTTP headers).
  * **Composite** (combining URL and headers).
* Best practices:
  * Maintain clear **documentation** and **migration guides**.
  * Use **deprecation warnings** for old versions.
  * Provide **backward compatibility** as much as possible.
  * Encourage clients to upgrade gradually.
* Example:
  * Version 1 might return product data with a **data** property.
  * Version 2 changes the response format to **products** property (breaking change).
* Proper versioning allows multiple API versions to coexist, supporting diverse client needs.

***

#### Markdown Table: HTTP Status Codes Discussed <a href="#markdown-table-http-status-codes-discussed" id="markdown-table-http-status-codes-discussed"></a>

| Status Code | Meaning               | When to Use                   |
| ----------- | --------------------- | ----------------------------- |
| 200         | OK                    | Successful requests           |
| 304         | Not Modified          | Cached response still valid   |
| 400         | Bad Request           | Invalid request data          |
| 401         | Unauthorized          | Authentication required       |
| 403         | Forbidden             | Insufficient permissions      |
| 404         | Not Found             | Resource does not exist       |
| 422         | Unprocessable Entity  | Validation or semantic errors |
| 500         | Internal Server Error | Server-side failure           |

***

#### Best Practices & Key Insights <a href="#best-practices-key-insights" id="best-practices-key-insights"></a>

* Implement **partial responses** to avoid over-fetching.
* Use **query parameters** for flexible data access.
* Keep **error responses consistent** and traceable.
* Leverage **Cache-Control** and **ETag** for performance.
* Apply **API versioning** for breaking changes.
* Maintain documentation and migration guides.
* Monitor API usage and deprecate responsibly.
* Use **Redis or server-side caching** for heavy reads.
* Consider **GraphQL** only when REST becomes limiting.

***

#### Conclusion <a href="#conclusion" id="conclusion"></a>

This workshop delivers a practical, real-world blueprint for building scalable REST APIs. By combining partial responses, caching strategies, consistent error handling, and disciplined versioning, teams can ship APIs that scale cleanly while keeping clients happy and unbroken.

***

#### FAQ (Extracted from Q\&A) <a href="#faq-extracted-from-q-a" id="faq-extracted-from-q-a"></a>

**Should we switch to GraphQL for partial responses?**\
Not required. REST with partial responses works for most use cases.

**How to handle ETag in Node.js?**\
Implement via middleware or custom hashing logic.

**How to avoid duplication in versioned APIs?**\
Separate routing layers, reuse service logic.

**When to use API versioning?**\
Always for breaking changes.

**How to cache frequently changing data?**\
Don’t. Cache only stable or semi-static data.

#### Resources <a href="#resources" id="resources"></a>

This summary is based on the following workshop:

[**Day 2: REST API Partial Responses, Error Handling, Caching, and Version Control | REST API Design**](https://www.youtube.com/watch?v=orn9N90dcVY\&t=4459s)

```
Credits: Stack Learner - REST API Design Workshop Series
```

***
