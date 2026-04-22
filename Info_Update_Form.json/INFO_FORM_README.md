# Info Update Form

An n8n workflow that provides a simple web form where authorized users can enter text, which is automatically chunked and stored in a Pinecone vector database. A WhatsApp confirmation is sent after successful upload.

---

## How It Works

```
Form Trigger (text input)
      ↓
Code Node (split into chunks)
      ↓
Pinecone Vector Store (store embeddings)
      ↓
HTTP Request (WhatsApp confirmation)
```

---

## What Each Node Does

**On Form Submission** — A web form with a large text area. The authorized user pastes or types the information they want to store. Accessible via a direct URL.

**Split Into Chunks** — Splits the submitted text into overlapping chunks of 800 words with 80-word overlap. Adds metadata including source, timestamp, chunk index, and word count to each chunk.

**Pinecone Vector Store** — Converts each chunk into a vector embedding using Mistral and stores it in the Pinecone index.

**HTTP Request** — Sends a WhatsApp confirmation message via Green API to notify that the data was saved successfully.

---

## Credentials Required

| Node | Credential Type |
|------|----------------|
| Embeddings Mistral Cloud | Mistral Cloud API |
| Pinecone Vector Store | Pinecone API |
| HTTP Request | Green API (configured manually in node URL) |

---

## Pinecone Setup

- Create a Pinecone index with **dimension: 1024** (required for Mistral embeddings)
- After importing, open the Pinecone Vector Store node and enter your index name
- This workflow **writes** to Pinecone — the Auto Reply Agent workflow **reads** from it
- Both workflows must use the **same Pinecone index**

---

## Green API Setup (WhatsApp Confirmation)

Update the HTTP Request node URL:
```
https://api.green-api.com/waInstance[YOUR_ID_INSTANCE]/sendMessage/[YOUR_API_TOKEN_INSTANCE]
```

Update the `chatId` value to your WhatsApp number:
```
91XXXXXXXXXX@c.us
```
- `91` = India country code (change for other countries)
- `XXXXXXXXXX` = 10 digit WhatsApp number
- `@c.us` = required suffix

---

## How to Use

1. Activate the workflow
2. Open the Form URL (shown in the Form Trigger node)
3. Paste or type the information in the text area
4. Click Submit
5. You will receive a WhatsApp confirmation when the data is saved

---

## Best Practices for Adding Data

**For permanent information** (fees, schedule, faculty):
Just paste the full text and submit. No special format needed.

**For date-specific information** (holidays, exam dates, special events):
Always include the date clearly in the text:
```
1 May 2026 - Holiday hai, class nahi hogi
15 June 2026 - Exam result day, batch timing changed to 2 PM
```
This allows the Auto Reply Agent to match queries with the correct date.

**For updating existing information:**
Pinecone does not auto-delete old data. Before submitting updated information, delete the old entry from the Pinecone dashboard first, then submit the new text. This avoids conflicting answers.

---

## Customizing for Other Use Cases

This workflow is not limited to coaching centers. You can use it for any knowledge base that needs to be searchable via AI:

- **Restaurants** — store menu, timings, special offers
- **Clinics** — store doctor schedules, services, pricing
- **E-commerce** — store product details, return policies, FAQs
- **HR teams** — store company policies, leave rules, onboarding info

To adapt: change the Form Trigger field label to match your content type. The chunking and Pinecone storage logic works for any plain text.

---

## Chunking Configuration

You can adjust these values inside the **Split Into Chunks** code node:

| Parameter | Default | What It Does |
|-----------|---------|-------------|
| CHUNK_SIZE | 800 words | How large each stored chunk is |
| OVERLAP_WORDS | 80 words | How much context is shared between chunks |

Smaller chunks = more precise retrieval. Larger chunks = more context per result.

---

## Important Notes

- Keep this form URL private — share only with authorized users who should be able to update the knowledge base
- Activate the workflow before using — form submissions do not work in inactive state
- Monthly cleanup of outdated data is recommended via the Pinecone dashboard

---

## Related Workflows

- **WhatsApp Auto Reply Agent** — Reads from the same Pinecone index to answer student questions automatically. Add data here first before the agent can answer questions about it.
