# 🚀 Dedicated Test Data Endpoints - API Documentation

## Overview

The Python AI Analysis Service now provides **5 dedicated endpoints** for each test data type, making it easier to generate specific types of test data without specifying the `testDataType` parameter.

---

## 🎯 New Dedicated Endpoints

### Base URL
```
http://localhost:8000
```

### Endpoints Summary

| Endpoint | Type | Description | Metadata Fields |
|----------|------|-------------|-----------------|
| `/api/testdata/generate/security` | Security | SQL injection, XSS, command injection | `_attack_vector`, `_description` |
| `/api/testdata/generate/boundary` | Boundary | Min, max, overflow, underflow | `_boundary_type`, `_description` |
| `/api/testdata/generate/equivalence` | Equivalence | Valid/invalid partitions | `_partition_class`, `_partition_type` |
| `/api/testdata/generate/positive` | Positive | Valid data, proper formats | `_scenario_type`, `_description` |
| `/api/testdata/generate/negative` | Negative | Invalid data, edge cases | `_invalid_type`, `_description` |

---

## 📍 1. Security Test Data

### Endpoint
```http
POST /api/testdata/generate/security
```

### Description
Generate AI-powered security test data with attack vectors including SQL injection, XSS, command injection, path traversal, and more.

### Request Body
```json
{
  "template": {
    "email": "{{faker.email}}",
    "password": "{{faker.password}}"
  },
  "count": 15
}
```

**Note:** `testDataType` is automatically set to `'security'` - no need to specify it!

### Response Example
```json
{
  "success": true,
  "data": [
    {
      "email": "admin'--",
      "password": "anything",
      "_testDataType": "security",
      "_index": 0,
      "_description": "SQL Injection - Authentication Bypass",
      "_attack_vector": "sql_injection",
      "_test_type": "security"
    },
    {
      "email": "<script>alert(document.cookie)</script>@test.com",
      "password": "test123",
      "_testDataType": "security",
      "_index": 1,
      "_description": "XSS - Cookie Stealing via Email",
      "_attack_vector": "xss",
      "_test_type": "security"
    },
    {
      "email": "'; DROP TABLE users;--",
      "password": "ignored",
      "_testDataType": "security",
      "_index": 2,
      "_description": "SQL Injection - Table Drop Attack",
      "_attack_vector": "sql_injection",
      "_test_type": "security"
    }
  ],
  "metadata": {
    "count": 15,
    "testDataType": "security",
    "attack_vectors": [
      "sql_injection",
      "xss",
      "command_injection",
      "path_traversal",
      "ldap_injection",
      "xml_injection",
      "auth_bypass",
      "csrf",
      "nosql_injection",
      "ssti"
    ],
    "owasp_coverage": true,
    "endpoint": "/api/testdata/generate/security",
    "source": "gpt4o_dynamic"
  }
}
```

### CURL Example
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

### Attack Vectors Covered
- ✅ SQL Injection (Classic, Union, Blind)
- ✅ XSS (Reflected, Stored, DOM-based)
- ✅ Command Injection
- ✅ Path Traversal
- ✅ LDAP Injection
- ✅ XML/XXE Injection
- ✅ Authentication Bypass
- ✅ CSRF Attacks
- ✅ NoSQL Injection
- ✅ SSTI (Server-Side Template Injection)

---

## 📐 2. Boundary Value Analysis

### Endpoint
```http
POST /api/testdata/generate/boundary
```

### Description
Generate AI-powered boundary value analysis test data including min, max, off-by-one, overflow/underflow conditions.

### Request Body
```json
{
  "template": {
    "age": "{{faker.number(0-120)}}",
    "email": "{{faker.email}}"
  },
  "count": 20
}
```

