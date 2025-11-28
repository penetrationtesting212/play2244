# 📊 Test Data API Usage Guide - Current Implementation

## Overview

This document describes how the **5 test data types** (Security, Boundary, Equivalence, Positive, Negative) are currently used in the system.

---

## 🎯 Current API Architecture

### **Python AI Service (Port 8000)**
- Base URL: `http://localhost:8000`
- Handles AI-powered test data generation using GPT-4o

### **Node.js Backend (Port 3001)**
- Base URL: `http://localhost:3001`
- Acts as proxy to Python service
- Provides TypeScript API endpoints

---

## 📍 Available Endpoints

### **1. Python AI Service Endpoints**

#### **Main Unified Endpoint**
```http
POST http://localhost:8000/api/dynamic/generate-testdata
```

**Request Body:**
```json
{
  "template": {
    "email": "{{faker.email}}",
    "password": "{{faker.password}}"
  },
  "count": 10,
  "testDataType": "security"  // Options: security, boundary, equivalence, positive, negative, all
}
```

**Response:**
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
      "_attack_vector": "sql_injection"
    }
  ],
  "metadata": {
    "count": 10,
    "testDataType": "security",
    "source": "gpt4o_dynamic"
  }
}
```

#### **Test Data Recommendation Endpoint**
```http
POST http://localhost:8000/api/ai-analysis/recommend-testdata
```

**Request Body:**
```json
{
  "script_content": "await page.fill('#email', 'test@test.com');",
  "test_data_type": "security"
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "detected_fields": [
      {"field": "email", "type": "email", "example": "{{faker.email}}"}
    ],
    "recommended_template": {
      "email": "{{faker.email}}"
    },
    "gpt4_generated_data": [
      {
        "email": "admin'--",
        "_description": "SQL Injection - Authentication Bypass",
        "_attack_vector": "sql_injection"
      }
    ],
    "test_scenarios": [
      {"type": "sql_injection", "description": "SQL injection attack patterns", "count": 10}
    ]
  }
}
```

---

### **2. Node.js Backend Endpoints**

#### **Security Testing**
```http
POST http://localhost:3001/api/testing-strategies/security
```

**Request Body:**
```json
{
  "count": 10,
  "options": {},
  "useAI": true
}
```

#### **Boundary Value Analysis**
```http
POST http://localhost:3001/api/testing-strategies/boundary
```

**Request Body:**
```json
{
  "count": 9,
  "fieldName": "transferAmount",
  "fieldType": "number",
  "minValue": 0.01,
  "maxValue": 999999.99,
  "options": {},
  "useAI": true
}
```

#### **Equivalence Partitioning**
```http
POST http://localhost:3001/api/testing-strategies/equivalence
```

**Request Body:**
```json
{
  "count": 10,
  "fieldName": "transferAmount",
  "partitionType": "all",
  "options": {},
  "useAI": true
}
```

**Note:** These Node.js endpoints currently call the **external Python service** at:
- URL: `http://34.46.36.105:3000/genieapi/assistant/generate-security-tests`
- This is NOT the local AI analysis service

---

## 🔄 Current Workflow

### **Frontend → Python AI Service (Direct)**

The frontend (`ScriptEnhancementModal.tsx`) **directly calls** the Python AI service:

```typescript
// Step 1: Get AI recommendations
const recResponse = await axios.post(
  `http://localhost:8000/api/ai-analysis/recommend-testdata`,
  {
    script_content: scriptCode,
    test_data_type: testDataType  // 'security', 'boundary', etc.
  }
);

const recommendation = recResponse.data.data;

