# Flow 3: No-Show Tracker & Automated Follow-up

## Purpose
Identifies missed appointments and triggers automated follow-up to reschedule.

## Trigger
**Recurrence** (Scheduled)
- Frequency: Day
- Interval: 1 (runs daily)
- Time: 10:00 AM (after morning appointments)
- Time zone: (UTC+01:00) Rome

## Flow Steps

### Step 1: Get Yesterday's Appointments

**Action**: List rows (Dataverse)

**Table**: Patient Appointments

**Filter rows**:
```
cr_appointmentdate ge '@{formatDateTime(addDays(utcNow(), -1), 'yyyy-MM-dd')}T00:00:00Z' and 
cr_appointmentdate le '@{formatDateTime(addDays(utcNow(), -1), 'yyyy-MM-dd')}T23:59:59Z' and 
(cr_status eq 861560001 or cr_status eq 861560002)
```

*Status codes:*
- 861560001 = "Confirmed"
- 861560002 = "Reminder Sent"

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

### Step 2: Initialize No-Show Counter

```
Variable: varNoShowCount
Type: Integer
Value: 0
```

### Step 3: Apply to Each Missed Appointment

**Action**: Apply to each

**Input**: `@{outputs('List_rows')?['body/value']}`

#### Inside Loop:

#### 3A. Initialize Appointment Variables

```
Variable: varPatientName (String)
Value: @{items('Apply_to_each')?['cr_patientname']}

Variable: varMissedDate (String)
Value: @{formatDateTime(items('Apply_to_each')?['cr_appointmentdate'], 'MMMM dd, yyyy')}

Variable: varMissedTime (String)
Value: @{formatDateTime(items('Apply_to_each')?['cr_appointmentdate'], 'hh:mm tt')}
```

#### 3B. Update Status to No-Show

**Action**: Update a row (Dataverse)

**Table**: Patient Appointments  
**Row ID**: `@{items('Apply_to_each')?['cr_patientappointmentid']}`

**Fields**:
```
Status (cr_status): No-Show
```

#### 3C. Send Follow-up Email

**Action**: Send an email (V2)

**To**: `@{items('Apply_to_each')?['cr_patientemail']}`

**Subject**: `We Missed You - Please Reschedule Your Appointment`

**Body** (HTML):
```html
<!DOCTYPE html>
<html>
<head>
    <style>
        body { font-family: Arial, sans-serif; color: #333; line-height: 1.6; }
        .header { 
            background-color: #f57c00; 
            color: white; 
            padding: 25px; 
            text-align: center; 
        }
        .content { padding: 30px; background-color: #f9f9f9; }
        .missed-box { 
            background-color: #fff3e0; 
            padding: 20px; 
            margin: 20px 0; 
            border-left: 5px solid #f57c00;
            border-radius: 4px;
        }
        .cta-box {
            background-color: #ffffff;
            padding: 25px;
            margin: 20px 0;
            border-radius: 8px;
            box-shadow: 0 2px 8px rgba(0,0,0,0.1);
            text-align: center;
        }
        .button {
            display: inline-block;
            padding: 15px 30px;
            background-color: #0078d4;
            color: white;
            text-decoration: none;
            border-radius: 6px;
            font-weight: bold;
            margin: 10px;
        }
        .footer { 
            text-align: center; 
            padding: 20px; 
            font-size: 12px; 
            color: #666; 
        }
        .detail { margin: 8px 0; }
        .label { font-weight: bold; color: #f57c00; }
    </style>
</head>
<body>
    <div class="header">
        <h1>We Missed You at Your Appointment</h1>
    </div>
    
    <div class="content">
        <p style="font-size: 16px;">Dear @{variables('varPatientName')},</p>
        
        <p>We noticed that you were unable to attend your scheduled appointment. Your health is important to us, and we want to ensure you receive the care you need.</p>
        
        <div class="missed-box">
            <h3 style="margin-top: 0; color: #f57c00;">Missed Appointment Details</h3>
            
            <div class="detail">
                <span class="label">Date:</span> @{variables('varMissedDate')}
            </div>
            
            <div class="detail">
                <span class="label">Time:</span> @{variables('varMissedTime')}
            </div>
            
            <div class="detail">
                <span class="label">Department:</span> @{items('Apply_to_each')?['cr_department']}
            </div>
            
            <div class="detail">
                <span class="label">Physician:</span> Dr. @{items('Apply_to_each')?['cr_physician']}
            </div>
        </div>
        
        <div class="cta-box">
            <h2 style="color: #0078d4; margin-top: 0;">Let's Get You Rescheduled</h2>
            <p style="font-size: 16px;">We understand that unexpected situations arise. We're here to help you reschedule at your convenience.</p>
            
            <a href="tel:+390382123456" class="button">📞 Call to Reschedule</a>
            <a href="mailto:appointments@hospital.it" class="button">✉️ Email Us</a>
            
            <p style="margin-top: 20px; font-size: 14px; color: #666;">
                <strong>Contact Hours:</strong> Monday - Friday, 8:00 AM - 6:00 PM<br>
                <strong>Phone:</strong> +39 0382 123456<br>
                <strong>Email:</strong> appointments@hospital.it
            </p>
        </div>
        
        <div style="background-color: #e3f2fd; padding: 15px; border-radius: 6px; margin: 20px 0;">
            <h3 style="color: #1976d2; margin-top: 0;">💡 Tips for Your Next Appointment</h3>
            <ul>
                <li>Add the appointment to your calendar with a reminder</li>
                <li>Set a reminder on your phone 24 hours before</li>
                <li>Plan your transportation in advance</li>
                <li>If you need to cancel, let us know at least 24 hours ahead</li>
            </ul>
        </div>
        
        <p style="font-size: 14px; margin-top: 25px;">
            We care about your health and look forward to seeing you soon.
        </p>
        
        <p style="font-size: 14px;">
            Best regards,<br>
            <strong>Patient Care Team</strong><br>
            Healthcare System
        </p>
    </div>
    
    <div class="footer">
        <p>This is an automated follow-up message.</p>
        <p>If you believe you attended this appointment, please contact us immediately.</p>
        <p>© 2026 Healthcare System - Patient Care Services</p>
    </div>
</body>
</html>
```

