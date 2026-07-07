## FutureDev WhatsApp Maintenance Chatbot

SendPulse Flow Build Guide — Phase 1 Implementation
Companion to the Phase 1 User Journey Brief (2 June 2026). This guide translates that brief into buildable steps inside your existing SendPulse WhatsApp bot, and defines the webhook contract used to connect SendPulse to the Google Sheet database.
Bot reference: triggers/6a3553c2a95e1213e0015827 (existing flow, build onto this).

## 1. Architecture

SendPulse's Visual Flow Builder cannot safely generate duplicate-proof, per-complex sequential ticket numbers, and cannot run the cross-tab lookups the brief requires (Section 2 and Section 10). The brief allows Google Sheets as the Phase 1 database only if duplicate-prevention and sequencing controls are clearly implemented — so the flow must call out to a small backend rather than writing to the sheet directly.
Tenant (WhatsApp) <-> SendPulse Flow <-> HTTP Request node <-> Google Apps Script Web App <-> Google Sheet
The Apps Script Web App (Code.gs, provided alongside this guide) exposes four actions your flow will call over HTTPS:

## 2. One-time setup before building the flow

Code.gs reads tenant data directly from your existing per-property tabs (135 Coleraine, 146 Coleraine, 37 North, 10 Davies, 16 Michelle, 39 North, 51 Devonshire, 11 Charles, 30 Green, Other Managed) — no new Tenants tab and no re-entry of data required.
In the Google Sheet, run setupSheets() once from the Apps Script editor (Code.gs) to create the Tickets, TicketSequence, Media and AuditLog tabs.
Known gap: the existing tabs have no Email or Active/Inactive column, so verification is phone-number-only for now and there is no way yet to block a former tenant. Add a STATUS column (Active/Inactive) to any tab and Code.gs will start respecting it automatically.
Known gap: 'Other Managed' properties without a leading complex number (Sandton Towers, 77 On Grayston, Odyssey) get a placeholder code instead of a real FutureDev complex number — get the real numbers from FutureDev before going live with those properties.
In Apps Script, set Script Properties (Project Settings) for SHARED_SECRET, SENDPULSE_API_ID, SENDPULSE_API_SECRET and SENDPULSE_BOT_ID — these power both the SendPulse flow calls and the dashboard's live-messages/reply feature.
Deploy Code.gs as a Web App (Execute as: Me, Who has access: Anyone) and copy the Web App URL.
In SendPulse, go to your bot's Variables section and create the flow variables listed in Section 5 below.
Store the Web App URL and shared secret as SendPulse Bot Variables (e.g. {{webhook_url}}, {{webhook_token}}) so they aren't hardcoded into every HTTP node.

## 3. Flow variables to create in SendPulse


## 4. Journey 1 — First-time verification (build onto entry point)

Build these as sequential flow steps after your existing trigger/entry node.

## 5. Journey 2 — Returning verified user

Run verifyTenant on every incoming session (not just first contact) so a number that has since become inactive is blocked, per brief Journey 2 step 2.
Main menu buttons: 1 Maintenance, 2 Lease query, 3 Accounts query, 4 Utilities query.
Only "Maintenance" continues into Journey 3; the other three options should show a short "coming soon — contact FutureDev directly" message and end the flow branch, per brief Section 7.

## 6. Journey 3 — Maintenance ticket logging


## 7. Internal notifications

Add a SendPulse "Send Email" action immediately after a successful createTicket response, to your nominated maintenance recipients, subject including ticket number, complex, unit and emergency flag.
For emergency_flag = true, add a second, separate email/alert action to your nominated emergency recipients — this must not be the only place the emergency surfaces (brief Section 12).
Optional: trigger a "Dispatch notification" flow later by re-entering this contact's flow via a SendPulse scheduled/triggered message once FutureDev marks a ticket Dispatched on the dashboard (the dashboard's updateTicketStatus call can also fire a SendPulse outbound message via SendPulse's own API if you want this automated rather than manual).

## 8. Webhook request/response contract

Example HTTP Request node body for createTicket (set Content-Type: application/json):
{ "token": "{{webhook_token}}", "action": "createTicket",
  "tenantName": "{{tenant_name}}", "cellphone": "{{contact_phone}}",
  "email": "{{input_email}}", "complexNumber": "{{selected_complex}}",
  "complexName": "{{selected_complex_name}}", "unitNumber": "{{selected_unit}}",
  "category": "{{category}}", "subcategory": "{{subcategory}}",
  "description": "{{description}}", "mediaLinks": "{{media_link}}",
  "emergency": {{emergency_flag}} }
Response: { ok: true, ticketReference: "01-146-0001" } — map ticketReference to {{ticket_reference}} using SendPulse's "Save response to variable" option on the HTTP Request node.

## 9. Mapping to the UAT checklist (brief Section 14)

Every line in the brief's UAT checklist maps to a specific node above: verification gating to Steps 2-8 of Journey 1, ticket-number formatting and duplicate prevention to the Apps Script generateTicketReference lock, and dashboard/sheet field capture to the Tickets tab schema in Code.gs. Run the full UAT list against this build before go-live, per brief Section 16.

## 10. What still needs FutureDev/Keagan input before this goes live

Final maintenance terms, call-out wording, privacy notice and opt-in text to drop into the message templates above.
Approved OTP provider/process for Journey 1 Step 6.
Nominated normal and emergency email recipients.
Completed Tenants tab migration with Email and Status columns populated for the pilot complexes.
Signed POPIA operator agreement before any tenant data is loaded into the sheet (brief Section 2 and 15).
