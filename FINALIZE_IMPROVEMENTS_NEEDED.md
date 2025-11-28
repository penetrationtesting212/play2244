# 🔧 Finalize and Run - Improvements Needed

## 🚨 Critical Issues Found

Based on the memory requirements, the following improvements are mandatory:

---

## Issue 1: Workflow State Machine Must Be Simplified

### **Memory Requirement:**
```
| Current State → | draft | reviewed | finalized |
|----------------|-------|----------|-----------|
| **draft**      | -     | ✓        |           |
| **reviewed**   |       | -        | ✓         |
| **finalized**  |       |          | -         |
```

### **Current Implementation:**
```typescript
draft → ai_enhanced → testdata_ready → human_review → finalized → archived
```

### **Required Change:**
```typescript
export type WorkflowStatus = 
  | 'draft'
  | 'reviewed'  // RENAME from 'human_review'
  | 'finalized';
  // REMOVE: ai_enhanced, testdata_ready, archived
```

### **Allowed Transitions:**
```typescript
export const WORKFLOW_STATES = {
  draft: {
    allowedTransitions: ['reviewed'],  // ONLY forward to reviewed
    requiresHumanApproval: false
  },
  reviewed: {
    allowedTransitions: ['finalized'],  // ONLY forward to finalized
    requiresHumanApproval: true  // MANDATORY human validation
  },
  finalized: {
    allowedTransitions: [],  // NO transitions allowed (terminal state)
    canRunInCI: true
  }
};
```

---

## Issue 2: Finalize Modal Must Allow Current Script or Database Script Selection

### **Memory Requirement:**
> "When the user clicks 'finalize and run', there must be an option to select either the current script or a script from the database."

### **Current Implementation:**
Execute modal (Dashboard.tsx Line 1153) only shows database scripts.

### **Required Change:**
Add radio buttons to choose script source:

```tsx
<div className="script-source-selector">
  <label>
    <input type="radio" name="scriptSource" value="current" />
    📝 Use Current Script (from editor)
  </label>
  
  <label>
    <input type="radio" name="scriptSource" value="database" />
    🗃️ Select Script from Database
  </label>
</div>

{/* Show current script details if 'current' selected */}
{scriptSource === 'current' && (
  <div className="current-script-preview">
    <h4>Current Script:</h4>
    <pre>{currentScriptCode}</pre>
  </div>
)}

{/* Show database script selector if 'database' selected */}
{scriptSource === 'database' && (
  <div className="database-script-selector">
    {/* Existing script list */}
  </div>
)}
```

---

## Issue 3: Mandatory Human Validation Gate

### **Memory Requirement:**
> "The HumanFinalize step is a mandatory gate before finalization, ensuring human oversight in the approval process."

### **Current Implementation:**
```typescript
// Line 103 - WRONG: Admin can skip to finalized
{ from: 'draft', to: 'finalized', action: 'quick-finalize', requiredRole: 'admin' }
```

### **Required Change:**
```typescript
// REMOVE admin bypass - ALL scripts MUST go through reviewed state
export const WORKFLOW_TRANSITIONS = [
  { from: 'draft', to: 'reviewed', action: 'submit-for-review' },
  { from: 'reviewed', to: 'finalized', action: 'approve' }
  // NO direct draft → finalized transition
];
```

---

## Issue 4: No Backward Transitions Allowed

### **Memory Requirement:**
> "Only forward transitions via review and HumanFinalize are permitted. No backward or skipping transitions allowed."

### **Current Implementation:**
```typescript
// Line 122 - WRONG: Backward transition
{ from: 'finalized', to: 'human_review', action: 'reopen' }
```

### **Required Change:**
```typescript
export const WORKFLOW_STATES = {
  finalized: {
    allowedTransitions: [],  // EMPTY - finalized is terminal state
    allowedActions: ['run-in-ci', 'view-insights', 'archive']
  }
};
```

---

## Issue 5: Only Finalized Scripts Can Run in CI

