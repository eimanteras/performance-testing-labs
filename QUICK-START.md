# Performance Testing Labs - Quick Start Guide

Welcome! This guide will help you complete both performance testing exercises using JMeter.

---

## 📋 Workspace Structure

```
performance-testing-labs/
├── README.md                           # Project overview
├── JMETER-SETUP.md                    # JMeter installation guide
├── QUICK-START.md                     # This file
│
└── performance-tests/
    ├── exercise1-api-test/            # Restful Booker API test
    │   ├── README.md                  # Detailed exercise instructions
    │   ├── api_test.jmx               # JMeter test plan (ready to use)
    │   ├── ANALYSIS-TEMPLATE.md       # Report template
    │   ├── results.jtl                # Generated after test run
    │   ├── html-report/               # Generated HTML dashboard
    │   └── screenshots/               # Store test result screenshots
    │
    └── exercise2-web-test/            # Demoblaze website test
        ├── README.md                  # Detailed exercise instructions
        ├── web_test.jmx               # JMeter test plan (ready to use)
        ├── ANALYSIS-TEMPLATE.md       # Report template
        ├── results.jtl                # Generated after test run
        ├── html-report/               # Generated HTML dashboard
        └── screenshots/               # Store test result screenshots
```

---

## 🚀 Getting Started in 5 Steps

### Step 1: Install JMeter

Follow the guide: See [JMETER-SETUP.md](./JMETER-SETUP.md)

**Quick Option (Chocolatey)**:
```powershell
# Install Java (if not already installed)
choco install openjdk

# Install JMeter
choco install jmeter

# Verify
jmeter -version
```

### Step 2: Choose an Exercise

- **Exercise 1**: API Performance Test (Restful Booker)
  - Go to: `performance-tests/exercise1-api-test/`
  - Read: `README.md`
  - Test file: `api_test.jmx`

- **Exercise 2**: Website Performance Test (Demoblaze)
  - Go to: `performance-tests/exercise2-web-test/`
  - Read: `README.md`
  - Test file: `web_test.jmx`

### Step 3: Run the Test

**Exercise 1 (API Test)**:
```powershell
cd performance-tests/exercise1-api-test

# Run test and generate HTML report
jmeter -n -t api_test.jmx -l results.jtl -e -o html-report

# View results
Start-Process html-report/index.html
```

**Exercise 2 (Website Test)**:
```powershell
cd performance-tests/exercise2-web-test

# Before running JMeter:
# 1. Open https://www.demoblaze.com in browser (F12 → Network tab)
# 2. Manually perform each step and record times
# 3. Save screenshots of Network tab timings

# Run JMeter test
jmeter -n -t web_test.jmx -l results.jtl -e -o html-report

# View results
Start-Process html-report/index.html
```

### Step 4: Analyze Results

1. **Open HTML Report**:
   - Navigate to `html-report/index.html`
   - Review Statistics, Response Times, Transaction Controllers

2. **Extract Key Metrics**:
   - From HTML report or `results.jtl` file
   - Record: throughput, avg response time, 99th percentile, error rate

3. **Fill Analysis Template**:
   - Exercise 1: Open `ANALYSIS-TEMPLATE.md`
   - Exercise 2: Open `ANALYSIS-TEMPLATE.md`
   - Fill in all metrics and analysis sections

### Step 5: Submit Report

- **Save as**: `exercise1-report.md` or `exercise2-report.md`
- **Include**: Filled analysis template + screenshots
- Submit both file and HTML report folder

---

## 📊 Exercise 1: API Performance Test

### Quick Facts
| Property | Value |
|----------|-------|
| **Target URL** | https://restful-booker.herokuapp.com |
| **Test Duration** | 15 minutes |
| **Threads** | 5 users |
| **Performance Goal** | 10,000 t/d, <500ms 99% |
| **Expected Time** | 15-20 min (including setup) |

