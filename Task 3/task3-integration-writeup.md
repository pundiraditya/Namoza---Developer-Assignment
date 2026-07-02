# Task 03 — Integration Design: Landing Page → HubSpot → WhatsApp → Google Ads

## Architecture

I'd build a small serverless function (Node, hosted on Vercel or similar) that the form posts to directly, rather than using a HubSpot native embed or routing through Zapier/Make.

Flow:
1. Form submits → POST to `/api/consultation-submit` with name, phone, clinic preference.
2. Function checks HubSpot Contacts API for an existing contact matching that phone number (`idProperty=phone`, with phone set as a unique property in HubSpot's settings — more on why below).
3. If found, update the existing contact. If not, create a new one with Source = "Google Ads - Consultation Landing Page" and Lead Status = "New Enquiry".
4. At the same time, the function calls Karix's WhatsApp API to send the confirmation message, and fires the Google Ads conversion for `booking_confirmed`. These two run in parallel with the HubSpot call, not after it no reason for WhatsApp to wait on HubSpot finishing.
5. Each of the three calls is logged independently, so if one fails the other two still go through.

I went with a direct API call over Zapier/Make mainly because of the 2-minute WhatsApp SLA a workflow tool adds its own polling/trigger delay on top of the actual API calls, which eats into that budget for no good reason. It also gives me real error responses to retry on, instead of a generic "step failed" from a no-code platform. If this were a lower-stakes MVP without a hard SLA, Zapier would honestly be the faster way to ship it — just not the right call here with paid traffic about to hit the page.

## Biggest failure point

HubSpot dedupes contacts by email by default. This form doesn't collect email, so out of the box every repeat enquiry would create a new contact instead of updating the existing one — clinic staff lose the history, and the lead gets scored as if it's a new person each time.

Fix: set `phone` as a unique-value property in HubSpot and match on that during the upsert instead of email.

The harder case is two different people submitting under the same phone number — pretty common in Indian households where one number covers a family. I wouldn't want the second person's submission to silently overwrite the first person's name. So if a name mismatch shows up on an existing phone match, the function leaves the original contact alone and adds a note flagging the second name, then puts it in a "needs review" list for staff to sort out manually rather than guessing.

## What could break the WhatsApp SLA

A few things: Karix having downtime or rate-limiting during a traffic spike, a WhatsApp template not being pre approved for the exact variables being sent (which fails silently rather than throwing an error), or a sequential call chain where WhatsApp waits on HubSpot to finish first and eats up time it doesn't need to.

To monitor it, I'd log a start and delivery-webhook timestamp for every send, and alert if anything crosses 90 seconds a buffer before the actual SLA breaks. Karix accepting the API call isn't the same as WhatsApp actually delivering the message, so the delivery webhook is the thing worth watching, not just the initial response.
