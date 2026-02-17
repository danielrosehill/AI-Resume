# Agent Resume Standard -- Specification v1.0.0

## 1. Motivation

Resumes were designed for humans. They are loosely structured, visually formatted documents optimized for a recruiter skimming a PDF for 30 seconds. This is a problem for AI agents.

When an AI agent -- whether a recruiter bot, a talent matching system, or a screening pipeline -- encounters a traditional resume, it faces a series of compounding failures:

- **Unstructured text requires NLP parsing with error-prone results.** A PDF or DOCX resume must be converted to text, then parsed with heuristics or language models to extract structured fields. This process is fragile, lossy, and inconsistent across formats. A section labeled "Professional Experience" in one resume might be "Work History" in another and "Career" in a third.

- **Salary expectations, work style, and engagement preferences are never included.** Traditional resumes omit the information that matters most for programmatic matching: compensation ranges, remote/async preferences, timezone constraints, and engagement type (contract, fractional, retainer). This data exists only in the candidate's head or in a separate, unlinked form.

- **No machine-readable matching hints or filtering criteria.** There is no way for a candidate to express what they are looking for or what they want to avoid. An agent cannot filter on deal-breakers, preferred industries, or ideal project duration because these fields simply do not exist.

- **Timezone, availability, and communication preferences are buried or absent.** For distributed and remote work, these are first-class concerns. A traditional resume provides none of this information in a structured way.

- **No way to express deal-breakers, values, or agent-specific instructions.** A candidate cannot tell a consuming agent "always verify with me before scheduling" or "my compensation data is approximate" because there is no mechanism for candidate-to-agent communication.

