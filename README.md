# Website Lead Auto-Reply (n8n template)

An n8n workflow that turns a website contact form into an instant, AI-drafted reply — and logs every lead to a spreadsheet at the same time.

## What it does

1. **Form submission** — someone fills in your website's contact form (name, email, company, service interested in, message, plus an optional marketing-consent checkbox and a privacy policy link).
2. **Two things happen in parallel:**
   - The lead is appended as a new row in a Google Sheet, so nothing is ever lost even if the AI step fails.
   - An AI Agent (Claude) reads the message against a fixed set of business rules and drafts a reply.
3. **The AI decides whether the reply is safe to send automatically or needs a human to check it first** (`needs_review`), based on rules you define — e.g. it should never invent prices, book meetings, or promise a named person will follow up.
4. **If it's safe:** the reply waits a random 1–5 minutes (so it doesn't look like a robot fired instantly) and is sent for real.
5. **If it needs review:** the reply is saved as a Gmail draft for you to check, edit, and send yourself.

## Setup

1. **Import** `website-lead-auto-reply.json` into your n8n instance.
2. **Connect credentials** for each of these nodes (n8n will prompt you — nothing is pre-filled):
   - **Google Sheets** ("Append row in sheet") — OAuth2, then point the `Document` field at your own sheet.
   - **Gmail** ("Save Draft Reply for Review" and "Send Auto-Reply") — OAuth2.
   - **Anthropic** ("Claude Sonnet 5") — API key.
3. **Point the Google Sheet at your own file.** The `Document` field is set to a placeholder (`YOUR_SHEET_ID`) — replace it with your own sheet's URL, and update the `Sheet` field to match your tab name. Give the sheet these columns: `Name`, `Email`, `Company`, `Service`, `Message`, `Date`, `Marketing Consent`.
4. **Rewrite the System Message** on the AI Agent node. It currently contains a worked example for a fictional accounting firm ("BDG") — replace the business context (services, sectors, offices) and the hard rules (what it can/can't say) with your own. This is the most important step: the quality of the auto-replies depends entirely on how specific and restrictive these rules are.
5. **Add your own form fields** to the "Website Form" trigger node to match what you actually want to collect.
6. **Test** using the Test URL first, then **Publish** the workflow to activate the Production URL and go live.

## GDPR / consent notes

- The marketing-consent checkbox is unticked by default and separate from submitting the enquiry — this is required if you intend to email leads again beyond just replying to what they asked.
- The privacy policy link next to the submit button is a disclosure requirement.
- Replying to an existing enquiry doesn't require marketing consent (legitimate interest), but any further marketing contact does — that's what the checkbox is for.
- You're responsible for your own Data Processing Agreements with each processor (n8n, Google, Anthropic) and your own retention policy.

## Notes

- The "Human-like Delay" (1–5 min random wait) is deliberate — an instant reply reads as automated; a short, human-scale delay doesn't.
- The Structured Output Parser enforces a strict `{ reply, needs_review }` shape from the model. If you see "Model output doesn't fit required format" errors occasionally, that's an intermittent model/tool-call quirk, not a config issue — a retry usually works.