### **Memory Requirement:**
> "Only finalized scripts are allowed in CI."

### **Current Implementation:**
✅ This is correct (Line 81):
```typescript
finalized: {
  canRunInCI: true
}
```

### **Validation Needed:**
Ensure execution endpoints check workflow status:
```typescript
// In execution controller
if (script.workflowStatus !== 'finalized') {
  throw new Error('Only finalized scripts can run in CI');
}
```

---

## 📋 Implementation Checklist

### **Backend Changes:**

- [ ] **Simplify workflowStatus.ts**
  - Remove states: `ai_enhanced`, `testdata_ready`, `archived`
  - Rename `human_review` → `reviewed`
  - Update transitions to only allow forward flow
  - Remove admin bypass transition

- [ ] **Update pipeline.controller.ts**
  - Update finalize logic to check `reviewed` status
  - Add validation: script must be in `reviewed` state to finalize
  - Remove backward transition logic

- [ ] **Update database schema**
  - Migrate existing scripts to new state names
  - Update constraints to enforce state machine

### **Frontend Changes:**

- [ ] **Create/Update FinalizeExecuteModal.tsx**
  - Add radio button: "Current Script" vs "Database Script"
  - Show current script preview when "current" selected
  - Show database script list when "database" selected
  - Display workflow status badge
  - Enforce: Only show finalize button if status = 'reviewed'

- [ ] **Update Dashboard.tsx**
  - Replace execute modal with new FinalizeExecuteModal
  - Pass current script code to modal
  - Handle both script sources in execution logic

- [ ] **Update ScriptCueCards.tsx**
  - Update workflow visualization to show 3 states
  - Update step descriptions to match new flow

### **Validation Changes:**

- [ ] **Add status checks in execution endpoints**
  - Validate `workflowStatus === 'finalized'` before running
  - Return clear error if script not finalized

- [ ] **Update UI badges**
  - Replace 6 status badges with 3: draft, reviewed, finalized
  - Add color coding: draft (gray), reviewed (yellow), finalized (green)

---

## 🎯 Expected Flow After Fixes

### **Correct Workflow:**
```
1. User creates script → Status: draft

2. User clicks "Submit for Review" → Status: reviewed
   (AI enhancement and test data happen BEFORE status change)

3. Human validates in ValidationModal
   - Option 1: Approve → Status: finalized
   - Option 2: Reject → Status: draft (start over)

4. User clicks "Finalize and Run"
   - Choose: Current Script OR Database Script
   - System checks: workflowStatus === 'finalized'
   - Execute script in CI
```

### **Enforced Rules:**
✅ All scripts MUST go through human review (no bypass)
✅ Only forward transitions allowed (draft → reviewed → finalized)
✅ Finalized is terminal (no backwards)
✅ Only finalized scripts run in CI

---

## 🚀 Priority

**CRITICAL** - These changes are required to comply with the governance rules in memory.

**Suggested Order:**
1. Fix workflowStatus.ts (backend state machine)
2. Update controllers to enforce new rules
3. Create FinalizeExecuteModal with script selection
4. Update UI to show correct workflow

---

## 📝 Migration Strategy

Since existing scripts use the old states, migration is needed:

```sql
-- Map old states to new states
UPDATE "Script" 
SET "workflowStatus" = CASE
  WHEN "workflowStatus" IN ('draft', 'ai_enhanced', 'testdata_ready') THEN 'draft'
  WHEN "workflowStatus" = 'human_review' THEN 'reviewed'
  WHEN "workflowStatus" IN ('finalized', 'archived') THEN 'finalized'
END;
```

---

## ✅ Summary

The current implementation has:
- ❌ Too many states (6 instead of 3)
- ❌ Missing script selection in finalize modal
- ❌ Admin bypass violates mandatory review
- ❌ Backward transitions not allowed
- ❌ State names don't match memory spec

All issues must be fixed to comply with governance rules.
