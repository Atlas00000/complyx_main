# Logging Test Scripts

This document describes how to test the logging implementation.

## Prerequisites

- Node.js >= 18.0.0
- pnpm installed
- Server dependencies installed: `cd server && pnpm install`

## Server-Side Tests

### 1. Basic Logging Test

Tests all logging levels and features:

```bash
cd server
pnpm test:logging
```

**What it tests:**
- Basic log levels (debug, info, warn, error)
- HTTP request/response logging
- Database query logging
- External API logging
- Performance logging
- Security event logging
- Complex context logging
- File logging (if enabled)

### 2. Request Logging Middleware Test

Tests the request logging middleware with simulated HTTP requests:

```bash
cd server
pnpm test:request-logging
```

**What it tests:**
- Request ID generation
- Request/response timing
- Status code logging
- Concurrent request handling
- Slow request detection

### 3. Integration Test

Tests the complete logging system integration:

```bash
cd server
pnpm test:logging-integration
```

**What it tests:**
- End-to-end request/response flow
- Error handling and logging
- Request ID propagation
- Context preservation

## Client-Side Tests

### Basic Logging Test

Tests client-side logging (requires Node.js with tsx):

```bash
cd client
npx tsx scripts/testLogging.ts
```

**What it tests:**
- Basic log levels
- API request/response logging
- User action logging
- Performance logging
- Error handling

## Docker Logging Test

To test Docker container logging:

```bash
cd server/docker
docker-compose up -d

# Check PostgreSQL logs
docker logs complyx-postgres

# Check Redis logs
docker logs complyx-redis

# View logs in real-time
docker logs -f complyx-postgres
docker logs -f complyx-redis
```

## File Logging Test

To test file logging:

1. Enable file logging in `.env`:
```env
ENABLE_FILE_LOGGING=true
LOG_DIR=./logs
```

2. Run the logging test:
```bash
cd server
pnpm test:logging
```

3. Check the logs directory:
```bash
ls -la server/logs/
cat server/logs/app-$(date +%Y-%m-%d).log
```

## Expected Output

### Server Logging Test Output

You should see:
- ✅ Emoji-prefixed log messages
- ✅ Timestamps for each log
- ✅ Context information in JSON format
- ✅ Request IDs for HTTP requests
- ✅ Performance metrics with durations
- ✅ Error stack traces

Example:
```
ℹ️ [INFO] [14:30:45] [Request: req-1234567890] Server started {"port":3001,"environment":"development"}
🔍 [DEBUG] [14:30:45] Request: GET /api/test {"requestId":"req-123","method":"GET","url":"/api/test"}
ℹ️ [INFO] [14:30:45] [Request: req-123] Response: GET /api/test - 200 (150ms) {"statusCode":200,"duration":150}
```

### Request Logging Test Output

You should see:
- Unique request IDs for each request
- Request method and URL
- Response status codes
- Request duration in milliseconds
- User context (if available)

## Troubleshooting

### Issue: "Cannot find module 'tsx'"

**Solution:**
```bash
cd server
pnpm add -D tsx
```

### Issue: File logging not working

**Check:**
1. `ENABLE_FILE_LOGGING=true` in `.env`
2. `LOG_DIR` is set and writable
3. Logs directory exists: `mkdir -p server/logs`

### Issue: Docker logs not showing

**Check:**
1. Containers are running: `docker ps`
2. Logging driver is configured in `docker-compose.yml`
3. Check container logs: `docker logs complyx-postgres`

## Next Steps

After running tests:
1. Review log output for proper formatting
2. Check file logs (if enabled) for JSON structure
3. Verify request IDs are unique and traceable
4. Test error scenarios to ensure proper error logging
5. Monitor performance metrics in logs

## Performance Considerations

- Logging is asynchronous and non-blocking
- File logging uses append operations (fast)
- Remote logging uses sendBeacon (client) or fetch (server)
- Log levels can be adjusted to reduce verbosity in production
