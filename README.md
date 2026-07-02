# Namoza Developer Assignment — OrthoNow

## What's here
- `task1-gtm-schema.md` — GTM event schema + booking funnel dataLayer tracking + which event to import into Google Ads
- `task2-landing-page/index.html` — the landing page, single file, just open it in a browser
- `task3-integration-writeup.md` — how the form connects to HubSpot, WhatsApp (Karix), and Google Ads
- `pagespeed-screenshot.png` — add this after running PageSpeed Insights (see below)

## Running the landing page
No build step, no server — just open `task2-landing-page/index.html` in a browser. To check the tracking works, open the console, fill the form, and submit. You'll see the `consultation_form_submitted` event land in `window.dataLayer`.

## PageSpeed screenshot
Deploy the page somewhere (Vercel/Netlify/GitHub Pages all work), run the live URL through PageSpeed Insights on the Mobile tab, and drop the screenshot in this folder. The page doesn't load any fonts, images, or JS libraries, so it should clear 90+ without much effort.
