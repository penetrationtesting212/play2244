# 🚀 5 Dedicated Test Data API Endpoints - Quick Reference

## 📍 Base URL
```
http://localhost:8000
```

---

## 🔐 1. Security Test Data Endpoint

### **Endpoint**
```http
POST /api/testdata/generate/security
```

### **Description**
Generates AI-powered security test data with attack vectors.

### **Includes**
- ✅ SQL Injection attacks
- ✅ XSS (Cross-Site Scripting)
- ✅ Command Injection
- ✅ Path Traversal
- ✅ LDAP Injection
- ✅ XML/XXE Injection
- ✅ Authentication bypass
- ✅ CSRF attacks
- ✅ NoSQL Injection
- ✅ SSTI (Server-Side Template Injection)

### **Request Body**
```json
{
  "template": {
    "email": "{{faker.email}}",
    "password": "{{faker.password}}"
  },
  "count": 15
}
```

### **Response Example**
```json
{
  "success": true,
  "data": [
    {
      "email": "admin'--",
      "password": "anything",
      "_description": "SQL Injection - Authentication Bypass",
      "_attack_vector": "sql_injection",
      "_test_type": "security"
    },
    {
      "email": "test@example.com<script>alert('XSS')</script>",
      "password": "test123",
      "_description": "XSS attack in email field",
      "_attack_vector": "xss",
      "_test_type": "security"
    }
  ],
  "metadata": {
    "testDataType": "security",
    "attack_vectors": [
      "sql_injection", "xss", "command_injection", "path_traversal",
      "ldap_injection", "xml_injection", "auth_bypass", "csrf",
      "nosql_injection", "ssti"
    ],
    "owasp_coverage": true,
    "endpoint": "/api/testdata/generate/security"
  }
}
```

### **CURL Example**
```bash
curl -X POST http://localhost:8000/api/testdata/generate/security \
  -H "Content-Type: application/json" \
  -d '{
    "template": {
      "email": "{{faker.email}}",
      "password": "{{faker.password}}"
    },
    "count": 15
  }'
```

---

## 📏 2. Boundary Value Test Data Endpoint

### **Endpoint**
```http
POST /api/testdata/generate/boundary
```

### **Description**
Generates AI-powered boundary value analysis test data.

### **Includes**
- ✅ Min, Max values
- ✅ Min-1, Max+1 (off-by-one)
- ✅ Zero, Null, Empty
- ✅ Overflow/Underflow conditions
- ✅ String length boundaries
- ✅ Precision limits

### **Request Body**
```json
{
  "template": {
    "amount": "{{faker.number}}",
    "age": "{{faker.number}}"
  },
  "count": 10
}
```

### **Response Example**
```json
{
  "success": true,
  "data": [
    {
      "amount": "0.01",
      "age": "0",
      "_description": "Minimum boundary values",
      "_boundary_type": "min",
      "_test_type": "boundary"
    },
    {
      "amount": "999999999.99",
      "age": "150",
      "_description": "Maximum boundary values",
      "_boundary_type": "max",
      "_test_type": "boundary"
    },
    {
      "amount": "0.00",
      "age": "-1",
      "_description": "Below minimum (min-1)",
      "_boundary_type": "min-1",
      "_test_type": "boundary"
    }
  ],
  "metadata": {
    "testDataType": "boundary",
    "boundary_types": [
      "min", "max", "min-1", "max+1", "zero", 
      "null", "empty", "overflow", "underflow"
    ],
    "coverage_level": "comprehensive",
    "endpoint": "/api/testdata/generate/boundary"
  }
}
```

### **CURL Example**
```bash
curl -X POST http://localhost:8000/api/testdata/generate/boundary \
  -H "Content-Type: application/json" \
  -d '{
    "template": {
      "amount": "{{faker.number}}",
      "age": "{{faker.number}}"
    },
    "count": 10
  }'
```

---

## ⚖️ 3. Equivalence Partitioning Test Data Endpoint

### **Endpoint**
```http
POST /api/testdata/generate/equivalence
```

### **Description**
Generates AI-powered equivalence partitioning test data.

### **Includes**
- ✅ Valid partitions (representative valid values)
- ✅ Invalid partitions (representative invalid values)
- ✅ Boundary partitions (values at class boundaries)
- ✅ Domain-specific partitions
- ✅ Format variations

### **Request Body**
```json
{
  "template": {
    "phone": "{{faker.phone}}",
    "email": "{{faker.email}}"
  },
  "count": 12
}
```