### Response Example
```json
{
  "success": true,
  "data": [
    {
      "age": "0",
      "email": "a@b.c",
      "_testDataType": "boundary",
      "_index": 0,
      "_description": "Age - Minimum Valid (0), Email - Minimum Length",
      "_boundary_type": "min",
      "_test_type": "boundary"
    },
    {
      "age": "-1",
      "email": "test@test.com",
      "_testDataType": "boundary",
      "_index": 1,
      "_description": "Age - Below Minimum (Min-1)",
      "_boundary_type": "min-1",
      "_test_type": "boundary"
    },
    {
      "age": "120",
      "email": "test@test.com",
      "_testDataType": "boundary",
      "_index": 2,
      "_description": "Age - Maximum Realistic Value",
      "_boundary_type": "max",
      "_test_type": "boundary"
    },
    {
      "age": "121",
      "email": "test@test.com",
      "_testDataType": "boundary",
      "_index": 3,
      "_description": "Age - Above Maximum (Max+1)",
      "_boundary_type": "max+1",
      "_test_type": "boundary"
    }
  ],
  "metadata": {
    "count": 20,
    "testDataType": "boundary",
    "boundary_types": [
      "min",
      "max",
      "min-1",
      "max+1",
      "zero",
      "null",
      "empty",
      "overflow",
      "underflow"
    ],
    "coverage_level": "comprehensive",
    "endpoint": "/api/testdata/generate/boundary"
  }
}
```

### CURL Example
```bash
curl -X POST http://localhost:8000/api/testdata/generate/boundary \
  -H "Content-Type: application/json" \
  -d '{
    "template": {
      "age": "{{faker.number(0-120)}}",
      "amount": "{{faker.number(0-999999)}}"
    },
    "count": 20
  }'
```

### Boundary Types Covered
- ✅ Min (minimum valid value)
- ✅ Max (maximum valid value)
- ✅ Min-1 (below minimum - off-by-one)
- ✅ Max+1 (above maximum - off-by-one)
- ✅ Zero
- ✅ Null
- ✅ Empty
- ✅ Overflow
- ✅ Underflow

---

## ⚖️ 3. Equivalence Partitioning

### Endpoint
```http
POST /api/testdata/generate/equivalence
```

### Description
Generate AI-powered equivalence partitioning test data with valid partitions, invalid partitions, and boundary partitions.

### Request Body
```json
{
  "template": {
    "email": "{{faker.email}}",
    "transferAmount": "{{faker.number(1-1000000)}}"
  },
  "count": 12
}
```

### Response Example
```json
{
  "success": true,
  "data": [
    {
      "email": "user@gmail.com",
      "transferAmount": "500",
      "_testDataType": "equivalence",
      "_index": 0,
      "_description": "Valid Email - Standard Format, Small Amount",
      "_partition_class": "valid_standard",
      "_partition_type": "valid",
      "_test_type": "equivalence"
    },
    {
      "email": "admin@company.co.uk",
      "transferAmount": "50000",
      "_testDataType": "equivalence",
      "_index": 1,
      "_description": "Valid Email - Subdomain, Medium Amount",
      "_partition_class": "valid_standard",
      "_partition_type": "valid",
      "_test_type": "equivalence"
    },
    {
      "email": "notanemail",
      "transferAmount": "1000",
      "_testDataType": "equivalence",
      "_index": 2,
      "_description": "Invalid Email - Missing @ Symbol",
      "_partition_class": "invalid_format",
      "_partition_type": "invalid",
      "_test_type": "equivalence"
    },
    {
      "email": "test@test.com",
      "transferAmount": "0.01",
      "_testDataType": "equivalence",
      "_index": 3,
      "_description": "Boundary - Minimum Transfer Amount",
      "_partition_class": "boundary_min",
      "_partition_type": "boundary",
      "_test_type": "equivalence"
    }
  ],
  "metadata": {
    "count": 12,
    "testDataType": "equivalence",
    "partition_types": [
      "valid_partition",
      "invalid_partition",
      "boundary_partition"
    ],
    "partition_classes": [
      "valid_standard",
      "valid_edge",
      "invalid_format",
      "invalid_range",
      "boundary_min",
      "boundary_max"
    ],
    "endpoint": "/api/testdata/generate/equivalence"
  }
}
```

