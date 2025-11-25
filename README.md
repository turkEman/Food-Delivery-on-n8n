# Food Ordering System — n8n Automation Workflow

A simple but complete food-ordering backend pipeline implemented as an n8n automation workflow. A small frontend (HTML/CSS/JS) deployed on Netlify sends order JSON to an n8n webhook. The workflow formats and validates the data in a Code node and stores finalized orders in MongoDB.

---

## Demo / Frontend

Live frontend (Netlify):

[https://task1-n8n.netlify.app/](https://task1-n8n.netlify.app/)

---

## Table of contents

* ✅ Features
* 🚀 Quick start
* 🔧 Workflow & node-by-node explanation
* 🧪 How to test
* 🛠 Technical details & configuration
* 🔮 Suggested improvements
* 📁 Files to include in a portfolio
* 📜 License

---

## ✅ Features

* **Webhook trigger** — n8n receives order JSON from the frontend.
* **Code node** — validates, sanitizes and formats incoming orders.
* **MongoDB insert** — stores finalized orders for later retrieval and processing.
* **End-to-end** — frontend → webhook → processing → database.
* **Simple deployment** — frontend hosted on Netlify for demo purposes.

---

## 🚀 Quick start

### Requirements

* n8n (self-hosted or n8n.cloud)
* MongoDB (Atlas or local)
* Access to the Netlify frontend provided above

### High-level steps

1. Import or recreate the n8n workflow in your n8n instance.
2. Create a MongoDB credential in n8n or configure a MongoDB node with a connection string.
3. Set the Webhook node's endpoint and update the frontend to point to that URL (if using your own n8n host).
4. Trigger orders from the frontend and confirm they appear in MongoDB.

---

## 🔧 Workflow overview

**Data flow:** Frontend → Webhook (n8n trigger) → Code node (format/validate) → MongoDB — Insert Documents

**Visual**: include a screenshot of your n8n workflow in this README or portfolio.

### Node-by-node explanation

1. **Webhook (Trigger)**

   * Listens for `POST` requests from the Netlify frontend.
   * Expects JSON with fields such as: `customer` (name, email), `items` (array), `total`.

2. **Code Node (JavaScript formatter)**

   * Validates required fields.
   * Normalizes and sanitizes items.
   * Generates an `orderId`, `createdAt` timestamp, and sets `status: "pending"`.
   * Returns a database-ready JSON object.

   **Code node snippet (JavaScript):**

```javascript
// n8n Code node (JavaScript)
// Input: items[0].json should contain the incoming payload
const input = items[0].json || {};

function sanitizeString(s) {
  if (!s) return '';
  return String(s).trim();
}

// Basic validation
if (!input.customer || !input.items || !Array.isArray(input.items) || input.items.length === 0) {
  throw new Error('Invalid payload: customer and items are required.');
}

const orderId = `ORD-${Date.now()}`;
const customer = {
  name: sanitizeString(input.customer.name || input.customer),
  email: sanitizeString((input.customer && input.customer.email) || ''),
};

const items = input.items.map(it => ({
  name: sanitizeString(it.name),
  qty: Number(it.qty || it.quantity || 1),
  price: Number(it.price || 0),
}));

const total = Number(input.total || items.reduce((s, i) => s + (i.qty * i.price), 0));

const output = {
  orderId,
  customer,
  items,
  total,
  status: 'pending',
  createdAt: new Date().toISOString(),
};

return [{ json: output }];
```

3. **MongoDB — Insert Documents**

   * Accepts the formatted JSON and inserts it into an `orders` collection.
   * Use MongoDB Atlas or a local instance. Configure the MongoDB node's connection string or credential in n8n.

---

## 🧪 How to test

1. Open the Netlify frontend: [https://task1-n8n.netlify.app/](https://task1-n8n.netlify.app/)
2. Select items and place an order.
3. In n8n, check the webhook execution history to confirm it received the payload.
4. Inspect the Code node output to verify formatting/validation.
5. Confirm a new document appears in your MongoDB `orders` collection.

**Example webhook payload** (what the frontend should POST):

```json
{
  "customer": { "name": "Jane Doe", "email": "jane@example.com" },
  "items": [
    { "name": "Burger", "qty": 2, "price": 5.5 },
    { "name": "Fries", "qty": 1, "price": 2.5 }
  ],
  "total": 13.5
}
```

---

## 🛠 Technical details & recommended environment variables

**Credentials / environment**

* `MONGODB_URI` — full MongoDB connection string (if using environment variables outside n8n)
* `MONGODB_DB` — database name (e.g. `food_orders`)
* `MONGODB_COLLECTION` — collection name (e.g. `orders`)

**n8n configuration tips**

* Create a dedicated credential for MongoDB in n8n and use the MongoDB node's "Insert" operation.
* If using n8n.cloud, use its provided webhook URL; if self-hosted, ensure your instance is reachable from Netlify (or use a tunnel like ngrok for testing).

---

## 🔮 Future Work

1. **Webhook authentication** — require an API key / header signature to prevent unauthorized requests.
2. **Notifications** — send email or SMS to customer/admin on order creation using n8n integrations.
3. **Error handling & retry** — route failed MongoDB inserts to a fallback collection and notify ops.
4. **Schema validation** — use a JSON schema or a stricter validator in the Code node.
5. **Admin dashboard** — build a simple UI that queries MongoDB for order management.
6. **Frontend UX** — add an order confirmation page and display orderId.

---


## 📜 License & author

This project is provided as an example/demo.

---

