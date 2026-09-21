# Smart Service Desk & AI Triage Workflow (n8n + Claude API)

*Русская версия: [README.ru.md](README.ru.md)*

An event-driven system that automates B2B support intake: it receives requests, classifies incidents with an AI agent, and routes them intelligently.

> **Project status:** portfolio project built to demonstrate the architecture. It is not a client deployment.

---

## The problem and the solution

**The problem:** first-line support teams and managers spend much of their time sorting chaotic requests by hand, filtering spam, forwarding tickets to the right specialists and writing standard replies. Critical incidents get lost in the general stream, and SLAs are missed.

**The solution:** an autonomous pipeline built on **n8n** and the **Claude API (Anthropic)**. It picks up requests in real time, classifies their severity with an LLM, instantly escalates outages to engineers in Telegram, and automatically closes simple requests or drafts replies.

---

## Architecture: three independent flows

Instead of one bulky scenario, the logic is split into three isolated, asynchronous flows (event-driven architecture).

### 1. Intake and first-line triage
Receives requests (Webhook / form), generates a ticket ID (`INC-XXXX`), recognizes the intent with the Claude API, and routes the ticket with a Switch node. `High` tickets trigger an instant Telegram alert; `Medium` and `Low` tickets get an email reply to the customer.

![Intake and triage flow](flow-triage.png)

### 2. Email feedback handling and spam protection
Runs on its own `IMAP Trigger`. It monitors the mailbox, filters out spam with a Filter node (checks for the `INC-` marker), cleans the model's JSON output of Markdown formatting with a custom JavaScript Code node, and updates statuses in Google Sheets using Upsert logic (no duplicate rows).

![Email processing and Upsert flow](flow-feedback.png)

### 3. Error handling and monitoring
An independent `Error Trigger` catches errors and timeouts in any node of the process and sends the administrator a detailed failure log. The design follows a **Human-in-the-Loop** approach: the AI does not send replies to critical incidents without human approval.

![Error monitoring flow](flow-error.png)

---

## Demo: alerts and notifications

Instant Telegram notifications let support react to incidents within seconds, with the AI's analysis and a draft reply already in front of them:

| Urgent incident alert (High priority) | Status notification / failure (error log) |
| :---: | :---: |
| ![Telegram alert](tg-alert.png) | ![Telegram result](tg-result.png) |

*Examples of automatic messages generated and sent by the system to a first-line support Telegram channel.*

---

## Tech stack
* **Orchestration:** n8n (Docker, self-hosted on a Linux VPS with SSL/HTTPS)
* **AI / LLM:** Anthropic Claude Sonnet 4.5 API (prompt engineering, JSON output, intent recognition)
* **Integrations:** REST API, Webhooks, IMAP, Telegram Bot API, Google Sheets API (Upsert)
* **Code and parsing:** JavaScript (ES6+), JSON, regular expressions (RegEx)

---

## How to deploy it yourself
1. Download `smart_service_desk_workflow.json` from this repository.
2. In n8n, open the `...` menu, choose **Import from File**, and select the downloaded file.
3. Set up your own credentials:
   * **Anthropic API key** (for the Claude node)
   * **Telegram bot token and chat ID** (for notifications)
   * **Google account** (for the spreadsheet)
   * **IMAP / SMTP** (for reading and sending email)
4. Switch the workflow to **Active** in the top-right corner.

> Never commit API keys or tokens to the repository. Keep them only in n8n credentials.

---

## What it delivers (design goals)
* **Faster first response to critical incidents:** High-priority tickets trigger an instant Telegram alert instead of waiting in a shared inbox.
* **Less manual routine for first-line support:** classification, routing and standard replies are automated.
* **No duplicate records:** Upsert logic keeps all communication about one incident on a single row in the database.