### CURL Example
```bash
curl -X POST http://localhost:8000/api/testdata/generate/equivalence \
  -H "Content-Type: application/json" \
  -d '{
    "template": {
      "email": "{{faker.email}}",
      "transferAmount": "{{faker.number(1-1000000)}}"
    },
    "count": 12
  }'
```

### Partition Types Covered
- ✅ Valid Partition (representative valid values)
- ✅ Invalid Partition (representative invalid values)
- ✅ Boundary Partition (values at class boundaries)
- ✅ Domain-specific partitions
- ✅ Format variations

---

## ✅ 4. Positive Test Data

### Endpoint
```http
POST /api/testdata/generate/positive
```

### Description
Generate AI-powered positive (valid) test data with realistic user scenarios and proper formats.

### Request Body
```json
{
  "template": {
    "email": "{{faker.email}}",
    "name": "{{faker.name}}",
    "phone": "{{faker.phone}}"
  },
  "count": 15
}
```

### Response Example
```json
{
  "success": true,
  "data": [
    {
      "email": "john.doe@gmail.com",
      "name": "John Doe",
      "phone": "555-123-4567",
      "_testDataType": "positive",
      "_index": 0,
      "_description": "Standard Email and Full Name",
      "_scenario_type": "standard",
      "_test_type": "positive"
    },
    {
      "email": "sarah.jones@company.com",
      "name": "Dr. Sarah Jones",
      "phone": "+1-555-987-6543",
      "_testDataType": "positive",
      "_index": 1,
      "_description": "Corporate Email and Professional Name",
      "_scenario_type": "corporate",
      "_test_type": "positive"
    },
    {
      "email": "jose.garcia@hotmail.com",
      "name": "José García",
      "phone": "(555) 234-5678",
      "_testDataType": "positive",
      "_index": 2,
      "_description": "International Name with Accents",
      "_scenario_type": "international",
      "_test_type": "positive"
    }
  ],
  "metadata": {
    "count": 15,
    "testDataType": "positive",
    "scenario_types": [
      "standard",
      "corporate",
      "international",
      "formatted",
      "edge_valid"
    ],
    "validation_status": "all_valid",
    "endpoint": "/api/testdata/generate/positive"
  }
}
```

### CURL Example
```bash
curl -X POST http://localhost:8000/api/testdata/generate/positive \
  -H "Content-Type: application/json" \
  -d '{
    "template": {
      "email": "{{faker.email}}",
      "name": "{{faker.name}}",
      "phone": "{{faker.phone}}"
    },
    "count": 15
  }'
```

### Scenario Types Covered
- ✅ Standard (typical valid values)
- ✅ Corporate (business formats)
- ✅ International (global formats with Unicode)
- ✅ Formatted (various valid formats)
- ✅ Edge Valid (valid but unusual)

---

## ❌ 5. Negative Test Data

### Endpoint
```http
POST /api/testdata/generate/negative
```

### Description
Generate AI-powered negative (invalid) test data to test error handling and validation.

### Request Body
```json
{
  "template": {
    "email": "{{faker.email}}",
    "name": "{{faker.name}}",
    "phone": "{{faker.phone}}"
  },
  "count": 15
}
```

### Response Example
```json
{
  "success": true,
  "data": [
    {
      "email": "",
      "name": "John Doe",
      "phone": "555-1234",
      "_testDataType": "negative",
      "_index": 0,
      "_description": "Email - Empty String",
      "_invalid_type": "empty",
      "_test_type": "negative"
    },
    {
      "email": "notanemail",
      "name": "John Doe",
      "phone": "555-1234",
      "_testDataType": "negative",
      "_index": 1,
      "_description": "Email - Missing @ Symbol",
      "_invalid_type": "invalid_format",
      "_test_type": "negative"
    },
    {
      "email": "test@test.com",
      "name": "!!!@@@###",
      "phone": "555-1234",
      "_testDataType": "negative",
      "_index": 2,
      "_description": "Name - Special Characters Only",
      "_invalid_type": "special_chars",
      "_test_type": "negative"
    },
    {
      "email": "a".repeat(300) + "@test.com",
      "name": "John Doe",
      "phone": "555-1234",
      "_testDataType": "negative",
      "_index": 3,
      "_description": "Email - Extremely Long Local Part",
      "_invalid_type": "too_long",
      "_test_type": "negative"
    }
  ],
  "metadata": {
    "count": 15,
    "testDataType": "negative",
    "invalid_types": [
      "empty",
      "null",
      "invalid_format",
      "too_long",
      "too_short",
      "special_chars",
      "wrong_type",
      "missing_required"
    ],
    "validation_status": "all_invalid",
    "endpoint": "/api/testdata/generate/negative"
  }
}
```

