# ✅ 5 Dedicated API Endpoints - Ready to Use!

## 🎯 Overview

You **already have** 5 dedicated API endpoints for test data generation! They were created in a previous session and are ready to use.

---

## 📍 Your 5 API Endpoints

### **Base URL:** `http://localhost:8000`

| # | Endpoint | Type | Auto-Sets |
|---|----------|------|-----------|
| 1 | `/api/testdata/generate/security` | 🔒 Security | `testDataType='security'` |
| 2 | `/api/testdata/generate/boundary` | 📏 Boundary | `testDataType='boundary'` |
| 3 | `/api/testdata/generate/equivalence` | ⚖️ Equivalence | `testDataType='equivalence'` |
| 4 | `/api/testdata/generate/positive` | ✅ Positive | `testDataType='positive'` |
| 5 | `/api/testdata/generate/negative` | ❌ Negative | `testDataType='negative'` |

---

## 🚀 Quick Start

### **1. Start the AI Service**
```bash
cd c:\chandra-1212-main\ai-analysis-service
python main.py
```

### **2. Test an Endpoint**
```bash
# Windows PowerShell
$body = @{
    template = @{
        email = "{{faker.email}}"
        password = "{{faker.password}}"
    }
    count = 10
} | ConvertTo-Json

Invoke-WebRequest -Uri "http://localhost:8000/api/testdata/generate/security" `
    -Method POST `
    -ContentType "application/json" `
    -Body $body
```

**OR using Python:**
```python
import requests

response = requests.post(
    'http://localhost:8000/api/testdata/generate/security',
    json={
        'template': {
            'email': '{{faker.email}}',
            'password': '{{faker.password}}'
        },
        'count': 10
    }
)
print(response.json())
```

---

## 🔐 1. Security Endpoint

**POST** `/api/testdata/generate/security`

**What it generates:**
- SQL Injection: `admin'--`, `' OR '1'='1'--`
- XSS: `<script>alert('XSS')</script>`
- Command Injection: `; ls -la`, `| cat /etc/passwd`
- Path Traversal: `../../etc/passwd`
- Auth Bypass: `admin' OR '1'='1`
- CSRF, LDAP, XML, NoSQL injection

**Example Request:**
```json
{
  "template": {
    "email": "{{faker.email}}",
    "password": "{{faker.password}}"
  },
  "count": 15
}
```

**Example Response:**
```json
{
  "success": true,
  "data": [
    {
      "email": "admin'--",
      "password": "anything",
      "_description": "SQL Injection - Authentication Bypass",
      "_attack_vector": "sql_injection"
    }
  ],
  "metadata": {
    "attack_vectors": ["sql_injection", "xss", "command_injection", ...],
    "owasp_coverage": true
  }
}
```

---

## 📏 2. Boundary Endpoint

**POST** `/api/testdata/generate/boundary`

**What it generates:**
- Min/Max values
- Min-1, Max+1 (off-by-one errors)
- Zero, Empty, Null
- Overflow/Underflow
- Length boundaries

**Example Request:**
```json
{
  "template": {
    "amount": "{{faker.number}}",
    "age": "{{faker.number}}"
  },
  "count": 10
}
```

**Example Response:**
```json
{
  "success": true,
  "data": [
    {
      "amount": "0.01",
      "age": "0",
      "_boundary_type": "min"
    },
    {
      "amount": "999999999.99",
      "age": "150",
      "_boundary_type": "max"
    }
  ],
  "metadata": {
    "boundary_types": ["min", "max", "min-1", "max+1", "zero", ...]
  }
}
```

---

## ⚖️ 3. Equivalence Endpoint

**POST** `/api/testdata/generate/equivalence`

**What it generates:**
- Valid partitions (standard, formatted, international)
- Invalid partitions (wrong format, out of range)
- Boundary partitions (edge of valid classes)
- Format variations

**Example Request:**
```json
{
  "template": {
    "phone": "{{faker.phone}}",
    "email": "{{faker.email}}"
  },
  "count": 12
}
```

**Example Response:**
```json
{
  "success": true,
  "data": [
    {
      "phone": "1234567890",
      "_partition_class": "valid_standard",
      "_partition_type": "valid"
    },
    {
      "phone": "abc-def-ghij",
      "_partition_class": "invalid_format",
      "_partition_type": "invalid"
    }
  ]
}
```

---

## ✅ 4. Positive Endpoint

**POST** `/api/testdata/generate/positive`

**What it generates:**
- Valid data within expected ranges
- Proper formats (email, phone, date)
- Multiple valid scenarios (standard, corporate, international)
- Realistic user data

**Example Request:**
```json
{
  "template": {
    "firstName": "{{faker.name}}",
    "email": "{{faker.email}}"
  },
  "count": 10
}
```

**Example Response:**
```json
{
  "success": true,
  "data": [
    {
      "firstName": "John",
      "email": "john.doe@gmail.com",
      "_scenario_type": "standard"
    },
    {
      "firstName": "Sarah",
      "email": "sarah@company.com",
      "_scenario_type": "corporate"
    }
  ]
}
```

---

## ❌ 5. Negative Endpoint

**POST** `/api/testdata/generate/negative`

**What it generates:**
- Invalid formats
- Empty/null values
- Special characters
- Too long/too short
- Wrong data types

**Example Request:**
```json
{
  "template": {
    "email": "{{faker.email}}",
    "phone": "{{faker.phone}}"
  },
  "count": 10
}
```

**Example Response:**
```json
{
  "success": true,
  "data": [
    {
      "email": "",
      "phone": "",
      "_invalid_type": "empty"
    },
    {
      "email": "not-an-email",
      "phone": "123",
      "_invalid_type": "invalid_format"
    }
  ]
}
```

---

## 🎯 Key Features

All 5 endpoints include:

✅ **Automatic Type Setting** - No need to specify `testDataType`
- Security endpoint automatically sets `testDataType='security'`
- Boundary endpoint automatically sets `testDataType='boundary'`
- Equivalence endpoint automatically sets `testDataType='equivalence'`
- Positive endpoint automatically sets `testDataType='positive'`
- Negative endpoint automatically sets `testDataType='negative'`

✅ **Rich Metadata**
- Attack vectors, boundary types, partition classes
- Test type, descriptions, scenario types
- Validation status, coverage level

✅ **AI-Powered Generation**
- Uses **GPT-4o** when available
- Leverages **enhanced script analyzer** (228 patterns, 25 field types)
- **Constraint-aware** generation (uses field constraints)

✅ **Fallback Support**
- Template-based generation if AI unavailable
- Always returns valid test data

---

## 📂 File Locations

**Main API File:**
```
c:\chandra-1212-main\ai-analysis-service\main.py
```

**Endpoints are defined at:**
- Security: Line 4940-4990
- Boundary: Line 4993-5040
- Equivalence: Line 5043-5090
- Positive: Line 5093-5135
- Negative: Line 5138-5184

**Documentation:**
- Full docs: `DEDICATED_TEST_DATA_ENDPOINTS.md`
- Quick reference: `API_ENDPOINTS_QUICK_REFERENCE.md`
- Test script: `ai-analysis-service/test_endpoints.py`

---

## 🧪 How to Test All Endpoints

### **Option 1: Use the Test Script**
```bash
cd c:\chandra-1212-main\ai-analysis-service
python test_endpoints.py
```

### **Option 2: Use API Documentation**
1. Start the service: `python main.py`
2. Open browser: `http://localhost:8000/docs`
3. Try each endpoint interactively

