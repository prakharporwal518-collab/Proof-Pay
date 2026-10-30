# ProofPay

**AI-verified payments on PayPal.** Pay only for work that is actually delivered.

ProofPay locks a payment with PayPal's authorize-and-capture flow and releases it only when an AI verifier confirms that the uploaded proof meets the agreed conditions. If the proof falls short, the receiver gets clear reasons. If the deal is rejected or expires, the authorization is voided and the payer is never charged.

Built for the **PayPal AI Hackathon 2026**.

🎥 **Demo video:** [Watch on YouTube](https://youtube.com/your-demo-link)
🌐 **Live app:** [proofpay.vercel.app](https://your-live-link)

---

## The problem

Online deals between strangers run on blind trust. Clients fear paying before work is done; freelancers fear doing the work and never getting paid. Donors can't see whether their money reached its goal. Today the only fix is a slow, manual dispute after something goes wrong.

## How ProofPay works

1. **Create a deal.** The payer describes what they want in plain language, e.g. *"5 logo concepts in PNG, brand colours blue and white."*
2. **AI builds a checklist.** The brief becomes 3–6 testable conditions that the payer confirms.
3. **Funds are locked.** The payer approves a PayPal payment with `intent: AUTHORIZE`. Money is held, not taken.
4. **Proof is uploaded.** The receiver submits images, PDFs or a link.
5. **AI verifies.** Each condition gets a pass/fail result, a confidence score and a reason.
6. **Money moves.**
   - All conditions pass with high confidence → payment is **captured** automatically.
   - Any condition fails → **Needs changes**, with reasons; the receiver can resubmit.
   - AI is unsure → the **payer reviews** and approves or rejects.
   - Rejected or expired → authorization is **voided**; the payer is not charged.

## Features

- 📝 **Plain-language deals**: no forms full of fields; just describe the job
- 🤖 **AI condition parser**: turns a brief into a confirmable checklist
- 🔒 **Real payment hold**: PayPal Orders v2 authorize → capture / void
- 🖼️ **Vision-based proof verification**: checks images and PDFs against each condition
- ✅ **Explainable verdicts**: pass/fail, confidence and a reason for every condition
- 🧑‍⚖️ **Human override**: low-confidence verdicts always go to the payer
- 🔁 **Resubmission flow**: receivers fix and re-upload without starting over
- 🕒 **Deal timeline**: every event from creation to release, visible to both sides
- 🛡️ **Tamper resistance**: rule checks before AI, proof-file hashing, and prompt-injection-safe prompts

### Use cases

| Who | Pays for | Proof |
| --- | --- | --- |
| Clients hiring freelancers | Logos, websites, content | Files, PDFs, live links |
| Donors funding NGOs | Books, food kits, repairs | Photos, bills |
| Parents | Pocket money for tasks | Photo of completed homework |
| Open-source maintainers | Bug bounties | Merged GitHub PR |

## Tech stack

| Layer | Technology |
| --- | --- |
| Frontend | React, Vite, Tailwind CSS, PayPal JS SDK |
| Backend | Python, FastAPI |
| Payments | PayPal Orders v2 + Payments v2 REST APIs (Sandbox) |
| AI | Vision-capable LLM API |
| PDF parsing | pdfplumber |
| Database | PostgreSQL (Supabase) |
| File storage | Supabase Storage |
| Hosting | Vercel (frontend), Render (backend) |

## Architecture

```
React app ──REST──► FastAPI backend ──► PayPal REST API (authorize / capture / void)
    │                     ├──────────► Vision LLM (parse brief, verify proof)
    │                     ├──────────► PostgreSQL (deals, conditions, proofs)
    │                     └──────────► File storage (proof images and PDFs)
    └── PayPal JS SDK popup (buyer approves payment)
```

The PayPal client secret and AI API keys live only on the backend.

## PayPal integration

| Step | Endpoint |
| --- | --- |
| Create order | `POST /v2/checkout/orders` with `intent: AUTHORIZE` |
| Authorize after buyer approval | `POST /v2/checkout/orders/{order_id}/authorize` |
| Release payment | `POST /v2/payments/authorizations/{authorization_id}/capture` |
| Cancel / reject | `POST /v2/payments/authorizations/{authorization_id}/void` |
| Refund after capture | `POST /v2/payments/captures/{capture_id}/refund` |

Webhooks handled: `PAYMENT.AUTHORIZATION.CREATED`, `PAYMENT.AUTHORIZATION.VOIDED`, `PAYMENT.CAPTURE.COMPLETED`, `PAYMENT.CAPTURE.REFUNDED`.

## Project structure

```
proofpay/
├── frontend/          # React + Vite app
│   ├── src/
│   └── .env           # not committed
├── backend/           # FastAPI app
│   ├── app/
│   │   ├── main.py
│   │   ├── paypal.py
│   │   ├── verifier.py
│   │   └── models.py
│   ├── requirements.txt
│   └── .env           # not committed
└── README.md
```

## Setup

### Prerequisites

- Node.js 18+
- Python 3.10+
- A [PayPal Developer](https://developer.paypal.com/) account
- An API key for a vision-capable LLM
- A PostgreSQL database (a free Supabase project works)

### 1. Clone the repo

```bash
git clone https://github.com/prakharporwal518-collab/proofpay.git
cd proofpay
```

### 2. Create PayPal Sandbox credentials

1. Log in to the PayPal Developer Dashboard.
2. Go to **Apps & Credentials → Sandbox → Create App**.
3. Copy the **Client ID** and **Secret**.
4. Under **Sandbox → Accounts**, note the default **Personal** (buyer) and **Business** (receiver) test accounts.

### 3. Backend

```bash
cd backend
python -m venv venv
source venv/bin/activate        # Windows: venv\Scripts\activate
pip install -r requirements.txt
```

Create `backend/.env`:

```env
PAYPAL_CLIENT_ID=your_sandbox_client_id
PAYPAL_CLIENT_SECRET=your_sandbox_secret
PAYPAL_API_BASE=https://api-m.sandbox.paypal.com
PAYPAL_WEBHOOK_ID=your_webhook_id
LLM_API_KEY=your_llm_api_key
DATABASE_URL=postgresql://user:password@host:5432/proofpay
STORAGE_URL=your_storage_url
STORAGE_KEY=your_storage_key
FRONTEND_URL=http://localhost:5173
```

Run it:

```bash
uvicorn app.main:app --reload
```

The API runs at `http://localhost:8000` (interactive docs at `/docs`).

### 4. Frontend

```bash
cd ../frontend
npm install
```

Create `frontend/.env`:

```env
VITE_API_URL=http://localhost:8000
VITE_PAYPAL_CLIENT_ID=your_sandbox_client_id
```

Run it:

```bash
npm run dev
```

Open `http://localhost:5173`.

## Sandbox test credentials (for judges)

The live demo runs entirely on **PayPal Sandbox**. No real money is used.

| Role | Email | Password |
| --- | --- | --- |
| Payer (buyer) | `buyer-xxxx@personal.example.com` | `shared on Devpost` |
| Receiver (business) | `receiver-xxxx@business.example.com` | `shared on Devpost` |

**Try it:**

1. Open the live app and create a deal with a sample brief.
2. Click the PayPal button and log in with the **buyer** Sandbox account.
3. Upload a sample proof from `/samples` (one passing set, one failing set).
4. Watch the AI verdict, then check the capture in the receiver's Sandbox account.

> ⚠️ Never commit real or Sandbox secrets to this repo. Test account passwords are shared only in the Devpost submission.

## Demo

- 🎥 Video walkthrough: [YouTube link](https://youtube.com/your-demo-link)
- 🌐 Live app: [proofpay.vercel.app](https://your-live-link)
- 📄 Devpost: [ProofPay on Devpost](https://paypalaihackathon.devpost.com/)

## Roadmap

- [ ] Milestone payments (split one deal into stages)
- [ ] Ready templates for freelance, NGO, homework and bounty deals
- [ ] Receiver trust score from past verdicts
- [ ] GitHub integration for automatic bounty verification

## Author

**Prakhar Porwal**, B.Tech CSE, LNCT Bhopal
[GitHub](https://github.com/prakharporwal518-collab) · [LinkedIn](https://linkedin.com/in/prakhar-porwal-a7a117405)

## License

MIT
