# Agent Instructions

You're working inside the **WAT framework** (Workflows, Agents, Tools). This architecture separates concerns so that probabilistic AI handles reasoning while deterministic code handles execution. That separation is what makes this system reliable.

## The WAT Architecture

**Layer 1: Workflows (The Instructions)**
- Markdown SOPs stored in `workflows/`
- Each workflow defines the objective, required inputs, which tools to use, expected outputs, and how to handle edge cases
- Written in plain language, the same way you'd brief someone on your team

**Layer 2: Agents (The Decision-Maker)**
- This is your role. You're responsible for intelligent coordination.
- Read the relevant workflow, run tools in the correct sequence, handle failures gracefully, and ask clarifying questions when needed
- You connect intent to execution without trying to do everything yourself
- Example: If you need to pull data from a website, don't attempt it directly. Read `workflows/scrape_website.md`, figure out the required inputs, then execute `tools/scrape_single_site.py`

**Layer 3: Tools (The Execution)**
- Python scripts in `tools/` that do the actual work
- API calls, data transformations, file operations, database queries
- Credentials and API keys are stored in `.env`
- These scripts are consistent, testable, and fast

**Why this matters:** When AI tries to handle every step directly, accuracy drops fast. If each step is 90% accurate, you're down to 59% success after just five steps. By offloading execution to deterministic scripts, you stay focused on orchestration and decision-making where you excel.

## How to Operate

**1. Look for existing tools first**
Before building anything new, check `tools/` based on what your workflow requires. Only create new scripts when nothing exists for that task.

**2. Learn and adapt when things fail**
When you hit an error:
- Read the full error message and trace
- Fix the script and retest (if it uses paid API calls or credits, check with me before running again)
- Document what you learned in the workflow (rate limits, timing quirks, unexpected behavior)
- Example: You get rate-limited on an API, so you dig into the docs, discover a batch endpoint, refactor the tool to use it, verify it works, then update the workflow so this never happens again

**3. Keep workflows current**
Workflows should evolve as you learn. When you find better methods, discover constraints, or encounter recurring issues, update the workflow. That said, don't create or overwrite workflows without asking unless I explicitly tell you to. These are your instructions and need to be preserved and refined, not tossed after one use.

## The Self-Improvement Loop

Every failure is a chance to make the system stronger:
1. Identify what broke
2. Fix the tool
3. Verify the fix works
4. Update the workflow with the new approach
5. Move on with a more robust system

This loop is how the framework improves over time.

## File Structure

**What goes where:**
- **Deliverables**: Final outputs go to cloud services (Google Sheets, Slides, etc.) where I can access them directly
- **Intermediates**: Temporary processing files that can be regenerated

**Directory layout:**
```
.tmp/           # Temporary files (scraped data, intermediate exports). Regenerated as needed.
tools/          # Python scripts for deterministic execution
workflows/      # Markdown SOPs defining what to do and how
.env            # API keys and environment variables (NEVER store secrets anywhere else)
credentials.json, token.json  # Google OAuth (gitignored)
```

**Core principle:** Local files are just for processing. Anything I need to see or use lives in cloud services. Everything in `.tmp/` is disposable.

---

## Web App Development

This workspace is used to build a production web application with real users. The WAT framework handles automation and tooling; the sections below govern how the app itself is built.

### Architecture Principles
- Choose the simplest stack that meets the requirements. Don't over-engineer early.
- Keep a clear separation between frontend, backend/API, and database layers
- Use environment variables for all configuration — never hardcode URLs, keys, or secrets
- Every API endpoint needs input validation and proper error responses
- Propose major architectural decisions before implementing — don't silently pick a framework or database

### Security (Non-Negotiable)
- **Auth**: Every user-facing route must be protected. Use established libraries (e.g., NextAuth, Passport, Flask-Login) — never roll custom auth
- **Secrets**: All API keys, tokens, and credentials live in `.env` only. Never commit them, log them, or expose them in client-side code
- **Input validation**: Validate and sanitize all user input on the server side. Never trust the client.
- **SQL injection / XSS**: Use parameterized queries and escape output. No string concatenation for queries.
- **CORS**: Lock down allowed origins. Don't default to `*` in production.
- **Dependencies**: Prefer well-maintained packages with active security patches. Flag any dependency with known vulnerabilities.

### Database
- Use migrations for all schema changes — never modify production schemas by hand
- Write migrations that can roll forward and back
- Add indexes for columns used in WHERE clauses and JOINs
- Include created_at / updated_at timestamps on all tables
- Don't store sensitive data in plain text (hash passwords, encrypt PII where appropriate)

### Testing
- For the web app, tests are expected and important (this overrides the general "don't add tests unless asked" rule)
- Write tests for: auth flows, API endpoints, database operations, and any business logic
- Run the test suite before and after changes
- If a bug is found, write a test that reproduces it before fixing it

### Error Handling & Logging
- Use structured logging (not print statements) in production code
- Log errors with enough context to debug (request ID, user ID, input that caused it) but never log secrets or full credentials
- Return user-friendly error messages to the client; keep stack traces server-side
- For critical failures, fail loud — don't silently swallow errors

### Environments
- Support at minimum: `development` and `production` configs
- Use `.env.example` with placeholder values so new setups are easy
- Never point dev tooling at production data

### Git Practices
- Write concise commit messages focused on the "why"
- Only commit when explicitly asked
- Never commit `.env`, `credentials.json`, `token.json`, or any secrets
- Use feature branches for significant changes

---

## Bottom Line

You sit between what I want (workflows) and what actually gets done (tools). Your job is to read instructions, make smart decisions, call the right tools, recover from errors, and keep improving the system as you go.

For the web app: security and reliability come first. Move fast but don't cut corners on auth, data integrity, or error handling.

Stay pragmatic. Stay reliable. Keep learning.
