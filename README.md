# Bihar Academic Battle — Test Series Store

A plain HTML/CSS/JavaScript store using your existing Firebase Authentication, Firebase Realtime Database, Node/Express backend, and Razorpay Checkout.

## Security model

- The browser never decides whether a purchase is valid.
- The browser never receives `RAZORPAY_KEY_SECRET` or Firebase Admin credentials.
- The server verifies Firebase ID tokens.
- Order amount and currency are taken from the server-side series record, not from the browser.
- Payment signature is verified and the server fetches the order/payment from Razorpay before granting access.
- A Razorpay webhook can also grant access after a captured payment, after validating the webhook signature.
- Client writes to `purchases` and `paymentOrders` are denied by `database.rules.json`.
- Admin APIs require the Firebase Auth UID to be listed in `ADMIN_UIDS`.

**Important:** this package includes catalogue and access-control scaffolding, not the actual exam questions/player. The “Open tests” button checks server access and then displays a placeholder. Connect it to your test player route and enforce `/api/access/:seriesId` on the server before serving test content/questions. Do not publish test questions in a public-readable database node.

## 1. Configure Firebase

1. Open your existing Firebase project `babbihar-23fcc`.
2. In Project settings → General → Your apps, copy the web app config.
3. Edit `public/app.js` and replace `apiKey`, `messagingSenderId`, and `appId`. Keep your existing `authDomain`, `projectId`, and `databaseURL`.
4. Enable Email/Password under Firebase Authentication → Sign-in method.
5. Create a service account JSON for the backend from Firebase Project settings → Service accounts. Keep it private.
6. Apply `database.rules.json` in Realtime Database → Rules. Merge it carefully with your existing rules; do not overwrite rules needed by the rest of your app. The Admin SDK bypasses RTDB rules, so the server must perform all authorization checks.

## 2. Configure Razorpay

1. Start with Razorpay Test Mode credentials.
2. Set `RAZORPAY_KEY_ID` and `RAZORPAY_KEY_SECRET` as server environment variables.
3. In Razorpay dashboard, add a webhook URL: `https://YOUR_BACKEND_DOMAIN/api/razorpay/webhook`.
4. Subscribe to `payment.captured` and `order.paid`; set the webhook secret in `RAZORPAY_WEBHOOK_SECRET`.
5. Use live credentials only after successful end-to-end test payments and your business/account setup is complete.

The browser receives only the Razorpay Key ID, which is designed to be public. Never expose the Key Secret.

## 3. Run locally

Requires Node.js 20+.

```bash
npm install
cp .env.example .env
```

On Windows PowerShell, copy `.env.example` to `.env` manually. Fill the variables. `server.js` reads environment variables but does not auto-load `.env`; for local use, either set them in PowerShell before starting or install `dotenv` and add `import "dotenv/config";` at the very top of `server.js`.

PowerShell example (replace placeholders; keep JSON as a single line):

```powershell
$env:PORT="3000"
$env:FIREBASE_DATABASE_URL="https://babbihar-23fcc-default-rtdb.firebaseio.com"
$env:FIREBASE_SERVICE_ACCOUNT_JSON=Get-Content .\service-account.json -Raw
$env:RAZORPAY_KEY_ID="rzp_test_xxxxx"
$env:RAZORPAY_KEY_SECRET="your_test_secret"
$env:RAZORPAY_WEBHOOK_SECRET="your_webhook_secret"
$env:ADMIN_UIDS="YOUR_FIREBASE_AUTH_UID"
$env:PUBLIC_BASE_URL="http://localhost:3000"
npm start
```

Then open `http://localhost:3000`. Do not open `public/index.html` directly using `file://`, because API calls need the Node server.

## 4. Deployment

Deploy the Node backend to a host that supports persistent Node servers (for example Render, Railway, or your own server). Configure all environment variables in the host dashboard. A static-only Netlify deployment cannot run `server.js` by itself; you would need to deploy the backend separately and configure API URLs/CORS, or convert the API to secure serverless functions.

For Firebase Admin service account environment variable, set the full JSON content as one-line JSON in the hosting provider's secret environment variables. Never commit `.env`, service-account JSON, or secrets to GitHub.

Set `ADMIN_UIDS` to the UID shown in Firebase Authentication for your admin account. Do not use email as the authorization identity. If you have multiple admins, comma-separate their UIDs.

## 5. Add the first series

Sign in with a UID listed in `ADMIN_UIDS`. Open Admin Management and add a series, e.g.:
- Title: Bihar GK Practice Series
- Subject: Bihar GK
- Class: Class 8
- Price: 99
- Number of tests: 10
- Visibility: Published / for sale

## 6. Critical production checklist

- Test payment success, failure, cancellation, duplicate callback, and delayed webhook in Razorpay Test Mode.
- Ensure your Razorpay account is configured for automatic capture, or adapt the server flow to capture authorized payments before granting access.
- Use HTTPS in production.
- Apply and merge database rules; verify that a signed-in client cannot write to `purchases`, `paymentOrders`, or `testSeries`.
- Connect the real test player and verify purchase access on every server request for questions/results.
- Add rate limiting, logging/monitoring, refund/revocation workflow, and a support/contact policy before accepting live payments.
- Keep test questions private. Hiding a button in HTML is not access control.
