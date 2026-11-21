# Project: AI assistant for Zoho CRM

An AI assistant that gets data into Zoho CRM (cards, files), works with it (chat), and gets it out (files saved to the record). All five features can be built with the Claude API plus Zoho's APIs.

---

## 1. Chat with AI

- **Where:** a CRM widget (web), a Cliq bot, or a screen in the mobile app.
- **How:** your server calls Claude with tools such as `search_records`, `get_record`, `create_record`, `update_record` and `generate_file`. Claude decides which tools to call, and your server runs them against the Zoho CRM API.
- **Example:** "Show my deals closing this month over $5k" or "Log a call with Rahim and set a follow-up for Friday."
- **Watch for:** run actions with the **user's own Zoho permissions**, not an admin token, and ask for confirmation before changing or deleting anything.

## 2. File generation based on settings

- **Settings:** an admin picks templates and default formats per report type (e.g. quote → PDF, pipeline → XLSX, summary → DOCX).
- **How:** Claude returns structured JSON, and your server builds the file from the template. Use Claude's code execution (where Claude builds the file in its sandbox) only for one-off custom files, because it's slower and costs more.
- **Output:** attach the file to the record, save it to WorkDrive, or send it by email or in Cliq.

## 3. Read files from storage → create records

- **Sources:** WorkDrive, Google Drive, email attachments, or an upload in the widget.
- **How:** your server downloads the file and sends it to Claude (PDFs and images directly, spreadsheets converted to text or CSV). Claude extracts the records as JSON that matches the CRM fields.
- **Example:** a purchase order PDF becomes a deal plus products, or a lead list in Excel becomes 200 leads.
- **Watch for:** show a **preview and duplicate check** before creating anything, and use the Batch API for large files.

## 4. Scan a card on mobile → create a record

- **How:** the user takes a photo, Claude reads the image directly (no separate OCR needed), and Claude returns clean fields.
- **Then:** check for duplicates in the CRM, optionally enrich with web search, then create the lead or contact.
- **Extras:** event tag, a note on where you met, and a draft follow-up email.
- **Watch for:** the phone app sends the photo to **your server**, never directly to Claude, so your API key stays safe.

## 5. Generate a file from the current record and save it to that record

**Flow**

```
User opens a Deal → clicks "Generate" in the widget → picks a type (Quote, Proposal, Summary)
   → widget sends the module + record ID to your server
   → server reads the record and its related data from the CRM
   → Claude writes the content as JSON
   → server builds the file from the template (PDF / DOCX / XLSX)
   → server uploads it to that record's Attachments
   → widget shows "Saved to this deal" with a link
```

**Details**

- **Knowing the current record:** a CRM widget gets the module name and record ID from the page it's on, so the user doesn't choose anything.
- **Related data:** besides the record's own fields, pull related lists (contacts, products, notes, past activities, emails) so the file has full context.
- **Settings decide the output:** templates and formats per module. A Deal offers Quote (PDF) and Proposal (DOCX), a Contact offers Profile summary (PDF), an Account offers Account review (XLSX).
- **Saving:** upload through the CRM Attachments API to that record. For large or shared files, save to a WorkDrive folder linked to the record and attach the link.
- **Naming:** a clear pattern such as `Quote - Acme Ltd - 2026-11-18.pdf`.
- **Version history:** when generating again, save a new version and don't overwrite. Note who generated it and when.
- **Watch for:** the user can generate only from fields they're allowed to see. Check Zoho's attachment size limit, and have a person review a file before it goes to a customer.

---

## Shared pieces all five need

- **One backend** (e.g. Zoho Catalyst) that holds the Claude API key and the Zoho OAuth tokens.
- **Field mapping:** read the CRM's field list (`getFields`) so Claude fills the real fields, including custom ones.
- **Audit log:** a record of what the AI created, changed or generated, and for which user.
- **Usage limits:** a monthly cap per org or user to control costs.
