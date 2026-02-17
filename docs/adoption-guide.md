# Adoption Guide -- Agent Resume Standard

The Agent Resume Standard (`agent-resume.json`) is a machine-readable resume format designed for AI agents, recruitment platforms, and automated talent-matching systems. This guide explains how candidates, platforms, and recruiters can adopt it.

---

## For Candidates

### Creating Your `agent-resume.json`

#### Step 1: Start from the Template or Schema

Clone the repository or download the schema file. The schema defines every field, its type, and whether it is required. Start with a copy of the example `resume-agent.json` and replace the values with your own.

```bash
# Clone the repo
git clone https://github.com/danielrosehill/AI-Resume.git
cd AI-Resume

# Copy the example as your starting point
cp resume-agent.json my-agent-resume.json
```

#### Step 2: Fill Required Sections First

The following sections contain required fields and should be completed before anything else:

1. **`meta`** -- Set `format` to `"agent-resume"`, `version` to `"1.0.0"`, and `lastUpdated` to today's date.
2. **`person`** -- Your full name and headline at minimum.
3. **`location`** -- City, country, and IANA timezone.
4. **`summary`** -- A 2-4 sentence professional summary.
5. **`availability`** -- Your current search status.
6. **`skills`** -- At least your `primary` skills array.
7. **`experience`** -- Your work history with roles, organizations, and dates.
8. **`education`** -- Degrees, fields, and institutions.

#### Step 3: Add Optional Sections That Matter for Your Search

Not every section applies to every candidate. Focus on the ones that will help agents match you accurately:

- If you freelance or consult, fill out **`compensation`** so agents can budget-match before contacting you.
- If remote work matters to you, complete **`workPreferences`** with your `remotePreference`, `asyncPreference`, and timezone details.
- If you have strong opinions about engagement types, fill in **`engagementPreferences`** including `dealBreakers`.
- If you have a public portfolio (GitHub, Hugging Face, npm), fill in **`portfolio`** so agents can verify your work.
- If you know exactly what you want, write a detailed **`matchCriteria`** section. The `idealEngagement` field is the single richest signal for semantic matching.

#### Step 4: Validate Against the Schema

If a JSON Schema file is available in the repository, validate your file against it:

```bash
# Using ajv-cli (install with: npm install -g ajv-cli)
ajv validate -s schema/agent-resume-schema.json -d my-agent-resume.json

# Or use any online JSON Schema validator
```

Fix any validation errors before publishing.

#### Step 5: Host It at a Stable URL

Place your `agent-resume.json` alongside your existing resume on your personal website or a publicly accessible URL. Common patterns:

```
https://yoursite.com/agent-resume.json
https://yoursite.com/files/agent-resume.json
https://yoursite.com/.well-known/agent-resume.json
```

Record the canonical URL in `meta.canonical` so agents can deduplicate if they find your resume at multiple locations. If you also have HTML, PDF, or Markdown versions of your resume, link them in `meta.alternateFormats`.

#### Step 6: Keep It Updated

Set a recurring reminder based on your chosen `agentInstructions.updateFrequency`:

- **Weekly**: For active job seekers whose availability and preferences change often.
- **Monthly**: For most professionals who are passively open.
- **Quarterly**: For stable situations with infrequent changes.
- **Annually**: For candidates who are not actively searching.

Update `meta.lastUpdated` every time you revise the file.

### Tips on Key Sections

#### Compensation: Be Honest, Use Ranges

State real numbers. If your hourly rate is $100-175, say so. If you would accept $90 for an exceptional project, set `negotiable` to `true` and explain in `notes`. Ranges are better than single numbers because they signal flexibility. Do not inflate your numbers hoping to negotiate down -- agents will filter you out of budget-appropriate roles.

If you work across different engagement types (hourly consulting vs. full-time salary), fill in both `hourlyRate` and `annualSalary`. They serve different matching contexts.

#### Match Criteria: Be Specific About What You Want (and Do Not Want)

The `lookingFor` array is your positive signal. Be concrete: "AI agent development and orchestration projects" is far more matchable than "interesting technical work."

The `notLookingFor` array is equally important. If you know you do not want frontend work or entry-level roles, say so explicitly. This saves everyone time. Agents will respect these exclusions.

The `idealEngagement` field is your most powerful tool. Write a short paragraph describing your dream role. AI agents will use semantic similarity between this paragraph and job descriptions to produce match scores. The more specific you are, the better the matching.

#### Agent Instructions: Tell Agents What to Verify First

The `agentInstructions.humanVerification` field is your safety net. Use it to specify what an agent must check with you before acting. Common examples:

- "Contact me via email before scheduling any interviews."
- "Do not share my compensation expectations with third parties without my consent."
- "Verify my availability before making commitments on my behalf."

