---
name: Start and embed a Screenleap screen share
description: >-
  Create a Screenleap screen sharing session server-side, hand the presenter the
  screenShareData for screenleap.js, embed the viewer URL, poll session state,
  and stop the session cleanly.
api: openapi/screenleap-inc-screen-sharing-openapi.yml
operations: [createScreenShare, getScreenShare, stopScreenShare, listRecentScreenShares]
---

# Start and embed a Screenleap screen share

Use the Screenleap Screen Sharing API (base `https://api.screenleap.com/v2`) to add
live screen sharing to your product.

## Authentication
Send both credentials on every request as headers:
`accountid: <your account id>` and `authtoken: <your token>`. Get them from a free
developer account (Account menu -> Developer page). There is no OAuth or scopes.

## Steps

1. **Create the session** — `createScreenShare` (`POST /screen-shares`).
   Optionally pass `externalId` (<= 192 chars) to correlate the session with your
   own record, plus feature toggles such as `videoMode`, `allowChat`,
   `allowRecording`, `title`. The response contains `screenShareData` and a
   `viewerUrl`.

2. **Start the presenter** — on the presenter's page, load
   `https://integration.screenleap.com/screenleap.js` and call
   `screenleap.startSharing('DEFAULT', screenShareData)` with the data from step 1.

3. **Show viewers** — redirect viewers to the returned `viewerUrl` or embed it in
   an iframe.

4. **Track state** — poll `getScreenShare` (`GET /screen-shares/{screenShareCode}`)
   for `isActive`, `totalViewers`, `userMinutes`, `costInCents` and the
   `participants` array. `listRecentScreenShares` (`GET /recent-screen-shares`)
   returns sessions pending, active, or ended in the last 5 minutes. Prefer the
   per-session `apiCallbackUrl` webhook over tight polling.

5. **Stop the session** — `stopScreenShare`
   (`POST /screen-shares/{screenShareCode}/stop`).

## Rules
- Timestamps (`startTime`, `endTime`, `dateCreated`) are epoch milliseconds.
- `isActive` may lag the true state by a few minutes — do not treat it as real-time.
- No idempotency key exists; do not blindly retry `createScreenShare` on timeout, or
  you may open duplicate sessions — reconcile with `listRecentScreenShares` first.
- Errors are plain HTTP: `401` (bad credentials), `403` (not your session),
  `404` (unknown or already-ended share code). See
  `errors/screenleap-inc-problem-types.yml`.