### **Response Example**
```json
{
  "success": true,
  "data": [
    {
      "phone": "1234567890",
      "email": "user@example.com",
      "_description": "Standard valid partition",
      "_partition_class": "valid_standard",
      "_partition_type": "valid",
      "_test_type": "equivalence"
    },
    {
      "phone": "123-456-7890",
      "email": "user+tag@example.com",
      "_description": "Formatted valid partition",
      "_partition_class": "valid_formatted",
      "_partition_type": "valid",
      "_test_type": "equivalence"
    },
    {
      "phone": "abc-def-ghij",
      "email": "invalid-email",
      "_description": "Invalid format partition",
      "_partition_class": "invalid_format",
      "_partition_type": "invalid",
      "_test_type": "equivalence"
    }
  ],
  "metadata": {
    "testDataType": "equivalence",
    "partition_types": [
      "valid_partition", "invalid_partition", "boundary_partition"
    ],
    "partition_classes": [
      "valid_standard", "valid_edge", "invalid_format",
      "invalid_range", "boundary_min", "boundary_max"
    ],
    "endpoint": "/api/testdata/generate/equivalence"
  }
}
```

### **CURL Example**
```bash
curl -X POST http://localhost:8000/api/testdata/generate/equivalence \
  -H "Content-Type: application/json" \
  -d '{
    "template": {
      "phone": "{{faker.phone}}",
      "email": "{{faker.email}}"
    },
    "count": 12
  }'
```

---

## ✅ 4. Positive (Valid) Test Data Endpoint

### **Endpoint**
```http
POST /api/testdata/generate/positive
```

### **Description**
Generates AI-powered positive (valid) test data.

### **Includes**
- ✅ Valid data within expected ranges
- ✅ Proper formats (email, phone, date)
- ✅ Representative valid values
- ✅ Realistic user scenarios
- ✅ Multiple valid formats (standard, corporate, international)

### **Request Body**
```json
{
  "template": {
    "firstName": "{{faker.name}}",
    "email": "{{faker.email}}",
    "phone": "{{faker.phone}}"
  },
  "count": 10
}
```

### **Response Example**
```json
{
  "success": true,
  "data": [
    {
      "firstName": "John",
      "email": "john.doe@gmail.com",
      "phone": "555-123-4567",
      "_description": "Standard valid scenario",
      "_scenario_type": "standard",
      "_test_type": "positive"
    },
    {
      "firstName": "Sarah",
      "email": "sarah.johnson@company.com",
      "phone": "(555) 987-6543",
      "_description": "Corporate scenario",
      "_scenario_type": "corporate",
      "_test_type": "positive"
    },
    {
      "firstName": "Wei",
      "email": "wei.chen@example.co.uk",
      "phone": "+44 20 1234 5678",
      "_description": "International scenario",
      "_scenario_type": "international",
      "_test_type": "positive"
    }
  ],
  "metadata": {
    "testDataType": "positive",
    "scenario_types": [
      "standard", "corporate", "international", 
      "formatted", "edge_valid"
    ],
    "validation_status": "all_valid",
    "endpoint": "/api/testdata/generate/positive"
  }
}
```

### **CURL Example**
```bash
curl -X POST http://localhost:8000/api/testdata/generate/positive \
  -H "Content-Type: application/json" \
  -d '{
    "template": {
      "firstName": "{{faker.name}}",
      "email": "{{faker.email}}",
      "phone": "{{faker.phone}}"
    },
    "count": 10
  }'
```

---

## ❌ 5. Negative (Invalid) Test Data Endpoint

### **Endpoint**
```http
POST /api/testdata/generate/negative
```

### **Description**
Generates AI-powered negative (invalid) test data.

### **Includes**
- ✅ Invalid data formats
- ✅ Empty values and null inputs
- ✅ Special characters and symbols
- ✅ Extra long strings
- ✅ Missing required parts
- ✅ Wrong data types

### **Request Body**
```json
{
  "template": {
    "email": "{{faker.email}}",
    "age": "{{faker.number}}",
    "phone": "{{faker.phone}}"
  },
  "count": 10
}
```

### **Response Example**
```json
{
  "success": true,
  "data": [
    {
      "email": "",
      "age": "",
      "phone": "",
      "_description": "Empty values",
      "_invalid_type": "empty",
      "_test_type": "negative"
    },
    {
      "email": "not-an-email",
      "age": "abc",
      "phone": "12345",
      "_description": "Invalid formats",
      "_invalid_type": "invalid_format",
      "_test_type": "negative"
    },
    {
      "email": "a".repeat(300) + "@example.com",
      "age": "999999",
      "phone": "1".repeat(50),
      "_description": "Too long values",
      "_invalid_type": "too_long",
      "_test_type": "negative"
    }
  ],
  "metadata": {
    "testDataType": "negative",
    "invalid_types": [
      "empty", "null", "invalid_format", "too_long",
      "too_short", "special_chars", "wrong_type", "missing_required"
    ],
    "validation_status": "all_invalid",
    "endpoint": "/api/testdata/generate/negative"
  }
}
```

