# 🏨 WSP Concierge — Automated WhatsApp Hospitality & Property Management AI Engine

<div align="center">

[![n8n](https://img.shields.io/badge/Orchestrator-n8n--as--code%20(154%20Nodes)-ea4b71?style=for-the-badge&logo=n8n)](https://n8n.io)
[![WhatsApp](https://img.shields.io/badge/Messaging-Meta%20WhatsApp%20Cloud%20API-25d366?style=for-the-badge&logo=whatsapp)](https://developers.facebook.com/docs/whatsapp)
[![Hospitable](https://img.shields.io/badge/PMS%20Integration-Hospitable.com%20API-4a90e2?style=for-the-badge)](https://hospitable.com)
[![OpenAI](https://img.shields.io/badge/Cognitive%20AI-GPT--4o%20%2B%20Embeddings-412991?style=for-the-badge&logo=openai)](https://openai.com)
[![Qdrant](https://img.shields.io/badge/Vector%20Search-Qdrant%20RAG-dc2626?style=for-the-badge&logo=qdrant)](https://qdrant.tech)
[![Redis](https://img.shields.io/badge/Memory%20%26%20Locks-Redis%207-dc382d?style=for-the-badge&logo=redis)](https://redis.io)
[![HubSpot](https://img.shields.io/badge/CRM-HubSpot%20Sync-ff7a59?style=for-the-badge&logo=hubspot)](https://hubspot.com)
[![License](https://img.shields.io/badge/License-MIT-orange?style=for-the-badge)](LICENSE)

**An autonomous 24/7 AI Concierge and Property Management platform built with n8n, Meta WhatsApp Cloud API, and Hospitable.com. Manages traveler reservations, real-time availability quotes, midnight arrivals, payment verification, and property owner onboarding in Algerian Darija, French, and Arabic.**

[Architecture](#-system-architecture) • [Key Capabilities](#-key-capabilities) • [The 7 Sub-Workflows](#-the-7-sub-workflow-modules) • [Intent Disambiguation](#-intent-disambiguation--darija-nlp) • [Quick Start](#-quick-start) • [Environment Setup](#-environment-variables)

</div>

---

## 📖 Overview

Managing premium short-term rental properties (Airbnb, Booking.com, direct bookings) demands immediate responsiveness: travelers expect instant photo galleries, live pricing, and seamless check-in instructions at all hours of the night, while prospective property owners need prompt qualification and revenue estimates.

**WSP Concierge** is an end-to-end, multi-agent automation platform designed specifically for hospitality concierge agencies:
- **For Guests (Voyageurs)**: Instantly queries **Hospitable.com** for live calendar availability, generates dynamic price quotes, returns curated photo carousels, and assists late-night travelers (*"Voyageur Minuit"*) with automated lockbox codes and Wi-Fi instructions.
- **For Property Owners (Propriétaires)**: Qualifies new landlords seeking property management, gathers apartment specifications, and automatically triggers **Gamma AI** to synthesize branded presentation pitch decks.
- **Cognitive Dialect Engine**: Fine-tuned for **Algerian Darija**, distinguishing subtle ambiguities (e.g. differentiating between a guest wanting to sleep somewhere versus a host offering their flat) without human intervention.

---

## 💬 Live WhatsApp Automation & Concierge Experience

<div align="center">

| Guest Interaction (WhatsApp) | PMS & Backend Execution (n8n + Hospitable) |
|:-----------------------------|:-------------------------------------------|
| **Traveler**: *"Salam Kho rani 7ab nekri partma lghedoua f Hydra pour 3 nuits"* | ⚡ `AiRouteur1` detects `voyageur_potentiel` via temporal marker `lghedoua`. Queries Hospitable calendar for Hydra listings. |
| **Bot**: *"Wa alaykoum salam ! 'Le Panoramique Hydra' est disponible. 💵 24,000 DZD (3 nuits). Voici les photos 👇"* | 📸 `WaImageVoyageur` pulls and dispatches CDN property photo album directly into WhatsApp thread. |
| **Traveler (01:30 AM)**: *"Salam lha9t l'immeuble habit na3ref code boîte à clés stp"* | 🌙 **Midnight Traveler Engine** triggers: verifies confirmed booking, retrieves lockbox code & Wi-Fi password. |
| **Bot**: *"Bienvenue à Alger ! 🔑 Porte: 1489A / 📦 Boîte à clés: 7291 / 📶 Wifi: Residence_5G"* | 🔒 Redis locks session state and sends instant WhatsApp confirmation to agency on-call staff. |

</div>

---

## 🏛️ System Architecture & Subsystems

| Subsystem | Components | Function & Output |
|:----------|:-----------|:------------------|
| **Master Orchestrator** | `wsp_main.workflow.ts` (154 nodes) | Meta Graph Webhook verification, phone sanitizer, LangChain OpenAI GPT-4o agent execution, and photo delivery. |
| **PMS Synchronization** | `wsp_hospitable` & `wsp_dispo` | Real-time calendar availability lookup, multi-listing quotes in DZD/EUR, and proactive T+6h/T+12h payment reminders. |
| **Midnight Concierge** | `wsp_voyageur_minuit` | Scheduled property status audit and automated late-night check-in delivery (keys, Wi-Fi, directions). |
| **Payment Verification** | `wsp_verifier_paiement` | Validates customer payment receipts, updates booking status, and clears admin verification gates. |
| **RAG Knowledge Base** | `wsp_recherche_catalogue` + Qdrant | Vector similarity search answering guest questions on building amenities, parking, and house rules. |
| **CRM & Business Sync** | `wsp_crm_export` + HubSpot | Automatically logs guest contact records, booking values, and host onboarding leads to HubSpot & Google Sheets. |

---

## ✨ Key Capabilities

- 🤖 **154-Node Master Orchestrator**: High-availability n8n workflow coordinating 4 distinct AI agents, tools, and error handlers.
- 📅 **Direct Hospitable PMS Sync**: Real-time calendar lookup, instant quote calculation, and automated reservation holding via Redis locks.
- 🌙 **Midnight Traveler Protocol (`wsp : voyageur minuit`)**: Automatically triggers for guest messages past midnight, providing verified self-check-in lockbox codes, parking instructions, and concierge emergency line.
- 💳 **Automated Payment Proof Verification (`wsp : verifier paiement`)**: Ingests bank transfer screenshots and receipts, verifies payment status, and unblocks pending calendar reservations.
- 🔍 **Qdrant Vector Store Semantic Search**: Uses OpenAI text embeddings to match guest questions with property house rules, amenities, and nearby attractions.
- 🗂️ **Automated HubSpot CRM Pipeline**: Seamlessly logs owner leads, guest contact info, and booking statuses to HubSpot and Google Sheets.

---

## 🧩 The 7 Sub-Workflow Modules

The platform is decomposed into modular, isolated workflows for maximum reliability:

| Workflow File | Nodes | Purpose & Role |
|:--------------|:-----:|:---------------|
| [`workflows/wsp_main.workflow.ts`](workflows/wsp_main.workflow.ts) | **154** | Core orchestrator: WhatsApp webhook listener, AI router, agent execution, and photo delivery. |
| [`workflows/wsp_hospitable.workflow.ts`](workflows/wsp_hospitable.workflow.ts) | **20** | Hospitable reservation webhook listener, proactive WhatsApp guest onboarding, and T+6h/T+12h payment reminders. |
| [`workflows/wsp_dispo.workflow.ts`](workflows/wsp_dispo.workflow.ts) | **10** | Calculates real-time calendar availability, quote calculations in DZD/EUR, and multi-listing pagination. |
| [`workflows/wsp_voyageur_minuit.workflow.ts`](workflows/wsp_voyageur_minuit.workflow.ts) | **9** | Midnight arrival assistant: scheduled property status audit and automated late-night check-in delivery. |
| [`workflows/wsp_verifier_paiement.workflow.ts`](workflows/wsp_verifier_paiement.workflow.ts) | **8** | Validates customer payment receipts, updates booking status, and clears admin verification gates. |
| [`workflows/wsp_recherche_catalogue.workflow.ts`](workflows/wsp_recherche_catalogue.workflow.ts) | **12** | Semantic property catalogue search leveraging Qdrant vector database embeddings. |
| [`workflows/wsp_crm_export.workflow.ts`](workflows/wsp_crm_export.workflow.ts) | **14** | Syncs customer profiles, reservation details, and owner leads to HubSpot CRM and Google Sheets. |

---

## 🧠 Intent Disambiguation & Darija NLP

In North African dialects, the root verb **"k-r-y" (كرى / نكري)** is used by both guests and property owners. The system applies strict grammatical disambiguation rules:

```
"Salam Kho rani 7ab nekri partma lghedoua nchallah"
───► "nekri" + Temporal Marker ("lghedoua" / tomorrow) 
───► Intent: voyageur_potentiel (Guest looking for accommodation)
───► Action: Trigger Hospitable calendar search & photo gallery

"3ndi partma f sidi yahia hab nmdha gestion"
───► "3ndi partma" (I have an apartment) + "gestion" (management)
───► Intent: proprietaire_potentiel (Landlord delegating management)
───► Action: Trigger Host questionnaire, Redis lead CRM & Gamma deck
```

*For complete intent specs and scenario traces, see [chat_history/INTENT_DISAMBIGUATION.md](chat_history/INTENT_DISAMBIGUATION.md) and [chat_history/SAMPLE_CONVERSATIONS.md](chat_history/SAMPLE_CONVERSATIONS.md).*

---

## 🚀 Quick Start

### 1. Prerequisites
- **Node.js**: 18+ or 20+
- **n8n**: 1.0+ (`n8n-as-code` compatible)
- **Redis**: 7+ (for locks and chat memory)
- **Qdrant**: (Vector DB instance or Cloud)
- **Meta Business Account**: Configured for WhatsApp Cloud API
- **Hospitable.com Account**: API Key with property permissions

### 2. Installation & Setup

```bash
# Clone the repository
git clone https://github.com/Maliksouhildev/wsp-hospitality-concierge-bot.git
cd wsp-hospitality-concierge-bot

# Install dependencies
npm install

# Setup environment configuration
cp .env.example .env
```

### 3. Deploy Workflows to n8n

#### Option A — Using n8n-as-code CLI:
```bash
npx n8nac push
```

#### Option B — Manual Import via n8n GUI:
1. Open n8n at `http://localhost:5678`.
2. Go to **Workflows → Import from File**.
3. Import `workflows/wsp_main.json` and associated subworkflows (`wsp_hospitable.json`, `wsp_dispo.json`, etc.).
4. Link credentials for WhatsApp, Hospitable, Redis, and OpenAI.

---

## 🔑 Environment Variables

```ini
# Meta WhatsApp Cloud API
META_WA_PHONE_NUMBER_ID=109283746501928
META_WA_ACCESS_TOKEN=EAAG...
META_WA_VERIFY_TOKEN=your_secure_verify_token

# Hospitable.com PMS
HOSPITABLE_API_KEY=your_hospitable_token

# OpenAI & Qdrant
OPENAI_API_KEY=sk-proj-...
QDRANT_HOST=localhost
QDRANT_PORT=6333

# Redis
REDIS_HOST=127.0.0.1
REDIS_PORT=6379

# CRM Integrations
HUBSPOT_API_KEY=pat-na1-...
GOOGLE_SHEETS_SPREADSHEET_ID=1BxiMVs0XRA5nFMdKvBdBZjgmUUqptlbs74OgvE2upms
```

---

## 🤝 Contributing & License

Pull requests, improvements, and custom PMS adapters are welcome.

Distributed under the **MIT License**. Created & maintained by [Maliksouhildev](https://github.com/Maliksouhildev).
