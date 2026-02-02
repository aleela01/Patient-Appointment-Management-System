# Flow 1: Appointment Scheduler & Confirmation

## Purpose
Automatically sends confirmation email when a new patient appointment is created in the system.

## Trigger
**When a row is added** (Dataverse)
- Table: Patient Appointments
- Scope: Organization

## Flow Steps

### Step 1: Initialize Variables
Create these variables at the start:

```
Variable: varPatientName
Type: String
Value: @{triggerOutputs()?['body/cr_patientname']}

Variable: varAppointmentDate
Type: String  
Value: @{formatDateTime(triggerOutputs()?['body/cr_appointmentdate'], 'dddd, MMMM dd, yyyy')}

Variable: varAppointmentTime
Type: String
Value: @{formatDateTime(triggerOutputs()?['body/cr_appointmentdate'], 'hh:mm tt')}

Variable: varPhysician
Type: String
Value: @{triggerOutputs()?['body/cr_physician']}

Variable: varDepartment
Type: String
Value: @{triggerOutputs()?['body/cr_department']}
```

### Step 2: Send Confirmation Email

**Action**: Send an email (V2) - Office 365 Outlook

**To**: `@{triggerOutputs()?['body/cr_patientemail']}`

**Subject**: `Appointment Confirmation - @{variables('varAppointmentDate')}`

**Body** (HTML):
```html
<!DOCTYPE html>
<html>
<head>
    <style>
        body { font-family: Arial, sans-serif; line-height: 1.6; color: #333; }
        .header { background-color: #0078d4; color: white; padding: 20px; text-align: center; }
        .content { padding: 20px; background-color: #f9f9f9; }
        .appointment-details { 
            background-color: white; 
            padding: 15px; 
            margin: 15px 0; 
            border-left: 4px solid #0078d4; 
        }
        .detail-row { margin: 8px 0; }
        .label { font-weight: bold; color: #0078d4; }
        .footer { 
            padding: 15px; 
            text-align: center; 
            font-size: 12px; 
            color: #666; 
        }
        .button {
            display: inline-block;
            padding: 12px 24px;
            background-color: #0078d4;
            color: white;
            text-decoration: none;
            border-radius: 4px;
            margin: 10px 0;
        }
    </style>
</head>
<body>
    <div class="header">
        <h1>Appointment Confirmed</h1>
    </div>
    
    <div class="content">
        <p>Dear @{variables('varPatientName')},</p>
        
        <p>Your appointment has been successfully scheduled. Please review the details below:</p>
        
        <div class="appointment-details">
            <div class="detail-row">
                <span class="label">Date:</span> @{variables('varAppointmentDate')}
            </div>
            <div class="detail-row">
                <span class="label">Time:</span> @{variables('varAppointmentTime')}
            </div>
            <div class="detail-row">
                <span class="label">Department:</span> @{variables('varDepartment')}
            </div>
            <div class="detail-row">
                <span class="label">Physician:</span> Dr. @{variables('varPhysician')}
            </div>
            <div class="detail-row">
                <span class="label">Appointment Type:</span> @{triggerOutputs()?['body/cr_appointmenttype']}
            </div>
        </div>
        
        <h3>Important Information:</h3>
        <ul>
            <li>Please arrive 15 minutes before your scheduled time</li>
            <li>Bring a valid ID and insurance card</li>
            <li>If you need to reschedule, please contact us at least 24 hours in advance</li>
        </ul>
        
        <p>You will receive a reminder 24 hours before your appointment.</p>
        
        <p>If you have any questions, please don't hesitate to contact us.</p>
    </div>
    
    <div class="footer">
        <p>This is an automated message. Please do not reply to this email.</p>
        <p>© 2026 Healthcare System - Patient Appointment Management</p>
    </div>
</body>
</html>
```

**Importance**: Normal

### Step 3: Update Appointment Status

**Action**: Update a row (Dataverse)

**Table**: Patient Appointments  
**Row ID**: `@{triggerOutputs()?['body/cr_patientappointmentid']}`

**Fields to Update**:
```
Status (cr_status): Confirmed
Reminders Sent (cr_reminderssent): 0
```

### Step 4: (Optional) Log to Analytics

**Action**: Create a row (Dataverse) - Appointment Logs table

Or use **Compose** action to log JSON for later analysis:

```json
{
  "AppointmentID": "@{triggerOutputs()?['body/cr_appointmentid']}",
  "PatientName": "@{variables('varPatientName')}",
  "Action": "Confirmation Sent",
  "Timestamp": "@{utcNow()}",
  "EmailSent": true
}
```

---

## Error Handling

### Add Scope for Error Catching

Wrap Steps 2-3 in a **Scope** named "Send Confirmation"

**Configure run after**: Create parallel branch

**Action**: Condition
- **Condition**: `result('Send Confirmation')` is equal to `Failed`

**If Yes**:
- **Update a row**: Set Status to "Scheduled" (rollback)
- **Send email** to admin: "Failed to send confirmation for appointment @{triggerOutputs()?['body/cr_appointmentid']}"

---

## Testing Checklist

- [ ] Create test appointment with your email
- [ ] Verify confirmation email received
- [ ] Check HTML formatting renders correctly
- [ ] Verify appointment status updated to "Confirmed"
- [ ] Test with invalid email (error handling)
- [ ] Test with appointments in different departments

---

## Key Features Demonstrated

✅ **Dataverse Integration** - CRUD operations  
✅ **Email Automation** - Professional HTML templates  
✅ **Data Transformation** - Date/time formatting  
✅ **Error Handling** - Scopes and conditions  
✅ **Healthcare Context** - Patient communication compliance

---

## Next: Flow 2 - Reminder System
