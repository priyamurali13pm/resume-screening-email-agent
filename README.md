# Resume Screening & Interview Email Agent

An n8n workflow that reads resumes forwarded by a hiring manager, scores each candidate against a role using Gemini, logs the result, and automatically emails shortlisted candidates with a Google Meet interview link.

Built for the **Customer Outreach Executive** hiring process at Twinn.live, and running live.

## What it does

1. **Trigger:** A Gmail trigger checks every minute for emails from the manager that have a PDF attachment.
2. **Read:** The PDF resume is converted to text.
3. **Extract:** Gemini pulls out the candidate's name and email address. If no email is found, an alert is sent so the resume can be handled manually.
4. **Score:** Gemini scores the resume from 0 to 100 against the role requirements set in the Config node, and gives a short reason.
5. **Log:** Name, email, score, reason and date are appended to a Google Sheet.
6. **Decide:** The score is compared with the pass mark (default 70).
   - **Pass:** the candidate moves to the interview flow below.
   - **Fail:** a polite decline email is sent.
7. **Interview flow (shortlisted candidates):**
   - Wait until 10 AM the next day, then send the welcome email.
   - Wait 5 hours, then count that day's shortlisted candidates in the sheet.
   - Assign an interview window: 10 candidates per one-hour window, starting at 11:00. The 11th candidate goes to 12:00, and so on.
   - Create a Google Calendar event with an automatic Google Meet link.
   - Send the interview details email to the candidate.

## Workflow diagram (simplified)

```
Gmail trigger -> Extract PDF text -> Config -> Gemini: name + email
   -> Email found? -- no --> Alert email
   -> yes -> Gemini: score -> Log to Google Sheet -> Score >= 70?
        -> no  -> Decline email
        -> yes -> Wait to 10 AM -> Welcome email -> Wait 5 hours
               -> Count candidates -> Pick slot -> Calendar + Meet link -> Interview email
```

## Tech stack

- n8n (workflow automation)
- Gemini API (`gemini-3.1-flash-lite`) for extraction and scoring
- Gmail, Google Calendar (with Google Meet) and Google Sheets

## Setup

1. In n8n, create a new workflow and choose **Import from file**, then select `workflow/email-agent-workflow.json`.
2. Connect your own credentials on the Gmail, Google Calendar and Google Sheets nodes.
3. Create a Google Sheet with these columns: `Date`, `Candidate Name`, `Candidate Email`, `Score`, `Reason`, `Welcome Sent`, `Interview Sent`. Select it in both Google Sheets nodes.
4. Open the **Config (EDIT ME per role)** node and replace every placeholder:
   - `gemini_api_key`: your Gemini API key
   - `whatsapp_contact`: the contact number shown in candidate emails
   - `notify_email`: where the missing-email alerts go
   - `position`, `job_requirements`, `min_score`, `interview_hour`, `interview_minute`: your role details
5. In the **Resume Forwarded by Manager** node, replace `MANAGER_EMAIL@example.com` in the filter with the manager's email.
6. In the **Create Interview Slot** node, select your own calendar.
7. Switch the workflow to **Active**.

`test_mode` in the Config node shortens both waits to one minute so you can test quickly. Keep it `false` in real use.

## Notes

- The Twilio WhatsApp notification node is included but **disabled**. It is not part of the live workflow.
- The slot-picking step counts candidates with a score of 70 or above. If you change `min_score`, update the number in the **Pick Slot** node too.
- Never commit real API keys or personal contact details. All secrets in this repo are placeholders.

## Author

Priyadarshini
[LinkedIn](https://linkedin.com/in/priyadarshini-b-9828a3339) | [Kaggle](https://kaggle.com/priyadarshinipm)
