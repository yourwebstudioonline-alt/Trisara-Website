# Trisara — Next.js Site

A Next.js 15 (App Router) rebuild of the Trisara wedding & events venue site.
JSX (not TSX), Tailwind CSS, lucide-react icons.

## Getting started

```bash
npm install
npm run dev
```

Open http://localhost:3000.

## Project structure

```
app/                  Routes (App Router)
  layout.jsx           Root layout: fonts, global <head> metadata, LocalBusiness schema, Nav/Footer
  page.jsx              Home (hero, about, spaces, stories, CTA, offer popup)
  events/page.jsx        "The Events We Do"
  gallery/page.jsx        Filterable photo gallery (reads ?filter= from the URL)
  booking/page.jsx        Multi-step booking form
  sitemap.js               -> /sitemap.xml (auto-generated)
  robots.js                 -> /robots.txt (auto-generated)

components/
  Nav.jsx, Footer.jsx, PageHero.jsx, CtaBanner.jsx, Reveal.jsx, JsonLd.jsx
  home/                 Hero, StatStrip, About, SpacesGrid, OfferPopup,
                        TestimonialCarousel, StoryGrid (the old Stories page,
                        now embedded directly on the home page)
  events/               EventsList, EventRow
  gallery/              GalleryGrid (masonry + filter), GalleryFilter, Lightbox
  booking/               BookingForm (orchestrator), BookingSidebar (the
                        numbered step list), one component per step,
                        DatePicker (custom calendar), ContactPanel

lib/
  site-config.js        ⭐ Business name, address, phone, WhatsApp number,
                        hours, domain — edit this once and it flows into the
                        footer, metadata, floating contact buttons, and the
                        LocalBusiness JSON-LD schema.
  data.js                Events, spaces, gallery images, testimonials — edit
                        copy/content here without touching component code.
  schema.js               Builds the LocalBusiness JSON-LD from site-config.
  icon-map.js             Maps icon name strings (used in data.js) to
                        lucide-react components.
  mongodb.js               Cached Mongoose connection helper.

models/
  Booking.js              Mongoose schema for booking form submissions.
  Lead.js                  Mongoose schema for the 20%-off popup leads.

app/api/
  booking/route.js         POST — saves a booking to MongoDB.
  offer/route.js            POST — saves an offer-popup lead to MongoDB.
```

## Font

The whole site uses a single simple sans-serif font (**Inter**, via
`next/font/google`) — no more decorative serif/italic styling. Both
`font-display` and `font-body` Tailwind utilities point to the same Inter
variable, so existing className usage across components didn't need to
change, but if you want a different font, swap it in one place:
`app/layout.jsx` (the `Inter(...)` call) and `tailwind.config.js`
(`fontFamily.display` / `fontFamily.body`).

## MongoDB

Booking form submissions and offer-popup leads are saved to MongoDB via
Mongoose.

1. Copy `.env.local.example` to `.env.local` and fill in your connection
   string:
   ```
   MONGODB_URI=mongodb+srv://<user>:<password>@<cluster>.mongodb.net/trisara
   ```
   (`.env.local` is already git-ignored — you're filling this in yourself, as
   discussed.)
2. That's it — `POST /api/booking` and `POST /api/offer` will connect and
   write documents automatically. The connection is cached (`lib/mongodb.js`)
   so it isn't re-opened on every request.
3. Collections created: `bookings` and `leads` (pluralized automatically by
   Mongoose from the `Booking` / `Lead` model names). Each document has a
   `status` field (`new` by default) so you can track follow-ups, and
   `createdAt`/`updatedAt` timestamps.

If `MONGODB_URI` isn't set, the API routes return a clear error instead of
crashing the whole app — the rest of the site keeps working.

## Floating Call / WhatsApp buttons

`components/FloatingContact.jsx` renders two fixed buttons in the bottom-right
corner on every page:

- **Call** → `tel:` link using `siteConfig.phoneHref`, dials the business.
- **WhatsApp** → opens `wa.me` with `siteConfig.whatsappNumber`, pre-filled
  with: *"Hi! I just saw your venue and it looks amazing 😍 Can I get more
  details on this?"* — edit the `WHATSAPP_MESSAGE` constant at the top of
  that file to change the wording.

Update `phoneHref` and `whatsappNumber` in `lib/site-config.js` with your real
numbers before launch.


## Images

All images live in `public/images/...` and are referenced as plain paths
(e.g. `/images/hero.jpg`), no CDN. **The placeholder images currently in this
folder are generated solid-color placeholders** — swap them out for real
photography using the exact same filenames and everything (including the
gallery's aspect-ratio-aware masonry layout) will pick them up automatically:

```
public/images/
  hero.jpg, about.jpg, events-hero.jpg, gallery-hero.jpg, booking-hero.jpg, og-image.jpg
  spaces/hall.jpg, lawn.jpg, stage.jpg
  events/wedding.jpg, engagement.jpg, sangeet.jpg, corporate.jpg, birthday.jpg, babyshower.jpg
  stories/couple1.jpg, couple2.jpg, couple3.jpg
  gallery/g1.jpg … g12.jpg
```

To add more gallery photos: add the file under `public/images/gallery/`, then
add an entry to the `galleryImages` array in `lib/data.js` with its real
pixel width/height and a `category` of `hall`, `lawn`, `stage`, or `decor`.
The masonry grid (CSS columns) adjusts automatically for portrait or
landscape images — no manual layout tweaking needed.

## Editing content

- **Copy/text for events, gallery, testimonials, spaces:** `lib/data.js`
- **Business name, address, phone, opening hours, domain:** `lib/site-config.js`
  — this single file drives the footer, page metadata, and the
  `LocalBusiness` schema.org JSON-LD (including `openingHoursSpecification`).
- **Booking form steps:** each step is its own component in
  `components/booking/`, so you can edit fields/copy per step in isolation.

## SEO

- Per-page `metadata` exports (title, description, canonical, Open Graph, Twitter card).
- `/sitemap.xml` and `/robots.txt` are generated automatically via Next.js's
  `app/sitemap.js` / `app/robots.js` conventions — no static XML file to
  maintain by hand.
- `LocalBusiness` JSON-LD (with address, geo coordinates, phone, and opening
  hours) is injected once in the root layout.

**Before launch:** update `siteConfig.url` in `lib/site-config.js` to your
real production domain — the sitemap, robots.txt, canonical tags, and schema
`@id`/`url` fields all derive from it.

## Notes

- Booking and offer-popup submissions now write to MongoDB (see the MongoDB
  section above) — no more `// TODO` stubs.
- Phone number, WhatsApp number, and address in `lib/site-config.js` are
  still placeholders — update with the real values.

## Admin lead management

The site now includes a protected admin portal at `/admin` with unified Booking, Enquiry and Offer leads. Admins can search by name/phone, filter by lead type/status/date range, and move leads through `new`, `qualified`, `unqualified`, `spam`, and `booked`. A dedicated Booked Leads tab is included.

Add these environment variables in production:

Deploy v4 branch
META_ACCESS_TOKEN=your-meta-conversions-api-access-token
```

Status changes are sent server-side to the Meta Pixel through the Meta Conversions API, with the selected status in `custom_data.lead_status`. Browser-side lead submissions continue to fire the existing Lead pixel event. The forms also persist `_fbp`/`_fbc` attribution cookies so Meta can better match CRM status feedback to the original ad click.
