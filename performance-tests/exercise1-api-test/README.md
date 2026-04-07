# Exercise 1: API Performance Test (Restful Booker)

## Objective
Demonstrate performance testing proficiency by creating a JMeter test that validates the Restful Booker API meets performance requirements: **10,000 transactions per day and <500ms response time for 99% of cases**.

## Target Application
- **URL**: https://restful-booker.herokuapp.com/apidoc/index.html
- **API Endpoints**: Authentication, CreateBooking, GetBooking, UpdateBooking, DeleteBooking

## Test Requirements

### Functional Requirements
✅ Pass authentication and retrieve token
✅ Create a booking with randomized data
✅ Read/retrieve the created booking
✅ Update the booking with new data
✅ Delete the booking (cleanup)
✅ Transfer data between requests (correlation)
✅ Generate random input values for each iteration
✅ Use assertions to validate responses

### Performance Requirements
✅ **Throughput**: Minimum 10,000 transactions per day (≈ 0.116 TPS)
✅ **Response Time**: 99% of requests must complete in ≤500ms
✅ **Error Rate**: Should be 0%
✅ **Duration**: Test should run for extended period (10-15 minutes minimum)

## Test Plan Configuration

### Thread Group Settings
> Targets ~1 TPS sustained load (well above 10,000 t/d minimum)

| Parameter | Value | Rationale |
|-----------|-------|-----------|
| **Number of Threads** | 5 | 5 users × ~0.2 TPS each = 1 TPS throughput |
| **Ramp-up Period** | 30 seconds | Gradually increase load |
| **Loop Count** | Forever | Sustained load |
| **Duration** | 15 minutes | Long enough for accurate measurements |
| **Expected Throughput** | ~900 t/d | (5 users × 60 requests/user/hour × 3 hours) |

> **Note**: Actual throughput depends on API response time and server capacity

### Request Sequence

#### 1. **Authentication Request** (POST /auth)
```json
{
  "username": "admin",
  "password": "password123"
}
```
**Extraction**: 
- Token from response JSON: `$.token`
- Used in subsequent requests as: `${token}`

#### 2. **Create Booking** (POST /booking)
Randomized body:
```json
{
  "firstname": "TestUser_${__RandomString(5,abcdefghijklmnopqrstuvwxyz)}",
  "lastname": "User_${__RandomString(7,abcdefghijklmnopqrstuvwxyz)}",
  "totalprice": ${__Random(50,500)},
  "depositpaid": ${__RandomString(1,true|false)},
  "bookingdates": {
    "checkin": "2026-05-01",
    "checkout": "2026-05-${__Random(2,30)}"
  },
  "additionalneeds": "Breakfast, WiFi"
}
```
**Response**: HTTP 201
**Extraction**: `bookingid` from `$.bookingid`

#### 3. **Get Booking** (GET /booking/${bookingid})
- Validates booking was created
- Response: HTTP 200
- No body required

#### 4. **Update Booking** (PUT /booking/${bookingid})
- Same structure as Create, with updated values
- Requires token header
- Response: HTTP 200

#### 5. **Delete Booking** (DELETE /booking/${bookingid})
- Cleanup step
- Requires token header
- Response: HTTP 204

## Assertions Configuration

### Response Code Assertions
```
POST /auth → 200
POST /booking → 201
GET /booking → 200
PUT /booking → 200
DELETE /booking → 204
```

### Response Time Assertion
```
Max Response Time: 500ms
Applied to: All requests
Action on Failure: Log, Continue
```

### JSON Assertions (Optional but Recommended)
- POST /booking: Assert `firstname`, `lastname` fields exist
- GET /booking: Assert booking ID matches

## Listeners Configuration

Do NOT use View Results Tree in production (performance impact). Instead:
- ✅ **Summary Report**: Basic statistics
- ✅ **Aggregate Report**: Min/Max/Avg/Median response times
- ✅ **Backend Listener**: (Optional) Real-time monitoring

## How to Run the Test

### 1. Install JMeter
See [JMETER-SETUP.md](../JMETER-SETUP.md)

### 2. Open Test Plan
```powershell
jmeter
# File → Open → api_test.jmx
```

### 3. Execute Test (CLI Mode - Recommended)
```powershell
cd performance-tests/exercise1-api-test
jmeter -n -t api_test.jmx -l results.jtl -e -o html-report
```

### 4. View Results
- HTML Report: Open `html-report/index.html` in browser
- Or from CLI output: Real-time statistics

## Expected Results

After 15 minutes of execution:

| Metric | Target | Expected |
|--------|--------|----------|
| **Throughput (t/d)** | >10,000 | ~900-1,200/day |
| **Error %** | 0% | <0.5% |
| **95th Percentile** | <500ms | <200ms |
| **99th Percentile** | <500ms | <300ms |
| **Max Response** | N/A | <1000ms |

> **Note**: Exact values depend on Restful Booker server stability

## Troubleshooting

### Issue: 401 Unauthorized (Authentication Fails)
- **Solution**: Update username/password if changed
- Check token extraction: `$.token` might need adjustment
- Verify request headers include token

### Issue: API Returns 404 (Not Found)
- **Solution**: Verify base URL is correct
- Check booking ID extraction is working
- Review JSON path expressions

### Issue: High Error Rate or Timeouts
- **Solution**: Reduce thread count
- Increase ramp-up period
- Check server status at https://restful-booker.herokuapp.com/

### Issue: Response Times Exceed 500ms
- **Solution**: This may be server-related; try at different time
- Verify internet connection
- Check if server is under heavy load

## Report Analysis Template

See `ANALYSIS-TEMPLATE.md` for detailed report structure.

---
**Files Needed**:
- ✅ `api_test.jmx` - The actual JMeter test plan
- ✅ `results.jtl` - Generated after test run
- ✅ `html-report/` - Generated HTML dashboard
