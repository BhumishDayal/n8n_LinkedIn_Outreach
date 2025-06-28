# 🔍 LinkedIn Job Scraping & Outreach Automation

This `n8n` workflow is designed to automate the end-to-end process of B2B lead generation from LinkedIn job listings. It enables users to search for recent job postings, extract relevant company and contact details, verify contact information, generate personalized icebreakers using AI, and distribute the qualified leads into email and LinkedIn outreach platforms.

## 🧭 How the Workflow Works

1. **Manual Trigger**: Initiates the workflow manually for testing or batch runs.

2. **Scrape LinkedIn Job Listings**:
   - Calls the Apify API to scrape recent job listings based on predefined LinkedIn search URLs.
   - The API returns a list of job listings with metadata such as job title, company name, company website, and job poster name.

3. **Filter by Company Size**:
   - Filters out companies with fewer than 200 employees to focus on mid-to-large size businesses.

4. **Remove Duplicate Companies**:
   - Deduplicates results based on company LinkedIn URLs to avoid redundant outreach.

5. **Clean Company Name**:
   - Strips common suffixes like "Inc.", "LLC", "Ltd" from company names for consistency.

6. **Validate Job Poster Name**:
   - Ensures the job poster's name exists before proceeding to email lookup.

7. **Find Professional Email**:
   - Sends first name, last name, and company website to the Prospeo API to retrieve valid professional email addresses.
   - Filters only those contacts whose email status is "VALID".

8. **Fetch and Parse Company Website**:
   - Fetches the company's homepage HTML.
   - Converts HTML to readable markdown to prepare content for AI analysis.

9. **Generate Icebreaker Using OpenAI**:
   - Sends the parsed company info to GPT-4.
   - GPT-4 returns a short, specific, personalized compliment suitable for cold outreach.

10. **Log Data to Google Sheets**:
    - Appends all cleaned and enriched lead data into a Google Sheet.
    - Fields include names, titles, emails, company info, and AI-generated icebreaker.

11. **Send Leads to Smartlead and HeyReach**:
    - Automatically posts the lead to both Smartlead and HeyReach campaigns using their respective APIs.
    - The payload includes personal and company details, email, and icebreaker.

12. **Status Update and Throttling**:
    - Wait nodes ensure rate-limiting.
    - Status updates are logged in another Google Sheet to track outreach success.

13. **Optional: Email Verification and Fallbacks**:
    - If Prospeo fails, the workflow optionally calls Reoon or AnymailFinder APIs for alternate verification or contact discovery.

This `n8n` workflow automates the entire process of scraping job listings from LinkedIn, extracting decision-makers' names and emails, enriching them with AI-generated icebreakers, and sending the results to Google Sheets, Smartlead, and HeyReach.

---

## ⚙️ Prerequisites

