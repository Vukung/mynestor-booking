# Nestor Booking Page

A single self-contained `index.html` that powers online booking for **[MyNestor.org](https://www.mynestor.org)** (Nestor Technologies LLC). It's hosted on GitHub Pages and embedded into the Wix site as a "Embed a site" element, because Wix Bookings has no built-in way to show only the Nesters who serve a customer's ZIP/PIN code.

Live embed: https://www.mynestor.org/Nestor-Booking
Standalone: https://vukung.me/mynestor-booking/

## What it does

The page walks a customer through:

1. **Choose a service** - Nestor™ Business, Home, Admin, Digital, Connect, etc. Each card shows the name, tagline, duration and price pulled live from Wix, plus an "i" button with a brief of what's included.
2. **Enter a PIN/postal code** - filters to Nesters who serve that area.
3. **Choose a Nester** - only ones who serve that PIN *and* offer the chosen service.
4. **Pick a date and time** - availability pulled live from Wix Bookings, shown in Eastern Time regardless of the visitor's own time zone.
5. **Enter contact details** - first name, last name, email, phone.

The page then:
- Creates the booking via the Wix Bookings API (anonymous visitor token - no login required).
- Creates a Wix eCommerce checkout for that booking (this is the step that actually **confirms** the booking - Wix leaves a booking as "Incomplete" until its checkout completes, even for $0 pay-in-person services).
- Redirects the visitor to Wix's own hosted checkout page to click **Confirm**.
- Sends the visitor back to a Thank You page on mynestor.org afterward.

## Why it's built this way

- **No custom backend.** Everything runs client-side against Wix's public REST API using an OAuth app with the `anonymous` grant type. There's no server, no hosting cost beyond GitHub Pages, and no secret API key in the page.
- **Wix's confirmation step is unavoidable.** I tried skipping it (create booking only); those bookings stayed "Incomplete" and didn't hold the time slot. The checkout redirect is what makes Wix treat the booking as real.
- **The site is on Harmony**, which doesn't support Velo/backend code, so a "confirm booking automatically" backend function isn't an option here.

## Data source

Nester names, roles, PIN codes and their Wix Bookings `resourceId` live in the **`nestors-data`** Wix CMS collection, not in this file. That collection's **read** permission is set to *Anyone* (write/delete stay Admin-only), because the page needs to read it anonymously. It holds no emails or other sensitive contact info.

## Configuration

All of the settings that are likely to change live at the top of the `<script>` block in `index.html`, in the `CFG` object:

| Key | Purpose |
|---|---|
| `clientId` | Wix OAuth app client ID (safe to be public) |
| `collection` | Wix CMS collection name for the Nester roster |
| `businessTimeZone` / `tzLabel` | Must match the site's actual business time zone in Wix |
| `thankYouUrl` | Where Wix sends visitors after they click Confirm |
| `bookingPageUrl` | Where Wix sends visitors if they leave checkout without confirming |
| `hideServices` | Service names (lowercase) to exclude, e.g. internal test services |
| `serviceInfo` | Per-service "what's included" bullets shown in the info popup |

## Known limitations

- The Wix checkout page only auto-fills the customer's **email**; name and phone must be re-entered there. I haven't found a way around this from the REST API.
- A booking that isn't confirmed at checkout stays "Incomplete" in the Wix calendar and does **not** hold the time slot.
- Each Nester's evening availability window is short relative to a 2-hour service, so one booking can fill most of it.
- The `nestors-data` collection is publicly readable (by design - see Data source above).

## Deploying a change

1. Edit `index.html` (locally or directly on GitHub).
2. Commit to the `main` branch of this repo.
3. Wait ~1–2 minutes for GitHub Pages to redeploy, then hard-refresh the live embed to confirm.

## Testing checklist

- [ ] Enter a known-good PIN → correct Nesters appear
- [ ] Enter an unserved PIN → friendly "we're growing" message, not an error
- [ ] Pick a date/Nester with no openings → suggests trying another date or Nester
- [ ] Complete a booking through Confirm → shows Confirmed with an order number in the Wix Bookings calendar, on the Thank You page
- [ ] Abandon checkout before confirming → booking shows "Incomplete" and the slot is still bookable by someone else
