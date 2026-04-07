# Exercise 2: Website Performance Test (Demoblaze)

## Objective
Demonstrate web application performance testing by creating a realistic JMeter test that simulates a complete e-commerce user journey and validate results match real-world browser measurements.

## Target Application
- **URL**: https://www.demoblaze.com/
- **Scenario**: User registration → login → product selection → add to cart → checkout

## Test Requirements

### Functional Requirements
✅ User registration (Sign up form)
✅ User login (Log in form)
✅ Product selection and browsing
✅ Add product to cart
✅ Open cart and verify items
✅ Complete purchase flow (Place Order)
✅ Fill order details and confirm
✅ Transfer data between requests (correlation)
✅ Use assertions to validate key responses
✅ Randomize user/email data for each iteration
✅ Add realistic think time (user delays)

### Performance Requirements
✅ **Test Execution**: Run with 1 virtual user (baseline for comparison)
✅ **Response Validation**: Total transaction time matches browser DevTools Network measurements
✅ **Test Trustworthiness**: Results should closely align with manual navigation

## Test Plan Structure

### Controllers (Transaction Level Timing)
Use **Transaction Controllers** to group steps and measure total time. Each transaction should represent a logical user action:

```
Test Plan
├── Thread Group (1 user)
├── Transaction: Sign Up
│   ├── Open Website
│   ├── Click Sign Up
│   ├── Fill Form
│   └── Submit
├── Transaction: Log In
│   ├── Click Log In
│   ├── Fill Credentials
│   └── Submit
├── Transaction: Browse Products
│   ├── Open Products Page
│   └── Select Product
├── Transaction: Add to Cart
│   ├── Add Item
│   └── Confirm
├── Transaction: Open Cart
│   └── View Cart Items
├── Transaction: Checkout
│   ├── Click Place Order
│   ├── Fill Order Details
│   └── Confirm Purchase
└── Listeners
```

## Request Recording Strategy

### Recommended Method: JMeter Test Script Recorder

1. **Configure Proxy**:
   ```powershell
   jmeter
   # Workbench → HTTP(S) Test Script Recorder
   # Port: 8888
   # Start
   ```

2. **Configure Browser Proxy**:
   - Chrome: Settings → Advanced → System → Proxy Settings
   - Set HTTP Proxy: `localhost:8888`

3. **Record Actions**:
   1. Navigate to https://www.demoblaze.com/
   2. Click "Sign up"
   3. Register: Email `testuser_${RANDOM}@test.com`, Password `TestPass123`
   4. Click "Log in"
   5. Log in with credentials
   6. Browse products
   7. Select one product
   8. Click "Add to cart"
   9. Click "Cart" menu
   10. Click "Place order"
   11. Fill order details (name, country, city, card, etc.)
   12. Click "Purchase"

### Alternative: Manual Request Configuration
If recording is problematic, manually add HTTP requests with recorded URLs.

## Post-Recording Cleanup

### Remove Unwanted Requests
Delete requests for:
- ❌ Image files (*.jpg, *.png, *.gif)
- ❌ CSS files (*.css)
- ❌ Font files (*.woff, *.ttf)
- ❌ Analytics (Google Analytics, tracking pixels)
- ❌ Advertisement requests
- ❌ Unnecessary OPTIONS calls (preflight requests)

### Keep These Requests
- ✅ HTML page loads
- ✅ API calls (product list, cart operations)
- ✅ JSON responses
- ✅ Form submissions
- ✅ Checkout/payment processing

## Timers Configuration

### Uniform Random Timer
Add after each major step to simulate user think time:

| Step | Min Delay | Max Delay | Rationale |
|------|-----------|-----------|-----------|
| After page load | 2 sec | 5 sec | User reads page |
| After product view | 3 sec | 8 sec | User decides |
| Before checkout | 5 sec | 10 sec | User verification |

**Configuration**:
- Random Delay Maximum: 5000 ms
- Constant Delay Offset: 2000 ms
- Result: 2000-7000ms random wait

## Data Correlation & Extraction

### Extract and Reuse Dynamic Values

#### User Registration (POST /api/signup)
**Extract from response**:
```json
Response contains: "User created successfully" or similar
Verify: Email is registered
```

#### User Login (POST /api/login)
**Extract**:
- `sessionId` or `token` (if provided in response)
- Store as: `${sessionToken}`

#### Product Selection (GET /api/products)
**Extract**:
- First 5 product IDs from JSON response
- Pick random: `${__RandomElement(product_ids)}`

