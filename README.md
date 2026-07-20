# Rate Limiter

A Node.js and Express project that demonstrates API rate limiting with Redis, basic bot detection, WAF-style checks, and a browser dashboard powered by WebSockets.

## Features

- Express API server
- Redis-backed request tracking
- Rate limiting middleware for `/api` routes
- Static dashboard served from `public`
- WebSocket updates for live monitoring
- Health endpoint that bypasses rate limiting

## Requirements

- Node.js
- npm
- Redis

## Installation

```bash
npm install
```

## Configuration

Create a `.env` file in the project root and set your Redis connection URL:

```env
REDIS_URL=redis://localhost:6379
```

## Running The Project

```bash
npm start
```

The server runs at:

```text
http://localhost:3000
```

The dashboard is available at:

```text
http://localhost:3000/dashboard.html
```

## API Endpoints

```text
GET /health
GET /api/hello
GET /api/data
```

## Project Structure

```text
.
|-- botDetector.js
|-- middleware.js
|-- public/
|-- script.js
|-- server.js
|-- tokenBucket.js
`-- waf.js
```
