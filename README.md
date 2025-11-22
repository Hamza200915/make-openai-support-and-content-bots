# Make + OpenAI Support & Content Bots

Two no‑code workflows built with [Make](https://www.make.com), Google Forms, Google Sheets, OpenAI, and Gmail:

1. **Support Email Bot (Project A)** – answers customer questions for a sports store and emails the reply.
2. **Content Ideas Bot (Project B)** – takes a topic from a form, generates 5 social media post ideas + 1 full caption, saves them to Sheets, and emails them back.

This repo documents the setup so anyone can recreate or adapt these automations.

---

## Table of Contents

- [Overview](#overview)
- [Architecture](#architecture)
  - [Project A – Support Email Bot](#project-a--support-email-bot)
  - [Project B – Content Ideas Bot](#project-b--content-ideas-bot)
- [Prerequisites](#prerequisites)
- [Setup – Project A (Support Email Bot)](#setup--project-a-support-email-bot)
- [Setup – Project B (Content Ideas Bot)](#setup--project-b-content-ideas-bot)
  - [OpenAI prompt for content ideas](#openai-prompt-for-content-ideas)
- [Customization ideas](#customization-ideas)
- [Notes & Troubleshooting](#notes--troubleshooting)
- [License](#license)

---

## Overview

These automations are built entirely in **Make.com** (no traditional code files).

- **Google Forms** collect input from users.
- **Google Sheets** store form responses.
- **OpenAI** generates natural‑language answers and content.
- **Gmail** sends the responses automatically.

You can adapt these flows for customer support, FAQs, content planning, and more.

---

## Architecture

### Project A – Support Email Bot

**Goal:**  
Customer submits a question via a support form → AI generates a helpful answer → answer is stored in a sheet and emailed back.

**Data flow:**

1. **Google Form** – “Support form”
   - Q1: *What is your question?*
   - Q2: *Your email address*

2. **Google Sheet** – `Inbox`
   - Sheet: `Form_Responses`
   - Columns:
     - A: `Timestamp`
     - B: `What is your question?`
     - C: `Your email address`
     - D: `AI_Reply`

3. **Make Scenario (Project A):**

   - **Module 1 – Google Sheets: Watch New Rows**  
     Triggers when a new form response is added.

   - **Module 2 – OpenAI: Chat completion**  
     Uses a system prompt describing the store’s policies, tone, and rules to answer the question.

   - **Module 3 – Google Sheets: Update a Row**  
     Writes the AI reply into column **D (AI_Reply)** of the same row.

   - **Module 4 – Gmail: Send an email**  
     Sends the answer back to the address in column **C**.

---

### Project B – Content Ideas Bot

**Goal:**  
User submits a topic → AI generates **5 post ideas** + **1 full caption** → all saved to Sheets → emailed to the user.

**Data flow:**

1. **Google Form** – “Content Ideas”
   - Q1: *What topic do you want posts about?*
   - Q2: *Your email address*

2. **Google Sheet** – `ContentIdeas`
   - Sheet: `Form responses 1`
   - Columns:
     - A: `Timestamp`
     - B: `What topic do you want posts about?`
     - C: `Your email address`
     - D: `AI_Ideas` (the 5 ideas + first post)

3. **Make Scenario (Project B):**

   - **Module 1 – Google Sheets: Watch New Rows**
     - Spreadsheet: `ContentIdeas`
     - Sheet: `Form responses 1`
     - Row with headers: `A1:D1`
     - Limit: `1`

   - **Module 2 – OpenAI: Generate a completion**
     - Method: `Create a Chat Completion (GPT and other models)`
     - Model: `gpt-4o-2024-05-13 (system)`
     - Response format: `text`
     - Messages:
       - Message 1 (role: Developer/System) – long instructions for how to format IDEAS + FIRST_POST (see below).
       - Message 2 (role: User) – `Topic: {{1. What topic do you want posts about? (B)}}`
     - Important output: **`Result`** (contains the entire formatted text).

   - **Module 3 – Google Sheets: Update a Row**
     - Row number: `{{1. Row number}}` (from Watch New Rows)
     - Only set:
       - `AI_Ideas (D) = {{2. Result}}`
     - Leave columns A–C empty in this module to avoid overwriting.

   - **Module 4 – Gmail: Send an email**
     - To: `{{1. Your email address (C)}}`
     - Subject: `Your content Ideas`
     - Body type: `Collection of contents (text, images, etc.)`
     - Body contents:
       - 1 text block with:

         ```text
         Hi,

         {{1. What topic do you want posts about? (B)}}

         {{2. Result}}

         Thanks,
         Your content bot
         ```

---

## Prerequisites

- Google account with:
  - Google Sheets
  - Google Forms
  - Gmail
- [Make.com](https://www.make.com) account
- [OpenAI API key](https://platform.openai.com/)
- Basic familiarity with:
  - Creating Google Forms linked to Sheets
  - Creating scenarios in Make

---

## Setup – Project A (Support Email Bot)

1. **Create the Sheet**
   - Create `Inbox` spreadsheet.
   - Add sheet/tab: `Form_Responses`.
   - Make sure headers **A–D** match:
     - `Timestamp`, `What is your question?`, `Your email address`, `AI_Reply`.

2. **Create the Form**
   - Link the Form to the `Inbox` sheet.
   - Add questions:
     1. *What is your question?* (short answer)
     2. *Your email address* (short answer / email)

3. **Build the Make scenario**
   - Add **Google Sheets → Watch New Rows**.
   - Select the `Inbox` spreadsheet, `Form_Responses` sheet.
   - Add **OpenAI → Chat completion**.
     - Model: any supported GPT‑4‑style model that works with Make.
     - System message: describe your store, policies (shipping, returns), tone, etc.
     - User message: `Customer question: {{1. What is your question?}}`
   - Add **Google Sheets → Update a Row**.
     - Row number: `{{1. Row number}}`
     - Only map `AI_Reply (D) =` the AI output field (e.g., `choices[1].message.content` or `Result`, depending on the model).
   - Add **Gmail → Send an email**.
     - To: `{{1. Your email address}}`
     - Subject: something like `Reply to your question`
     - Body (text):

       ```text
       Hi,

       {{1. What is your question?}}

       {{2. AI reply field}}

       Thanks,
       Your support bot
       ```

4. **Test**
   - Turn scenario to *Run once*.
   - Fill the form.
   - Confirm:
     - Column D (`AI_Reply`) is filled.
     - Email arrives with the answer.

---

## Setup – Project B (Content Ideas Bot)

1. **Create the Sheet**
   - Create `ContentIdeas` spreadsheet.
   - Use sheet/tab: `Form responses 1`.
   - Headers:
     - A: `Timestamp`
     - B: `What topic do you want posts about?`
     - C: `Your email address`
     - D: `AI_Ideas`

2. **Create the Form**
   - Link the Form to `ContentIdeas`.
   - Questions:
     1. *What topic do you want posts about?*
     2. *Your email address*

3. **Build the Make scenario**

   #### Module 1 – Watch New Rows

   - Google Sheets → Watch New Rows
   - Spreadsheet: `ContentIdeas`
   - Sheet: `Form responses 1`
   - Row with headers: `A1:D1`
   - Limit: `1`

   #### Module 2 – OpenAI

   - OpenAI → Generate a completion
   - Method: Create a Chat Completion
   - Model: `gpt-4o-2024-05-13 (system)`
   - Response format: `text`
   - **Messages:**

     - Message 1 (Developer/System): use the prompt below.
     - Message 2 (User):

       ```text
       Topic: {{1. What topic do you want posts about? (B)}}
       ```

   #### Module 3 – Update a Row

   - Google Sheets → Update a Row
   - Row number: `{{1. Row number}}`
   - Values:
     - Timestamp (A): *(leave empty)*
     - What topic do you want posts about? (B): *(leave empty)*
     - Your email address (C): *(leave empty)*
     - **AI_Ideas (D):** `{{2. Result}}`

   #### Module 4 – Gmail

   - Gmail → Send an email
   - To: `{{1. Your email address (C)}}`
   - Subject: `Your content Ideas`
   - Body type: `Collection of contents (text, images, etc.)`
   - Body contents:
     - Add one **Text** body content with:

       ```text
       Hi,

       {{1. What topic do you want posts about? (B)}}

       {{2. Result}}

       Thanks,
       Your content bot
       ```

4. **Test**

   - Turn the scenario to *Run once*.
   - Submit the form.
   - Check:
     - Column D (`AI_Ideas`) is filled with `IDEAS:` and `FIRST_POST:`.
     - Email contains the same formatted text with clean line breaks.

---

## OpenAI prompt for content ideas

**Project B – Message 1 (System / Developer):**

```text
You are a social media strategist.

When I give you a TOPIC, you must:

1) Create 5 social media post ideas for that topic. Each idea should be 1–2 sentences and clear enough that someone could write a post from it.
2) Write one full, ready-to-post caption for IDEA #1.

Format your reply EXACTLY like this:

IDEAS:
1) ...
2) ...
3) ...
4) ...
5) ...

FIRST_POST:

[Write a full, engaging caption for idea #1, 3–8 sentences, and include 3–7 relevant hashtags.]

Do not change the headings IDEAS: or FIRST_POST:. Do not add any extra text before or after this format.