### **Option 3: Manual Testing**

**Python:**
```python
import requests

BASE_URL = "http://localhost:8000"
endpoints = [
    "/api/testdata/generate/security",
    "/api/testdata/generate/boundary",
    "/api/testdata/generate/equivalence",
    "/api/testdata/generate/positive",
    "/api/testdata/generate/negative"
]

template = {
    "email": "{{faker.email}}",
    "password": "{{faker.password}}"
}

for endpoint in endpoints:
    response = requests.post(
        f"{BASE_URL}{endpoint}",
        json={"template": template, "count": 5}
    )
    print(f"✅ {endpoint}: {len(response.json()['data'])} items")
```

---

## 🔗 Integration with Frontend

The endpoints are already integrated with your frontend. Check:

**Frontend Component:**
```
c:\chandra-1212-main\playwright-crx-enhanced\frontend\src\components\ScriptEnhancementModal.tsx
```

**Usage in Frontend:**
```typescript
// Frontend already uses these endpoints!
const response = await axios.post(
  `${AI_SERVICE_URL}/api/testdata/generate/${testDataType}`,
  {
    template: recommendation.recommended_template,
    count: testDataCount
  }
);
```

---

## 📊 Enhanced with Script Analyzer V2

All endpoints now benefit from the **enhanced script analyzer**:

✅ **228 patterns** detected (was 76)
✅ **25 field types** identified (was 10)
✅ **Rich constraints** extracted (pattern, min/max, formats)

**This means:**
- **Security tests** are field-specific (email XSS, password SQLi, amount overflow)
- **Boundary tests** use exact min/max from constraints
- **Equivalence tests** include format variations
- **Positive tests** respect validation rules
- **Negative tests** violate specific constraints

---

## ✅ Summary

You **already have** all 5 API endpoints! They are:

1. ✅ **Defined** in `main.py` (lines 4940-5184)
2. ✅ **Documented** in multiple files
3. ✅ **Tested** with test scripts
4. ✅ **Integrated** with frontend
5. ✅ **Enhanced** with script analyzer V2
6. ✅ **Production-ready** and working

**To use them:**
1. Start the AI service: `cd ai-analysis-service; python main.py`
2. Call any of the 5 endpoints
3. Get AI-powered, constraint-aware test data!

---

**Version:** 2.0  
**Status:** ✅ Production Ready  
**Created:** Previous session  
**Enhanced:** November 26, 2025 (Script Analyzer V2)