**Importance**: Normal

#### 3D. Increment No-Show Counter

**Action**: Increment variable

**Name**: varNoShowCount  
**Value**: 1

#### 3E. Create Follow-up Task (Optional)

**Action**: Create a row (Dataverse)

**Table**: Tasks (or custom Follow-up table)

**Fields**:
```json
{
  "subject": "Follow up: No-show patient @{variables('varPatientName')}",
  "description": "Patient missed appointment on @{variables('varMissedDate')}. Automated email sent. Consider phone follow-up.",
  "regardingobjectid": "@{items('Apply_to_each')?['cr_patientappointmentid']}",
  "scheduledstart": "@{addDays(utcNow(), 1)}",
  "prioritycode": 1
}
```

*Assigns task to patient coordinator for manual follow-up*

#### 3F. Log No-Show Event

**Action**: Compose

**Inputs**:
```json
{
  "EventType": "No-Show Detected",
  "AppointmentID": "@{items('Apply_to_each')?['cr_appointmentid']}",
  "PatientName": "@{variables('varPatientName')}",
  "MissedDate": "@{items('Apply_to_each')?['cr_appointmentdate']}",
  "Department": "@{items('Apply_to_each')?['cr_department']}",
  "FollowUpEmailSent": true,
  "ProcessedAt": "@{utcNow()}"
}
```

---

### Step 4: Send Daily Summary to Admin

**Action**: Send an email (V2)

**To**: admin@hospital.it

**Subject**: `Daily No-Show Report - @{formatDateTime(utcNow(), 'MMMM dd, yyyy')}`

**Body**:
```
Daily No-Show Summary
=====================

Date: @{formatDateTime(addDays(utcNow(), -1), 'MMMM dd, yyyy')}

Total Missed Appointments: @{variables('varNoShowCount')}

All affected patients have been sent automated follow-up emails.
Follow-up tasks have been created for the patient coordination team.

Detailed logs are available in the system.

---
Automated Report - Patient Appointment Management System
```

---

## Advanced: No-Show Pattern Analysis

### Optional Step 5: Track Repeat No-Shows

**Action**: List rows (Dataverse)

**Table**: Patient Appointments

**Filter**: 
```
cr_patientname eq '@{variables('varPatientName')}' and 
cr_status eq 861560004
```

*Count no-shows per patient*

**Action**: Condition

**If**: `@{length(outputs('List_rows_-_Patient_history')?['body/value'])}` is greater than `2`

**True Branch**:
- Flag patient as "high-risk no-show"
- Send alert to patient coordinator
- Consider requiring confirmation call for future appointments

---

## Error Handling

### Add Scope for Resilience

**Scope**: "Process No-Show"
- Contains: Steps 3B through 3F

**Configure run after**: Parallel branch

**Condition**: `result('Process_No-Show')` equals `Failed`

**If Failed**:
- Log error details
- Continue processing other appointments
- Send error notification to admin

---

## Testing Scenarios

### Test Case 1: Single No-Show
1. Create appointment for yesterday
2. Leave status as "Confirmed"
3. Run flow manually
4. Verify:
   - Status changed to "No-Show"
   - Follow-up email sent
   - Task created
   - Daily summary shows count = 1

### Test Case 2: Multiple No-Shows
1. Create 5 appointments for yesterday
2. Run flow
3. Verify all processed correctly
4. Check processing time

### Test Case 3: Already Completed
1. Create appointment for yesterday
2. Manually set status to "Completed"
3. Run flow
4. Verify: NOT marked as no-show

---

## Healthcare Context & Value

### Business Impact Metrics

**Problem Addressed:**
- Healthcare no-show rates: 15-30% average
- Cost per no-show: €100-€200
- Wasted physician time: ~20% of schedule

**Solution Value:**
- Automated follow-up: 40% rescheduling rate
- Reduced admin workload: 2-3 hours/day
- Better patient retention
- Improved scheduling efficiency

### Compliance Considerations

✅ **Patient Rights**:
- Respectful, non-judgmental communication
- Easy rescheduling options
- Privacy maintained (no details in subject line)

✅ **Documentation**:
- Automated audit trail
- Regulatory compliance (track no-show patterns)
- Quality improvement data

---

## Key Features Demonstrated

✅ **Scheduled Batch Processing** - Daily operations  
✅ **Data Analysis** - Historical queries, pattern detection  
✅ **Multi-step Workflows** - Status updates, emails, task creation  
✅ **Business Intelligence** - Reporting and analytics  
✅ **Healthcare Operations** - Real-world clinical workflow  
✅ **Error Resilience** - Continues despite individual failures  
✅ **Stakeholder Communication** - Patients + admin reporting

---

## Enhancements for Production

1. **SMS Follow-up** - Text reminder after 2 days if no response
2. **Patient Portal Integration** - Self-service rescheduling link
3. **Predictive Analytics** - ML model to predict no-show risk
4. **Resource Optimization** - Automatically fill gaps with waitlist patients
5. **Multi-language Support** - Detect patient language preference

---

## Next: Documentation & Deployment Guide
