# WhatsApp Auto Reply Agent

An n8n workflow that automatically receives WhatsApp messages via Green API webhook, classifies them, searches a Pinecone knowledge base, and sends intelligent replies back to the student — with language detection and translation support.

---

## How It Works

```
Webhook (Green API)
      ↓
Classifier Agent
(is message related to coaching?)
      ↓
IF Node (related / mixed / unrelated)
      ↓
Code Node (language detection)
      ↓
Output Sender Agent
(search Pinecone + craft reply)
      ↓
Code Node (format token count)
      ↓
Translator Agent
(translate reply to student's language)
      ↓
HTTP Request (send reply via Green API)
```

---

## What Each Node Does

**Webhook** — Receives incoming WhatsApp messages from Green API. Triggers the workflow on every new message.

**Classifier Agent** — Reads the student's message and classifies it as `related`, `unrelated`, or `mixed`. Extracts the query type, related parts, unrelated parts, and whether the query is time-sensitive.

**IF Node** — Routes related and mixed messages to the Output Sender. Unrelated messages are handled directly without searching Pinecone.

**Code Node (language detection)** — Detects what language the student wrote in (Hindi, English, Hinglish, or Marathi) so the Translator knows which language to reply in.

**Output Sender Agent** — The main answering agent. Searches Pinecone with short keywords, checks memory before searching, uses the Date & Time tool for time-sensitive queries, handles sensitive topics with redirects, and crafts a clean structured reply.

**Translator Agent** — Translates the reply into the student's detected language. Keeps bullet formatting intact.

**HTTP Request** — Sends the final reply back to the student on WhatsApp via Green API.

---

## Credentials Required

| Node | Credential Type |
|------|----------------|
| Classifier | Mistral Cloud API |
| Output Sender | Mistral Cloud API |
| Translator | Mistral Cloud API |
| Embeddings Mistral Cloud | Mistral Cloud API |
| Pinecone Vector Store | Pinecone API |
| HTTP Request | Green API (configured manually in node URL) |

---

## Green API Setup

1. Go to [console.green-api.com](https://console.green-api.com) and create a free account
2. Click **Create Instance** and choose a plan
3. Open the instance and scan the QR code using WhatsApp: **Settings → Linked Devices → Link a Device**
4. After scanning, instance status will show **Authorized**
5. Enable **"Receive webhooks on incoming messages"** in instance settings
6. Paste your n8n **Webhook URL** in the webhook URL field — use Production URL (not test)
7. Copy your **idInstance** and **apiTokenInstance** from the dashboard

### Configure the HTTP Request Node

Update the URL in the HTTP Request node:
```
https://api.green-api.com/waInstance[YOUR_ID_INSTANCE]/sendMessage/[YOUR_API_TOKEN_INSTANCE]
```

The `chatId` is automatically taken from the incoming message — no manual setup needed.

---

## Pinecone Setup

- This workflow **reads** from Pinecone — it does not write data
- Use the same Pinecone index as the **Info Update Form** workflow
- Index dimension must be **1024** (Mistral embedding size)
- After importing, open the Pinecone Vector Store node and select your index

---

## Customizing the AI Prompts

All three AI agents have system prompts you can edit:

**Classifier** — Edit what counts as "related" or "unrelated" for your use case. Add or remove topic categories based on your business.

**Output Sender** — This is the most important prompt to customize:
- Change the coaching center name and contact number
- Add or remove sensitive topics that should be redirected
- Edit the fallback message when information is not found
- Change emoji and formatting style
- Add new keyword examples for better Pinecone search

**Translator** — Edit which languages are supported. You can add more languages or remove ones you don't need.

To edit: open any agent node → click the **System Message** field → update the text.

---

## Memory Configuration

Two memory nodes are used:

**Simple Memory (Classifier)** — Remembers recent conversation for the classifier so it understands context across messages.

**Simple Memory1 (Output Sender)** — Remembers what was already answered so the same Pinecone search is not repeated for follow-up questions. Default window: last 10 messages.

Session key for both is the student's WhatsApp `chatId` — each student gets their own separate memory automatically.

---

## Language Support

The workflow automatically detects and replies in:
- Hindi (Devanagari)
- English
- Hinglish (Roman script mixed with English)

To add more languages (e.g. Marathi, Gujarati, Tamil) — edit the Translator node's system prompt and add the new language with its detection rule in the Classifier node's STEP 5.

---

## Important Notes

- Activate the workflow before testing — Green API webhook only sends to production URL
- The workflow ignores messages from the bot's own number automatically via the `chat id condition` IF node
- Pinecone Top K is set to retrieve the most relevant chunks — adjust in the Vector Store node if answers feel incomplete
- All sensitive topics (fee discounts, complaints, refunds) are redirected to the contact number — never answered by the AI

---

## Related Workflows

- **Info Update Form** — Used to add or update knowledge in the Pinecone database that this workflow searches. Both workflows use the same Pinecone index.
