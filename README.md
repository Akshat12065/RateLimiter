# Rate Limiter

A Node.js project that protects Express API routes using a Redis-backed token bucket rate limiter, simple bot detection, and Web Application Firewall checks. It also includes a browser dashboard that shows live request activity through WebSockets.

## Project Overview

This project demonstrates how incoming API traffic can be inspected before it reaches protected routes.

Request flow:

```text
Client request
|-- WAF security checks
|-- Bot detection checks
|-- Redis token bucket rate limit
`-- Protected API route
```

The `/health` route is not rate limited, while routes under `/api` pass through the protection middleware.

## Features

- Express server with JSON request handling
- Redis connection through environment variables
- Token bucket rate limiting per IP address
- Bot detection using User-Agent and request speed checks
- WAF checks for SQL injection, XSS, command injection, path traversal, bad headers, and oversized payloads
- Live dashboard served from the `public` folder
- WebSocket broadcasts for allowed, limited, bot-blocked, and WAF-blocked requests
- Helpful `X-Tokens-Remaining` response header for allowed API requests

## Tech Stack

- Node.js
- Express
- Redis
- WebSocket (`ws`)
- dotenv

## Requirements

- Node.js installed
- npm installed
- Redis database available locally or remotely

## Installation

Install project dependencies:

```bash
npm install
```

## Environment Variables

Create a `.env` file in the project root.

```env
REDIS_URL=redis://username:password@host:port
```

For a local Redis server, this may look like:

```env
REDIS_URL=redis://localhost:6379
```

Do not commit real Redis credentials to version control.

## Running The Project

Start the server:

```bash
npm start
```

The app runs at:

```text
http://localhost:3000
```

Open the dashboard at:

```text
http://localhost:3000/dashboard.html
```

## API Endpoints

### Health Check

```text
GET /health
```

Returns a simple server status response. This route does not use rate limiting.

### Hello API

```text
GET /api/hello
```

Protected by WAF, bot detection, and token bucket rate limiting.

### Data API

```text
GET /api/data
```

Protected route that returns sample data.

## Rate Limiting

Rate limiting is implemented in `tokenBucket.js`.

Current settings:

```text
Maximum tokens: 10
Refill rate: 0.2 tokens per second
Redis key: bucket:<ip>
Key expiry: 3600 seconds
```

Each request consumes one token. If an IP address has no tokens left, the request is rejected with:

```text
429 Too Many Requests
```

## Bot Detection

Bot detection is implemented in `botDetector.js`.

The detector assigns a score to each request:

```text
No User-Agent header: +40
Known bot tool in User-Agent: +60
More than 10 requests per second: +20
```

If the score is `60` or higher, the request is blocked with:

```text
403 Forbidden
```

Known bot tools include:

```text
curl, python, wget, scrapy, httpie
```

## WAF Protection

WAF logic is implemented in `waf.js`.

It checks requests for:

- Invalid HTTP methods
- Path traversal attempts
- SQL injection patterns
- XSS patterns
- Command injection patterns
- Missing or suspicious headers
- Oversized headers
- Invalid content length
- Missing content type for `POST`, `PUT`, and `PATCH`
- JSON body payloads larger than 10 KB

Blocked WAF requests return:

```text
403 Forbidden
```

## Dashboard

The dashboard is available at:

```text
/dashboard.html
```

It displays:

- Allowed requests
- Rate-limited requests
- Bot-blocked requests
- WAF-blocked requests
- Token bucket fill level
- WAF rule hit counts
- Bot threat score
- Live request log

The server sends dashboard updates through WebSockets.

## Project Structure

```text
.
|-- botDetector.js
|-- middleware.js
|-- package.json
|-- public/
|   `-- dashboard.html
|-- server.js
|-- tokenBucket.js
`-- waf.js
```

## Main Files

```text
server.js        Main Express and WebSocket server
middleware.js    Connects WAF, bot detection, and rate limiting
tokenBucket.js   Redis-backed token bucket algorithm
botDetector.js   Bot scoring and detection logic
waf.js           Web Application Firewall checks
public/          Static dashboard files
```

## Example Test Commands

Check server health:

```bash
curl http://localhost:3000/health
```

Send a protected API request:

```bash
curl http://localhost:3000/api/hello
```

Send multiple requests quickly to test rate limiting:

```bash
for i in {1..15}; do curl http://localhost:3000/api/hello; done
```
