# Icespot Pulincunnoo — Cloudflare Pages + D1

This project preserves the existing single-page shop design and adds a Cloudflare Pages Functions API backed by D1. Orders are stored centrally, customer Order History retrieves the order records associated with saved order references in that browser, and the admin screen can view all orders and update statuses.

## What is included

- Existing shop/menu design and cart
- Cakes & Desserts and Cakes & Bento inventory categories
- Cake preorder date/time, inscription and special notes
- Server-side order creation with price and availability checked against the database
- WhatsApp message handoff after the order request has been saved
- Central order list and statuses in the admin panel
- Admin sessions issued by the API (12-hour expiry)
- Cloudflare D1 schema migration

**Order status:** creating an order stores a request as `Pending confirmation`. It is not a confirmed booking until the shop confirms it on WhatsApp. No online payment is taken.

## Deploy using GitHub and Cloudflare (no local server needed)

1. Create a new private GitHub repository for the site.
2. Extract this ZIP on your computer. Upload the extracted project files/folders to the repository (drag the extracted contents into GitHub's **Add file → Upload files** page). Do not upload the ZIP itself as the repository source.
3. In Cloudflare Dashboard, open **Workers & Pages → D1 SQL Database → Create database**. Name it `icespot-db` and copy the database ID.
4. In GitHub, open `wrangler.toml`, replace `REPLACE_WITH_YOUR_D1_DATABASE_ID` with the real ID, then commit the change.
5. Open the new D1 database's **Console** and run all SQL from `migrations/0001_initial.sql` once. This creates the menu, settings, orders and sessions tables.
6. In Cloudflare, go to **Workers & Pages → Create → Pages → Connect to Git**. Connect the repository. Set the build command to `exit 0` and the build output directory to `.`. Deploy.
7. Open the Pages project **Settings → Functions → D1 database bindings** (wording can vary). Add a binding with variable name `DB` and select the `icespot-db` database. Save and trigger a new deployment.
8. Under **Settings → Variables and Secrets → Environment variables**, add `ADMIN_USER` and `ADMIN_PASSWORD` for Production. Use a unique username and a strong password; mark the password as a secret. Optional variables `SHOP_WHATSAPP` (digits with country code) and `SHOP_UPI` can also be set. Do not put these secrets in GitHub or in `index.html`.
9. Save and redeploy. Open the generated `*.pages.dev` URL and test the customer menu, then admin login and a test order.

If Cloudflare shows a D1 binding configuration error on the first deployment, finish steps 7–8 and redeploy. Pages Functions run at `/api/*` from `functions/api/[[path]].ts`.

## Important setup and security notes

- Change `SHOP_WHATSAPP` / shop WhatsApp number in the admin settings to the real shop number, including country code. Confirm the UPI ID before taking customer orders.
- The first API request seeds the sample menu items only if the menu table is empty. Replace those sample products/prices with the real inventory in Admin.
- `ADMIN_USER` and `ADMIN_PASSWORD` must be configured in Cloudflare. There are no default production admin credentials.
- Customer Order History uses unguessable order references saved in that browser's local storage to retrieve centrally stored order records. It is not a customer account system and does not sync a customer's history across different devices/browsers. The admin order list is central and shared.
- Do not treat WhatsApp opening as acceptance of an order. The shop must confirm it.
- This package has not been deployed to your Cloudflare account because I do not have access to your GitHub or Cloudflare account. The D1 ID, binding and secrets are account-specific and must be set by you.
