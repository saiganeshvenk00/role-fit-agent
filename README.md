
# Job Filter Agent – Resume-Based Job Scorer

This project is a fully automated job discovery and filtering system designed using **n8n**, **OpenAI GPT-4**, and **Google Sheets API**. It scrapes job postings from LinkedIn RSS feeds, evaluates each posting against your resume using an AI-based scoring model, and filters out roles that are not a good fit. The final output is a curated list of job opportunities appended to a Google Sheet for review.

## Objective

The goal is to automate the manual effort involved in reviewing job postings by:

- Collecting job descriptions from publicly available RSS feeds.
- Evaluating how well each job matches your resume.
- Filtering out low-scoring roles based on predefined thresholds.
- Logging only relevant roles to a Google Sheet for easy access.

## How to Use This Project

You can either import a prebuilt version of the workflow into your n8n instance or follow a step-by-step guide to build it from scratch.

## Option 1: Use the Provided Workflow File

1. Clone or download this repository.
2. Open your n8n instance.
3. Import the JSON file from the `workflow/` folder.
4. Set up the required credentials:
   - OpenAI API Key
   - Google Sheets OAuth2 credentials
5. Upload your resume document into the `src/` directory or host it in cloud storage with a shareable link.
6. Trigger the workflow manually or wait for the scheduled trigger.
7. Check the output in your connected Google Sheet.

## Option 2: Build the Workflow Manually in n8n

Follow the steps below to replicate the system entirely from scratch in your own environment.

### Step 1: Set Up Job Feed Ingestion

**1. Schedule Trigger**  
Configure a daily schedule to initiate the job search.

**2. RSS Feed Reader**  
Connect to a LinkedIn RSS feed filtered for a specific job category (e.g., Product Management roles in the United States).

**3. Limit Node**  
Restrict the number of jobs processed per run to prevent overload. Limit to 15 items.

### Step 2: Score Resume Relevance for Each Role

**4. Loop Over Items**  
Use a loop to iterate through each job retrieved from the RSS feed.

**5. HTTP Request - Pull Job Description HTML**  
Scrape the job posting’s full HTML content using the job’s URL.

**6. Timeout/Wait Node**  
Add a delay (e.g., 5–10 seconds) to avoid hitting OpenAI’s rate limits or token quotas.

**7. JD-Resume Match Scorer (OpenAI Chat Agent)**  
Send the job description and your resume to GPT-4 with a custom prompt. The model should return a score (1–10) based on:

- Role alignment
- Relevant skills
- Industry/domain familiarity
- Prior accomplishments

**8. Conditional Branch (If Node)**  
Check whether the score is greater than or equal to 7:
- If **yes**, continue to the sheet update process.
- If **no**, skip to the next item.

### Step 3: Extract Job Details and Write to Google Sheet

**9. Wait Node**  
Insert another brief delay to manage API usage.

**10. Extract JD Info (OpenAI Model or JavaScript)**  
Use another GPT call or custom code to extract key information:
- Job Title
- Location
- Score
- Description snippet
- Source URL

**11. Code Node**  
Format the extracted data into a structured JSON object compatible with the Google Sheets API.

**12. Google Sheets Node**  
Append the formatted data as a new row in your target spreadsheet.

## Visual Workflow Reference

The repository includes a visual diagram of the entire pipeline. Refer to `docs/architecture.png` for an architectural overview.

## Technologies Used

- **n8n**: Open-source workflow automation platform.
- **OpenAI GPT-4 (Mini)**: For resume-job matching and information extraction.
- **Google Sheets API**: For storing filtered job listings.
- **RSS Feeds**: For retrieving job opportunities.

## Prerequisites

- OpenAI API access (GPT-4)
- Active Google account and enabled Google Sheets API access
- n8n instance (local or hosted)
- Resume file in `.pdf`, `.docx`, or plain text format

## Example Output

Each qualifying job added to the Google Sheet includes:

- Job Title  
- Job Location  
- Score out of 10  
- Short description (truncated)  
- Source link (LinkedIn or other)

## Potential Enhancements

- Add support for comparing against multiple resumes
- Introduce vector-based similarity using embeddings
- Automate job applications based on high-score thresholds

## Author

**Saiganesh Venkataraman**  
Solutions Architect at Dell Technologies  
Graduate, Duke University – Master of Engineering Management  
Portfolio: https://saivportfolio.netlify.app  
LinkedIn: https://www.linkedin.com/in/saiganeshvenk00

## License

This project is licensed under the MIT License.
