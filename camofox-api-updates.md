# Camofox Browser API Updates

I have implemented several new features to enhance browser steering and data extraction capabilities. This document summarizes the new functionality and providing usage examples.

## 1. Global Prettified JSON

All API responses are now automatically prettified (indented with 2 spaces). This makes it much easier to read the output of `curl` commands directly in the terminal without needing an external formatter like `jq`.

---

## 2. Remote JavaScript Evaluation
**Endpoint**: `POST /tabs/:tabId/eval`

This allows you to execute arbitrary JavaScript within the context of the page. It uses Playwright's `page.evaluate()`.

### Usage
- **Method**: `POST`
- **Body**:
  - `userId`: (string) Your agent/user identifier.
  - `expression`: (string) The JavaScript code to evaluate.

### Example
```bash
curl -X POST http://localhost:9377/tabs/abc123/eval \
  -H "Content-Type: application/json" \
  -d '{
    "userId": "agent1",
    "expression": "document.title"
  }'
```

---

## 3. Cookie Export
**Endpoint**: `GET /sessions/:userId/cookies`

Export browser cookies for a specific user session. You can filter by URL or use a regex pattern to find specific cookies.

### Usage
- **Method**: `GET`
- **Query Params**:
  - `url`: (optional) Only return cookies applicable to this URL.
  - `pattern`: (optional) A case-insensitive regex to filter cookies by name.

### Example
```bash
# Export all cookies for a session
curl "http://localhost:9377/sessions/agent1/cookies"

# Filter cookies with "session" in the name
curl "http://localhost:9377/sessions/agent1/cookies?pattern=session"
```

---

## 4. Main Document Request Headers
**Endpoint**: `GET /tabs/:tabId/headers`

Retrieve the HTTP headers sent by the browser to the target website during the most recent page load or navigation.

### Usage
- **Method**: `GET`
- **Query Params**:
  - `userId`: (required) Your agent identifier.

### Example
```bash
curl "http://localhost:9377/tabs/abc123/headers?userId=agent1"
```

---

## 5. XHR/Fetch Request Headers
**Endpoint**: `GET /tabs/:tabId/xhr-headers`

Retrieve the URL and headers of the **most recent** background request (XHR or Fetch) that was sent to the same domain as the current page.

### Usage
- **Method**: `GET`
- **Query Params**:
  - `userId`: (required) Your agent identifier.

### Example
```bash
curl "http://localhost:9377/tabs/abc123/xhr-headers?userId=agent1"
```

### Note on Matching Logic
This endpoint only returns background requests targeting the same "base" domain as the current page. For example, if you are browsing `www.immowelt.de`, it will capture requests to `api.immowelt.de` but ignore requests to third-party tracking domains.

---

## Summary of Technical Changes
- **`server.js`**:
    - Added routes for `eval`, `cookies` (export), `headers`, and `xhr-headers`.
    - Implemented a background request listener that automatically attaches to every new page instance.
    - Added `app.set('json spaces', 2)`.
- **`lib/request-utils.js`**:
    - Updated the action classifier to ensure metrics are correctly tracked for the new endpoints.
- **Tab State**:
    - Extended the internal state tracking to persist headers and most recent XHR data.