### What Gets Tested
1. ✅ Authentication (POST /auth)
2. ✅ Create Booking (POST /booking) - with random data
3. ✅ Read Booking (GET /booking/{id})
4. ✅ Update Booking (PUT /booking/{id})
5. ✅ Delete Booking (DELETE /booking/{id})

### Expected Results
```
Total Requests:        ~900 (over 15 minutes)
Throughput:            ~900 t/d
99th Percentile:       <500ms           ✅
Error Rate:            ~0%              ✅
All Status Codes OK:   ✅
```

### Key Metrics to Track
- **Throughput (t/d)**: `(Total Requests / Duration) × 86400`
- **99th Percentile**: Find in HTML Report → "Statistics"
- **Error Rate**: `(Failed Requests / Total Requests) × 100`

---

## 📊 Exercise 2: Website Performance Test

### Quick Facts
| Property | Value |
|----------|-------|
| **Target URL** | https://www.demoblaze.com |
| **Test Duration** | Single run (~5-10 seconds) |
| **Threads** | 1 user (baseline only) |
| **Comparison** | JMeter vs Browser DevTools |
| **Expected Time** | 10-15 min (including browser timing) |

### What Gets Tested
1. ✅ Homepage load
2. ✅ User registration
3. ✅ User login
4. ✅ Product selection
5. ✅ Add to cart
6. ✅ View cart
7. ✅ Checkout form
8. ✅ Order confirmation

### Expected Results
```
Browser Total:       ~5,000-8,000 ms
JMeter Total:        ~5,200-8,200 ms
Variance:            ±15% (within acceptable range)
All Assertions:      PASS ✅
Test Trustworthy:    YES ✅
```

### Key Comparison Points
| Step | Browser | JMeter | Match? |
|------|---------|--------|--------|
| Homepage | _____ ms | _____ ms | ✅/❌ |
| Sign Up | _____ ms | _____ ms | ✅/❌ |
| Log In | _____ ms | _____ ms | ✅/❌ |
| ... | ... | ... | ... |

---

## 🛠️ Common Tasks

### Run Test and Generate Report
```powershell
cd performance-tests/exercise1-api-test
jmeter -n -t api_test.jmx -l results.jtl -e -o html-report
```

### View Test Plan in GUI
```powershell
cd performance-tests/exercise1-api-test
jmeter
# File → Open → api_test.jmx
```

### Find 99th Percentile Response Time
1. Open `html-report/index.html`
2. Go to "Statistics" section
3. Look for column "99% Line"

