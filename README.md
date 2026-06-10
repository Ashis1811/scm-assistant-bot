# SCM Assistant — Supply Chain Chatbot

> Trinamix Inc · Junior AI Engineer Hiring Task · Ref: TX-JrAI-003

A RAG-powered supply chain chatbot built in Flowise that answers questions about a supplier network using two data sources: `supplier_performance_data.csv` (2,000 purchase orders, 116 suppliers) and `SupplyChain_Governance_Policy_v3.2.pdf` (10-section governance policy).

---

## 🤖 Live Chatbot URL

**[https://cloud.flowiseai.com/chatbot/](https://cloud.flowiseai.com/chatbot/)**

---

## 🛠 Tech Stack

| Component | Choice |
|-----------|--------|
| **RAG Platform** | Flowise Cloud (flowiseai.com) |
| **LLM** | Groq — `llama-3.1-8b-instant` |
| **Embeddings** | _(HuggingFace Inference Embedding — sentence-transformers/all-MiniLM-L6-v2)_ |
| **Vector Store** | Document Store (Vector) — Flowise built-in |
| **Chain** | Conversational Retrieval QA Chain |
| **Streaming** | Enabled |

---

## 📦 Document Store — Chunk Configurations

Two chunk configurations were tested during the document store setup.

### Configuration A

| Setting | Value |
|---------|-------|
| Chunk Size | 500 tokens |
| Chunk Overlap | 50 tokens |
| Text Splitter | RecursiveCharacterTextSplitter |
| Chunks — CSV | _(2793)_ |
| Chunks — PDF | _(34)_ |

### Configuration B

| Setting | Value |
|---------|-------|
| Chunk Size | 1000 tokens |
| Chunk Overlap | 150 tokens |
| Text Splitter | RecursiveCharacterTextSplitter |
| Chunks — CSV | _(1479)_ |
| Chunks — PDF | _(18)_ |

**Observations:**
- Configuration A produced more chunks with tighter boundaries, which sometimes split a single supplier's row mid-record, causing incomplete answers on aggregation queries.
- Configuration B's larger windows kept related supplier rows together, improving accuracy on questions requiring totals and averages. It was selected for the final deployment.

---

## ✅ Sample Q&A (Verbatim Chatbot Answers)

### Q1: Which Tier-3 suppliers have an active disruption flag, and what response level applies per policy?

> _(According to the provided context, when a supplier has an Active Disruption flag, the following response tiers apply:

Level 1 — Monitor (Low Risk suppliers with disruption flag): Weekly status update required from supplier. BQBYTE Procurement team adjusts safety stock for affected SKU by +15%.
Level 2 — Manage (Medium Risk suppliers with disruption flag): Bi-weekly escalation calls. Safety stock adjusted by +30%. Alternate supplier placed on 48-hour readiness notice.
Level 3 — Activate (High Risk suppliers with any disruption flag, or any supplier with two simultaneous disruption flags): Immediate escalation to CPO. Alternate supplier activated for minimum 40% of volume within 10 business days. Safety stock adjusted by +50%. Full RCA required within 15 business days.
However, the context does not specify which Tier-3 suppliers have an active disruption flag. It only mentions that Level 3 — Activate applies to High Risk suppliers with any disruption flag, or any supplier with two simultaneous disruption flags.)_

---

### Q2: Which suppliers qualify for the annual Volume Rebate Program and how many are there?

> _(According to the provided context, Tier-1 suppliers qualify for the annual Volume Rebate Program. To qualify, a Tier-1 supplier must meet the following criteria:

On-Time Delivery (OTD) rate at or above 93.0%
Defect Rate below 0.5%
Sustainability Score at or above 85 for a full calendar year
The context does not specify the exact number of suppliers that qualify for the Volume Rebate Program. It only mentions that the rebate eligibility is assessed in January of the following year, implying that the number of eligible suppliers may change from year to year.)_

---

### Q3: Which region has the highest total PO value, and does it breach the concentration limit?

> _(To determine the region with the highest total PO value, we need to calculate the total PO value for each region.

The total PO value for each region is as follows:

EMEA: PO-10775 (31598.92 USD)
APAC: PO-11975 (440127.55 USD) + PO-10833 (112630.4 USD) + PO-10024 (155685.58 USD) = 440127.55 + 112630.4 + 155685.58 = 708443.53 USD
Since there is only one PO for the EMEA region, it is not possible to calculate the total PO value for this region. However, we can see that the APAC region has a higher total PO value.

The concentration limit is not specified in the provided context.)_

---

### Q4: Which suppliers are on Supplier Watch List (SWL) status and what does it restrict?

> _(According to the provided context, any supplier with a Compliance Score below 60 at any audit is placed on Supplier Watch List (SWL) status, regardless of tier designation.

The context does not specify the exact suppliers that are currently on SWL status. However, it does mention that SWL status limits new PO issuance to 20% of prior quarter volume.

)_

---

### Q5: Which product category has the highest average defect rate and does it exceed the Tier-2 limit?

> _(To determine the product category with the highest average defect rate, we need to analyze the provided data.

The provided data includes the following information for the product category "Packaging Materials":

Supplier_ID: SUP-107
Product_Category: Packaging Materials
Defect_Rate_Pct: 1.47
Since there is only one data point for the product category "Packaging Materials", we cannot calculate the average defect rate. However, we can see that the defect rate for this product category is 1.47%, which is below the Tier-2 maximum permissible defect rate of 2.50%.

The context does not provide enough information to determine the product category with the highest average defect rate.)_

---

## 💡 What I'd Improve

1. **Upgrade the LLM** — Swap `llama-3.1-8b-instant` for a larger model (e.g. `llama-3.3-70b-versatile` on Groq or Claude Sonnet) for more accurate multi-step reasoning on aggregation queries like regional PO totals.

2. **Pre-aggregate the CSV** — Rather than loading raw CSV rows as text chunks, pre-process the file into per-supplier summary records (total PO value, average OTD, average defect rate). This prevents the LLM from having to "add up" values scattered across dozens of fragmented chunks.

3. **Hybrid search** — Add a BM25 keyword retriever alongside the vector retriever. Exact supplier name lookups (e.g. "Deltaforge Vietnam") are more reliably found with keyword matching than pure semantic similarity.

4. **Metadata filtering** — Tag each chunk at ingestion with `{tier, region, supplier_id, doc_type}` so queries like "Tier-3 suppliers in EMEA" can pre-filter before semantic search, reducing noise in retrieved context.

5. **Tune the response prompt** — The default `"Hmm, I'm not sure"` fallback is too aggressive for a data-heavy use case. A custom prompt that instructs the LLM to reason over partial context and cite policy sections explicitly would produce more useful answers.

6. **Add streaming UI polish** — Stream tokens to the frontend for a more responsive feel, especially for long supplier list answers that currently have a noticeable delay.

---

## 📁 Repository Structure

```
scm-assistant-bot/
├── SCM Assistant AI Chatflow.json   # Exported Flowise chatflow
├── README.md                        # This file
├── .gitignore                       # Excludes .env and API key files
└── screenshots/
    ├── Chatflows_Architecture.png
    ├── Credentials.png
    ├── Document_store.png
    ├── Document_store(After upserted).png
    ├── Embeddings.png
    ├── Final chatflows.png
    └── Public_URL.png
```

---

## 📬 Submission
- **Live Chatbot:** `https://cloud.flowiseai.com/chatbot/f4d4955f-5f63-4903-a6dc-19b8f4673f12`
- **Contact:** Trinamix Talent Acquisition · Ref: TX-JrAI-003