This field exists because agents may act autonomously. You are defining the boundary of their autonomy.

---

## For Platforms and AI Agents

### Consuming `agent-resume.json`

#### Step 1: Fetch and Validate

Retrieve the JSON file and validate it against the schema before processing. Check `meta.format === "agent-resume"` and parse `meta.version` to ensure compatibility.

```
FUNCTION process_agent_resume(url):
    raw = HTTP_GET(url)
    data = JSON_PARSE(raw)

    IF data.meta.format != "agent-resume":
        RETURN error("Not an agent resume")

    IF MAJOR_VERSION(data.meta.version) > 1:
        LOG warning("Unknown major version, attempting best-effort parse")

    VALIDATE(data, AGENT_RESUME_SCHEMA)
    RETURN data
```

#### Step 2: Use `matchCriteria` for Initial Filtering

Before doing expensive semantic matching, use the structured match criteria for fast filtering:

```
FUNCTION passes_initial_filter(resume, job):
    // Check negative matches first (cheapest filter)
    FOR EACH item IN resume.matchCriteria.notLookingFor:
        IF SEMANTIC_OVERLAP(item, job.description) > THRESHOLD:
            RETURN false

    // Check deal breakers
    FOR EACH breaker IN resume.engagementPreferences.dealBreakers:
        IF MATCHES_JOB_ATTRIBUTE(breaker, job):
            RETURN false

    // Check industry exclusions
    IF job.industry IN resume.engagementPreferences.avoidIndustries:
        RETURN false

    // Check engagement type compatibility
    IF job.type NOT IN resume.engagementPreferences.preferredTypes:
        RETURN false  // or reduce score instead of hard filter

    RETURN true
```

#### Step 3: Use `compensation` for Budget Matching

Compare the job's budget against the candidate's stated ranges:

```
FUNCTION budget_compatible(resume, job):
    IF job.type == "hourly" AND resume.compensation.hourlyRate:
        IF job.maxRate < resume.compensation.hourlyRate.min:
            IF resume.compensation.negotiable:
                RETURN "marginal"  // worth a conversation
            RETURN false
        RETURN true

    IF job.type == "salary" AND resume.compensation.annualSalary:
        IF job.maxSalary < resume.compensation.annualSalary.min:
            IF resume.compensation.negotiable:
                RETURN "marginal"
            RETURN false
        RETURN true

    IF job.type == "project" AND resume.compensation.projectMinimum:
        IF job.budget < resume.compensation.projectMinimum.amount:
            RETURN false
        RETURN true

    RETURN "unknown"  // compensation data not provided
```

#### Step 4: Use `workPreferences` and `engagementPreferences` for Fit Scoring

Compute a compatibility score across multiple dimensions:

```
FUNCTION compute_fit_score(resume, job):
    score = 0
    max_score = 0

    // Remote compatibility
    max_score += 1
    IF resume.workPreferences.remotePreference == "remote_only" AND job.isRemote:
        score += 1
    ELSE IF resume.workPreferences.remotePreference == "remote_preferred":
        score += (1 IF job.isRemote ELSE 0.5)
    ELSE IF resume.workPreferences.remotePreference == "hybrid" AND job.isHybrid:
        score += 1

    // Timezone overlap
    max_score += 1
    overlap = COMPUTE_OVERLAP_HOURS(
        resume.workPreferences.typicalWorkingHours,
        job.teamTimezone
    )
    IF overlap >= resume.workPreferences.timezoneOverlapMinHours:
        score += 1
    ELSE:
        score += overlap / resume.workPreferences.timezoneOverlapMinHours

    // Industry preference
    max_score += 1
    IF job.industry IN resume.engagementPreferences.preferredIndustries:
        score += 1
    ELSE IF job.industry NOT IN resume.engagementPreferences.avoidIndustries:
        score += 0.5

    // Seniority match
    max_score += 1
    IF resume.experienceSummary.seniorityLevel == job.seniorityLevel:
        score += 1

    // Skills overlap
    max_score += 1
    primary_overlap = INTERSECTION(resume.skills.primary, job.requiredSkills)
    score += LEN(primary_overlap) / LEN(job.requiredSkills)

    RETURN score / max_score
```

#### Step 5: Respect `agentInstructions.humanVerification`

This is not optional. Before taking any action that affects the candidate externally (scheduling interviews, sharing data with third parties, making commitments), you must obtain human confirmation as specified in this field.

```
FUNCTION safe_to_act(resume, action):
    verification = resume.agentInstructions.humanVerification

    IF action.type IN ["schedule_interview", "share_data", "make_commitment"]:
        REQUIRE_HUMAN_APPROVAL(
            candidate_email = resume.person.email,
            action_description = action,
            verification_instructions = verification
        )
        RETURN false  // do not proceed until human approves

    RETURN true  // informational actions can proceed
```