### Extract Throughput
1. From CLI output after test completes, or
2. Open `results.jtl` file (it's a CSV)
3. Count total requests, divide by duration, multiply by 86400

### Check for Assertion Failures
1. Open `html-report/index.html`
2. Check "Summary Report" → Error % column
3. If >0%, review "View Results Tree" for details

---

## ⚠️ Troubleshooting

### "JMeter command not found"
**Solution**: 
1. Verify installation: `jmeter -version`
2. If not found, add JMeter/bin to PATH
3. Restart PowerShell and try again

### "Error: Cannot open test file"
**Solution**:
1. Verify file exists: `ls api_test.jmx`
2. Use absolute path if relative path fails
3. Check filename spelling

### "API returns 401 Unauthorized"
**Solution**:
1. Check credentials are correct (admin / password123)
2. Verify token extraction in /auth request
3. Check Bearer token format in subsequent requests

### "HTML Report not generated"
**Solution**:
1. Verify test ran successfully (check `results.jtl` was created)
2. Use `-e -o html-report` flags
3. Try again with newer JMeter version

### "Website test shows high variance (>20%)"
**Solution**:
1. Internet connection may be unstable
2. Website may have been under load
3. Browser think times may be inconsistent
4. Run test again at different time

### "Data not transferring between requests"
**Solution**:
1. Check JSON Extraction paths in test plan
2. Verify variable names match (use `${variableName}`)
3. In JMeter GUI, check "View Results Tree" for extracted values

---

## 📝 Analysis Report Checklist

### Before You Start Writing
- [ ] Test completed successfully (no critical errors)
- [ ] HTML report generated and reviewed
- [ ] All metrics extracted from HTML report
- [ ] Screenshots taken of key sections
- [ ] For Exercise 2: Browser timings recorded

### While Writing Report
- [ ] Fill all required fields in template
- [ ] Include actual measured values (not estimates)
- [ ] Answer all analysis questions
- [ ] Provide clear justifications for conclusions
- [ ] Check for spelling/grammar errors
- [ ] Include all required artifacts/screenshots

### Final Quality Check
- [ ] Report is complete (no blank sections)
- [ ] Math is correct (e.g., throughput calculation)
- [ ] Conclusions match data presented
- [ ] Report answers: "Are requirements met?"
- [ ] For Exercise 2: "Can test be trusted?" is answered

---

## 📚 Reference Documentation

### Full Guides
- [Exercise 1 Detailed Guide](./performance-tests/exercise1-api-test/README.md)
- [Exercise 2 Detailed Guide](./performance-tests/exercise2-web-test/README.md)
- [JMeter Setup Guide](./JMETER-SETUP.md)

### Templates
- [Exercise 1 Analysis Template](./performance-tests/exercise1-api-test/ANALYSIS-TEMPLATE.md)
- [Exercise 2 Analysis Template](./performance-tests/exercise2-web-test/ANALYSIS-TEMPLATE.md)

### JMeter Resources
- [JMeter Official Documentation](https://jmeter.apache.org/usermanual/)
- [Restful Booker API Docs](https://restful-booker.herokuapp.com/apidoc/index.html)
- [Demoblaze Website](https://www.demoblaze.com/)

---

## ⏱️ Estimated Timeline

### Exercise 1 (API Test)
- Setup & JMeter install: 10-15 min
- Understanding test plan: 5-10 min
- Running test (15 min duration): 15 min
- Analyzing results: 10-15 min
- Writing report: 20-30 min
- **Total: 60-85 minutes**

### Exercise 2 (Website Test)
- Setup & test plan review: 10-15 min
- Record browser timings manually: 15-20 min
- Run JMeter test: 5-10 min
- Comparing results: 10-15 min
- Writing report: 20-30 min
- **Total: 60-90 minutes**

### Total Lab Time
**2-3 hours** for both exercises with analysis and reporting

---

## 🎯 Success Criteria

- [ ] JMeter successfully installed and tested
- [ ] Exercise 1 test completes without critical errors
- [ ] Exercise 1 report identifies if 10,000 t/d requirement is met
- [ ] Exercise 1 report shows 99% response time performance
- [ ] Exercise 2 test completes with 1 virtual user
- [ ] Exercise 2 validates browser timings vs JMeter results
- [ ] Exercise 2 clearly states if test is trustworthy
- [ ] Both reports are complete with all metrics and analysis
- [ ] All assertions in test plans pass
- [ ] HTML reports generated successfully

---

## 📧 Support

If you encounter issues:

1. **Check the README** for your exercise first
2. **Review Troubleshooting** section above
3. **Check HTML report** for assertion/error details
4. **Look at results.jtl** file for raw data

---

## 🎓 Learning Objectives Covered

By completing these exercises, you will demonstrate:

✅ **Performance Testing Design**: Create realistic, production-like tests
✅ **JMeter Proficiency**: Test plan creation, execution, result analysis
✅ **Data Correlation**: Transfer data between requests realistically
✅ **Performance Analysis**: Interpret metrics and identify bottlenecks
✅ **Requirements Validation**: Verify systems meet defined SLOs
✅ **Real-World Alignment**: Compare synthetic tests vs actual behavior

---

Good luck! 🚀

**Ready to start?** → Read the README in your chosen exercise folder
