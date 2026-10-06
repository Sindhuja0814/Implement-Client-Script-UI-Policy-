# ServiceNow Incident – Client Script & UI Policy

## Project Overview
This micro project demonstrates how ServiceNow UI Policies and Client Scripts can be used to enforce data integrity and dynamic field behavior on Incident records.

## Objectives
- Make fields mandatory based on Incident conditions.
- Automatically set Urgency when Impact is High.
- Prevent saving a High Impact Incident when Assigned To is empty.
- Prevent direct State changes through list editing.
- Demonstrate reverse UI Policy behavior and form-based updates.

## Technologies / Skills
- ServiceNow Incident Management
- UI Policy
- UI Policy Actions
- Client Scripts
- JavaScript
- Form Validation

## Configuration

### 1. UI Policy – High Impact Control
**Table:** Incident  
**Condition:** Impact is `1 - High`  
**Active:** True  
**Reverse if false:** True

The UI Policy makes the Assignment Group mandatory when the condition is met.

### 2. UI Policy Action – Urgency
**Field:** Urgency  
**Read-only:** True  
**Condition:** Controlled by the High Impact Control UI Policy.

### 3. onChange Client Script – Auto Set Urgency
**Table:** Incident  
**Type:** onChange  
**Field:** Impact

When Impact is High, Urgency is automatically set to High.

### 4. onSubmit Client Script – Prevent Save if Assigned To Missing
**Table:** Incident  
**Type:** onSubmit

When Impact is High and Assigned To is empty, the record is prevented from saving.

### 5. onCellEdit Client Script – Prevent State Change via List Edit
**Table:** Incident  
**Type:** onCellEdit  
**Field:** State

Direct State changes through list editing are blocked. State changes through the Incident form are allowed.

## Client Scripts

### onChange – Auto Set Urgency
```javascript
function onChange(control, oldValue, newValue, isLoading) {
    if (isLoading || newValue == '') {
        return;
    }

    if (newValue == '1') {
        g_form.setValue('urgency', '1');
        g_form.addInfoMessage('Urgency set to High for High impact incident.');
    }
}
```

### onSubmit – Prevent Save if Assigned To Missing
```javascript
function onSubmit() {
    if (g_form.getValue('impact') == '1' &&
        g_form.getValue('assigned_to') == '') {

        g_form.showErrorBox(
            'assigned_to',
            'Assigned To is mandatory for High impact incidents.'
        );

        return false;
    }

    return true;
}
```

### onCellEdit – Prevent State Change via List Edit
```javascript
function onCellEdit(sysIDs, table, oldValues, newValue, callback) {
    alert('State cannot be updated using list editing. Please open the Incident.');
    callback(false);
}
```

## Testing

### Test 1 – Mandatory Enforcement
1. Open **Incident → Create New**.
2. Set Impact to **High**.
3. Leave Assigned To empty.
4. Click Submit.
5. The Incident should not be saved and an error should appear.

### Test 2 – Successful Save
1. Select a user in Assigned To.
2. Submit the Incident.
3. The record should save successfully.
4. Verify Urgency auto-setting and other configured behaviors.

### Test 3 – Reverse Condition
1. Open an Incident with Impact = High.
2. Change Impact to Medium.
3. Assigned To should no longer be mandatory.
4. Urgency should become editable.

### Test 4 – List Edit Blocking
1. Open the Incident list.
2. Try to edit State directly in the list.
3. An alert should appear and the State should remain unchanged.

### Test 5 – Form-Based State Update
State changes made through the Incident form should be allowed and saved successfully.

## Project Structure
```text
servicenow-incident-ui-policy-client-script/
├── README.md
├── .gitignore
├── client-scripts/
│   ├── onChange-auto-set-urgency.js
│   ├── onSubmit-prevent-save-if-assigned-to-missing.js
│   └── onCellEdit-prevent-state-change.js
├── ui-policy/
│   └── High-Impact-Control.md
└── docs/
    └── project-summary.md
```

## Conclusion
The project demonstrates how UI Policies and Client Scripts work together to enforce dynamic field behavior, automate updates, and prevent incorrect Incident submissions in ServiceNow.

> Note: This repository contains configuration documentation and Client Script source code. The ServiceNow configuration itself must be created inside a ServiceNow instance.