### CURL Example
```bash
curl -X POST http://localhost:8000/api/testdata/generate/negative \
  -H "Content-Type: application/json" \
  -d '{
    "template": {
      "email": "{{faker.email}}",
      "password": "{{faker.password}}"
    },
    "count": 15
  }'
```

### Invalid Types Covered
- ✅ Empty (empty strings)
- ✅ Null (null values)
- ✅ Invalid Format (wrong patterns)
- ✅ Too Long (exceeds max length)
- ✅ Too Short (below min length)
- ✅ Special Characters (unusual symbols)
- ✅ Wrong Type (type mismatch)
- ✅ Missing Required (incomplete data)

---

## 🔧 Usage Examples

### JavaScript/TypeScript (Axios)
```typescript
import axios from 'axios';

// Security Test Data
const securityData = await axios.post(
  'http://localhost:8000/api/testdata/generate/security',
  {
    template: {
      email: '{{faker.email}}',
      password: '{{faker.password}}'
    },
    count: 15
  }
);

// Boundary Test Data
const boundaryData = await axios.post(
  'http://localhost:8000/api/testdata/generate/boundary',
  {
    template: {
      age: '{{faker.number(0-120)}}',
      amount: '{{faker.number(0-999999)}}'
    },
    count: 20
  }
);

// Equivalence Test Data
const equivalenceData = await axios.post(
  'http://localhost:8000/api/testdata/generate/equivalence',
  {
    template: {
      email: '{{faker.email}}',
      transferAmount: '{{faker.number(1-1000000)}}'
    },
    count: 12
  }
);

// Positive Test Data
const positiveData = await axios.post(
  'http://localhost:8000/api/testdata/generate/positive',
  {
    template: {
      email: '{{faker.email}}',
      name: '{{faker.name}}'
    },
    count: 10
  }
);

// Negative Test Data
const negativeData = await axios.post(
  'http://localhost:8000/api/testdata/generate/negative',
  {
    template: {
      email: '{{faker.email}}',
      password: '{{faker.password}}'
    },
    count: 10
  }
);
```

### Python (Requests)
```python
import requests

# Security Test Data
security_response = requests.post(
    'http://localhost:8000/api/testdata/generate/security',
    json={
        'template': {
            'email': '{{faker.email}}',
            'password': '{{faker.password}}'
        },
        'count': 15
    }
)
security_data = security_response.json()

# Boundary Test Data
boundary_response = requests.post(
    'http://localhost:8000/api/testdata/generate/boundary',
    json={
        'template': {
            'age': '{{faker.number(0-120)}}',
            'amount': '{{faker.number(0-999999)}}'
        },
        'count': 20
    }
)
boundary_data = boundary_response.json()
```

---

## 📊 Comparison: Dedicated vs Unified Endpoint

### **Dedicated Endpoints** (NEW ✨)
```typescript
// Cleaner, more explicit
await axios.post('/api/testdata/generate/security', { template, count });
await axios.post('/api/testdata/generate/boundary', { template, count });
await axios.post('/api/testdata/generate/positive', { template, count });
```

**Pros:**
- ✅ No need to specify `testDataType`
- ✅ Clearer intent and purpose
- ✅ Type-specific metadata automatically included
- ✅ Better API documentation
- ✅ Easier to remember and use

