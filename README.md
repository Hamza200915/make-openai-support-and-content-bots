# Make Content Ideas Bot

A no‑code automation built with [Make](https://www.make.com), Google Forms, Google Sheets, OpenAI, and Gmail.

**Goal:**  
User submits a topic → AI generates **5 social media post ideas** + **1 full caption** → results are saved to Google Sheets → emailed back to the user.

---

## Overview

**Stack**

- Google Form – collects topic + email
- Google Sheet – stores all form responses + AI output
- Make.com – connects everything (no‑code scenario)
- OpenAI – generates the ideas and caption
- Gmail – sends the results back to the user

**Data flow**

1. User fills out the Google Form.
2. A new row appears in the linked Google Sheet.
3. Make scenario triggers on that new row.
4. The topic is sent to OpenAI, which returns:
   - `IDEAS:` 1–5
   - `FIRST_POST:` (full caption)
5. The AI output is written into column `AI_Ideas`.
6. Gmail sends a nicely formatted email with the ideas + caption.

---

## Google Form & Sheet Setup

### Google Sheet

Spreadsheet name: **`ContentIdeas`**  
Sheet/tab: **`Form responses 1`** (created automatically by the Form)

Columns:

| Column | Header                                | Description                          |
|--------|----------------------------------------|--------------------------------------|
| A      | Timestamp                             | Auto‑filled by Google Form           |
| B      | What topic do you want posts about?   | Topic text                           |
| C      | Your email address                    | Recipient address                    |
| D      | AI_Ideas                              | AI output (5 ideas + 1 caption)      |

### Google Form

Linked to the `ContentIdeas` spreadsheet.

Questions:

1. **What topic do you want posts about?** (short answer)  
2. **Your email address** (short answer / email)

---

## Make Scenario

The scenario has **4 modules**, in this order:

1. **Google Sheets – Watch New Rows**
2. **OpenAI – Generate a completion**
3. **Google Sheets – Update a Row**
4. **Gmail – Send an email**

### 1) Google Sheets – Watch New Rows

- Connection: your Google account
- Search Method: `Search by path`
- Drive: `My Drive`
- Spreadsheet ID / File: `/ContentIdeas`
- Sheet Name: `Form responses 1`
- Table contains headers: `Yes`
- Row with headers: `A1:D1`
- Limit: `1`

Important output fields used later:

- `Row number`
- `What topic do you want posts about? (B)`
- `Your email address (C)`

---

### 2) OpenAI – Generate a completion

- Connection: your OpenAI account
- Select Method: `Create a Chat Completion (GPT and other models)`
- Model: **`gpt-4o-2024-05-13 (system)`**
- Response format: **`text`**
- Max output tokens: **`1000`**

#### Messages

Make sure **Map = OFF** for the Messages section.

**Message 1**

- Role: `Developer / System`
- Text Content:

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
