# Deepen Diamonds

Diamond inventory & catalog platform for Deepen Group — replacing the existing
`deependiamonds.com/Stock` web tool with a mobile-first customer app, a storefront
kiosk display, an admin panel, and WhatsApp catalog sharing.

## What's here today

This is the **design phase**. The repo currently contains:

```
deepen-diamonds/
├── PLAN.md                 # Full architecture & product plan
├── brand/                  # Deepen Group logo + brand kit (colors, usage)
│   └── BRAND.md
└── design/mockups/
    └── index.html          # Interactive mobile UI mockups (open in a browser)
```

**View the mockups:** open `design/mockups/index.html` in any browser. It shows the
customer-app screens in a phone frame with a light/dark theme toggle:

- Home / landing (news · events · announcements · new arrivals)
- Login (phone OTP)
- Results — **4 view modes**: card · compact card · dense list · select-mode (batch hold)
- Filter (bottom sheet)
- Stone detail
- My Holds

## Scope

| Piece | Tech | Purpose |
|---|---|---|
| Customer app | Flutter (PWA + Android + iOS) | Filter diamonds → view list → hold. Also kiosk mode. |
| Admin panel | React | CSV inventory upload, customer approval, holds, catalog share. |
| Backend | Node + PostgreSQL | API, auth, notifications, deep links. |

Payments/orders are intentionally **out of scope** — after a customer holds a stone,
everything is manual / direct communication.

See [PLAN.md](./PLAN.md) for the full architecture.

## Brand

Primary blue `#1267E8`. Signature mark: the diamond in the "p" of the **deepen** logo.
Full kit in [brand/BRAND.md](./brand/BRAND.md).