// Step 2: Use GPT-4o generated data (Priority 1)
if (recommendation.gpt4_generated_data && recommendation.gpt4_generated_data.length > 0) {
  console.log('✅ Using GPT-4o AI-generated data');
  finalTestData = {
    success: true,
    data: recommendation.gpt4_generated_data.slice(0, testDataCount),
    metadata: {
      testDataType: testDataType,
      source: 'gpt4o_dynamic'
    }
  };
}
// Step 3: Use template-based generation (Priority 2)
else if (recommendation.recommended_template) {
  const genResponse = await axios.post(
    'http://localhost:8000/api/dynamic/generate-testdata',
    {
      template: recommendation.recommended_template,
      count: testDataCount,
      testDataType: testDataType
    }
  );
  finalTestData = genResponse.data;
}
```

---

## 🎨 Test Data Type Examples

### **1. Security Testing (`testDataType: 'security'`)**

**Generated Data:**
```json
[
  {
    "email": "admin'--",
    "password": "anything",
    "_description": "SQL Injection - Authentication Bypass",
    "_attack_vector": "sql_injection",
    "_test_type": "security"
  },
  {
    "email": "<script>alert(document.cookie)</script>@test.com",
    "password": "test123",
    "_description": "XSS - Cookie Stealing via Email",
    "_attack_vector": "xss",
    "_test_type": "security"
  }
]
```

**Metadata Fields:**
- `_description`: Attack explanation
- `_attack_vector`: Type (sql_injection, xss, command_injection, path_traversal)
- `_test_type`: "security"

---

### **2. Boundary Value Analysis (`testDataType: 'boundary'`)**

**Generated Data:**
```json
[
  {
    "email": "a@b.c",
    "age": "25",
    "_description": "Email - Minimum Valid Length",
    "_boundary_type": "min",
    "_test_type": "boundary"
  },
  {
    "email": "test@test.com",
    "age": "-1",
    "_description": "Age - Below Minimum (-1)",
    "_boundary_type": "min-1",
    "_test_type": "boundary"
  },
  {
    "email": "test@test.com",
    "age": "120",
    "_description": "Age - Maximum Realistic (120)",
    "_boundary_type": "max",
    "_test_type": "boundary"
  }
]
```

**Metadata Fields:**
- `_description`: Boundary type explanation
- `_boundary_type`: Type (min, max, min-1, max+1, zero, null, empty, overflow)
- `_test_type`: "boundary"

---

### **3. Equivalence Partitioning (`testDataType: 'equivalence'`)**

**Generated Data:**
```json
[
  {
    "email": "user@gmail.com",
    "_description": "Valid Email - Standard Format",
    "_partition_class": "valid_standard",
    "_partition_type": "valid",
    "_test_type": "equivalence"
  },
  {
    "email": "notanemail",
    "_description": "Invalid Email - Missing @ Symbol",
    "_partition_class": "invalid_format",
    "_partition_type": "invalid",
    "_test_type": "equivalence"
  }
]
```

**Metadata Fields:**
- `_description`: Partition explanation
- `_partition_class`: Class name (valid_standard, invalid_format, boundary_min)
- `_partition_type`: Type (valid, invalid, boundary)
- `_test_type`: "equivalence"

---

### **4. Positive Testing (`testDataType: 'positive'`)**

**Generated Data:**
```json
[
  {
    "email": "john.doe@gmail.com",
    "name": "John Doe",
    "phone": "555-123-4567",
    "_description": "Standard Email and Full Name",
    "_scenario_type": "standard",
    "_test_type": "positive"
  },
  {
    "email": "sarah.jones@company.com",
    "name": "Dr. Sarah Jones",
    "phone": "+1-555-987-6543",
    "_description": "Corporate Email and Professional Name",
    "_scenario_type": "corporate",
    "_test_type": "positive"
  }
]
```

**Metadata Fields:**
- `_description`: Scenario explanation
- `_scenario_type`: Type (standard, corporate, international, formatted)
- `_test_type`: "positive"

---

### **5. Negative Testing (`testDataType: 'negative'`)**

**Generated Data:**
```json
[
  {
    "email": "",
    "name": "John Doe",
    "_description": "Email - Empty String",
    "_invalid_type": "empty",
    "_test_type": "negative"
  },
  {
    "email": "notanemail",
    "name": "John Doe",
    "_description": "Email - Missing @ Symbol",
    "_invalid_type": "invalid_format",
    "_test_type": "negative"
  },
  {
    "email": "a@b.c",
    "name": "!!!@@@###",
    "_description": "Name - Special Characters Only",
    "_invalid_type": "special_chars",
    "_test_type": "negative"
  }
]
```

**Metadata Fields:**
- `_description`: Invalid scenario explanation
- `_invalid_type`: Type (empty, null, invalid_format, too_long, special_chars)
- `_test_type`: "negative"

---

### **6. All Types Mixed (`testDataType: 'all'`)**

**Distribution:**
- 40% Positive
- 20% Negative
- 20% Boundary
- 10% Security
- 10% Equivalence

**Generated Data:**
```json
[
  {
    "email": "user@test.com",
    "_testDataType": "positive",
    "_index": 0
  },
  {
    "email": "admin'--",
    "_testDataType": "security",
    "_attack_vector": "sql_injection",
    "_index": 1
  },
  {
    "email": "",
    "_testDataType": "negative",
    "_invalid_type": "empty",
    "_index": 2
  }
]
```

---

## 🔧 Implementation Details

### **Python Service (`main.py`)**

#### **Line 2291-2973: `/api/dynamic/generate-testdata`**
- Main endpoint for test data generation
- Supports all 5 test data types
- Uses `process_faker_template()` function
- Handles `{{faker.xxx}}` pattern replacement
- Returns metadata-enriched test data

#### **Line 3078-3785: `/api/ai-analysis/recommend-testdata`**
- Analyzes Playwright scripts
- Detects form fields automatically
- Calls GPT-4o with customized prompts per type
- Returns AI-generated test data + templates

### **Node.js Backend**

#### **`testing-strategies.controller.ts`**
- Lines 26-86: `generateSecurityTests()`
- Lines 92-170: `generateBoundaryTests()`
- Lines 176-192+: `generateEquivalenceTests()`

**Current Issue:** These call **external Python API** (`http://34.46.36.105:3000/genieapi`), NOT the local AI analysis service.

---

## 🚨 Current Gaps

