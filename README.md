# 🤖 AI Customer Support Bot

An autonomous customer support agent built with **n8n**, **Google Gemini**, and function-calling tools. The agent understands natural-language customer messages, extracts structured data, and independently decides which actions to take — logging details, creating tickets, sending confirmation emails, and escalating urgent issues to a human.

## ✨ Features

- 💬 **Conversational FAQ handling** — answers common questions directly (shipping, returns, business hours)
- 🧠 **Multi-turn memory** — asks clarifying questions (e.g. name, order ID) before acting, and remembers context across the conversation
- 📊 **Automatic customer logging** — writes structured customer data to Google Sheets
- 🎫 **Automated ticket creation** — logs support tickets with subject, description, and priority
- 📧 **Email confirmations** — sends the customer a summary of their issue and next steps via Gmail
- 🚨 **Smart escalation** — flags urgent issues and notifies a human support agent by email
- 🔧 **Tool-calling architecture** — the LLM decides *which* tools to call and *when*, rather than following a fixed script

## 🏗️ Architecture

```
                              ┌────────────────────┐
                              │   Chat Trigger      │
                              │ (public web widget) │
                              └──────────┬──────────┘
                                         │
                              ┌──────────▼──────────┐
                              │      AI Agent        │
                              │  (Google Gemini)     │
                              └──────────┬──────────┘
                                         │
                 ┌─────────────┬────────┼────────┬─────────────┐
                 │             │        │         │             │
         ┌───────▼──────┐ ┌────▼────┐ ┌─▼───────┐ ┌▼────────────────┐
         │ Simple Memory │ │  Send   │ │  Log    │ │ Create Support  │
         │ (buffer/win)  │ │ Support │ │Customer │ │    Ticket       │
         │               │ │  Email  │ │ Details │ │ (Google Sheets) │
         └───────────────┘ └─────────┘ └─────────┘ └─────────────────┘
                                                            │
                                                    ┌───────▼────────┐
                                                    │ Escalate to     │
                                                    │ Human (Gmail)   │
                                                    └─────────────────┘
```

The AI Agent uses **LLM function-calling**: given a customer message, Gemini decides autonomously which of the four tools to invoke (if any), what arguments to pass, and whether more information is needed from the customer first.

## 🛠️ Tech Stack

| Component | Tool |
|---|---|
| Workflow orchestration | [n8n](https://n8n.io) |
| LLM / reasoning engine | Google Gemini (via `@n8n/n8n-nodes-langchain`) |
| Conversation memory | n8n Buffer Window Memory |
| Email | Gmail API (OAuth2) |
| Data storage | Google Sheets API (OAuth2) |
| Interface | n8n hosted chat widget (embeddable) |

## 📋 Example Conversation

**Customer:** "My order hasn't arrived."

**Bot:** "I'm sorry to hear that. Could you please provide your order ID?"

**Customer:** "My name is Jane Doe, order ID ORD-4521, email jane@example.com"

**Bot:**
> Behind the scenes, the agent:
> 1. Logs Jane's details to the Customers sheet
> 2. Creates a support ticket in the Tickets sheet
> 3. Sends Jane a confirmation email
>
> "I've logged your issue and created a support ticket (ORD-4521). You'll receive a confirmation email shortly, and our team will follow up within 1-2 business days."

## 🚀 Setup

1. **Import the workflow**
   - In n8n: `Workflows → Import from File` → select [`ai-customer-support-bot.json`](./ai-customer-support-bot.json)

2. **Connect credentials**
   - **Google Gemini API** — for the language model
   - **Gmail OAuth2** — for sending confirmation and escalation emails
   - **Google Sheets OAuth2** — for logging customer/ticket data

3. **Create your tracking spreadsheet**
   - Create a Google Sheet with two tabs: `Customers` and `Tickets`
   - `Customers` columns: `Name | Email | Order ID | Issue | Timestamp`
   - `Tickets` columns: `Ticket Subject | Customer Email | Order ID | Description | Priority | Timestamp`
   - Copy the Spreadsheet ID into the `Log Customer Details` and `Create Support Ticket` nodes

4. **Update the escalation email**
   - Set the `Escalate to Human` node's recipient to your support team's email address

5. **Publish**
   - Click **Publish** in n8n, then share the chat widget URL or embed it on your site

## 🧩 Extending This Project

Ideas for taking it further:
- Connect a real ticketing system (Zendesk, Freshdesk) instead of Google Sheets
- Add a knowledge-base tool (vector search over your FAQ docs) for more accurate answers
- Swap the chat widget for WhatsApp, Telegram, or Slack
- Add an order-lookup tool that queries a real order-management API
- Add a fallback LLM (e.g. GPT-4o-mini) for resilience against provider outages

## 📄 License

MIT — free to use and adapt.

---

*Built as a demonstration of agentic AI workflow design — LLM tool-calling, multi-turn memory, and real-world API integration, orchestrated with n8n.*
