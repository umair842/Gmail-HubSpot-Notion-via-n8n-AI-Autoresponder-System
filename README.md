# TRT Australia AI Email Autoresponder Project

This project demonstrates a fully automated AI-based support system for **TRT Australia**, integrating **Gmail**, **HubSpot**, **Notion**, and **OpenRouter AI** via **n8n**. When a new email is received, the system identifies the sender, fetches the relevant deal and searches two Notion-based knowledge bases to generate a concise, human-like draft response using an AI agent. It allows manual review and supports fallback responses for unknown senders.

---

## 🔧 Project Scope

- Detect incoming emails in Gmail and identify if the sender is a customer.
- Search associated **HubSpot deals** and **Notion knowledge bases**.
- Use AI Agents to answer customer questions via structured queries.
- Create draft email responses for review.
- Auto-label and store drafts in Gmail folders.
- Provide fallback drafts for unknown contacts.

---

## 🧩 Tech Stack

- **n8n** (open-source workflow automation)
- **HubSpot CRM** (customer + deal data)
- **Notion** (knowledge base)
- **Gmail/Outlook** (email input/output)
- **OpenRouter** (AI agents via LLMs)

---

## 🔁 Workflow Overview

### 1. 📥 Email Trigger
- **Trigger:** New email received in Gmail  
- **Node:** Gmail Trigger  
- **Action:** Fetch full message with "Get Message" node  

### 2. 🔍 HubSpot Contact Search
- **Node:** HubSpot Contact Search  
- **Logic:** Check if sender email exists  
- **Switch Node:**
  - **Case 1:** Contact found → proceed with deal & Notion KB  
  - **Case 2:** Contact not found → send general info response  

### 3. 🔗 Fetch Associated Deal (if contact found)
- **Node:** HTTP Request to HubSpot  
- **Endpoint:** `/crm/v3/objects/contacts/{id}/associations/deals`  

### 4. 📚 Knowledge Base Search (Primary DB)
- **Node:** Notion Database → Blood Work Knowledge Base  
- **Fields Set:** Database Name & ID for use in next nodes  

### 5. 🧠 AI Agent – Generate Response
- **Model:** OpenRouter Chat Model  
- **Memory:** Window Buffer Memory  
- **Prompt:** Uses system message to instruct behavior, search via Notion tools  
- **Tools Used:**
  - Search Notion Database  
  - Get Page Content (block-level)  
- **Input:**
  - Email body (from Gmail)  
  - Session ID  
- **Output:**
  ```json
  {
    "status": "Answer found",
    "reply": "Here’s the info you requested..."
  }
### 6. 🧪 Result Check
Code Node: Parses AI response JSON

IF Node:

If status == Answer found → go to Step 7

Else → Try secondary KB

### 7. ✉️ Compose Draft Reply
Set Node: Format clean message

Gmail Node: Create draft

Message Body:

Hi {{name}},

{{ AI-generated reply }}

Best Regards,  
TRT Australia
Gmail Node: Add Label to Draft

🔄 Fallbacks
### 8. 📚 Secondary KB Search
If primary KB has no answer:

Query "TRT Australia Legitimacy & Trust – Private" in the same format

Same Notion, AI Agent, and response logic

### 9. 🧱 Full Page Block Merge (if both KBs fail)
Notion Block Node: Fetch child blocks from a Notion page

Code Node: Convert all text blocks into one searchable fullText

Agent Prompt: Modified to search inside {{ fullText }} only

Tools: Same as above

### 10. 🛑 Final Fallback – Contact Not Found
Switch Case 2:

Gmail Node: Create general info draft

Static Response Template:

Booking link

Blood test info

Policy and contact email

## 🧠 AI Agent Instructions (System Message Summary)
Only use facts from Notion DB

Never hallucinate or guess

Output strict JSON format with status and reply


Return fallback message if no results

## ⚙️ AI Tool Configuration
Tool 1 – Search Notion Database
Method: POST

Filter by: question keyword or tags (OR logic)

URL: https://api.notion.com/v1/databases/{{db_id}}/query

Tool 2 – Get Page Content
Method: GET

URL: https://api.notion.com/v1/blocks/{page_id}/children

Optimize Response: true

## 🚀 Technologies Used
Gmail API

HubSpot CRM API

Notion API

OpenRouter LLMs

n8n (workflow orchestration)

## 📈 Outcomes & Use Cases
Reduces manual inbox triage for TRT support staff

Responds to common medical/test queries using verified data

Filters out irrelevant or spam contacts with fallback handling

Can scale with additional KBs and workflows

📬 Want a Similar Setup?
This project showcases a multi-system AI responder framework that balances automation with review control. If you're looking to implement something similar for customer support, internal QA, or product FAQs—let’s chat!