1. An [n8n](https://n8n.io/) environment
2. API keys for:
   - Apify
   - Prospeo
   - AnymailFinder (optional)
   - OpenAI
   - Smartlead
   - HeyReach
   - Google Sheets OAuth credentials

---

## 🛠️ Node-by-Node Setup Instructions

### 1. **Manual Trigger: `When clicking ‘Test workflow’`**
- **Usage**: Starts the workflow manually for testing.
- **Setup**: No changes needed.

---

### 2. **HTTP Request (Apify Job Scraper)**
- **Purpose**: Triggers the Apify LinkedIn job scraper.
- **Setup**:
  1. Replace `hKByXkMQaC5Qt9UMN` in the URL with your Apify actor ID.
  2. Replace the `Authorization` header value with your own Apify API key.
     ```
     URL: https://api.apify.com/v2/acts/YOUR_ACTOR_ID/run-sync-get-dataset-items
     Header: Authorization: Bearer YOUR_APIFY_API_KEY
     ```
  3. Customize the `urls` field in the JSON body with your LinkedIn search result URL.

---

### 3. **Filter (Company Size > 200)**
- **Purpose**: Filters out companies with fewer than 200 employees.
- **Setup**: No changes unless you want to adjust the threshold.
  ```
  Condition: $json.companyEmployeesCount > 200
  ```

---

### 4. **Remove Duplicates (companyLinkedinUrl)**
- **Purpose**: Avoid contacting the same company twice.
- **Setup**:
  - Field to compare: `companyLinkedinUrl`

---

### 5. **Set (Clean up company name)**
- **Purpose**: Removes suffixes like Inc., Ltd, etc. from company names.
- **Setup**: No changes needed unless you want to modify the regex for other suffixes.

---

### 6. **If (Check if jobPosterName exists)**
- **Purpose**: Ensures we have a valid name to look up emails.
- **Setup**:
  - Condition: `jobPosterName exists`

---

### 7. **HTTP Request (Prospeo Email Finder)**
- **Purpose**: Gets emails using first and last name + company website.
- **Setup**:
  1. Replace `X-KEY` header with your Prospeo API key.
  2. Leave the name splitting and company value as is, unless your input format changes.

---

### 8. **Filter1 (Email Validity)**
- **Purpose**: Filters out results where `response.email_status != VALID`
- **Setup**:
  - Condition: `$json.response.email_status == "VALID"`

---

### 9. **Split In Batches (Loop Over Items)**
- **Purpose**: Processes results in chunks.
- **Setup**: No changes needed.

---

### 10. **HTTP Request2 (Fetch Company Website)**
- **Purpose**: Scrapes HTML content from the company site.
- **Setup**:
  - URL: `{{$json.companyWebsite}}`

---

### 11. **Markdown (Convert HTML to Markdown)**
- **Purpose**: Cleans up HTML into text for LLM input.
- **Setup**: No changes needed.

---

### 12. **OpenAI (Generate Icebreaker)**
- **Purpose**: Uses GPT to generate a 1-liner based on the website.
- **Setup**:
  1. Set your OpenAI credential under `Credentials` tab.
  2. Model: `gpt-4.1`
  3. Modify prompt only if changing tone or strategy.

---

### 13. **Google Sheets (Append Leads)**
- **Purpose**: Logs leads into a specific Google Sheet.
- **Setup**:
  1. Connect your `Google Sheets OAuth` credential.
  2. Select or paste your document ID and Sheet name.
  3. Match your field mappings as required.

---

### 14. **Smartlead (Add Lead to Email Campaign)**
- **Purpose**: Pushes leads into Smartlead.
- **Setup**:
  1. Replace API URL with your Smartlead campaign ID.
  2. Use `Query Auth` credential with valid token.
  3. Map fields: `email`, `first name`, `company`, `video`, etc.

---

### 15. **Wait**
- **Purpose**: Optional rate limiting.
- **Setup**: Adjust wait time in seconds if needed.

---

### 16. **HeyReach Campaign Add**
- **Purpose**: Adds lead to LinkedIn automation via HeyReach.
- **Setup**:
  1. Replace `X-API-KEY` header with your HeyReach API key.
  2. Update campaignId and linkedInAccountId with your own.
  3. Use variable mapping to structure the payload.

---

### 17. **Google Sheets2 (Mark Lead Status)**
- **Purpose**: Marks lead in a separate sheet as "YES/NO" depending on success.
- **Setup**:
  1. Connect appropriate spreadsheet.
  2. Map `linkedin url` as a matching key.
  3. Update the `Lead added to outreach` column.

---

### 18. **Optional Email Verifiers (Reoon / AnymailFinder)**
- **Purpose**: Validate or search for alternate decision-makers.
- **Setup**:
  - Replace API key in URL/headers as per each vendor's documentation.
  - Add conditions to validate `status == "valid"`

---

## 🔄 Summary of Major Modifications
| Feature              | Modification Description                                    |
|---------------------|-------------------------------------------------------------|
| Apify Scraper       | Custom actor ID and dynamic LinkedIn URLs                  |
| Company Clean-up    | Regex to strip business suffixes                            |
| Email Lookup        | Prospeo API with dynamic name parsing                       |
| Website Intelligence| Used company URL + Markdown + GPT-4 for first-liner crafting|
| Outreach Sync       | Smartlead + HeyReach campaign integrations                  |
| Lead Logging        | Custom structured export to Google Sheets                   |

---

## 🚀 Final Notes
- All credentials must be set up in `n8n` securely.
- You can extend this workflow by adding HubSpot, Lemlist, or webhooks for real-time notifications.
