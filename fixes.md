Application Fixes Documentation

Issues Found and Fixed

### 1. api/main.py - Line 8: Hardcoded Redis Host
**File:** `api/main.py`  
**Line:** 8  
**Problem:** Redis connection uses hardcoded `host="localhost"` which will fail in containerized environments. Containers cannot resolve "localhost" to other containers; they need the service name or hostname defined in the container orchestration setup.  
**What i changed:** Changed `redis.Redis(host="localhost", port=6379)` to use environment variables with fallback: `redis.Redis(host=os.getenv("REDIS_HOST", "localhost"), port=int(os.getenv("REDIS_PORT", 6379)))`  
**Impact expected:** Application will now work in containers when REDIS_HOST environment variable is set to the Redis service name.

---

### 2. frontend/app.js - Line 5: Hardcoded API URL
**File:** `frontend/app.js`  
**Line:** 5  
**Problem:** API endpoint is hardcoded to `"http://localhost:8000"` which will fail in containers. Frontend containers cannot reach backend on "localhost"; they need the service name or proper hostname.  
**What i Changed:** Changed `const API_URL = "http://localhost:8000";` to `const API_URL = process.env.API_URL || "http://localhost:8000";`  
**Impact expected:** Frontend can now connect to the API service when API_URL environment variable is set (e.g., `http://api:8000` in Docker).

---

### 3. frontend/views/index.html - Lines 27-33: Missing Error Handling in pollJob
**File:** `frontend/views/index.html`  
**Line:** 27-33 (pollJob function)  
**Problem:** The `pollJob` function doesn't handle error responses from the API. When a job is not found, the API returns `{"error": "not found"}` but the code tries to access `data.status` which will be undefined, causing the function to behave unexpectedly or enter an infinite loop.
**What i Changed:** Added error handling in pollJob function:
```javascript
async function pollJob(id) {
  const res = await fetch(`/status/${id}`);
  const data = await res.json();
  if (data.error) {
    renderJob(id, "error: not found");
    return;
  }
  renderJob(id, data.status);
  if (data.status !== 'completed') {
    setTimeout(() => pollJob(id), 2000);
  }
}
```  
**Impact expected:** Application will now gracefully handle cases where jobs don't exist instead of polling indefinitely or crashing.

---

### 4. worker/worker.py - Line 6: Hardcoded Redis Host
**File:** `worker/worker.py`  
**Line:** 6  
**Problem:** Redis connection uses hardcoded `host="localhost"` which will fail in containerized environments. Worker service won't be able to connect to Redis service.  
**What i Changed:** Changed `redis.Redis(host="localhost", port=6379)` to use environment variables with fallback: `redis.Redis(host=os.getenv("REDIS_HOST", "localhost"), port=int(os.getenv("REDIS_PORT", 6379)))`  
**Impact expected:** Worker will now work in containers when REDIS_HOST environment variable is set to the Redis service name.

---

## Summary of issues found

**Total Issues Found:** 4  
**Critical issues (Container-breaking):** 3 (hardcoded localhost connections)  
**High (Logic bug):** 1 (missing error handling)