# JMeter Reference Guide - Functions & Components Used

This guide provides quick reference for JMeter components and functions used in the test plans.

---

## 🧩 Components Overview

### Thread Group
Controls how many concurrent users (threads) run the test.

**Example Settings**:
```
Number of Threads:  5       (5 concurrent users)
Ramp-up Period:    30       (gradually add 1 user every 6 seconds)
Loop Count:        -1       (-1 = infinite loops, until duration ends)
Duration:          900      (900 seconds = 15 minutes)
```

**When to Use**:
- Multiple threads = simulating concurrent users
- Ramp-up = gradually increase load (realistic)
- Duration = run test for specific time period

---

### HTTP Sampler (Request)
Makes HTTP requests to the server.

**Example**:
```
Method:      POST
URL/Path:    /auth
Domain:      api.example.com
Headers:     Content-Type: application/json
Body:        {"username":"admin","password":"password123"}
```

**Status Codes**:
- 200 = OK
- 201 = Created
- 204 = No Content
- 400 = Bad Request
- 401 = Unauthorized
- 404 = Not Found
- 500 = Server Error

---

### JSON Post-Processor
Extracts data from JSON response to use in next requests.

**Purpose**: Data correlation between requests

**Syntax**:
```
JSON Path Expression: $.token
Reference Name:      token
Use as:             ${token}   (in next requests)
```

**Common JSON Paths**:
```
$.fieldname           → Top-level field
$.array[0]            → First item in array
$.data.nested.value   → Nested properties
$[*].id               → All IDs in array
```

**Example**:
1. Request 1 returns: `{"token":"xyz123","id":456}`
2. Post-Processor extracts:
   - `$.token` → stores as `${token}`
   - `$.id` → stores as `${id}`
3. Request 2 uses: `/api/users/${id}` and header `Authorization: ${token}`

---

### Response Assertion
Validates that responses match expected values.

**Common Assertions**:

1. **Response Code**:
   ```
   Expected:  200
   Field:     Response Code
   Pattern:   200 (exact match)
   ```

2. **Text Content**:
   ```
   Expected:  "success"
   Field:     Response Body
   Pattern:   success (substring match)
   ```

3. **JSON Field**:
   ```
   Expected:  true
   Field:     Response Body
   Pattern:   $.success
   ```

**Action on Failure**:
- Log and Continue (test keeps going)
- Log and Stop (test stops)
- Stop Thread (just this thread stops)

---

### Duration Assertion
Ensures response time is within acceptable range.

**Example**:
```
Duration:     500 ms
Success:      Request completes in ≤500ms
Failure:      Request takes >500ms (marked as failed assertion)
```

---

### Transaction Controller
Groups multiple requests and measures total time.

**Purpose**: Measure combined time for a user step

**Example** (Exercise 2):
```
Transaction: Sign Up
├── Load sign-up form
├── Submit form
└── Wait for confirmation
Total Time: ~2 seconds (all three requests combined)
```

---

### Timer
Adds delays between requests to simulate user "think time".

**Uniform Random Timer**:
```
Constant Delay Offset:    2000 ms (minimum)
Random Delay Range:       5000 ms (additional random)
Result:                   2000-7000 ms total delay
```

**When to Use**:
- After page loads (user reads page)
- After form submission (user reviews data)
- Between product browsing (user thinks)

---

## 🔧 JMeter Functions

Functions in JMeter use syntax: `${__FunctionName(args)}`

### Random Number Generator
```
${__Random(min,max)}

Example:  ${__Random(50,500)}     → Generates number between 50-500
Usage:    "totalprice": ${__Random(50,500)}
Result:   "totalprice": 347
```

### Random String Generator
```
${__RandomString(length, charset)}

Example:  ${__RandomString(5, abcdefghijklmnopqrstuvwxyz)}
Result:   xkqpz
Full:     "firstname": "User_${__RandomString(5,abc...)}"
          "firstname": "User_xkqpz"
```

### Random Element from List
```
${__RandomElement(item1,item2,item3)}

Example:  ${__RandomElement(true,false)}
Result:   true (or false, randomly)
```

---

## 📊 Understanding the Test Plans

### Exercise 1: API Test Flow

```
1. POST /auth
   ↓ Extract Token
   ↓
2. POST /booking (with token, random data)
   ↓ Extract BookingID
   ↓
3. GET /booking/{id} (using extracted ID)
   ↓
4. PUT /booking/{id} (update with new random data)
   ↓
5. DELETE /booking/{id} (cleanup)
   ↓
Loop back to step 1 (5 threads × N iterations)
```

**Key Extractions**:
- Step 1: `$.token` → `${token}`
- Step 2: `$.bookingid` → `${bookingid}`
- Used in: Steps 3, 4, 5

### Exercise 2: Website Test Flow

```
Transaction 1: Load Homepage
   ├── GET /
   └── Think Time (2-5 sec)
       ↓
Transaction 2: Sign Up
   ├── POST /api/signup
   └── Think Time (2-3 sec)
       ↓
Transaction 3: Log In
   ├── POST /api/login
   └── Think Time (3-5 sec)
       ↓
... (more transactions)
       ↓
Transaction 8: Complete Checkout
   ├── POST /api/order
   └── Extract Order ID
       ↓
End of Test (1 iteration, 1 user)
```