#### Step 6: Handle Missing Optional Fields Gracefully

Many fields in the schema are optional. Your system must not penalize candidates for omitting optional sections. Treat missing data as "unknown," not as "negative."

```
// BAD: Penalizing missing data
IF resume.compensation IS NULL:
    score -= 10  // DO NOT DO THIS

// GOOD: Neutral treatment of missing data
IF resume.compensation IS NULL:
    budget_match = "unknown"  // skip budget filtering for this candidate
```

A candidate who omits `engagementPreferences` is not saying "I have no preferences." They are saying "I chose not to fill this out." These are different signals. Default to inclusion when data is missing.

---

## For Recruiters

### What the Agent Resume Standard Enables

#### Instant Structured Data Instead of PDF Parsing

Traditional resumes are PDFs or Word documents that require OCR, layout parsing, and NLP extraction -- all of which introduce errors. An `agent-resume.json` file gives you perfectly structured data from the start. Every field is in a known location, with a known type.

This means:
- No more misparting "2015-2018" as a phone number.
- No more guessing whether "Python" in a resume refers to a skill or a project name.
- No more losing data buried in creative PDF layouts.

#### Honest Compensation Expectations Upfront

The `compensation` section eliminates the most common source of wasted time in recruiting: discovering at the end of a process that the candidate's salary expectations are 50% above the budget. With agent resumes, you know the range before the first email.

The `negotiable` flag and `notes` field add nuance. A candidate whose minimum is $120K but who marks `negotiable: true` with a note about non-profit discounts is giving you real information you can act on.

#### Work Style Compatibility Screening

The `workPreferences` section tells you whether a candidate will thrive in your team's environment before you ever speak to them:
- **Remote vs. onsite**: A `remote_only` candidate will not accept your hybrid role. Do not waste either party's time.
- **Async vs. sync**: A candidate who strongly prefers async communication will struggle in a team that runs daily standups and ad-hoc calls.
- **Meeting tolerance**: A candidate who marks `minimal` is telling you they are productive with deep focus time, not in meeting rooms.
- **Timezone overlap**: If your team is in San Francisco and the candidate needs 3 hours of overlap from Jerusalem, you can compute immediately whether that works.

#### Cultural Values Alignment

The `values` array lets you check alignment with your organization's culture before the first conversation. If your company values "move fast and break things" and the candidate values "documentation-driven development," that is a useful signal -- not necessarily a conflict, but something to discuss early.

#### Timezone and Availability Matching

Between `location.timezone`, `workPreferences.typicalWorkingHours`, and `workPreferences.timezoneOverlapMinHours`, you have everything you need to determine whether a distributed team arrangement will work logistically. No more discovering timezone incompatibility three interviews in.

#### Clear Deal-Breaker Filtering

The `engagementPreferences.dealBreakers` array is a gift to recruiters. If a candidate lists "Micromanagement culture" as a deal-breaker, you know immediately whether your organization is a fit. This is information candidates rarely share in traditional resumes but are willing to state in structured data that they control.

### How to Encourage Candidates to Adopt the Format

1. **Add it to your job postings.** Include a line like: "We accept agent-resume.json files. Learn more at [link]." Candidates who adopt early are signaling technical competence and openness to structured communication.

2. **Offer a converter.** Build or link to a tool that converts a traditional resume into an `agent-resume.json` file with human review. Lower the adoption barrier.

3. **Make it optional, not required.** The format is new. Do not gatekeep on it. Accept it as a supplement to traditional resumes, and over time it will become the primary format as candidates see the matching benefits.

4. **Show candidates the benefit.** When a candidate submits an agent resume and gets a better match as a result, tell them. Positive reinforcement drives adoption.

5. **Contribute back.** If you find fields missing that would help your matching, open an issue or PR on the repository. The standard improves through real-world usage.

---

## Quick Reference: Required vs. Optional Sections

| Section | Required Fields | Status |
|---------|----------------|--------|
| `meta` | `format`, `version`, `lastUpdated` | Required |
| `person` | `fullName`, `headline` | Required |
| `location` | `city`, `country`, `timezone.iana` | Required |
| `summary` | (the string itself) | Required |
| `availability` | `status` | Required |
| `compensation` | (none) | Optional |
| `experienceSummary` | (none) | Optional |
| `workPreferences` | (none) | Optional |
| `engagementPreferences` | (none) | Optional |
| `portfolio` | (none) | Optional |
| `languages` | (none) | Optional |
| `matchCriteria` | (none) | Optional |
| `communication` | (none) | Optional |
| `values` | (none) | Optional |
| `skills` | `primary` | Required |
| `experience` | `role`, `organization`, `startDate` per entry | Required |
| `education` | `degree`, `field`, `institution` per entry | Required |
| `agentInstructions` | (none) | Optional |
