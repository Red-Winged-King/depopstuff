# Quick Pickup · iPhone / Android USPS Carrier Pickup

A phone-friendly, installable web app to request the **free USPS Ground Advantage carrier pickup**, with a saved address and pickup instructions, package count, combined weight in pounds, confirmation number, and recent history on your device.

**Important:** This is not a scanner. A request does not mark packages accepted; your carrier still needs to scan at pickup. This app is not affiliated with USPS or Depop.

## Easy setup: Cloudflare Worker (no laptop or terminal required)

The **entire app is in `worker.js`**, so you can paste it into a Worker. GitHub Pages alone cannot safely schedule pickups because USPS requires a private OAuth client secret.

1. Get USPS Developer Portal credentials with **Carrier Pickup v3** production access: https://developers.usps.com/getting-started. A regular USPS.com account login is not enough.
2. Create a free Cloudflare account at https://dash.cloudflare.com/. Go to **Workers & Pages → Create → Worker**. Open the worker editor, replace the default script with the complete contents of **worker.js**, then **Deploy**. You will receive a private-to-you configuration on a public `*.workers.dev` URL.
3. In your Worker's **Settings → Variables and Secrets**, add these three variables, **Type: Secret**:
   - `USPS_CLIENT_ID` — USPS OAuth Consumer Key (private)
   - `USPS_CLIENT_SECRET` — USPS OAuth Consumer Secret (private)
   - `APP_ACCESS_KEY` — create a long random access phrase, **at least 16 characters**. You will enter this in the phone app; do NOT put it into GitHub.
4. Optionally, add `USPS_PACKAGE_TYPE` as a plaintext variable **only if USPS says the default `USPS_GROUND_ADVANTAGE` value is not accepted for your Carrier Pickup API version**. Confirm the supported Ground Advantage enum with the USPS API team's current OpenAPI specification. This remains a known integration dependency until live verification.
5. Open the Worker URL in **iPhone Safari**. Enter pickup name, address, contact, pickup spot, instructions and the `APP_ACCESS_KEY` from step 3; tap **Save setup**. The app stores these locally on that phone only.
6. In Safari, tap **Share → Add to Home Screen → Add**. You now have a home-screen app. Open it, enter package count and total weight in pounds, and submit. The actual confirmation appears only if USPS accepts the pickup request.

The default pickup instructions are:

> Please scan each package using Prepaid Acceptance when picking them up so tracking updates promptly. Thank you!

You can edit that message before scheduling. USPS does not guarantee drivers will follow the request.

## Security and behavior

- **Do not put your address, USPS Client Secret, or APP_ACCESS_KEY in a public GitHub repo.** The public source code contains none of these.
- The Worker keeps USPS OAuth credentials on the server as Cloudflare **Secrets**, rather than exposed to the browser. The browser only stores name/address/contact/key locally on your phone.
- Protect your phone and avoid sharing its web app access key. A lost phone could expose the locally stored pickup information and app access key.
- The app asks before performing an actual booking and prevents a second booking from the same phone on the same Eastern calendar date. This is a **basic** client-side duplicate guard, **not** a global booking guarantee. Before retrying after timeouts, check your USPS confirmation email/account first.
- The date is calculated from Eastern/Central time rules and submitted with `nextAvailablePickup=true` so USPS can move it forward for unavailable dates such as holidays. USPS is the source of truth for confirmed dates.
- USPS's documented earliest same-day cutoff is **2:00 a.m. Central Time**; otherwise the app requests a subsequent day.
- For free carrier pickup, packages must be prepaid and ready to go. An individual package cannot exceed 70 lb.
- **No live USPS reservations were made or verified in this build.** Test with your approved USPS API credentials and actual eligible address.

## Local test (optional)

`node --test test_worker.mjs`

## Files

- `worker.js` — deploy this single file to Cloudflare, complete HTML + server logic
- `index.html` — a readable copy of the phone UI. Browsing this standalone file does **not** schedule pickups; the Worker is required.
- `worker.core.js` — readable server-side source; `worker.js` embeds `index.html` at the top.
- `test_worker.mjs` — mocked tests, no USPS calls.
- `wrangler.toml` — optional config for developers; you can deploy through the Cloudflare dashboard without it.

### Official references
- USPS Carrier Pickup v3 API: https://developers.usps.com/carrierpickupv3
- USPS OAuth API: https://developers.usps.com/Oauth
- Cloudflare Worker dashboard: https://developers.cloudflare.com/workers/get-started/dashboard/
