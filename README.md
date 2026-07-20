# 🦷 QuensultingAI Dental Receptionist Agent

> An AI-powered voice receptionist for QuensultingAI Dental Clinic — built with **Retell AI**, **n8n**, **Google Calendar**, and **Google Sheets** to automate appointment booking end-to-end.

---

## 📋 Overview

This project contains the full workflow configuration for an intelligent dental clinic receptionist that:

- **Answers inbound calls** via a Retell AI voice agent (English & Hindi)
- **Collects patient details** through a natural conversation flow
- **Checks calendar availability** in real-time before confirming a slot
- **Books appointments** on Google Calendar automatically
- **Logs every booking** to a Google Sheets spreadsheet
- **Sends confirmation emails** to patients via Gmail
- **Handles edge cases** — slot conflicts, booking failures, FAQs, dental emergencies

---

## 🏗️ Architecture

```
Caller (Phone)
     │
     ▼
┌─────────────────────────────────────┐
│         Retell AI Voice Agent        │
│  • Bilingual: English / Hindi        │
│  • Conversation Flow (GPT/Claude)    │
│  • Collects: Name, Phone, Email,     │
│    Service, Doctor, Date, Time       │
└──────────────┬──────────────────────┘
               │  HTTP POST (Webhook)
               ▼
┌─────────────────────────────────────┐
│          n8n Automation Workflow     │
│                                      │
│  1. Validate all required fields     │
│  2. Compute slot start/end times     │
│  3. Check Google Calendar            │
│  4. Evaluate slot availability       │
│       ├─ Slot Free → Create Event   │
│       │    ├─ Append to Sheets      │
│       │    ├─ Send Gmail confirm    │
│       │    └─ Return success        │
│       └─ Slot Taken → Return failed │
└─────────────────────────────────────┘
```

---

## 📂 Project Structure

```
QuensultingAI Agent/
├── RetellAI_workflow.json    # Retell AI agent & conversation flow config
├── N8N_workflow.json         # n8n booking automation workflow
├── .env.example              # Template for required credentials
├── .gitignore                # Excludes secrets from version control
└── README.md                 # This file
```

---

## ✨ Features

| Feature | Details |
|---|---|
| 🗣️ Voice AI | Retell AI with `retell-Kate` voice, claude-4.6-sonnet LLM |
| 🌐 Bilingual | English & Hindi — language selected at start of call |
| 📅 Availability Check | Real-time Google Calendar conflict detection |
| 📊 Data Logging | Auto-appends to Google Sheets with Timestamp, Patient, Doctor, Call ID |
| 📧 Email Confirmation | HTML appointment confirmation via Gmail OAuth2 |
| 🔄 Retry Logic | 1 automatic retry on booking failure; graceful fallback to human receptionist |
| 🚨 Emergency Triage | Detects emergencies and routes caller to immediate help |
| 🧠 Smart Extraction | Converts spoken dates/times (e.g., "next Tuesday at 2:30 PM") to ISO format |

---

## 🔧 Setup & Configuration

### Prerequisites

- [n8n](https://n8n.io/) instance (self-hosted or cloud)
- [Retell AI](https://www.retellai.com/) account
- Google Cloud project with OAuth2 credentials for:
  - Google Calendar API
  - Google Sheets API
  - Gmail API

### Step 1 — Configure Environment Variables

Copy `.env.example` to `.env` and fill in your actual values:

```bash
cp .env.example .env
```

### Step 2 — Import n8n Workflow

1. Open your n8n instance
2. Go to **Workflows → Import**
3. Upload `N8N_workflow.json`
4. Configure Google credentials (OAuth2) inside n8n
5. Update the following inside the workflow nodes:
   - **Google Sheets** → set your spreadsheet ID
   - **Google Calendar** → set your calendar ID
   - **Webhook** path (default: `clinic-booking`)
6. **Activate** the workflow

### Step 3 — Import Retell AI Agent

1. Log in to [Retell AI Dashboard](https://app.retellai.com/)
2. Go to **Agents → Import**
3. Upload `RetellAI_workflow.json`
4. Update the `book_appointment` tool URL to your n8n webhook URL:
   ```
   https://<your-n8n-instance>/webhook/clinic-booking
   ```
5. Update any phone numbers referenced in the global prompt to your actual receptionist number
6. Publish the agent and connect a phone number

---

## 🔑 Credentials Reference

All sensitive values must be stored as environment variables or inside your n8n credential store — **never hardcoded in the JSON files**.

| Variable | Description |
|---|---|
| `N8N_WEBHOOK_URL` | Full URL of the n8n clinic-booking webhook |
| `GOOGLE_SHEETS_ID` | ID of the Google Sheets spreadsheet for logging appointments |
| `GOOGLE_CALENDAR_ID` | ID of the Google Calendar used for booking |
| `GMAIL_OAUTH2_CREDENTIAL_ID` | n8n credential ID for Gmail OAuth2 |
| `GOOGLE_SHEETS_OAUTH2_CREDENTIAL_ID` | n8n credential ID for Google Sheets OAuth2 |
| `GOOGLE_CALENDAR_OAUTH2_CREDENTIAL_ID` | n8n credential ID for Google Calendar OAuth2 |
| `RECEPTIONIST_PHONE` | Clinic receptionist's direct phone number |

> See `.env.example` for a complete template.

---

## 📞 Conversation Flow

```
Incoming Call
     │
     ▼
[Greeting] → Select Language (English / Hindi)
     │
     ├─── Book Appointment
     │         │
     │         ▼
     │    Collect Details → Extract Variables → Call Booking Function
     │         │                                        │
     │         │                              ┌─────────┴──────────┐
     │         │                         Success              Failure
     │         │                              │                    │
     │         │                    Confirm & End          Retry / Escalate
     │
     ├─── FAQ → Answer → (Book | End)
     │
     ├─── Emergency Triage → Provide Receptionist Number
     │
     └─── Transfer Human → Provide Receptionist Number
```

---

## 🛡️ Security Notes

- **No secrets are committed** to this repository. All credentials are referenced by ID inside n8n's encrypted credential store.
- The `agent_id` field in `RetellAI_workflow.json` has been cleared before publishing.
- Webhook IDs in `N8N_workflow.json` will auto-regenerate on import — this is safe.
- Rotate your Google OAuth2 tokens and Retell API keys periodically.

---

## 🩺 Clinic Information (Configured in Agent)

| Detail | Value |
|---|---|
| Working Hours | Monday – Saturday, 9:00 AM – 6:00 PM |
| Closed | Sunday |
| Consultation Fee | ₹500 |
| Payment Methods | Cash, Card, UPI |
| Walk-ins | Accepted (based on availability) |
| Appointment Slot | 30 minutes |
| Languages | English, Hindi |

**Services available:**
- Dental Cleaning
- Root Canal Treatment
- Teeth Whitening
- Braces Consultation
- Tooth Extraction
- General Dental Consultation

---

## 🤝 Contributing

Pull requests are welcome. For major changes, please open an issue first to discuss what you'd like to change.

---

## 📄 License

This project is licensed under the [MIT License](LICENSE).

---

*Built with ❤️ by [QuensultingAI](https://github.com/Alok-28)*
