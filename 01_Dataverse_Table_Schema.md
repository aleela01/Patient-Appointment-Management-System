# Patient Appointment Management - Dataverse Table Schema

## Table: Patient Appointments

### Table Details
- **Display Name**: Patient Appointments
- **Plural Name**: Patient Appointments
- **Schema Name**: cr_patientappointment
- **Primary Column**: Appointment ID (auto-number)

### Columns

| Column Name | Display Name | Data Type | Required | Description |
|------------|--------------|-----------|----------|-------------|
| cr_patientappointmentid | Appointment ID | Primary Key (GUID) | Yes | Unique identifier |
| cr_appointmentid | Appointment Number | Autonumber | Yes | Human-readable ID (e.g., APT-001) |
| cr_patientname | Patient Name | Single Line of Text (100) | Yes | Full name of patient |
| cr_patientemail | Patient Email | Email | Yes | Contact email |
| cr_patientphone | Patient Phone | Phone | No | Mobile number for SMS |
| cr_appointmentdate | Appointment Date & Time | Date and Time | Yes | Scheduled appointment datetime |
| cr_appointmenttype | Appointment Type | Choice | Yes | Type of consultation |
| cr_department | Department | Choice | Yes | Hospital department |
| cr_physician | Physician Name | Single Line of Text (100) | Yes | Assigned doctor |
| cr_status | Status | Choice | Yes | Current appointment status |
| cr_notes | Notes | Multiple Lines of Text | No | Additional information |
| cr_reminderssent | Reminders Sent | Whole Number | No | Count of reminders sent |
| cr_lastremindersent | Last Reminder Sent | Date and Time | No | Timestamp of last reminder |

### Choice Column Details

#### Appointment Type (cr_appointmenttype)
- Initial Consultation
- Follow-up Visit
- Diagnostic Test
- Surgical Procedure
- Telemedicine Consultation

#### Department (cr_department)
- Cardiology
- Orthopedics
- Neurology
- Oncology
- Pediatrics
- General Medicine

#### Status (cr_status)
- Scheduled (default)
- Confirmed
- Reminder Sent
- Completed
- Cancelled
- No-Show

### Business Rules
1. Appointment Date must be in the future
2. Email format validation
3. Status transitions: Scheduled → Confirmed → Reminder Sent → Completed/No-Show
4. Automatic reminder count increment

### Security
- Enable for: Activities, Notes, Connections
- Change tracking: Enabled
- Audit: Enabled (for compliance)

---

## Alternative: Simplified Excel/SharePoint Version

If you don't have Dataverse access, use SharePoint List with these columns:

| Column | Type | Settings |
|--------|------|----------|
| Title | Single line text | Appointment ID (auto) |
| PatientName | Single line text | Required |
| PatientEmail | Single line text | Email validation |
| PatientPhone | Single line text | |
| AppointmentDate | Date/Time | Required, Include time |
| AppointmentType | Choice | (see choices above) |
| Department | Choice | (see choices above) |
| Physician | Single line text | Required |
| Status | Choice | Default: Scheduled |
| Notes | Multiple lines | Plain text |
| RemindersSent | Number | Default: 0 |

---

## Next Steps
1. Create this table in your Dataverse environment
2. Or create SharePoint List if no Dataverse access
3. Proceed to Flow 1: Appointment Scheduler
