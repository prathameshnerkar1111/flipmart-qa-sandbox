# FlipMart — E-Commerce QA & Playwright Automation Sandbox

A standalone, zero-dependency Flipkart-inspired e-commerce web application built specifically for manual functional testing and **Playwright + TypeScript** end-to-end test automation practice.

Everything—120 products, styles, client-side state (`localStorage`), Open Shadow DOM widgets, and an isolated payment `<iframe>`—is self-contained in a single HTML file so you can run it locally or deploy it to GitHub Pages in under two minutes.

---

## Key Features

### Core E-Commerce Workflows
* **120-Product Catalog:** 20 distinct products across 6 categories (**Mobiles, Electronics, Fashion, Home, Appliances, Grocery**), including low-stock (`≤ 3`) and out-of-stock (`0`) items for boundary and negative testing.
* **Live Search & Autocomplete:** Real-time product/brand/category suggestions dropdown plus full PLP filtering.
* **Product Listing Page (PLP):** Category chips with live counts, max-price slider, multi-brand checkboxes, minimum rating filter, *Exclude Out of Stock* toggle, 4 sort modes, and 12-per-page pagination.
* **Product Detail Modal (PDP):** Dynamic variant switching (updates price and SKU), **Pincode Delivery Checker**, and persistent **Customer Reviews** submission.
* **Wishlist, Compare & Recently Viewed:** Heart-toggle wishlist, side-by-side comparison modal (enforces a max of 3 products), and automatic recently viewed tracking.
* **Cart, Save for Later & Coupons:** Quantity controls with stock-limit enforcement, Save for Later / Move to Cart, automatic free-delivery rules (₹999+ or Plus VIP), and coupon validation.
* **Multi-Step Checkout:** Account verification, Delivery Address CRUD (with 10-digit phone & 6-digit pincode validation), and 4 payment methods (UPI, No-Cost EMI, COD, and Card Gateway).
* **Orders & Invoice Download:** 4-step order tracker (`Order Confirmed` → `Shipped` → `Out for Delivery` → `Delivered`), cancellation modal with reason dropdown, and real `.txt` tax invoice downloads.

### Playwright & QA Automation Edge Cases
* **Open Shadow DOM:** `<supercoin-widget>` custom web component with interactive daily coin claims.
* **Isolated Payment Iframe (`srcdoc` + `postMessage`):** Credit/Debit card verification runs inside an isolated `<iframe>` and communicates verification status back to the parent checkout flow.
* **Async Auto-Waiting Lab:** 1.5-second simulated warehouse inventory sync in the **QA Lab** view.
* **Persistent QA Control Bar:** Inject artificial network latency (`0ms`, `1500ms`, `3000ms`), toggle **Simulate 500 Error** for negative/retry testing, open the in-app **QA Cheat Sheet**, or reset all sandbox data with one click.

---

## Quick Start

### Run Locally
1. Save the application file as `index.html` (or `flipmart.html`).
2. Double-click the file to open it in any modern browser, or serve it locally:
   ```bash
   npx serve .
   ```

### Deploy Free on GitHub Pages
1. Rename `flipmart.html` to **`index.html`**.
2. Create a new **Public** GitHub repository (e.g., `flipmart-qa-sandbox`).
3. Upload `index.html` and this `README.md` to the root of the `main` branch.
4. Go to **Settings → Pages → Build and deployment**, set **Source** to **Deploy from a branch**, select **`main`** (`/ root`), and click **Save**.
5. Wait 1–2 minutes for your live URL:
   `https://<your-username>.github.io/flipmart-qa-sandbox/`

---

## Test Data & Credentials

### Demo Accounts
| Persona | Email | Password | Behavior / Perks |
| :--- | :--- | :--- | :--- |
| **Standard Shopper** | `shopper@flipmart.in` | `Pass@1234` | Standard delivery (Free ≥ ₹999) |
| **FlipMart Plus VIP ✦** | `plus@flipmart.in` | `Plus@1234` | Free delivery on all orders |
| **Blocked User** | `blocked@flipmart.in` | `Block@123` | Blocked at login (negative test) |

