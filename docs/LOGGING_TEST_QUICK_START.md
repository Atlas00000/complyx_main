# Quick Start: Testing Logging

## 🚀 Quick Test Commands

### Server-Side Logging Test
```bash
cd server
NODE_ENV=development LOG_LEVEL=debug pnpm test:logging
```

### Request Logging Middleware Test
```bash
cd server
NODE_ENV=development pnpm test:request-logging
```

### Integration Test
```bash
cd server
NODE_ENV=development pnpm test:logging-integration
```

## ✅ What to Expect

### Successful Test Output Should Show:

1. **Emoji-prefixed logs:**
   - 🔍 Debug messages
   - ℹ️ Info messages
   - ⚠️ Warning messages
   - ❌ Error messages

2. **Formatted output:**
   ```
   🔍 [DEBUG] [10:40:46 PM] Debug message {"test":"debug","value":123}
   ℹ️ [INFO] [10:40:46 PM] Info message {"test":"info","value":456}
   ```

3. **Request/Response logging:**
   ```
   🔍 [DEBUG] [10:40:46 PM] Request: GET /api/dashboard/score {"requestId":"req-123",...}
   ℹ️ [INFO] [10:40:46 PM] [Request: req-123] Response: GET /api/dashboard/score - 200 (150ms)
   ```

4. **Error details:**
   - Error messages with stack traces
   - Full log entries for errors and debug messages

## 📝 Test File Logging

To test file logging:

```bash
cd server
NODE_ENV=development LOG_LEVEL=debug ENABLE_FILE_LOGGING=true pnpm test:logging

# Check the logs
ls -la logs/
cat logs/app-$(date +%Y-%m-%d).log
```

## 🐳 Test Docker Logging

```bash
cd server/docker
docker-compose up -d

# View logs
docker logs complyx-postgres
docker logs complyx-redis

# Follow logs in real-time
docker logs -f complyx-postgres
```

## 🎯 Test Scenarios Covered

✅ Basic log levels (debug, info, warn, error)  
✅ HTTP request/response logging  
✅ Database query logging  
✅ External API logging  
✅ Performance metrics  
✅ Security events  
✅ Error handling with stack traces  
✅ Complex context logging  
✅ Request ID tracking  
✅ File logging (when enabled)  

## 🔧 Troubleshooting

**No logs showing?**
- Set `NODE_ENV=development`
- Set `LOG_LEVEL=debug` to see all levels
- Check console output (logs go to console in development)

**File logging not working?**
- Set `ENABLE_FILE_LOGGING=true`
- Check `LOG_DIR` is writable
- Ensure logs directory exists

**Docker logs empty?**
- Verify containers are running: `docker ps`
- Check docker-compose.yml logging configuration
- View logs: `docker logs <container-name>`
