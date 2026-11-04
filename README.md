# lead-generation 
Node 1: Lead Intake Webhook (POST /lead-intake)

Acts as the primary API entry point that receives raw lead submissions in JSON format from website forms, ads, or external test scripts.

Nodes 2 & 3: Validate & Normalize Lead (Code & IF Nodes)

Sanitizes input data by trimming whitespace, normalizing the email to lowercase, and validating required fields (name, email regex, and message). Invalid submissions are blocked immediately, returning an HTTP 400 Bad Request without consuming AI tokens.

Nodes 4, 5 & 6: Deduplication Engine (Google Sheets Lookup & Filter)

Queries the Leads tab in Google Sheets to check if the normalized email already exists. Duplicate leads bypass the AI model, return a duplicate_skipped response, and preserve existing records to avoid re-triggering sales alerts.

Node 7: Gemini AI Classifier (gemini-2.5-flash via HTTP Request)

Evaluates unique leads by sending the budget, timeline, company, and message to Google Gemini API (temperature: 0.2, responseMimeType: application/json). It returns structured data: classification (Hot, Warm, Cold), an intent score (0–100), reasoning, recommended action, and customized follow-up email copy.

Node 8: Output Parser & Fallback Guard (Code Node)

Safely parses the JSON returned by Gemini and provides high system reliability. If the AI model times out or encounters quota limits, a deterministic rule-based fallback classifies the lead based on budget and timeline keywords so execution never breaks.

Node 9: Store Lead in Database (Google Sheets Append Node)

Appends the validated, AI-scored lead into the Leads tab with all 15 fields, including generated lead_id, contact details, intent score, AI reasoning, and routing status.

Nodes 10 & 11a/11b/11c: Priority Routing & Follow-Up (Switch & Action Nodes)

Routes leads dynamically based on their AI classification:

Hot Leads (Node 11a): Sends an instant priority VIP response to the prospect and triggers an urgent internal alert for senior Account Executives.

Warm Leads (Node 11b): Triggers an automated nurture email with product information and discovery scheduling.

Cold Leads (Node 11c): Suppresses direct sales alerts and queues the prospect into a monthly educational newsletter list.

Nodes 12, 14, 16, 18 & 19: Comprehensive Audit Trail & Global Error Handling

Every execution path logs its outcome to the dedicated Execution_Logs sheet:

LEAD_PROCESSED (SUCCESS) for completed leads.

DUPLICATE_SKIPPED for existing emails.

VALIDATION_FAILED (ERROR) for malformed payloads.

RUNTIME_EXCEPTION (CRITICAL_ERROR) captured by the Global Error Trigger (Node 18) for unexpected exceptions.

Nodes 13, 15 & 17: Webhook Response Nodes

Returns clean, standardized HTTP responses (200 OK on success/duplicate, 400 Bad Request on invalid payload) back to the client.
