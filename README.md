# Workflow Automation & Integration Engine (n8n)

## Description

This repository hosts the automation layer of the ecosystem, powered by n8n. It contains the backend logic, event-driven pipelines, and API integrations responsible for orchestrating data movement, triggering alerts, and connecting third-party services across departments.

---

## 📁 Repository Structure

```
/finance
  /PurchaseOrders
/hr
  /TalentHunting
/sales
  /WeeklyReport
README.md
```

Each folder contains the sanitized JSON exports of production-ready n8n workflows for that department. All sensitive values (API keys, credentials, webhook paths, instance IDs) have been replaced with placeholder variables. See the [Setup & Configuration](#️-setup--configuration) section before importing.

---

## 📋 Workflow Index

### 💰 Finance

| Workflow Name | File | Trigger | Description |
| :--- | :--- | :--- | :--- |
| **Extract Purchase Orders** | `finance/PurchaseOrders/extract_purchase_order_data.json` | 🌐 Webhook | Triggered by a CRM ticket stage change. Locates the associated purchase order file in Google Drive (PDF, image, or spreadsheet), extracts billing fields using AI vision or LLM depending on file type, and inserts structured data into MySQL. Unresolvable tickets are queued as pending for retry. |
| **Retry Pending Orders** | `finance/PurchaseOrders/retry_purchase_order_pending.json` | ⏱️ Cron (Daily) | Daily job that re-processes tickets stuck in pending state by re-invoking the Extract Purchase Orders workflow. Marks tickets as resolved on success, increments the attempt counter on failure, and discards after 30 failed attempts. |

---

### 👥 HR

| Workflow Name | File | Trigger | Description |
| :--- | :--- | :--- | :--- |
| **Talent Hunting Bot** | `hr/TalentHunting/talent_hunting_bot.json` | ⏱️ Cron (Mon–Fri 8am) | Reads open vacancies from the ATS via API, uses Claude to generate optimized LinkedIn search keywords per vacancy, launches a PhantomBuster scraper to collect candidate profiles, inserts each candidate into MySQL, and scores them against the vacancy using Claude. Results are stored with score, matched keywords, urgency level, and recommended action. |
| **Talent API** | `hr/TalentHunting/talent_api.json` | 🌐 Webhook (GET + POST) | Exposes two webhook endpoints: a GET to retrieve all candidates from MySQL ordered by score, and a POST to update a candidate's status by LinkedIn URL. Acts as the backend layer for any frontend or tool consuming Talent Hunting Bot data. |

---

### 📈 Sales

| Workflow Name | File | Trigger | Description |
| :--- | :--- | :--- | :--- |
| **Weekly Sales Report** | `sales/WeeklyReport/weekly_sales_report.json` | ⏱️ Cron (Fri 12pm) | Authenticates against Azure AD to get a Power BI bearer token, runs DAX queries against a configured dataset to extract key sales metrics (QTD and MTD KPIs by market, campaign detail, bookings), assembles an HTML email with KPI cards and summary tables, attaches CSV files as supporting detail, and sends the report via Gmail to a configured recipients list. |

---

## ⚙️ Setup & Configuration

All sensitive values have been removed from the exported JSON files and replaced with placeholder variables. Before importing any workflow into your n8n instance, replace the following placeholders with your actual credentials and endpoints.

### 🔑 API Keys & Tokens

Hardcoded inside `jsCode` nodes. Replace manually after import or configure via environment variables in your n8n instance.

| Placeholder | Description |
| :--- | :--- |
| `{{GOOGLE_API_KEY}}` | Google Gemini API key used for PDF and image extraction |
| `{{ANTHROPIC_API_KEY}}` | Anthropic API key (`sk-ant-...`) used by Claude for spreadsheet extraction and candidate scoring |
| `{{HUBSPOT_PAT_TOKEN}}` | HubSpot Private App Token (`pat-...`) for reading CRM properties |
| `{{PHANTOMBUSTER_API_KEY}}` | PhantomBuster API key for launching LinkedIn scraper agents |

### 🔗 API Endpoints

| Placeholder | Description |
| :--- | :--- |
| `{{GEMINI_API_URL}}` | Gemini Vision API endpoint |
| `{{ANTHROPIC_API_URL}}` | Anthropic messages endpoint (`https://api.anthropic.com/v1/messages`) |
| `{{HUBSPOT_API_URL}}` | HubSpot CRM tickets endpoint |
| `{{GOOGLE_DRIVE_API_URL}}` | Google Drive files API |
| `{{GOOGLE_SHEETS_API_URL}}` | Google Sheets spreadsheets API |
| `{{PEOPLEFORCE_API_URL}}` | PeopleForce ATS API endpoint (e.g. `https://yourcompany.peopleforce.io/api/public/v1/...`) |
| `{{PHANTOMBUSTER_RESULT_URL}}` | PhantomBuster S3 URL where the scraper stores its results |

### ☁️ Azure AD & Power BI

Required for the Weekly Sales Report workflow. Register an app in Azure AD with Power BI API permissions (`Dataset.Read.All`).

| Placeholder | Description |
| :--- | :--- |
| `{{AZURE_TENANT_ID}}` | Azure AD tenant ID (found in Azure Portal → Azure Active Directory) |
| `{{AZURE_CLIENT_ID}}` | App registration client ID |
| `{{AZURE_CLIENT_SECRET}}` | App registration client secret |
| `{{POWERBI_WORKSPACE_ID}}` | Power BI workspace (group) ID containing the dataset |
| `{{POWERBI_DATASET_ID}}` | Power BI dataset ID to run DAX queries against |
| `{{REPORT_RECIPIENTS}}` | Comma-separated list of email addresses to receive the report |
| `{{ADMIN_EMAIL}}` | Email address that receives workflow error notifications |

### 🔐 n8n Credentials

Prompted automatically by n8n on import. Create the following credential types in your instance:

| Placeholder | Credential Type |
| :--- | :--- |
| `{{CREDENTIAL_ID}}` / `{{CREDENTIAL_NAME}}` | `googleDriveOAuth2Api` — Google Drive OAuth2 |
| `{{CREDENTIAL_ID}}` / `{{CREDENTIAL_NAME}}` | `googleSheetsOAuth2Api` — Google Sheets OAuth2 |
| `{{CREDENTIAL_ID}}` / `{{CREDENTIAL_NAME}}` | `gmailOAuth2` — Gmail OAuth2 (used to send reports and error notifications) |
| `{{CREDENTIAL_ID}}` / `{{CREDENTIAL_NAME}}` | `mySql` — MySQL connection |
| `{{CREDENTIAL_ID}}` / `{{CREDENTIAL_NAME}}` | `httpHeaderAuth` — HTTP Header Auth (used for PeopleForce `X-API-Key`) |

### 🤖 PhantomBuster

| Placeholder | Description |
| :--- | :--- |
| `{{PHANTOMBUSTER_AGENT_ID}}` | ID of the LinkedIn scraper agent in your PhantomBuster account |
| `{{LINKEDIN_SESSION_COOKIE}}` | LinkedIn `li_at` session cookie used by PhantomBuster to authenticate scraping |

### 🔁 Linked Workflows

Some workflows invoke other workflows directly. After importing both, update these placeholders in the caller workflow:

| Placeholder | Description |
| :--- | :--- |
| `{{LINKED_WORKFLOW_ID}}` | Internal n8n ID of the target workflow |
| `{{LINKED_WORKFLOW_URL}}` | Internal URL path (e.g. `/workflow/{id}`) |
| `{{LINKED_WORKFLOW_NAME}}` | Display name as it appears in your n8n instance |

### 🔔 Webhooks

| Placeholder | Description |
| :--- | :--- |
| `{{WEBHOOK_PATH}}` | Auto-generated by n8n on activation. No manual setup required. |

### 🏷️ Workflow Metadata

Auto-generated by n8n on import. No manual replacement required.

| Placeholder | Description |
| :--- | :--- |
| `{{WORKFLOW_ID}}` | Internal workflow ID assigned by n8n |
| `{{WORKFLOW_VERSION_ID}}` | Version identifier managed by n8n |
| `{{INSTANCE_ID}}` | Unique ID of the n8n instance |

---

## 🗂️ Key Components

- **Workflow Blueprints:** Sanitized JSON exports of production-ready n8n pipelines.
- **API & Webhook Integrations:** Connections to CRM, cloud storage, spreadsheet, and AI APIs.
- **AI-Powered Processing:** Vision and language models for intelligent data extraction.
- **Error Handling & Retry Logic:** Automatic queuing of failed records with configurable retry and discard thresholds.

---

## 🛠️ Tech Stack

- n8n (Workflow Automation)
- JavaScript / Node.js
- Google Drive & Sheets API
- Gmail API
- HubSpot CRM API
- Google Gemini Vision API
- Anthropic Claude API
- Microsoft Power BI API + Azure AD OAuth2
- PhantomBuster (LinkedIn scraping)
- PeopleForce ATS API
- MySQL
