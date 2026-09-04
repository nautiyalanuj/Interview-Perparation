# What is Cursor-based pagination
- It is a database querying strategy that uses a unique, sequential identifier (the "cursor") to fetch the next set of results, rather than relying on a numeric page offset.
- It is the industry standard for real-time, frequently updating data feeds like social media timelines.
- Unlike Offset Pagination (LIMIT 10 OFFSET 50000), which forces the database to read through and discard thousands of unwanted rows to reach the target data, cursor pagination tells the database exactly where to start reading (WHERE id > 50000 LIMIT 10), completely bypassing the irrelevant data.
- A cursor is just an opaque token (a customized identifier) that the backend creates and passes to the frontend. When the frontend sends that exact token back, the backend decodes it to find the precise database anchor point.
- Because it targets an exact row using a WHERE clause instead of mathematically shifting a window (OFFSET), it fixes both major flaws of offset pagination:
  - **It prevents data drift**: It anchors your position to a specific record. If rows are added or deleted above your current view, your anchor doesn't move. You never see duplicates or skip records.
  - **It preserves database speed**: The database uses an index to jump straight to that cursor row instantly. With offset, if you ask for page 10,000, the database has to physically read and count through 100,000 rows just to discard them.
- For the backend to use a cursor efficiently, whatever data you choose to pack inside that token must follow two strict rules:
  - It must be sequentially sortable: The database must be able to use operators like greater-than (>) or less-than (<) against it.
  - It must be completely unique: If two rows have the exact same cursor value, the database won't know which one you left off on, causing missing data. This is why a cursor is almost always a combination of a property and a unique ID (like Timestamp + ID).
 
|Category | Advantages 👍 |Disadvantages 👎|
| - | -| -|
|Performance| Highly scalable. Query speeds remain lightning-fast and constant (O(1) efficiency), even when skipping to the end of billions of rows.| No direct page skipping. Users cannot skip directly to a specific page (e.g., jumping from Page 1 to Page 10) without fetching intermediate data.| 
|Data Consistency| No skipped or duplicate items. If new records are added or deleted while a user is scrolling, the feed remains accurate without breaking. | Complex reverse navigation. Moving backward through results requires reversing the query sort order, which complicates implementation. |
|Ideal Use Case| Perfect for infinite scroll feeds, real-time streams (like chat logs), and massive, fast-growing datasets.| Poorly suited for traditional search results or internal admin dashboards that require exact page numbers.| 
|Implementation |Reduces database memory overhead because the database only scans the exact number of rows requested. | Harder to implement. It requires a unique, sequentially ordered column (like an auto-incrementing ID or timestamp) to function properly.|

# Example 
Imagine an app where users post messages. We want to show 2 items per page, sorted by ID (newest first). While a user is looking at Page 1, a new post (Post 6) is created.

## Scenario A: Offset Pagination (The Broken Way)
Offset pagination relies on counting rows from the beginning.
- Page 1 Request: LIMIT 2 OFFSET 0 (Database returns Posts 5 and 4)
- Action: Post 6 is inserted at the top of the database.
- Page 2 Request: LIMIT 2 OFFSET 2 (Database skips the first 2 rows, which are now 6 and 5, and returns Posts 4 and 3)

⚠️ The Bug: The user sees Post 4 twice (at the bottom of Page 1 and the top of Page 2). If items were deleted instead, the user would completely skip a post.
```
Initial State:            [Post 5] [Post 4] | [Post 3] [Post 2] [Post 1]

                          |--- Page 1 ---|   |--- Page 2 ---|
```


```
New Post 6 Arrives:       [Post 6] [Post 5] | [Post 4] [Post 3] [Post 2] [Post 1]

                          |--- Page 1 ---|   |--- Page 2 ---|
                                              ^ Duplicate!
```

## Scenario B: Cursor Pagination (The Better Way)
Cursor pagination remembers a specific anchor point instead of counting lines.
- Page 1 Request: LIMIT 2 (Database returns Posts 5 and 4). The API tells the frontend: "Your cursor for the next page is 4."
- Action: Post 6 is inserted at the top of the database.
- Page 2 Request: WHERE id < 4 LIMIT 2 (Database looks only after Post 4 and returns Posts 3 and 2)

✅ The Success: Even though the database changed, the user gets the correct next items with no duplicates.

## How a Cursor is Created and Defined
A cursor is not a temporary database pointer or a session token. It is a piece of data extracted from the last record of the current page that points to its exact position in the index.

Assume you are sorting by a creation timestamp. If two things happen at the exact same millisecond, timestamps conflict. To prevent this, a cursor is usually defined by combining a unique property and the sort property (e.g., Timestamp + ID).

- The API Query:
```
  SELECT id, title, created_at 
  FROM posts 
  WHERE (created_at, id) < ('2026-09-04 12:00:00', 452)
  ORDER BY created_at DESC, id DESC
  LIMIT 10;
```

- Generating the Token:
  - The backend takes the values of the last item in the result (created_at and id), serializes them (often using Base64 to keep URLs clean), and sends it to the client.
  ```
  // Backend turns data into an opaque string
  const cursorData = JSON.stringify({ created_at: '2026-09-04 12:00:00', id: 452 });
  const nextCursor = Buffer.from(cursorData).toString('base64'); 
  // Result: "eyJjcmVhdGVkX2F0IjoiMjAyNi0wOS0wNCAxMjowMDowMCIsImlkIjo0NTJ9"

  ```
  - The client receives this string and sends it back in the next API request: GET /api/posts?cursor=eyJjcmVh...
