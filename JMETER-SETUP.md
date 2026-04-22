# JMeter Setup Guide

## Prerequisites
- ✅ Java 17 (already installed: OpenJDK Temurin-17.0.18)

## Installation Steps (Windows)

### Option 1: Automatic Installation via winget (Recommended)
```powershell
winget install --id DEVCOM.JMeter --accept-source-agreements --accept-package-agreements
```

### Option 2: Automatic Installation via Chocolatey
If `choco` is not recognized, install Chocolatey first: https://chocolatey.org/install

```powershell
choco install jmeter
```

### Option 3: Manual Installation
1. Download JMeter from [jmeter.apache.org](https://jmeter.apache.org/)
   - Download: `apache-jmeter-5.6.x.zip` (or latest stable version)
2. Extract to a location (e.g., `C:\tools\apache-jmeter-5.6.3`)
3. Add to PATH:
   - Open Environment Variables
   - Add: `C:\tools\apache-jmeter-5.6.3\bin` to System PATH
4. Verify installation:
   ```powershell
   jmeter -version
   ```

### Option 4: Download and Run Portably (No Install)
1. Download from `/bin/ApacheJMeter.exe` in the ZIP
2. Extract anywhere and run directly

## Verification
```powershell
jmeter -version
# Expected output: ApacheJMeter 5.6.x (or later)
```

If `jmeter` is still not recognized right after install, open a new PowerShell window and run again.

## Quick Start

### GUI Mode (Test Design)
```powershell
jmeter
```
- Used for creating and debugging test plans

### CLI Mode (Test Execution - Recommended for Reports)
```powershell
jmeter -n -t test_plan.jmx -l results.jtl -e -o html-report
```
- `-n`: Non-GUI mode (faster, less memory)
- `-t`: Test plan file
- `-l`: Results file (JTL format)
- `-e`: Generate HTML report
- `-o`: Output directory for HTML report

## System Recommendations
- Minimum: 2GB RAM for 1000+ concurrent users
- For this lab: Default settings fine
- Disable GUI elements for larger tests (use CLI mode)
