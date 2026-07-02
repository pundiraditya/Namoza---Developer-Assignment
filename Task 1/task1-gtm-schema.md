# Task 01 — GTM Event Schema: OrthoNow

## Event Schema

| Event Name | Trigger Type | Key Parameters | GA4 Report / Audience |
|---|---|---|---|
| `booking_step_complete` (step 1) | Custom Event, listens for dataLayer push on step transition | `step_number`, `step_name`, `clinic_location`, `specialty` | Funnel Exploration step 1; audience: "started booking, no completion" |
| `booking_step_complete` (step 2) | Custom Event trigger | `step_number`, `step_name`, `clinic_location`, `has_phone` | Funnel Exploration step 2; remarketing audience for step-2 drop-offs |
| `booking_confirmed` | Custom Event, fired on server-confirmed booking (not just the client-side click) | `step_number: 3`, `clinic_location`, `specialty`, `booking_id` | Conversions report — this is the one I'd import into Google Ads |
| `call_now_click` | Click trigger, `href` contains `tel:` | `page_type`, `clinic_location`, `phone_number_masked` | Engagement report; audience: "high-intent, no booking" |
| `whatsapp_widget_click` | Click trigger, `href` contains `wa.me` | `page_type`, `clinic_location`, `widget_position` | Engagement report |
| `patient_guide_download` | Form submission trigger + click on the resulting download link | `guide_name`, `lead_source_page`, `has_phone` | Lead-gen conversion; nurture audience for non-bookers |
| `clinic_page_view` | GA4 config tag, automatic page_view with a custom dimension attached | `clinic_name`, `clinic_city`, `page_location` | Pages/screens report filtered by clinic; per-city intent audience |
| `blog_scroll_depth` | Built-in Scroll Depth trigger (25/50/75/90%) | `percent_scrolled`, `article_title`, `article_category` | Engagement report; "engaged readers" audience |

Quick note — the JSON example in the brief uses single quotes around keys, which isn't actually valid JSON and would break if anything tried to parse it downstream. I've used double quotes everywhere below.

## Tracking the booking funnel

OrthoNow's booking form is a single-page component it swaps sections in and out, the URL never changes, nothing reloads. GTM only reacts to things like DOM events, URL changes, or dataLayer pushes, and there's no browser event that means "user just finished step 2 of a form the JS is managing internally." So GTM genuinely can't see step progress on its own here. The front-end dev has to add a `dataLayer.push()` inside each step's "next" handler, right after validation passes, and GTM just listens for that via a Custom Event trigger. It's not detecting anything it's just picking up what the dev already sent it.

**Step 1 — location + specialty selected:**
```json
{
  "event": "booking_step_complete",
  "step_number": 1,
  "step_name": "location_specialty_selected",
  "clinic_location": "Indiranagar - Bengaluru",
  "specialty": "Knee & Joint Care"
}
```

**Step 2 — contact details entered:**
```json
{
  "event": "booking_step_complete",
  "step_number": 2,
  "step_name": "contact_details_entered",
  "clinic_location": "Indiranagar - Bengaluru",
  "preferred_date": "2026-07-04"
}
```

**Step 3 — booking confirmed.** This one I'd tie to an actual backend confirmation rather than just the client-side confirm click, otherwise someone could trigger it without a real booking going through:
```json
{
  "event": "booking_confirmed",
  "step_number": 3,
  "step_name": "booking_confirmed",
  "clinic_location": "Indiranagar - Bengaluru",
  "specialty": "Knee & Joint Care",
  "booking_id": "ORTH-20260701-4471"
}
```

For the GTM side: one Custom Event trigger per step (or a single trigger on `booking_step_complete` with tag-level conditions on `step_number` — probably easier to maintain), each firing a GA4 event tag that passes `step_number`/`step_name` along. Those two need to be registered as event-scoped custom dimensions in GA4 Admin, otherwise they won't show up as usable breakdowns later.

For the funnel report itself — open funnel in Explorations, three steps filtered on the right event name + step_number combo. I'd turn on elapsed time too, since someone finishing step 1 and then sitting for ten minutes before step 2 tells you something different than someone who never comes back at all. Breaking it down by clinic_location is probably the most useful cut for OrthoNow specifically, since drop-off will likely vary by clinic depending on slot availability.

## Which event to import into Google Ads

`booking_confirmed`, not the call clicks or the WhatsApp clicks or the guide download.

The click events are tempting because they're higher volume, but they're not real conversions they're someone tapping a button, which could be accidental or just browsing. If you import those as the optimization target, Smart Bidding starts chasing clicks instead of actual bookings, and your CPA looks great while patient volume doesn't move at all. The guide download is a genuine lead but it's still early funnel downloading a PDF isn't the same as wanting an appointment. `booking_confirmed` is the only one that's a backend-verified, completed booking, which is the actual thing OrthoNow is paying for.
