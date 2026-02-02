# Flow 2: Automated 24-Hour Appointment Reminder

## Purpose
Automatically sends reminder emails/SMS 24 hours before scheduled appointments.

## Trigger
**Recurrence** (Scheduled)
- Frequency: Hour
- Interval: 1 (runs every hour)
- Time zone: (UTC+01:00) Rome, Stockholm, Vienna

*Alternative: Run every 4 hours to reduce API calls*

## Flow Steps

### Step 1: Get Upcoming Appointments

**Action**: List rows (Dataverse)

**Table**: Patient Appointments

**Filter rows**:
```
cr_appointmentdate ge '@{addHours(utcNow(), 23)}' and 
cr_appointmentdate le '@{addHours(utcNow(), 25)}' and 
cr_status eq 861560001
```

*Note: 861560001 = "Confirmed" status value (check your environment)*

**Additional filters**:
- Status: Confirmed
- Reminders Sent: less than 1

**Select columns**: 
- cr_patientappointmentid
- cr_appointmentid
- cr_patientname
- cr_patientemail
- cr_patientphone
- cr_appointmentdate
- cr_physician
- cr_department
- cr_appointmenttype

### Step 2: Apply to Each Appointment

**Action**: Apply to each

**Input**: `@{outputs('List_rows')?['body/value']}`

#### Inside the loop:

#### 2A. Initialize Reminder Variables

```
Variable: varReminderPatient
Type: String
Value: @{items('Apply_to_each')?['cr_patientname']}

Variable: varReminderDate
Type: String
Value: @{formatDateTime(items('Apply_to_each')?['cr_appointmentdate'], 'dddd, MMMM dd, yyyy')}

Variable: varReminderTime
Type: String
Value: @{formatDateTime(items('Apply_to_each')?['cr_appointmentdate'], 'hh:mm tt')}

Variable: varHoursUntil
Type: Integer
Value: @{div(sub(ticks(items('Apply_to_each')?['cr_appointmentdate']), ticks(utcNow())), 36000000000)}
```

#### 2B. Send Reminder Email

**Action**: Send an email (V2)

**To**: `@{items('Apply_to_each')?['cr_patientemail']}`

**Subject**: `Reminder: Your Appointment Tomorrow - @{variables('varReminderDate')}`

**Body** (HTML):
```html
<!DOCTYPE html>
<html>
<head>
    <style>
        body { font-family: Arial, sans-serif; color: #333; }
        .header { 
            background: linear-gradient(135deg, #0078d4 0%, #005a9e 100%);
            color: white; 
            padding: 30px; 
            text-align: center; 
        }
        .content { padding: 30px; background-color: #f5f5f5; }
        .reminder-box { 
            background-color: white; 
            padding: 25px; 
            margin: 20px 0; 
            border-radius: 8px;
            box-shadow: 0 2px 4px rgba(0,0,0,0.1);
        }
        .highlight { 
            background-color: #fff3cd; 
            padding: 15px; 
            border-left: 4px solid #ffc107;
            margin: 15px 0;
        }
        .detail { margin: 10px 0; font-size: 16px; }
        .label { font-weight: bold; color: #0078d4; }
        .value { color: #333; }
        .checklist { 
            background-color: #e8f5e9; 
            padding: 15px; 
            border-radius: 6px;
            margin: 15px 0;
        }
        .footer { 
            text-align: center; 
            padding: 20px; 
            font-size: 12px; 
            color: #666; 
        }
    </style>
</head>
<body>
    <div class="header">
        <h1>🏥 Appointment Reminder</h1>
        <p style="font-size: 18px; margin: 10px 0;">Your appointment is in approximately @{variables('varHoursUntil')} hours</p>
    </div>
    
    <div class="content">
        <p style="font-size: 16px;">Dear @{variables('varReminderPatient')},</p>
        
        <div class="highlight">
            <strong>⏰ This is a friendly reminder about your upcoming appointment</strong>
        </div>
        
        <div class="reminder-box">
            <h2 style="color: #0078d4; margin-top: 0;">Appointment Details</h2>
            
            <div class="detail">
                <span class="label">📅 Date:</span> 
                <span class="value">@{variables('varReminderDate')}</span>
            </div>
            
            <div class="detail">
                <span class="label">🕐 Time:</span> 
                <span class="value">@{variables('varReminderTime')}</span>
            </div>
            
            <div class="detail">
                <span class="label">🏥 Department:</span> 
                <span class="value">@{items('Apply_to_each')?['cr_department']}</span>
            </div>
            
            <div class="detail">
                <span class="label">👨‍⚕️ Physician:</span> 
                <span class="value">Dr. @{items('Apply_to_each')?['cr_physician']}</span>
            </div>
            
            <div class="detail">
                <span class="label">📋 Type:</span> 
                <span class="value">@{items('Apply_to_each')?['cr_appointmenttype']}</span>
            </div>
        </div>
        
        <div class="checklist">
            <h3 style="margin-top: 0; color: #2e7d32;">✓ Pre-Appointment Checklist</h3>
            <ul style="line-height: 1.8;">
                <li>Arrive 15 minutes early for check-in</li>
                <li>Bring your ID and insurance card</li>
                <li>Bring any recent test results or medical records</li>
                <li>Write down any questions for your doctor</li>
                <li>Bring a list of current medications</li>
            </ul>
        </div>
        
        <div style="background-color: #fff; padding: 15px; border-radius: 6px; margin: 15px 0;">
            <h3 style="color: #d32f2f; margin-top: 0;">Need to Cancel or Reschedule?</h3>
            <p>Please contact us at least 12 hours in advance:</p>
            <p>📞 Phone: +39 0382 123456<br>
            ✉️ Email: appointments@hospital.it</p>
        </div>
        
        <p style="font-size: 14px; color: #666; margin-top: 20px;">
            We look forward to seeing you!
        </p>
    </div>
    
    <div class="footer">
        <p>This is an automated reminder. Please do not reply to this email.</p>
        <p>If you have received this in error, please contact us immediately.</p>
        <p>© 2026 Healthcare System - Patient Care Services</p>
    </div>
</body>
</html>
```