### **Unified Endpoint** (Still Available)
```typescript
// More verbose, requires testDataType
await axios.post('/api/dynamic/generate-testdata', { 
  template, 
  count, 
  testDataType: 'security' 
});
```

**Pros:**
- ✅ Single endpoint for all types
- ✅ Flexible for dynamic type selection
- ✅ Backward compatible

**Both approaches work and produce identical results!**

---

## 🎯 API Response Structure

All endpoints return the same structure:

```typescript
interface TestDataResponse {
  success: boolean;
  data: Array<{
    [field: string]: any;
    _testDataType: string;     // Type of test data
    _index: number;            // Record index
    _description?: string;     // Test case description
    _test_type: string;        // Same as _testDataType
    // Type-specific metadata:
    _attack_vector?: string;   // Security only
    _boundary_type?: string;   // Boundary only
    _partition_class?: string; // Equivalence only
    _scenario_type?: string;   // Positive only
    _invalid_type?: string;    // Negative only
  }>;
  metadata: {
    count: number;
    testDataType: string;
    endpoint: string;
    source?: string;
    // Type-specific metadata arrays
    attack_vectors?: string[];     // Security
    boundary_types?: string[];     // Boundary
    partition_types?: string[];    // Equivalence
    scenario_types?: string[];     // Positive
    invalid_types?: string[];      // Negative
  };
}
```

---

## 🚀 Quick Start

### 1. Start the Python AI Service
```bash
cd ai-analysis-service
python main.py
```

Service runs on: `http://localhost:8000`

### 2. Test the Endpoints
```bash
# Security
curl -X POST http://localhost:8000/api/testdata/generate/security \
  -H "Content-Type: application/json" \
  -d '{"template": {"email": "{{faker.email}}"}, "count": 10}'

# Boundary
curl -X POST http://localhost:8000/api/testdata/generate/boundary \
  -H "Content-Type: application/json" \
  -d '{"template": {"age": "{{faker.number(0-120)}}"}, "count": 10}'

# Equivalence
curl -X POST http://localhost:8000/api/testdata/generate/equivalence \
  -H "Content-Type: application/json" \
  -d '{"template": {"email": "{{faker.email}}"}, "count": 10}'

# Positive
curl -X POST http://localhost:8000/api/testdata/generate/positive \
  -H "Content-Type: application/json" \
  -d '{"template": {"name": "{{faker.name}}"}, "count": 10}'

# Negative
curl -X POST http://localhost:8000/api/testdata/generate/negative \
  -H "Content-Type: application/json" \
  -d '{"template": {"email": "{{faker.email}}"}, "count": 10}'
```

### 3. View API Documentation
Open your browser:
- **Swagger UI**: http://localhost:8000/docs
- **ReDoc**: http://localhost:8000/redoc

All 5 endpoints will be listed under the **"Test Data"** tag.

---

## 📚 Related Documentation

- [`TEST_DATA_API_USAGE_GUIDE.md`](TEST_DATA_API_USAGE_GUIDE.md) - Complete usage guide
- [`COMPLETE_AI_TEST_DATA_GENERATION.md`](COMPLETE_AI_TEST_DATA_GENERATION.md) - AI-powered generation details
- [`AI_POWERED_ALL_TEST_TYPES.md`](AI_POWERED_ALL_TEST_TYPES.md) - Implementation details
- [`STATE_TRANSITION_AND_DECISION_MAKING_RESEARCH.md`](STATE_TRANSITION_AND_DECISION_MAKING_RESEARCH.md) - Advanced AI techniques

---

## 🎉 Summary

✅ **5 dedicated endpoints created**  
✅ **Automatic test type assignment**  
✅ **Type-specific metadata enrichment**  
✅ **Comprehensive API documentation**  
✅ **Full Swagger/OpenAPI integration**  
✅ **Backward compatible with unified endpoint**  

**All endpoints are AI-powered using GPT-4o and support script-aware test data generation!** 🚀

---

**Last Updated:** November 26, 2025  
**Version:** 1.0  
**Python AI Service:** v2.0.0
