# Namoza Developer Assignment — OrthoNow

Submission for the Developer (Client Web + Martech) assignment. Three files, matching the three tasks.

## What's here

- `task1-gtm-schema.md` — event tracking plan for OrthoNow, including the booking funnel and which event to feed into Google Ads  
- `task2-landing-page/index.html` — the rebuilt "Book a Consultation" landing page  
- `task3-integration-writeup.md` — how the form connects to HubSpot, WhatsApp, and Google Ads

## Running the landing page

Just open `task2-landing-page/index.html` in a browser no server or build step needed. To see the tracking fire, open the console, fill in the form, and submit. The `consultation_form_submitted` push will show up in `window.dataLayer`.

## PageSpeed
<img width="1889" height="901" alt="image" src="https://github.com/user-attachments/assets/a2077c2a-9e4f-4f9d-a2be-a244228931ee" />

Once this is deployed somewhere (Vercel/Netlify), I'll run it through PageSpeed Insights on mobile and drop the screenshot in this folder. The page has no external fonts, images, or libraries, so it should clear 90+ without much extra work.  