#### 2C. (Optional) Send SMS Reminder

**Action**: Condition

**Condition**: `@{empty(items('Apply_to_each')?['cr_patientphone'])}` is equal to `false`

**If Yes**:

**Action**: Send SMS (Twilio or other SMS provider)

**To**: `@{items('Apply_to_each')?['cr_patientphone']}`

**Message**:
```
Appointment Reminder
Hello @{variables('varReminderPatient')},

Your appointment is tomorrow:
📅 @{variables('varReminderDate')}
🕐 @{variables('varReminderTime')}
👨‍⚕️ Dr. @{items('Apply_to_each')?['cr_physician']}
🏥 @{items('Apply_to_each')?['cr_department']}

Please arrive 15 min early. Bring ID & insurance card.

To cancel/reschedule: +39 0382 123456
```

#### 2D. Update Appointment Record

**Action**: Update a row (Dataverse)

**Table**: Patient Appointments  
**Row ID**: `@{items('Apply_to_each')?['cr_patientappointmentid']}`

**Fields**:
```
Status (cr_status): Reminder Sent
Reminders Sent (cr_reminderssent): @{add(items('Apply_to_each')?['cr_reminderssent'], 1)}
Last Reminder Sent (cr_lastremindersent): @{utcNow()}
```

#### 2E. Log Reminder Activity

**Action**: Compose

**Inputs**:
```json
{
  "AppointmentID": "@{items('Apply_to_each')?['cr_appointmentid']}",
  "PatientName": "@{variables('varReminderPatient')}",
  "ReminderSentAt": "@{utcNow()}",
  "AppointmentDateTime": "@{items('Apply_to_each')?['cr_appointmentdate']}",
  "EmailSent": true,
  "SMSSent": "@{not(empty(items('Apply_to_each')?['cr_patientphone']))}"
}
```

---

## Error Handling

### Wrap email sending in Try-Catch

**Add Scope**: "Try Send Reminder"
- Contains: Steps 2B, 2C, 2D

**Configure run after**: Add parallel branch

**Action**: Condition
- **If**: `result('Try_Send_Reminder')` is equal to `Failed`

**True branch**:
- **Send email** to admin with error details
- **Create row** in Error Log table
- **Do NOT update** appointment status

---

## Performance Optimization

### Add concurrency control to loop

**Apply to each** settings:
- Concurrency control: ON
- Degree of Parallelism: 5

*Sends up to 5 reminders simultaneously*

---

## Testing Scenarios

### Test Case 1: Happy Path
1. Create appointment for tomorrow same time
2. Wait for hourly trigger (or manually trigger)
3. Verify reminder email received
4. Check appointment status = "Reminder Sent"
5. Verify Reminders Sent count = 1

### Test Case 2: Multiple Appointments
1. Create 3 appointments for tomorrow
2. Trigger flow
3. Verify all 3 receive reminders
4. Check processing time

### Test Case 3: Edge Cases
- Appointment exactly 24h away
- Appointment 23.5h away
- Already received reminder (should skip)
- Invalid email address (error handling)

---

## Healthcare Compliance Notes

✅ **GDPR Compliance**: 
- Patient consent for reminders (assumed in real system)
- Unsubscribe option should be added

✅ **HIPAA Considerations**:
- No sensitive medical details in email subject
- Generic reminder content
- Secure transmission (Office 365 encryption)

✅ **Accessibility**:
- HTML email works in text-only mode
- Clear, readable font sizes
- Semantic HTML structure

---

## Key Features Demonstrated

✅ **Scheduled Automation** - Time-based triggers  
✅ **Complex Filtering** - Date/time queries in Dataverse  
✅ **Loops & Iterations** - Apply to each with concurrency  
✅ **Multi-channel Communication** - Email + SMS  
✅ **Error Handling** - Try-catch patterns  
✅ **Professional Templates** - Healthcare-grade HTML emails  
✅ **Audit Trail** - Logging and tracking

---

## Monitoring & Analytics

Add a **Flow Checker** connection to monitor:
- Number of reminders sent per day
- Success/failure rate
- Average processing time
- Patients who didn't receive reminders (investigation needed)

---

## Next: Flow 3 - No-Show Tracker
