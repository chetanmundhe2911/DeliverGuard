# 📦 DeliverGuard

> A real-time delivery verification demo that detects fraudulent order cancellations using live GPS and automated IVR calls.

---

## 🧩 Problem Statement

Last-mile delivery fraud is a growing problem. Two common scenarios:

**Scenario 1 — Legitimate cancellation:** Delivery boy reaches the customer's address, finds no one home, calls the customer, and the customer requests cancellation.

**Scenario 2 — Fraudulent cancellation:** Delivery boy never visits the address and cancels the order without contacting the customer — pocketing the item or avoiding the trip.

DeliverGuard cross-verifies both scenarios using two automated flags.

---

## ✅ How It Works

### 🛰️ Flag 1 — Geo Verification
When a delivery boy marks an order as cancelled, the system captures his real GPS coordinates via the browser and computes the distance to the customer's delivery address using the **Haversine formula**.

- If distance ≤ threshold (default 100m) → 🟢 **GREEN FLAG** — delivery boy was present
- If distance > threshold → 🔴 **RED FLAG / Fraud Alert** — delivery boy was not at the address

### 📞 Flag 2 — Automated IVR Call
Once cancellation is triggered, the system auto-places a call to the customer via **Twilio** with an IVR menu:

1. *"Did you cancel your order? Press 1 for Yes, Press 2 for No."*
2. *"Would you like to reschedule your delivery? Press 1 for Yes, Press 2 for No."*

The customer's response is logged and the order status updated accordingly.

---

## 🚀 Demo Setup

### Prerequisites
- A modern browser (Chrome recommended) with location permissions enabled
- A [Twilio](https://twilio.com) account (free trial works)

### Twilio Setup

1. Sign up at [twilio.com/try-twilio](https://www.twilio.com/try-twilio) — no credit card needed for trial
2. Go to **Phone Numbers → Verified Caller IDs** and verify both:
   - Delivery boy's number
   - Customer's number
3. Go to **Phone Numbers → Buy a Number** → select **United States** → purchase any Voice-enabled number (~$1.15 from your free credits)
4. From the **Account Dashboard**, copy:
   - Account SID
   - Auth Token
   - Your Twilio phone number

### Running the Demo

This is a single self-contained HTML file — no build step, no dependencies, no server required.

```bash
# Clone the repo
git clone https://github.com/your-username/deliverguard.git
cd deliverguard

# Open directly in browser
open delivery-verify-demo.html
```

Or just double-click `delivery-verify-demo.html` in your file explorer.

### Configuration

Fill in the sidebar fields in the app at runtime — **no credentials are stored in the code**:

| Field | Where to find it |
|---|---|
| Account SID | Twilio Console → Account Dashboard |
| Auth Token | Twilio Console → Account Dashboard → Show |
| Twilio Phone Number | Twilio Console → Phone Numbers → Active Numbers |
| Delivery Boy Number | Your verified caller ID (e.g. `+919XXXXXXXXX`) |
| Customer Number | Your verified caller ID (e.g. `+917XXXXXXXXX`) |

> All fields are left blank in the repo intentionally. Fill them in the browser UI each session — nothing is persisted or sent anywhere except directly to Twilio.

---

## 📌 Pinning the Delivery Address

For maximum accuracy, instead of relying on hardcoded map coordinates:

1. **Stand at the customer's actual delivery address**
2. Click **"📌 Pin Nishant's Address Here"** in the app
3. The system saves your exact GPS as the target — precision to 6 decimal places

This eliminates any map geocoding error.

---

## 🗂️ Project Structure

```
deliverguard/
│
├── delivery-verify-demo.html   # Complete app — single file, zero dependencies
└── README.md                   # This file
```

---

## 🛠️ Tech Stack

| Layer | Technology |
|---|---|
| Frontend | Vanilla HTML, CSS, JavaScript |
| Fonts | Space Grotesk + IBM Plex Mono (Google Fonts) |
| Geolocation | Browser Geolocation API (`navigator.geolocation`) |
| Distance Calc | Haversine formula (pure JS) |
| IVR Calls | Twilio Voice API + TwiML |
| Voice | Amazon Polly — Aditi (Indian English) |

---

## ⚠️ Known Limitations

**CORS on IVR calls:** Browsers block direct API calls to Twilio from a webpage. If the call fails with a network error, use one of these approaches:

**Option A — Open file directly:**
Open the HTML file from your local filesystem (`file://`) rather than a web server — some browsers permit this.

**Option B — Node.js proxy (recommended for production):**
```bash
npm install express twilio cors
```
```js
const express = require('express');
const twilio = require('twilio');
const cors = require('cors');
const app = express();
app.use(cors());
app.use(express.json());

app.post('/call', async (req, res) => {
  const client = twilio(process.env.TWILIO_SID, process.env.TWILIO_TOKEN);
  const call = await client.calls.create({
    to: req.body.to,
    from: process.env.TWILIO_FROM,
    twiml: req.body.twiml
  });
  res.json({ sid: call.sid, status: call.status });
});

app.listen(3000);
```
Then update the fetch URL in the HTML to `http://localhost:3000/call`.

**Option C — Twilio Functions:**
Deploy the call logic as a [Twilio Serverless Function](https://www.twilio.com/docs/serverless/functions-assets/functions) and call that endpoint instead.

---

## 📈 Roadmap / Future Improvements

- [ ] Backend server (Node.js/Express) to resolve CORS and store logs in a database
- [ ] Dashboard showing all cancellation events with geo + IVR outcomes
- [ ] Webhook to capture customer keypress response from Twilio
- [ ] WhatsApp message fallback if call goes unanswered
- [ ] Mobile app wrapper (React Native / PWA) for delivery boys
- [ ] Integration with delivery management systems (Shiprocket, Delhivery API)

---

## 🤝 Built For   

This project was built as a proof-of-concept demo for a startup stakeholder to demonstrate how delivery fraud can be detected and cross-verified in real time using off-the-shelf APIs — with zero backend infrastructure for the demo.

---

## 📄 License

MIT License — free to use, modify, and distribute.