#### Add to Cart (POST /api/cart)
**Extract**:
- `cartId` from response
- Store as: `${cartId}`

#### Checkout (POST /api/order)
**Extract on success**:
- Order ID from response
- Use in confirmation step

**Example JSON Extraction**:
```
Response JSON Path: $.cartId
Reference Name: cartId
```

## Assertions Configuration

### Response Code Assertions
```
GET / → 200 (homepage)
POST /api/signup → 200 or 201 (registration)
POST /api/login → 200 (login)
GET /api/products → 200 (product list)
POST /api/cart → 200 or 201 (cart operations)
POST /api/order → 200 or 201 (order placement)
```

### Text Assertions
- Signup: Assert response contains "successfully registered" or similar
- Login: Assert no error message
- Order: Assert contains "Purchase successful" or confirmation number

### Duration Assertions
- Per request: < 5 seconds (should be much faster normally)
- Entire transaction: < 30 seconds

## How to Run the Test

### 1. Install JMeter
See [JMETER-SETUP.md](../JMETER-SETUP.md)

### 2. Open Test Plan
```powershell
jmeter
# File → Open → web_test.jmx
```

### 3. Execute Test (1 Virtual User)
```powershell
cd performance-tests/exercise2-web-test
jmeter -n -t web_test.jmx -l results.jtl -e -o html-report
```

### 4. Compare with Browser Measurements

**Real-World Baseline (Browser DevTools)**:
1. Open https://www.demoblaze.com/ with DevTools Network tab
2. Record timings for each step (manually):
   - Sign up: ____ ms
   - Log in: ____ ms
   - Product page load: ____ ms
   - Add to cart: ____ ms
   - Cart page: ____ ms
   - Checkout: ____ ms
   - **Total**: ____ ms

**JMeter Results**:
- Open `html-report/index.html`
- Note transaction times under "Statistics"
- Compare with browser measurements

## Expected Results & Validation

### Success Criteria
✅ All requests return HTTP 200-201 (no errors)
✅ No assertions fail
✅ Total transaction time within ±20% of browser DevTools
✅ No timeout errors
✅ Test completes successfully with purchased order

### Example Comparison Table

| Step | Browser (DevTools) | JMeter Result | Variance | Status |
|------|-------------------|---------------|----------|--------|
| Sign Up | 850ms | 920ms | +8.2% | ✅ PASS |
| Log In | 650ms | 710ms | +9.2% | ✅ PASS |
| Product Browse | 1200ms | 1280ms | +6.7% | ✅ PASS |
| Add to Cart | 450ms | 520ms | +15.6% | ✅ PASS |
| Checkout | 2100ms | 2350ms | +11.9% | ✅ PASS |
| **TOTAL** | **5250ms** | **5780ms** | **+10.1%** | ✅ PASS |

> **Variance up to ±20% is acceptable due to network variability**

### Analysis Conclusion Example
> "The JMeter test results closely matched real-world browser measurements, with a total variance of 10.1%. The test can be trusted for baseline performance evaluation. The add-to-cart transaction showed the highest variance (15.6%), likely due to backend processing time variability."

## Troubleshooting

### Issue: Cannot Access www.demoblaze.com
- **Solution**: Check internet connectivity
- Website may be temporarily down; try again later

### Issue: "Purchase Successful" Assertion Fails
- **Solution**: Check if order was actually placed (site may block repeated orders)
- Verify email extraction is working
- Check form field names using browser DevTools

### Issue: Data Correlation Error (productId not found)
- **Solution**: Verify JSON extraction path using actual response
- Right-click request → View Results Tree → Check Response
- Adjust JSON path expression accordingly

### Issue: Timeouts (>10 seconds)
- **Solution**: Internet connection issue
- Website may be slow; test at different time
- Increase timeout values in test plan

## How to Proceed

1. ✅ Run browser at https://www.demoblaze.com/ with DevTools Network open
2. ✅ Record timings for each user step
3. ✅ Create/open `web_test.jmx` in JMeter
4. ✅ Execute test with 1 virtual user
5. ✅ Generate HTML report
6. ✅ Compare JMeter timings with browser DevTools
7. ✅ Complete analysis (see ANALYSIS-TEMPLATE.md)

---
**Files Needed**:
- ✅ `web_test.jmx` - The actual JMeter test plan
- ✅ `results.jtl` - Generated after test run
- ✅ `html-report/` - Generated HTML dashboard
- ✅ Browser Network timings (recorded manually with DevTools)