- **The JSON Resume standard does not cover these fields.** JSON Resume (https://jsonresume.org) is an excellent step toward structured resumes, but it was designed for human rendering, not agent consumption. It lacks sections for compensation, work preferences, engagement preferences, match criteria, agent instructions, and many other fields that AI agents need for effective candidate evaluation.

The Agent Resume Standard addresses all of these gaps. It is a JSON format designed from the ground up for machine consumption, with every field chosen for its utility to an AI agent performing talent search, candidate matching, or recruitment automation.

---

## 2. Design Principles

### Agent-first

Every field in the schema is designed for programmatic consumption. Field names are camelCase for direct deserialization into JavaScript/TypeScript objects. Values use enums over free text wherever possible. The document is not a rendering format -- it is a data interchange format.

### Human-extensible

The format is JSON. Humans can read it, edit it in any text editor, and version-control it with Git. No special tooling is required to author or maintain an Agent Resume.

### Opt-in depth

Only the following sections are **required**: `meta`, `person`, `location`, `summary`, `availability`, `skills`, `experience`, and `education`. Everything else is optional. A minimal Agent Resume can be written in under 30 lines. A comprehensive one can include compensation data, work preferences, match criteria, portfolio links, and agent instructions.

### Semantic precision

Use enums over free text wherever possible. Availability status is not a sentence -- it is one of `actively_looking`, `open_to_opportunities`, `not_looking`, or `employed_not_looking`. Seniority level is not inferred from years of experience -- it is explicitly declared as `junior`, `mid`, `senior`, `staff`, `principal`, or `executive`.

### Self-describing

The `agentInstructions` section tells consuming agents how to interpret the document: what data is approximate, how often the document is updated, and what actions require human verification. This enables agents to calibrate their confidence and behavior.

### Versionable

The `meta.version` field uses semantic versioning (semver). Consuming agents can check the version and handle schema differences accordingly. This enables the standard to evolve without breaking existing consumers.

---

## 3. Document Structure

The top-level structure of an Agent Resume document is a single JSON object with the following sections:

```
{
  "$schema":                  // Optional. Path or URL to JSON Schema file.
  "meta":                     // REQUIRED. Format metadata and versioning.
  "person":                   // REQUIRED. Identity and contact information.
  "location":                 // REQUIRED. Geographic and timezone data.
  "summary":                  // REQUIRED. Professional summary (string).
  "availability":             // REQUIRED. Current availability and contact method.
  "compensation":             // Optional. Salary, rate, and project minimum.
  "experienceSummary":        // Optional. Aggregate career statistics.
  "workPreferences":          // Optional. Remote, async, and scheduling preferences.
  "engagementPreferences":    // Optional. Engagement types, industries, deal-breakers.
  "portfolio":                // Optional. Links to public work and publications.
  "languages":                // Optional. Spoken/written language proficiencies.
  "matchCriteria":            // Optional. What the candidate is and is not looking for.
  "communication":            // Optional. How and when to reach the candidate.
  "values":                   // Optional. Professional values (array of strings).
  "skills":                   // REQUIRED. Primary skills, secondary skills, and tools.
  "experience":               // REQUIRED. Work history (array of positions).
  "education":                // REQUIRED. Educational background (array of entries).
  "agentInstructions":        // Optional. Instructions for consuming AI agents.
}
```

### Required vs. Optional Summary

| Section                | Required |
|------------------------|----------|
| `meta`                 | Yes      |
| `person`               | Yes      |
| `location`             | Yes      |
| `summary`              | Yes      |
| `availability`         | Yes      |
| `compensation`         | No       |
| `experienceSummary`    | No       |
| `workPreferences`      | No       |
| `engagementPreferences`| No       |
| `portfolio`            | No       |
| `languages`            | No       |
| `matchCriteria`        | No       |
| `communication`        | No       |
| `values`               | No       |
| `skills`               | Yes      |
| `experience`           | Yes      |
| `education`            | Yes      |
| `agentInstructions`    | No       |

---

## 4. Field Reference

### 4.1 `meta`

| Field              | Type     | Required | Description                                                                 |
|--------------------|----------|----------|-----------------------------------------------------------------------------|
| `format`           | string   | Yes      | Must be `"agent-resume"`. Identifies the document type.                     |
| `version`          | string   | Yes      | Semver version of the schema (e.g., `"1.0.0"`).                            |
| `lastUpdated`      | string   | Yes      | ISO 8601 date of last update (e.g., `"2026-02-17"`).                       |
| `canonical`        | string   | No       | URL to the canonical version of this resume.                                |
| `alternateFormats` | object   | No       | URLs to alternate format versions. Keys: `html`, `pdf`, `json`, `markdown`. |

### 4.2 `person`

| Field        | Type   | Required | Description                                           |
|--------------|--------|----------|-------------------------------------------------------|
| `fullName`   | string | Yes      | Full legal or professional name.                      |
| `headline`   | string | Yes      | One-line professional headline.                       |
| `website`    | string | No       | Personal or professional website URL.                 |
| `email`      | string | Yes      | Contact email address.                                |
| `linkedin`   | string | No       | LinkedIn profile URL.                                 |
| `github`     | string | No       | GitHub profile URL.                                   |
| `huggingface`| string | No       | Hugging Face profile URL.                             |
| `business`   | object | No       | Business entity. Fields: `name` (string), `url` (string). |

### 4.3 `location`

| Field          | Type    | Required | Description                                                                 |
|----------------|---------|----------|-----------------------------------------------------------------------------|
| `city`         | string  | Yes      | City of residence.                                                          |
| `country`      | string  | Yes      | Country of residence.                                                       |
| `countryCode`  | string  | Yes      | ISO 3166-1 alpha-2 country code.                                            |
| `timezone`     | object  | Yes      | Timezone details. See Timezone Object below.                                |
| `openToRemote` | boolean | No       | Whether the candidate is open to remote work. Default: `true`.              |

**Timezone Object:**

| Field       | Type   | Required | Description                                               |
|-------------|--------|----------|-----------------------------------------------------------|
| `iana`      | string | Yes      | IANA timezone identifier (e.g., `"Asia/Jerusalem"`).      |
| `standard`  | object | Yes      | Standard time. Fields: `abbreviation` (string), `utcOffset` (string, e.g., `"+02:00"`). |
| `daylight`  | object | No       | Daylight saving time. Fields: `abbreviation` (string), `utcOffset` (string), `start` (ISO 8601 date), `end` (ISO 8601 date). |

### 4.4 `summary`

| Field     | Type   | Required | Description                                                      |
|-----------|--------|----------|------------------------------------------------------------------|
| (value)   | string | Yes      | A professional summary. Free text. 1-3 paragraphs recommended.   |

This section is a top-level string, not an object. Example:

```json
"summary": "AI developer and technical communications professional with..."
```

### 4.5 `availability`

| Field          | Type            | Required | Description                                                    |
|----------------|-----------------|----------|----------------------------------------------------------------|
| `status`       | enum (string)   | Yes      | Current availability status. See Enums Reference.              |
| `types`        | array of strings| Yes      | Types of work the candidate is available for. Free text array. |
| `contactMethod`| enum (string)   | Yes      | Preferred initial contact method. See Enums Reference.         |
| `contactUrl`   | string          | No       | URL for initiating contact (e.g., contact form).               |

### 4.6 `compensation`

| Field            | Type    | Required | Description                                                   |
|------------------|---------|----------|---------------------------------------------------------------|
| `hourlyRate`     | object  | No       | Hourly rate range. Fields: `min` (number), `max` (number), `currency` (string, ISO 4217), `preferred` (number, optional). |
| `annualSalary`   | object  | No       | Annual salary range. Fields: `min` (number), `max` (number), `currency` (string, ISO 4217). |
| `projectMinimum` | object  | No       | Minimum project engagement value. Fields: `amount` (number), `currency` (string, ISO 4217). |
| `negotiable`     | boolean | No       | Whether compensation is negotiable. Default: `true`.          |
| `notes`          | string  | No       | Free text notes on compensation (e.g., discount for long-term). |

### 4.7 `experienceSummary`

| Field              | Type            | Required | Description                                                  |
|--------------------|-----------------|----------|--------------------------------------------------------------|
| `totalYears`       | number          | No       | Total years of professional experience.                      |
| `careerStartYear`  | number          | No       | Year the candidate's career began.                           |
| `domainExperience` | array of objects| No       | Breakdown by domain. Each object: `domain` (string), `years` (number), `level` (enum). See Enums Reference. |
| `seniorityLevel`   | enum (string)   | No       | Overall seniority level. See Enums Reference.                |

### 4.8 `workPreferences`

| Field                      | Type            | Required | Description                                                        |
|----------------------------|-----------------|----------|--------------------------------------------------------------------|
| `remotePreference`         | enum (string)   | No       | Remote work preference. See Enums Reference.                       |
| `asyncPreference`          | enum (string)   | No       | Asynchronous communication preference. See Enums Reference.        |
| `communicationTools`       | array of strings| No       | Tools the candidate prefers (e.g., `"Slack"`, `"Notion"`).        |
| `meetingTolerance`         | enum (string)   | No       | Tolerance for meetings. See Enums Reference.                       |
| `typicalWorkingHours`      | object          | No       | Working hours. Fields: `start` (HH:MM), `end` (HH:MM), `timezone` (IANA string). |
| `timezoneOverlapMinHours`  | number          | No       | Minimum required timezone overlap in hours with collaborators.     |

### 4.9 `engagementPreferences`

| Field                  | Type            | Required | Description                                                         |
|------------------------|-----------------|----------|---------------------------------------------------------------------|
| `preferredTypes`       | array of enums  | No       | Preferred engagement types. See Enums Reference.                    |
| `preferredIndustries`  | array of strings| No       | Industries the candidate prefers.                                   |
| `avoidIndustries`      | array of strings| No       | Industries the candidate wants to avoid.                            |
| `idealProjectDuration` | object          | No       | Ideal project length. Fields: `min` (number), `max` (number), `unit` (enum: `"days"`, `"weeks"`, `"months"`, `"years"`). |
| `noticePeriod`         | string          | No       | Notice period before starting (e.g., `"2 weeks"`).                 |
| `teamSizePreference`   | enum (string)   | No       | Preferred team size. See Enums Reference.                           |
| `dealBreakers`         | array of strings| No       | Conditions under which the candidate will decline. Free text array. |

### 4.10 `portfolio`

| Field                 | Type            | Required | Description                                                       |
|-----------------------|-----------------|----------|-------------------------------------------------------------------|
| `github`              | object          | No       | GitHub presence. Fields: `url` (string), `publicRepos` (number), `contributions` (string). |
| `huggingFace`         | object          | No       | Hugging Face presence. Fields: `url` (string), `datasets` (number), `spaces` (number), `models` (number). |
| `npm`                 | object          | No       | NPM presence. Fields: `url` (string), `packages` (number).       |
| `publications`        | array of objects| No       | Published works. Each object: `title` (string), `url` (string), `type` (enum). See Enums Reference. |
| `speakingEngagements` | array of objects| No       | Talks and presentations. Each object: `title` (string), `event` (string), `date` (ISO 8601 date), `url` (string, optional). |

### 4.11 `languages`

Array of objects, each with:

| Field        | Type          | Required | Description                                        |
|--------------|---------------|----------|----------------------------------------------------|
| `language`   | string        | Yes      | Language name in English (e.g., `"English"`).      |
| `proficiency`| enum (string) | Yes      | Proficiency level. See Enums Reference.            |
| `iso639`     | string        | No       | ISO 639-1 two-letter language code (e.g., `"en"`). |

### 4.12 `matchCriteria`

| Field              | Type            | Required | Description                                                      |
|--------------------|-----------------|----------|------------------------------------------------------------------|
| `lookingFor`       | array of strings| No       | What the candidate is seeking. Free text array.                  |
| `notLookingFor`    | array of strings| No       | What the candidate is not seeking. Free text array.              |
| `idealEngagement`  | string          | No       | Free text description of the ideal engagement.                   |
| `keywords`         | array of strings| No       | Keywords for search matching (e.g., `"RAG"`, `"MCP"`, `"LLM"`). |

### 4.13 `communication`

| Field               | Type          | Required | Description                                                     |
|---------------------|---------------|----------|-----------------------------------------------------------------|
| `preferredMethod`   | enum (string) | No       | Preferred communication method. See Enums Reference.            |
| `responseTimeSLA`   | string        | No       | Expected response time (e.g., `"24-48 hours"`).                 |
| `bestTimeToReach`   | object        | No       | Best contact window. Fields: `start` (HH:MM), `end` (HH:MM), `timezone` (IANA string). |
| `calendarBookingUrl`| string or null| No       | URL for scheduling a meeting. `null` if not available.          |

### 4.14 `values`

| Field   | Type            | Required | Description                                                 |
|---------|-----------------|----------|-------------------------------------------------------------|
| (value) | array of strings| No       | Professional values and principles. Free text array.        |

This section is a top-level array, not an object. Example:

```json
"values": ["Open source", "Async-first communication", "Transparency and directness"]
```

### 4.15 `skills`

| Field       | Type            | Required | Description                                                       |
|-------------|-----------------|----------|-------------------------------------------------------------------|
| `primary`   | array of strings| Yes      | Primary/core skills. Order implies priority.                      |
| `secondary` | array of strings| No       | Secondary/supporting skills.                                      |
| `tools`     | object          | No       | Tool proficiencies grouped by category. Each key is a category name (camelCase string), each value is an array of strings. |

**Tools object example categories:** `aiMl`, `cloud`, `devops`, `cms`, `automation`, `generativeAi`. Categories are not fixed -- authors may define their own.

### 4.16 `experience`

Array of objects, each with:

| Field          | Type            | Required | Description                                                    |
|----------------|-----------------|----------|----------------------------------------------------------------|
| `role`         | string          | Yes      | Job title or role.                                             |
| `organization` | string          | Yes      | Company or organization name.                                  |
| `url`          | string          | No       | Organization website URL.                                      |
| `startDate`    | string          | Yes      | Start date in ISO 8601 format (e.g., `"2022-01-01"`).         |
| `endDate`      | string or null  | No       | End date in ISO 8601 format. `null` if current.               |
| `current`      | boolean         | No       | Whether this is the current position. Default: `false`.        |
| `location`     | string          | No       | Location of the role (free text).                              |
| `type`         | enum (string)   | No       | Employment type. See Enums Reference.                          |
| `summary`      | string          | No       | Brief description of the role and achievements.                |
| `tags`         | array of strings| No       | Tags for categorization and search (e.g., `"ai"`, `"consulting"`). |

### 4.17 `education`

Array of objects, each with:

| Field         | Type   | Required | Description                                         |
|---------------|--------|----------|-----------------------------------------------------|
| `degree`      | string | Yes      | Degree type or name (e.g., `"MA"`, `"BSc"`, `"PhD"`). |
| `field`       | string | Yes      | Field of study.                                     |
| `institution` | string | Yes      | Name of the institution.                            |
| `location`    | string | No       | Location of the institution.                        |

### 4.18 `agentInstructions`

| Field                | Type          | Required | Description                                                        |
|----------------------|---------------|----------|--------------------------------------------------------------------|
| `purpose`            | string        | No       | Describes the purpose and intended audience of the document.       |
| `confidenceNotes`    | string        | No       | Notes on data accuracy and confidence levels for consuming agents. |
| `updateFrequency`    | enum (string) | No       | How often the document is updated. See Enums Reference.            |
| `humanVerification`  | string        | No       | Instructions for when and how to verify with the human candidate.  |

---

## 5. Enums Reference

### `availability.status`

| Value                    | Description                                        |
|--------------------------|----------------------------------------------------|
| `actively_looking`       | Actively seeking new opportunities.                |
| `open_to_opportunities`  | Not actively searching but receptive to offers.    |
| `not_looking`            | Not currently available for new work.              |
| `employed_not_looking`   | Employed and not interested in new opportunities.  |

### `availability.contactMethod`

| Value     | Description                |
|-----------|----------------------------|
| `email`   | Contact via email.         |
| `phone`   | Contact via phone.         |
| `linkedin`| Contact via LinkedIn.      |
| `form`    | Contact via web form.      |
| `other`   | Other contact method.      |

### `experienceSummary.domainExperience[].level`

| Value          | Description                              |
|----------------|------------------------------------------|
| `beginner`     | Foundational knowledge, limited practice.|
| `intermediate` | Solid working knowledge and experience.  |
| `advanced`     | Deep expertise, can lead in this domain. |
| `expert`       | Recognized authority, extensive track record. |

### `experienceSummary.seniorityLevel`

| Value       | Description                         |
|-------------|-------------------------------------|
| `junior`    | 0-2 years experience.              |
| `mid`       | 3-5 years experience.              |
| `senior`    | 6-10 years experience.             |
| `staff`     | 10+ years, technical leadership.   |
| `principal` | 15+ years, organizational impact.  |
| `executive` | C-level or VP-level leadership.    |

### `workPreferences.remotePreference`

| Value          | Description                                  |
|----------------|----------------------------------------------|
| `remote_only`  | Will only accept fully remote positions.     |
| `remote_first` | Prefers remote, open to occasional onsite.   |
| `hybrid`       | Open to a mix of remote and onsite.          |
| `onsite_ok`    | Open to fully onsite roles.                  |
| `no_preference`| No strong preference.                        |

### `workPreferences.asyncPreference`

| Value              | Description                                    |
|--------------------|------------------------------------------------|
| `async_only`       | Strongly prefers asynchronous communication.   |
| `async_preferred`  | Prefers async, accepts some synchronous.       |
| `balanced`         | Comfortable with both async and sync.          |
| `sync_preferred`   | Prefers real-time communication.               |

### `workPreferences.meetingTolerance`

| Value      | Description                                    |
|------------|------------------------------------------------|
| `none`     | Avoids meetings entirely.                      |
| `minimal`  | Accepts essential meetings only (1-2/week).    |
| `moderate` | Comfortable with regular meetings (3-5/week).  |
| `high`     | Fine with frequent meetings (daily standups, etc.). |

### `engagementPreferences.preferredTypes`

| Value            | Description                                 |
|------------------|---------------------------------------------|
| `full_time`      | Full-time permanent employment.             |
| `part_time`      | Part-time employment.                       |
| `contract`       | Fixed-term contract work.                   |
| `freelance`      | Independent freelance engagements.          |
| `consulting`     | Advisory and consulting work.               |
| `fractional`     | Fractional/part-time leadership role.       |
| `project_based`  | Discrete project engagements.               |
| `retainer`       | Ongoing retainer arrangements.              |
| `advisory`       | Board or advisory positions.                |

### `engagementPreferences.teamSizePreference`

| Value      | Description                    |
|------------|--------------------------------|
| `solo`     | Prefers working alone.         |
| `small`    | Small teams (2-10 people).     |
| `medium`   | Medium teams (11-50 people).   |
| `large`    | Large teams (50+ people).      |
| `any`      | No team size preference.       |

### `engagementPreferences.idealProjectDuration.unit`

| Value    | Description |
|----------|-------------|
| `days`   | Days.       |
| `weeks`  | Weeks.      |
| `months` | Months.     |
| `years`  | Years.      |

### `portfolio.publications[].type`

| Value           | Description                          |
|-----------------|--------------------------------------|
| `open_source`   | Open source project or repository.   |
| `article`       | Published article or blog post.      |
| `paper`         | Academic or research paper.          |
| `book`          | Published book.                      |
| `whitepaper`    | Industry white paper.                |
| `documentation` | Technical documentation.             |
| `video`         | Video content.                       |
| `podcast`       | Podcast episode or series.           |
| `other`         | Other publication type.              |

### `languages[].proficiency`

| Value                   | Description                                              |
|-------------------------|----------------------------------------------------------|
| `native`                | Native or bilingual proficiency.                         |
| `full_professional`     | Full professional proficiency.                           |
| `professional_working`  | Professional working proficiency.                        |
| `limited_working`       | Limited working proficiency.                             |
| `elementary`            | Elementary proficiency.                                  |

These levels follow the ILR (Interagency Language Roundtable) scale commonly used in professional contexts.

### `communication.preferredMethod`

| Value     | Description                    |
|-----------|--------------------------------|
| `email`   | Email communication.           |
| `phone`   | Phone or voice call.           |
| `slack`   | Slack or similar chat platform.|
| `linkedin`| LinkedIn messaging.            |
| `calendar`| Calendar booking link.         |
| `other`   | Other method.                  |

### `experience[].type`

| Value                    | Description                                        |
|--------------------------|----------------------------------------------------|
| `full_time`              | Full-time employment.                              |
| `part_time`              | Part-time employment.                              |
| `contract`               | Contract engagement.                               |
| `freelance`              | Freelance work.                                    |
| `self_employed`          | Self-employed or sole proprietor.                  |
| `internship`             | Internship position.                               |
| `volunteer`              | Volunteer work.                                    |
| `advisory`               | Advisory or board role.                            |
| `full_time_then_contract`| Transitioned from full-time to contract.           |

### `agentInstructions.updateFrequency`

| Value       | Description                      |
|-------------|----------------------------------|
| `daily`     | Updated daily.                   |
| `weekly`    | Updated weekly.                  |
| `biweekly`  | Updated every two weeks.         |
| `monthly`   | Updated monthly.                 |
| `quarterly` | Updated quarterly.               |
| `annually`  | Updated annually.                |
| `as_needed` | Updated as circumstances change. |

---

## 6. Versioning Strategy

The Agent Resume Standard uses **Semantic Versioning** (semver) to manage schema evolution.

The version is declared in the `meta.version` field of every document.

### Version Components

**MAJOR** (e.g., 1.x.x to 2.x.x)
: Breaking changes to required fields. This includes renaming required fields, changing their types, removing required fields, or restructuring required sections. Consuming agents must update their parsers.

**MINOR** (e.g., 1.0.x to 1.1.x)
: New optional sections or fields. Existing documents remain valid. Consuming agents can ignore fields they do not recognize. No parser changes are required for backward compatibility.

**PATCH** (e.g., 1.0.0 to 1.0.1)
: Description, documentation, or clarification changes only. No structural changes to the schema. Enum descriptions may be refined. Examples may be added or corrected.

### Current Version

```
1.0.0
```

### Compatibility Rules

- A consuming agent built for version `1.x.x` MUST be able to parse any `1.y.y` document where `y >= x`, by ignoring unrecognized fields.
- A consuming agent SHOULD check `meta.version` before parsing and warn if the major version is unsupported.
- Authors SHOULD update `meta.version` when modifying their document to reflect the schema version they are targeting.

---

## 7. MIME Type and File Naming

### Recommended Filename

```
resume-agent.json
```

This filename distinguishes the Agent Resume from traditional `resume.json` files (e.g., JSON Resume format) and makes the intended consumer clear.

### MIME Type

```
application/json
```

The Agent Resume uses standard JSON. No custom MIME type is required.

### Recommended HTTP Headers

When serving an Agent Resume over HTTP, the following custom header is recommended:

```
X-Agent-Resume-Version: 1.0.0
```

This allows consuming agents to check the schema version before downloading and parsing the full document.

### Discovery

Authors are encouraged to:

1. Host the file at a well-known path (e.g., `https://example.com/resume-agent.json`).
2. Reference it from the `<head>` of their website:
   ```html
   <link rel="alternate" type="application/json" href="/resume-agent.json" title="Agent Resume" />
   ```
3. Include it in the `meta.canonical` field of the document itself.
4. List alternate format URLs in `meta.alternateFormats` for cross-referencing.

---

## 8. Security and Privacy Considerations

### Sensitive Data

The Agent Resume format intentionally supports fields that contain sensitive personal and financial information, including compensation expectations, availability status, and contact details. Candidates have full control over which optional sections they include. The following guidelines apply:

- **Compensation data is sensitive.** Candidates choose what to include. An Agent Resume with no `compensation` section is valid. Consuming agents MUST NOT infer compensation expectations from the absence of this section.
- **Contact information is provided at the candidate's discretion.** The `person.email` field is required, but candidates may use a public-facing or role-specific address.

### Agent Behavior Requirements

- **Respect `agentInstructions.humanVerification`.** When this field is present, consuming agents SHOULD follow its directives before taking actions on behalf of or involving the candidate. Typical instructions include requiring email confirmation before scheduling interviews or sharing personal data.
- **Do not redistribute without permission.** Consuming agents SHOULD NOT forward, republish, or share the Agent Resume document with third parties without explicit permission from the candidate.
- **Do not cache indefinitely.** Agents SHOULD respect the `agentInstructions.updateFrequency` field and re-fetch the document at the indicated interval. Stale data leads to poor matching.

### Regulatory Compliance

- **GDPR (EU/EEA).** Processing an Agent Resume constitutes processing personal data under GDPR. Data controllers must have a lawful basis for processing, must respect data subject rights (access, rectification, erasure), and must not retain data beyond its useful life.
- **CCPA (California).** Similar protections apply for California residents. Candidates have the right to know what data is collected and to request deletion.
- **Local employment law.** Some jurisdictions restrict the collection or use of salary history information. Consuming agents must comply with applicable local regulations regardless of what data the candidate has chosen to include.

### Transport Security

- Agent Resume documents SHOULD be served over HTTPS.
- When stored, Agent Resume documents SHOULD be encrypted at rest if they contain compensation or other sensitive data.
- API endpoints that accept or serve Agent Resume documents SHOULD require authentication.

---

## Appendix A: Minimal Valid Document

The following is a minimal valid Agent Resume document containing only required sections:

```json
{
  "meta": {
    "format": "agent-resume",
    "version": "1.0.0",
    "lastUpdated": "2026-01-01"
  },
  "person": {
    "fullName": "Jane Doe",
    "headline": "Software Engineer",
    "email": "jane@example.com"
  },
  "location": {
    "city": "San Francisco",
    "country": "United States",
    "countryCode": "US",
    "timezone": {
      "iana": "America/Los_Angeles",
      "standard": {
        "abbreviation": "PST",
        "utcOffset": "-08:00"
      }
    }
  },
  "summary": "Software engineer with 5 years of experience in backend systems and distributed computing.",
  "availability": {
    "status": "open_to_opportunities",
    "types": ["full_time", "contract"],
    "contactMethod": "email"
  },
  "skills": {
    "primary": ["Go", "Python", "Distributed Systems", "Kubernetes"]
  },
  "experience": [
    {
      "role": "Senior Software Engineer",
      "organization": "Acme Corp",
      "startDate": "2021-03-01",
      "current": true
    }
  ],
  "education": [
    {
      "degree": "BSc",
      "field": "Computer Science",
      "institution": "UC Berkeley"
    }
  ]
}
```

---

## Appendix B: Full Document Example

For a complete, real-world example of the Agent Resume Standard, see [`resume-agent.json`](../resume-agent.json) in the root of this repository.