**Key Extractions**:
- Step 4: `$[0].id` → `${productId}`
- Step 5: `$.id` → `${cartId}`
- Step 8: `$.id` → `${orderId}`

---

## 🎯 Metrics Explained

### Throughput
Definition: Requests per unit time

**Calculation**:
```
Throughput (requests/second) = Total Requests / Duration (seconds)

Example:
Total Requests:   900
Duration:         900 seconds (15 minutes)
Throughput:       900 / 900 = 1 request/second

Convert to daily:
1 req/sec × 86,400 sec/day = 86,400 requests/day
```

### Response Time Percentiles
Definition: The response time below which X% of requests fall

**Example** (below 500ms requirement):
```
Min:        50 ms   (fastest request)
50th %ile:  150 ms  (median - half are faster, half slower)
95th %ile:  300 ms  (95% of requests are faster than this)
99th %ile:  450 ms  ← THIS IS THE REQUIREMENT (<500ms)
Max:        950 ms  (slowest request)
```

**Interpretation**:
- 99% of users experience ≤450ms response time
- Only 1% of users might see >450ms
- Requirement: 99% must be <500ms ✅ PASS

### Error Rate
Definition: Percentage of failed requests

**Calculation**:
```
Error Rate = (Failed Requests / Total Requests) × 100

Example:
Total Requests:    1000
Failed Requests:   5 (assertions failed, or timeouts)
Error Rate:        (5 / 1000) × 100 = 0.5%

Requirement: <1% ✅ PASS
```

---

## 🔍 Reading JMeter HTML Report

### Key Sections

**Summary Report**
- Total requests made
- Successful count
- Error count and rate
- Average response time

**Statistics Table**
- Min/Max/Mean/Median response times
- **95th and 99th percentiles** (what you need!)
- Standard deviation
- Requests per second

**Response Times Over Time**
- Graph showing how response times varied
- Helps identify if system slowed down during test

**Requests Per Second**
- Shows if load was consistent
- Helps verify test ran for full duration

### Where to Find Key Metrics

| Metric | Location |
|--------|----------|
| **Throughput (t/d)** | Summary Report or Stats table (Requests/sec) |
| **99th Percentile** | Statistics → "99% Line" column |
| **Error Rate** | Summary Report "Error %" or count in table |
| **Avg Response Time** | Statistics → "Average" column |
| **Per-request times** | Statistics → by request name |

---

## 💡 Common Mistakes & Fixes

### Mistake 1: Token extraction not working
**Problem**: Next request gets 401 Unauthorized
**Fix**:
```
1. Check JSON path is correct: $.token (not $[token] or $.data.token)
2. Verify extraction comes AFTER the auth request
3. Check variable is used correctly: ${token} (with $ and {})
4. View Results Tree to see extracted values
```

### Mistake 2: Assertions fail but test says "success"
**Problem**: "All green" in summary but some requests failed
**Fix**:
```
1. Assertions marked "Continue on Error" don't stop test
2. Check assertion failure count in report
3. Review specific failed assertions under "Assertions" tab
```

### Mistake 3: Response time way too high
**Problem**: Average >2000ms (seems slow)
**Fix**:
```
1. Check if Test Script Recorder is running (slows down traffic)
2. Verify internet connection
3. Server might be under actual load
4. Try test at different time
5. Disable View Results Tree listener (impacts performance)
```

### Mistake 4: "Variable not found" in URL
**Problem**: Request to `/booking/${bookingid}` fails
**Fix**:
```
1. Check spelling: ${bookingid} vs ${bookingId} (case-sensitive!)
2. Ensure Post-Processor runs BEFORE it's used
3. View Results Tree to confirm variable was extracted
```

---

## 🚀 Optimizations & Tips

### For Better Response Times in Reports
- Use **CLI mode** (`jmeter -n`) not GUI
  - GUI mode takes more resources
  - Results will be slower than real-world usage
  
- Remove **View Results Tree** listener in production
  - It stores all responses in memory
  - Can slow down test

- Use **CSV Data Set Config** for large datasets
  - More efficient than random generation
  - Better for load testing

### For More Accurate Tests
- Run test **multiple times** to get average
- Exclude first few iterations (warm-up period)
- Test at **off-peak hours** for APIs
- Match **real user behavior** (think times, randomization)

### For Better Data Correlation
- Always extract IDs/tokens from responses
- Don't hard-code values (every iteration should be unique)
- Use random delays (user think time)
- Verify extracted values in View Results Tree

---

## 📖 Additional Resources

- [JMeter Built-in Functions](https://jmeter.apache.org/usermanual/functions.html)
- [JSON Path Syntax](https://goessner.net/articles/JsonPath/)
- [HTTP Status Codes](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status)
- [Performance Testing Best Practices](https://jmeter.apache.org/usermanual/best-practices.html)

---

**Last Updated**: April 2026  
**JMeter Version**: 5.6+