### **Missing Dedicated Endpoints in Python Service**

Currently, there are **NO dedicated endpoints** for:
1. ❌ `/api/testdata/generate/security`
2. ❌ `/api/testdata/generate/boundary`
3. ❌ `/api/testdata/generate/equivalence`
4. ❌ `/api/testdata/generate/positive`
5. ❌ `/api/testdata/generate/negative`

**Workaround:** All types use the **single unified endpoint** `/api/dynamic/generate-testdata` with the `testDataType` parameter.

### **Node.js Backend Routing Issue**

The Node.js backend `testing-strategies` routes call an **external Python service** instead of the local AI analysis service:

```typescript
this.pythonApiUrl = process.env.PYTHON_API_URL || 'http://34.46.36.105:3000/genieapi';
// Should be: 'http://localhost:8000'
```

---

## ✅ Recommended Improvements

### **1. Create Dedicated Python Endpoints**

Add specific endpoints for each test data type:

```python
@app.post("/api/testdata/generate/security")
async def generate_security_testdata(request: DynamicTestDataRequest):
    request.testDataType = 'security'
    return await generate_dynamic_testdata(request)

@app.post("/api/testdata/generate/boundary")
async def generate_boundary_testdata(request: DynamicTestDataRequest):
    request.testDataType = 'boundary'
    return await generate_dynamic_testdata(request)

@app.post("/api/testdata/generate/equivalence")
async def generate_equivalence_testdata(request: DynamicTestDataRequest):
    request.testDataType = 'equivalence'
    return await generate_dynamic_testdata(request)

@app.post("/api/testdata/generate/positive")
async def generate_positive_testdata(request: DynamicTestDataRequest):
    request.testDataType = 'positive'
    return await generate_dynamic_testdata(request)

@app.post("/api/testdata/generate/negative")
async def generate_negative_testdata(request: DynamicTestDataRequest):
    request.testDataType = 'negative'
    return await generate_dynamic_testdata(request)
```

### **2. Update Node.js Backend Routes**

Point to local AI analysis service:

```typescript
constructor() {
  this.pythonApiUrl = process.env.PYTHON_API_URL || 'http://localhost:8000';
}

// Update endpoints
async generateSecurityTests(req: Request, res: Response) {
  const response = await axios.post(
    `${this.pythonApiUrl}/api/testdata/generate/security`,
    { template, count, testDataType: 'security' }
  );
}
```

### **3. Frontend Usage**

Direct calls to specific endpoints:

```typescript
// Option 1: Dedicated endpoint
const response = await axios.post(
  'http://localhost:8000/api/testdata/generate/security',
  { template: {...}, count: 10 }
);

// Option 2: Unified endpoint (current)
const response = await axios.post(
  'http://localhost:8000/api/dynamic/generate-testdata',
  { template: {...}, count: 10, testDataType: 'security' }
);
```

---

## 📊 Usage Statistics

### **Test Data Types Coverage**

| Type | GPT-4o | Scenarios | Metadata Fields | Script-Aware |
|------|--------|-----------|----------------|--------------|
| Security | ✅ | 15-20 | `_attack_vector`, `_description` | ✅ |
| Boundary | ✅ | 15-20 | `_boundary_type`, `_description` | ✅ |
| Equivalence | ✅ | 12-15 | `_partition_class`, `_partition_type` | ✅ |
| Positive | ✅ | 10-15 | `_scenario_type`, `_description` | ✅ |
| Negative | ✅ | 10-15 | `_invalid_type`, `_description` | ✅ |

---

## 🎯 Quick Reference

### **Generate Security Test Data**
```bash
curl -X POST http://localhost:8000/api/dynamic/generate-testdata \
  -H "Content-Type: application/json" \
  -d '{
    "template": {"email": "{{faker.email}}"},
    "count": 10,
    "testDataType": "security"
  }'
```

### **Generate Boundary Test Data**
```bash
curl -X POST http://localhost:8000/api/dynamic/generate-testdata \
  -H "Content-Type: application/json" \
  -d '{
    "template": {"age": "{{faker.number(1-100)}}"},
    "count": 10,
    "testDataType": "boundary"
  }'
```

### **Get AI Recommendations**
```bash
curl -X POST http://localhost:8000/api/ai-analysis/recommend-testdata \
  -H "Content-Type: application/json" \
  -d '{
    "script_content": "await page.fill(\"#email\", \"test\");",
    "test_data_type": "security"
  }'
```

---

## 📚 Documentation Files

- `COMPLETE_AI_TEST_DATA_GENERATION.md` - Complete guide for all 5 types
- `AI_POWERED_ALL_TEST_TYPES.md` - Security, Boundary, Equivalence details
- `SECURITY_TESTING_ENHANCEMENT.md` - Security testing focus
- `STATE_TRANSITION_AND_DECISION_MAKING_RESEARCH.md` - Advanced AI techniques

---

**Last Updated:** November 26, 2025  
**Version:** 1.0  
**Status:** ✅ All 5 test data types AI-powered and operational