* **2FA OTP Code:** `123456` (enable the *2FA OTP challenge* checkbox in the login modal)

### Coupon Codes
* `SAVE10` — 10% discount on cart subtotal
* `FLAT500` — Flat ₹500 off on orders ≥ ₹2,000

### Delivery Pincodes
* **Serviceable:** `411018`, `400001`, `560001` (any valid 6-digit pincode except `999999`)
* **Unserviceable (Negative Test):** `999999`

---

## Playwright + TypeScript Examples

### 1. Piercing the Open Shadow DOM (`<supercoin-widget>`)
```ts
import { test, expect } from '@playwright/test';

test('claim daily SuperCoins inside Shadow DOM', async ({ page }) => {
  await page.goto('/');
  const widget = page.getByTestId('supercoin-widget');
  await expect(widget.getByTestId('shadow-supercoins-balance')).toContainText('120 Coins');

  await widget.getByTestId('shadow-claim-coins-btn').click();
  await expect(widget.getByTestId('shadow-supercoins-balance')).toContainText('130 Coins');
});
```

### 2. Verifying a Card Inside the Payment Iframe
```ts
test('verify credit card inside isolated payment iframe', async ({ page }) => {
  await page.getByTestId('pay-method-card').check();

  const cardFrame = page.frameLocator('[data-testid="payment-iframe"]');
  await cardFrame.getByTestId('iframe-card-number').fill('4111111111114242');
  await cardFrame.getByTestId('iframe-card-expiry').fill('08/29');
  await cardFrame.getByTestId('iframe-card-cvv').fill('123');
  await cardFrame.getByTestId('iframe-verify-card-btn').click();

  await expect(page.getByTestId('card-verify-status')).toContainText('ending in 4242');
});
```

### 3. Verifying Invoice File Download
```ts
test('download order tax invoice', async ({ page }) => {
  await page.getByTestId('nav-orders').click();

  const [download] = await Promise.all([
    page.waitForEvent('download'),
    page.getByTestId('download-invoice-FM-2026-1001').click(),
  ]);

  expect(download.suggestedFilename()).toBe('FlipMart_Invoice_FM-2026-1001.txt');
});
```

### 4. Testing Chaos / HTTP 500 Error Handling
```ts
test('handles simulated 500 error during login', async ({ page }) => {
  await page.getByTestId('qa-error500-toggle').check();
  await page.getByTestId('auth-open-btn').click();
  await page.getByTestId('login-submit-btn').click();

  await expect(page.getByTestId('auth-error')).toContainText('HTTP 500');
});
```

---

## Key `data-testid` Reference

| Area | `data-testid` Examples |
| :--- | :--- |
| **Header & Nav** | `search-input`, `auth-open-btn`, `logout-btn`, `nav-shop`, `nav-orders`, `nav-wishlist`, `nav-compare`, `nav-cart`, `nav-qalab` |
| **Filters & Sort** | `filter-category-select`, `filter-max-price`, `filter-exclude-oos`, `clear-filters-btn`, `sort-price-asc`, `sort-rating` |
| **Product & PDP** | `product-card-MOB-01`, `quick-add-cart-MOB-01`, `pdp-modal`, `pdp-variant-1`, `pdp-pincode-input`, `pdp-review-form` |
| **Cart & Checkout** | `coupon-input`, `apply-coupon-btn`, `proceed-checkout-btn`, `payment-iframe`, `upi-id-input`, `place-order-btn` |
| **Orders** | `order-card-FM-2026-1001`, `download-invoice-FM-2026-1001`, `cancel-order-FM-2026-1002`, `cancel-reason-select` |
| **QA Control Bar** | `qa-latency-select`, `qa-error500-toggle`, `qa-reset-btn`, `qa-cheatsheet-btn` |

---

## Disclaimer
FlipMart is a **fictional QA training sandbox**. All products, prices, payments, and orders are simulated locally in browser `localStorage` (`flipmart_qa_sandbox_v2`) with no real backend or payment processing.
