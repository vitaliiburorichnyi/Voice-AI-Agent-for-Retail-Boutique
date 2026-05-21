# 🎙️ Voice AI Agent for Retail — Boutique

> Autonomous voice assistant that handles inbound customer calls for a clothing store — identifies callers, checks order status, updates delivery addresses, and logs every call automatically.

---

## 📌 Overview

Built a production-ready Voice AI Agent using **Happ.tools + n8n + Google Sheets**.

The agent replaces a human operator for routine inbound calls:
- Greets customers by name using caller phone lookup
- Provides real-time order status
- Updates delivery address on request
- Logs full call summary after every conversation

**Result:** 100% call success rate across 10 test scenarios. Zero human intervention for typical customer requests.

---

## 🏗️ Architecture

```
Incoming Call (SIP/Zadarma)
        │
        ▼
Happ.tools Voice Agent (Anika)
        │
        ├── [BEFORE CALL] Init Tool
        │       POST /webhook/boutique-init
        │       → n8n looks up caller phone in Google Sheets
        │       → Returns: customer_name, order_id, order_status, delivery_address
        │       → Agent greets customer by name with order details
        │
        ├── [DURING CALL] webhook_action: change_address
        │       POST /webhook/boutique-change-address
        │       → n8n updates delivery_address row in Google Sheets
        │       → Agent confirms new address to customer
        │
        └── [AFTER CALL] Postcall Tool
                POST /webhook/boutique-postcall
                → n8n extracts: phone, customer_name, order_id, summary, outcome
                → Appends new row to Call Log sheet
```

---

## 🛠️ Tech Stack

| Layer | Tool |
|---|---|
| Voice AI Platform | Happ.tools |
| Workflow Automation | n8n (self-hosted) |
| Data Layer | Google Sheets |
| Telephony | SIP / Zadarma |
| Webhook tunneling (dev) | ngrok |

---

## 📊 Google Sheets Structure

**Sheet 1: Orders** — customer lookup database
| phone | customer_name | order_id | order_status | delivery_address |
|---|---|---|---|---|
| 380991234567 | Олена | #1042 | В дорозі | Київ, Хрещатик 1 |

**Sheet 2: Call Log** — auto-populated after every call
| timestamp | phone | customer_name | order_id | summary | outcome |
|---|---|---|---|---|---|

---

## 🔄 n8n Workflows

### Workflow 1 — Init Tool
```
Webhook → Google Sheets (lookup by phone) → IF (customer found?)
  ├── TRUE  → Respond with customer data
  └── FALSE → Respond with empty JSON (unknown caller fallback)
```

### Workflow 2 — Change Address
```
Webhook → Google Sheets (update delivery_address row) → Respond with confirmation
```

### Workflow 3 — Postcall
```
Webhook → Code node (extract customer_name, order_id from summary) 
        → Google Sheets (append row to Call Log)
```

---

## 🧪 Test Results

| # | Scenario | Result |
|---|---|---|
| 1 | Known customer asks order status | ✅ Greeted by name, correct status |
| 2 | Known customer changes address | ✅ Old address confirmed, new address saved |
| 3 | Unknown number calls | ✅ Generic greeting, no crash |
| 4 | Customer asks delivery cost | ✅ Correct answer from Knowledge Base |
| 5 | Customer asks return policy | ✅ Correct answer from Knowledge Base |
| 6 | Angry customer | ✅ Calm response, human handoff offered |
| 7 | Out-of-scope question | ✅ Redirected to manager |
| 8 | Order not found | ✅ Informed customer, offered manager |
| 9 | Customer asks about sizes | ✅ Correct answer from Knowledge Base |
| 10 | Normal call ends | ✅ Postcall fired, Call Log updated |

---

## 📈 Metrics

| Metric | Value |
|---|---|
| Call Success Rate | 100% |
| Human Handoff Rate | 30% (edge cases only) |
| Knowledge Base Accuracy | 100% |
| Init Tool Success Rate | 100% |
| Postcall Logging Rate | 100% |
| Avg Call Duration | ~60 sec |

---

## 🔑 Key Challenges Solved

**1. Phone number format mismatch**
Google Sheets drops `+` from phone numbers. Fixed by stripping `+` in n8n filter expression before lookup.

**2. Unknown caller crashing Init Tool**
When phone not found, Google Sheets returned empty and n8n stopped. Fixed with `Always Output Data` + IF node that returns empty JSON fallback.

**3. Postcall field mapping**
Happ.tools sends call data in nested JSON structure. Built a Code node in n8n to extract `customer_name` and `order_id` from transcript summary using regex.

**4. Variable injection in prompts**
Happ.tools uses `{{variable}}` syntax for dynamic variables. Initial `[variable]` syntax caused agent to read placeholder text literally.

---

## 🚀 How to Replicate

1. Create Google Sheets with `Orders` and `Call Log` tabs
2. Set up n8n self-hosted instance
3. Import the 3 workflow JSONs (see `/workflows` folder)
4. Connect Google Sheets credentials in n8n
5. Create Happ.tools agent with the System Prompt (see `/prompts` folder)
6. Configure 3 Tools in Happ.tools pointing to your n8n webhook URLs
7. Connect SIP number in Happ.tools Phone Numbers tab
8. Test with Postman before going live

---

## 📁 Repository Structure

```
├── workflows/
│   ├── init-tool.json
│   ├── change-address.json
│   └── postcall.json
├── prompts/
│   └── system-prompt.md
├── knowledge-base/
│   └── boutique_knowledge_base.md
└── README.md
```

---

## 💡 Next Steps

- [ ] Replace Google Sheets with KeyCRM API
- [ ] Add SMS confirmation after address change
- [ ] Implement outbound calls for order confirmations
- [ ] Add multilingual support (UA/RU/EN)
- [ ] Reduce Init Tool latency from 3s to <1s

---
