# n8n Automation Workflows

This repository contains three automated workflows built in **n8n**. Below is the architecture, credential configuration, and chronological step-by-step execution flow for each workflow JSON file.

---

## 1. Code Summarizer
**Workflow ID:** `9Wx5pileZEeEBGwK` [cite: 4]  
**Description:** Monitors incoming emails containing code, uses OpenAI language models in parallel to generate inline comments and a plain-language summary, formats the combined output into syntax-highlighted HTML, and emails the result back [cite: 4].

### Required Credentials & Integrations
* **Gmail OAuth2:** Used for polling incoming messages and sending the final HTML email [cite: 4].
* **OpenAI / n8n AI Gateway:** Powers the LLM chains using `gpt-5-mini` and `gpt-6-astra` [cite: 4].

### Chronological Execution Steps
1. **Trigger (`Gmail Trigger`):** Polls the connected Gmail account every minute (`everyMinute`) for incoming emails from the specified sender (`vodafoneidea231996@gmail.com`) [cite: 4].
2. **Parallel AI Processing:** Upon receiving an email, the workflow splits the incoming text (`$json.text`) into two simultaneous branches [cite: 4]:
   * **Branch 1 — Add Code Comments (`Comments_Addition_node`):** Connected to **OpenAI Chat Model** (`gpt-6-astra`), this node adds line-by-line explanatory comments to the code without modifying any existing lines [cite: 4].
   * **Branch 2 — Summarize Code (`Code_summary`):** Connected to **OpenAI Chat Model1** (`gpt-5-mini`), this node generates a high-level summary of the code in simple language covering all important details [cite: 4].
3. **Synchronize Branches (`Merge`):** Waits for both parallel LLM nodes to finish and merges their outputs into a single item stream (`Comments_Addition_node` at index `0`, `Code_summary` at index `1`) [cite: 4].
4. **Consolidate Outputs (`Aggregate`):** Aggregates the `text` fields from both merged branches into a single array (`$json.text[]`) [cite: 4].
5. **Format Payload (`Edit Fields`):** Creates a string field named `email_body` that combines the commented code (`$json.text[0]`) and the summary (`$json.text`) under clear headings [cite: 4].
6. **Convert to HTML (`Basic LLM Chain`):** Connected to **OpenAI Chat Model2** (`gpt-5-mini`), this node converts `email_body` into raw HTML with syntax-highlighted code blocks [cite: 4].
7. **Send Email (`Send a message`):** Uses regex to strip any residual Markdown code fences (` ```html `) from the LLM output and sends the final HTML email to `vodafoneidea231996@gmail.com` with the subject `"Code Summary and comments"` [cite: 4].

---

## 2. Daily n8n Workflow Backup to GitHub
**Workflow ID:** `uExIokRk8qIJ8Hw3` [cite: 5]  
**Description:** An automated disaster-recovery workflow that triggers daily to export all workflows from the n8n instance and commit them as formatted `.json` files into a GitHub repository [cite: 5].

### Required Credentials & Integrations
* **GitHub API / OAuth2:** Required to create the target repository and commit `.json` backup files [cite: 5].
* **n8n API Credential:** Required by the internal n8n node to fetch all workflow schemas from the instance [cite: 5].

### Chronological Execution Steps
1. **Trigger (`Daily Schedule`):** Runs automatically once every day at **2:00 AM** (`triggerAtHour: 2`) [cite: 5].
2. **Initialize Repository (`Create Backup Repository`):** Sends a `POST` request to `https://github.com` to create a public repository named `n8n-workflow-backups` [cite: 5]. 
   * *Note:* Configured with `executeOnce: true` and `onError: continueRegularOutput` so the workflow continues seamlessly if the repository already exists [cite: 5].
3. **Retrieve Workflows (`Get All Workflows`):** Queries the host n8n instance to fetch the complete JSON data of all existing workflows [cite: 5].
4. **Update Existing Backup (`Update Backup File`):** Attempts to edit the existing workflow file in GitHub at `backups/<workflow-name-in-kebab-case>.json` with a date-stamped commit message (`Backup <Name> (yyyy-MM-dd)`) [cite: 5].
5. **Create New Backup (`Create Backup File`):** If the file does not exist yet in the repository, the `Update Backup File` node routes the error via `continueErrorOutput` to this fallback node, which creates a brand-new `.json` file at the same `backups/` path [cite: 5].

---

## 3. Sheet_Updation_Workflow
**Workflow ID:** `0Nhf3vn1fTgQkU9I` [cite: 6]  
**Description:** Collects customer feedback via an n8n web form, evaluates whether the feedback is positive to assign a discount flag, and appends the complete customer record into a Google Sheet [cite: 6].

### Required Credentials & Integrations
* **n8n Form Trigger:** Built-in webhook node that hosts the public feedback form [cite: 6].
* **Google Sheets OAuth2 API:** Connects to the `Customer Data` spreadsheet (`Sheet1`) [cite: 6].

### Chronological Execution Steps
1. **Trigger (`On form submission`):** Hosts a web form titled *"Collecting User Feedback"* that captures four required fields from the user [cite: 6]:
   * `Email Id` (Email) [cite: 6]
   * `Name` (Text) [cite: 6]
   * `Age` (Number) [cite: 6]
   * `Feedback` (Dropdown: `Positive`, `Neutral`, or `Negative`) [cite: 6]
2. **Evaluate Feedback (`If`):** Checks whether the submitted `$json.Feedback` value strictly equals `"Positive"` (case-sensitive) [cite: 6].
3. **Assign Discount Flag (Conditional Branching):**
   * **True Branch (`Discount_Yes`):** If `Feedback` is `"Positive"`, sets a new string field `"Give_Discount": "Yes"` while retaining all original form fields [cite: 6].
   * **False Branch (`Discount_No`):** If `Feedback` is `"Neutral"` or `"Negative"`, sets `"Give_Discount": "No"` while retaining all original form fields [cite: 6].
4. **Log to Spreadsheet (`Append row in sheet`):** Both branches converge into the Google Sheets node, which appends a new row to `Sheet1` of the `Customer Data` spreadsheet mapping `Email_id`, `Name`, `Age`, `Feedback`, and `Give_Discount` [cite: 6].

---

## How to Import & Setup
1. Open your **n8n** instance and create a new workflow.
2. Click the **•••** menu in the top-right corner and select **Import from File**.
3. Select the desired `.json` workflow file.
4. Configure your **Gmail**, **OpenAI**, **GitHub**, and **Google Sheets** credentials inside the respective nodes before toggling the workflow to **Active**.