### **CURL Example**
```bash
curl -X POST http://localhost:8000/api/testdata/generate/negative \
  -H "Content-Type: application/json" \
  -d '{
    "template": {
      "email": "{{faker.email}}",
      "age": "{{faker.number}}",
      "phone": "{{faker.phone}}"
    },
    "count": 10
  }'
```

---

## 🎯 All Endpoints Summary

| # | Endpoint | Test Type | Icon | Purpose |
|---|----------|-----------|------|---------|
| 1 | `/api/testdata/generate/security` | Security | 🔒 | Attack vectors, OWASP coverage |
| 2 | `/api/testdata/generate/boundary` | Boundary | 📏 | Min/Max values, edge cases |
| 3 | `/api/testdata/generate/equivalence` | Equivalence | ⚖️ | Valid/Invalid partitions |
| 4 | `/api/testdata/generate/positive` | Positive | ✅ | Valid data scenarios |
| 5 | `/api/testdata/generate/negative` | Negative | ❌ | Invalid data, error cases |

---

## 🔧 Common Request Parameters

All endpoints accept the same request body structure:

```typescript
{
  "template": {
    // Field definitions using Faker.js placeholders
    "fieldName": "{{faker.type}}"
  },
  "count": 10,  // Number of test data items to generate
  "testDataType": "auto"  // Automatically set by endpoint
}
```

### **Faker.js Placeholders:**
- `{{faker.email}}` - Email addresses
- `{{faker.name}}` - Person names
- `{{faker.phone}}` - Phone numbers
- `{{faker.password}}` - Passwords
- `{{faker.number}}` - Numbers
- `{{faker.date}}` - Dates
- `{{faker.address}}` - Addresses

---

## 🚀 Quick Test (All Endpoints)

### **Python**
```python
import requests

BASE_URL = "http://localhost:8000"

# Test all 5 endpoints
endpoints = {
    "security": "/api/testdata/generate/security",
    "boundary": "/api/testdata/generate/boundary",
    "equivalence": "/api/testdata/generate/equivalence",
    "positive": "/api/testdata/generate/positive",
    "negative": "/api/testdata/generate/negative"
}

template = {
    "email": "{{faker.email}}",
    "password": "{{faker.password}}"
}

for name, endpoint in endpoints.items():
    response = requests.post(
        f"{BASE_URL}{endpoint}",
        json={"template": template, "count": 5}
    )
    print(f"✅ {name}: {response.status_code}")
    print(f"   Data count: {len(response.json().get('data', []))}")
```

### **JavaScript/Node.js**
```javascript
const axios = require('axios');

const BASE_URL = 'http://localhost:8000';

const endpoints = {
  security: '/api/testdata/generate/security',
  boundary: '/api/testdata/generate/boundary',
  equivalence: '/api/testdata/generate/equivalence',
  positive: '/api/testdata/generate/positive',
  negative: '/api/testdata/generate/negative'
};

const template = {
  email: '{{faker.email}}',
  password: '{{faker.password}}'
};

// Test all endpoints
Object.entries(endpoints).forEach(async ([name, endpoint]) => {
  const response = await axios.post(`${BASE_URL}${endpoint}`, {
    template,
    count: 5
  });
  console.log(`✅ ${name}: ${response.status}`)
  console.log(`   Data count: ${response.data.data.length}`);
});
```

---

## 📚 Additional Resources

- **Full Documentation:** [DEDICATED_TEST_DATA_ENDPOINTS.md](./DEDICATED_TEST_DATA_ENDPOINTS.md)
- **API Docs:** http://localhost:8000/docs
- **ReDoc:** http://localhost:8000/redoc
- **Test Script:** [test_endpoints.py](./ai-analysis-service/test_endpoints.py)

---

## ✅ Features

All 5 endpoints include:
- ✅ **Automatic type setting** - No need to specify `testDataType`
- ✅ **Rich metadata** - Attack vectors, boundary types, partition classes
- ✅ **AI-powered generation** - Uses GPT-4o when available
- ✅ **Fallback support** - Template-based generation if AI unavailable
- ✅ **Enhanced script analysis** - Leverages 228 patterns, 25 field types
- ✅ **Constraint-aware** - Uses field constraints from script analysis

---

**Version:** 2.0  
**Status:** ✅ Production Ready  
**Created:** November 26, 2025
