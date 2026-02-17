# Field Reference -- Agent Resume Standard v1.0.0

This document provides a comprehensive, field-by-field reference for every property in the Agent Resume Standard schema. It is intended for developers building tools that produce or consume `agent-resume.json` files, and for candidates who want to understand what each field means and why it exists.

---

## `meta`

Metadata about the document itself. Agents use this section to verify they are reading the correct format, check the schema version, and locate alternate renderings of the same resume.

| Path | Type | Required | Description | Example | Rationale |
|------|------|----------|-------------|---------|-----------|
| `meta.format` | `string` | Yes | Fixed identifier that marks this file as an agent resume. Must be `"agent-resume"`. | `"agent-resume"` | Allows agents to detect this format programmatically without relying on file naming conventions alone. |
| `meta.version` | `string` (semver) | Yes | The semantic version of the Agent Resume Standard this document conforms to. | `"1.0.0"` | Enables forward-compatible parsing. Agents can handle schema changes by switching on the major version number. |
| `meta.lastUpdated` | `string` (ISO 8601 date) | Yes | The date the candidate last reviewed and updated this file. | `"2026-02-17"` | Tells agents how stale the data might be. An agent matching on compensation or availability can weight freshness accordingly. |
| `meta.canonical` | `string` (URI) | No | The canonical URL where the definitive version of this resume lives. | `"https://danielrosehill.com/about/cv"` | Prevents duplicate-matching when the same resume appears at multiple URLs. Agents should deduplicate by canonical URI. |
| `meta.alternateFormats` | `object` | No | A map of format names to URLs where the resume is available in other formats. Common keys: `html`, `pdf`, `json`, `markdown`. | `{ "pdf": "https://example.com/resume.pdf" }` | Lets agents retrieve a human-readable version for recruiter review, or a raw JSON version for programmatic processing, without scraping. |

---

## `person`

Core identity and contact information for the candidate.

| Path | Type | Required | Description | Example | Rationale |
|------|------|----------|-------------|---------|-----------|
| `person.fullName` | `string` | Yes | The candidate's full legal or professional name. | `"Daniel Rosehill"` | Primary identifier for the candidate across all systems. |
| `person.headline` | `string` | Yes | A short professional title or tagline (one line). | `"AI Developer & Technical Communications Professional"` | Gives agents and recruiters a quick summary of what the candidate does, useful for search result displays and initial filtering. |
| `person.website` | `string` (URI) | No | The candidate's personal or portfolio website. | `"https://danielrosehill.com"` | Allows agents to fetch additional context, verify identity, or present a link to recruiters. |
| `person.email` | `string` (email) | No | A contact email address. Should be one the candidate is comfortable sharing publicly. | `"public@danielrosehill.com"` | Primary communication channel. Agents should respect this as the initial outreach method unless `communication.preferredMethod` says otherwise. |
| `person.linkedin` | `string` (URI) | No | LinkedIn profile URL. | `"https://linkedin.com/in/danielrosehill/"` | Cross-references identity on the dominant professional network. Useful for recruiters who want to verify the profile. |
| `person.github` | `string` (URI) | No | GitHub profile URL. | `"https://github.com/danielrosehill"` | Signals technical activity. Agents can crawl public repositories to validate skills claims. |
| `person.huggingface` | `string` (URI) | No | Hugging Face profile URL. | `"https://huggingface.co/danielrosehill"` | Relevant for AI/ML candidates. Agents can verify model, dataset, and Spaces contributions. |
| `person.business.name` | `string` | No | The name of the candidate's business entity, if they operate through one. | `"DSR Holdings"` | Relevant for contract and consulting engagements where invoicing goes through a business entity rather than an individual. |
| `person.business.url` | `string` (URI) | No | The website of the candidate's business entity. | `"https://dsrholdings.cloud"` | Lets agents and recruiters verify the business entity. |

---

## `location`

Where the candidate is based and how they relate to geography.

| Path | Type | Required | Description | Example | Rationale |
|------|------|----------|-------------|---------|-----------|
| `location.city` | `string` | Yes | The candidate's primary city of residence. | `"Jerusalem"` | Enables geo-based matching for hybrid or onsite roles, and gives context for timezone inference. |
| `location.country` | `string` | Yes | The candidate's country of residence (full name). | `"Israel"` | Country-level filtering for roles with jurisdictional or tax requirements. |
| `location.countryCode` | `string` (ISO 3166-1 alpha-2) | No | Two-letter country code per the ISO 3166-1 standard. | `"IL"` | Machine-parseable country identifier. Avoids ambiguity from country name variations (e.g., "USA" vs. "United States" vs. "US"). |
| `location.timezone.iana` | `string` | Yes | The IANA timezone identifier for the candidate's primary location. | `"Asia/Jerusalem"` | Enables precise overlap calculations between candidate and team working hours. |
| `location.timezone.standard.abbreviation` | `string` | No | Abbreviation for the standard (non-DST) timezone. | `"IST"` | Human-readable timezone label for display purposes. |
| `location.timezone.standard.utcOffset` | `string` | No | UTC offset during standard time. | `"+02:00"` | Allows quick offset math without requiring a timezone database. |
| `location.timezone.daylight.abbreviation` | `string` | No | Abbreviation for the daylight saving timezone. | `"IDT"` | Human-readable DST label. |
| `location.timezone.daylight.utcOffset` | `string` | No | UTC offset during daylight saving time. | `"+03:00"` | Enables correct overlap calculations during DST periods. |
| `location.timezone.daylight.start` | `string` (ISO date) | No | Date when daylight saving time begins for the current year. | `"2026-03-27"` | Lets agents determine which offset applies on a given date. |
| `location.timezone.daylight.end` | `string` (ISO date) | No | Date when daylight saving time ends for the current year. | `"2026-10-25"` | Paired with `daylight.start` to define the DST window. |
| `location.openToRemote` | `boolean` | No | Whether the candidate is open to remote work. | `true` | Quick filter for remote-first searches. More nuanced preferences live in `workPreferences.remotePreference`. |

---

## `summary`

| Path | Type | Required | Description | Example | Rationale |
|------|------|----------|-------------|---------|-----------|
| `summary` | `string` | Yes | A free-text professional summary, typically 2-4 sentences. | `"AI developer and technical communications professional with over a decade of experience..."` | Serves the same purpose as a resume summary section. Agents can use it for semantic similarity matching against job descriptions. It is the primary field for natural-language understanding of the candidate. |

---

## `availability`

The candidate's current job search status and how to reach them.

| Path | Type | Required | Description | Example | Rationale |
|------|------|----------|-------------|---------|-----------|
| `availability.status` | `string` (enum) | Yes | Current job search status. One of: `actively_searching`, `open_to_opportunities`, `not_looking`, `employed_open`. | `"open_to_opportunities"` | The single most important signal for whether to contact this candidate. `actively_searching` means they are actively looking and welcome outreach. `open_to_opportunities` means they are employed but receptive. `not_looking` means do not contact for roles. `employed_open` means currently employed and passively open. |
| `availability.types` | `array` of `string` | No | Types of work the candidate is currently open to. Free-form strings describing engagement categories. | `["consulting", "contract", "technical_documentation"]` | Quick filter before diving into the more detailed `engagementPreferences` section. |
| `availability.contactMethod` | `string` (enum) | No | Preferred initial contact method. One of: `email`, `linkedin`, `phone`, `calendar`, `form`, `other`. | `"email"` | Tells agents which channel to use for first outreach, increasing response likelihood. |
| `availability.contactUrl` | `string` (URI) | No | A URL for the preferred contact method (e.g., a contact form, calendar booking link, or LinkedIn profile). | `"https://danielrosehill.com/contact"` | Provides a direct, actionable link for outreach. |

---

## `compensation`

Transparent compensation expectations. This section exists to eliminate wasted time on both sides by enabling budget-to-expectation matching before any conversation begins.

| Path | Type | Required | Description | Example | Rationale |
|------|------|----------|-------------|---------|-----------|
| `compensation.hourlyRate.min` | `number` | No | Minimum acceptable hourly rate. | `100` | Sets the floor for hourly engagements. Agents should not match roles that pay below this. |
| `compensation.hourlyRate.max` | `number` | No | Maximum expected hourly rate. | `175` | Sets the ceiling. Useful for clients with higher budgets to understand the candidate's range. |
| `compensation.hourlyRate.currency` | `string` (ISO 4217) | No | Currency code for the hourly rate. | `"USD"` | Prevents ambiguity. Agents should normalize to a common currency for cross-candidate comparison. |
| `compensation.hourlyRate.preferred` | `number` | No | The candidate's preferred hourly rate within their range. | `150` | Gives agents a target rate for quick matching. If a role's budget is near this number, it is a strong fit. |
| `compensation.annualSalary.min` | `number` | No | Minimum acceptable annual salary. | `120000` | Enables salary-range matching for full-time roles. |
| `compensation.annualSalary.max` | `number` | No | Maximum expected annual salary. | `180000` | Upper bound for salary expectations. |
| `compensation.annualSalary.currency` | `string` (ISO 4217) | No | Currency code for the annual salary. | `"USD"` | Ensures correct currency interpretation. |
| `compensation.projectMinimum.amount` | `number` | No | Minimum project size the candidate will accept. | `5000` | Filters out engagements that are too small to be worthwhile. Agents should skip roles below this threshold. |
| `compensation.projectMinimum.currency` | `string` (ISO 4217) | No | Currency code for the project minimum. | `"USD"` | Paired with `amount` for unambiguous interpretation. |
| `compensation.negotiable` | `boolean` | No | Whether the stated compensation figures are negotiable. | `true` | Signals flexibility. If `true`, agents can present roles slightly outside the stated range. |
| `compensation.notes` | `string` | No | Free-text notes about compensation expectations. | `"Rates vary based on project scope..."` | Captures nuance that structured fields cannot, such as discount policies for retainers or non-profit rates. |

---

## `experienceSummary`

A structured overview of the candidate's career trajectory, designed for quick programmatic evaluation without parsing individual experience entries.

| Path | Type | Required | Description | Example | Rationale |
|------|------|----------|-------------|---------|-----------|
| `experienceSummary.totalYears` | `number` | No | Total years of professional experience. | `13` | The most common filter in recruitment. Enables instant seniority-level matching. |
| `experienceSummary.careerStartYear` | `number` | No | The year the candidate began their professional career. | `2013` | Allows agents to independently verify `totalYears` and understand career vintage. |
| `experienceSummary.domainExperience` | `array` of `object` | No | Breakdown of experience by domain area. Each entry has `domain`, `years`, and `level`. | See below. | Enables matching at the domain level rather than just job titles. A candidate may have 13 total years but only 5 in AI/ML. |
| `experienceSummary.domainExperience[].domain` | `string` | Yes (within array) | The name of the domain or specialization. | `"Technical Communications"` | Free-form domain label. Agents should use fuzzy matching or keyword overlap. |
| `experienceSummary.domainExperience[].years` | `number` | Yes (within array) | Years of experience in this domain. | `13` | Enables domain-specific seniority filtering. |
| `experienceSummary.domainExperience[].level` | `string` (enum) | Yes (within array) | Self-assessed proficiency level. One of: `beginner`, `intermediate`, `advanced`, `expert`. | `"expert"` | Adds a qualitative dimension beyond raw years. Three years at `expert` level (intense focus) may outweigh ten years at `intermediate` (occasional involvement). |
| `experienceSummary.seniorityLevel` | `string` (enum) | No | Overall seniority level. One of: `junior`, `mid`, `senior`, `lead`, `principal`, `executive`. | `"senior"` | A single field that maps to the seniority tiers most job postings use. Agents can filter on this directly. |

---

## `workPreferences`

How the candidate prefers to work. These fields enable compatibility matching between a candidate's work style and a team's culture.

| Path | Type | Required | Description | Example | Rationale |
|------|------|----------|-------------|---------|-----------|
| `workPreferences.remotePreference` | `string` (enum) | No | The candidate's remote work preference. One of: `remote_only`, `remote_preferred`, `hybrid`, `onsite`. | `"remote_only"` | The most common work-style filter. `remote_only` means the candidate will not consider roles requiring any onsite presence. `remote_preferred` means remote is preferred but hybrid is acceptable. `hybrid` means a mix is expected. `onsite` means the candidate prefers or requires onsite. |
| `workPreferences.asyncPreference` | `string` (enum) | No | Preference for asynchronous vs. synchronous communication. One of: `async_only`, `async_preferred`, `balanced`, `sync_preferred`. | `"async_preferred"` | Signals cultural fit. An `async_preferred` candidate may struggle in a team that runs six hours of daily meetings. |
| `workPreferences.communicationTools` | `array` of `string` | No | Tools the candidate is comfortable using for work communication. | `["Email", "Slack", "Google Meet", "Zoom"]` | Practical compatibility check. If a team is all-in on Microsoft Teams and the candidate has never used it, this surfaces that gap early. |
| `workPreferences.meetingTolerance` | `string` (enum) | No | How many meetings the candidate is comfortable with. One of: `minimal`, `moderate`, `high`. | `"minimal"` | `minimal` means the candidate strongly prefers few meetings (perhaps 1-3 per week). `moderate` means a normal meeting load is fine. `high` means the candidate is comfortable in meeting-heavy cultures. |
| `workPreferences.typicalWorkingHours.start` | `string` (HH:MM) | No | When the candidate typically starts their workday. | `"09:00"` | Combined with `end` and `timezone`, agents can compute overlap windows with team members in other timezones. |
| `workPreferences.typicalWorkingHours.end` | `string` (HH:MM) | No | When the candidate typically ends their workday. | `"18:00"` | Paired with `start` for overlap calculations. |
| `workPreferences.typicalWorkingHours.timezone` | `string` (IANA) | No | Timezone for the working hours. | `"Asia/Jerusalem"` | Anchors the start/end times to a specific timezone for accurate overlap math. |
| `workPreferences.timezoneOverlapMinHours` | `number` | No | Minimum number of overlapping working hours the candidate requires with the team. | `3` | Enables agents to filter out roles where timezone overlap would be insufficient. A candidate requiring 3 hours of overlap with a US Pacific team can be evaluated instantly. |

---

## `engagementPreferences`

What types of engagements the candidate is looking for, and what they want to avoid.

| Path | Type | Required | Description | Example | Rationale |
|------|------|----------|-------------|---------|-----------|
| `engagementPreferences.preferredTypes` | `array` of `string` (enum) | No | The engagement types the candidate prefers. Enum values: `full_time`, `part_time`, `consulting`, `contract`, `fractional`, `project_based`, `retainer`, `advisory`. | `["consulting", "contract", "fractional"]` | Enables agents to match candidates to the correct engagement structure. A candidate who only wants `consulting` should not be shown full-time roles. |
| `engagementPreferences.preferredIndustries` | `array` of `string` | No | Industries the candidate is particularly interested in working with. | `["AI/ML", "Developer Tools", "Open Source"]` | Positive industry targeting. Agents can boost match scores for roles in these industries. |
| `engagementPreferences.avoidIndustries` | `array` of `string` | No | Industries the candidate does not want to work in. | `["Gambling", "Tobacco"]` | Hard exclusion filter. Agents must not present roles in these industries. |
| `engagementPreferences.idealProjectDuration.min` | `number` | No | Minimum project duration the candidate prefers. | `1` | Combined with `unit`, filters out engagements that are too short. |
| `engagementPreferences.idealProjectDuration.max` | `number` | No | Maximum project duration the candidate prefers. | `12` | Combined with `unit`, filters out engagements that are too long. |
| `engagementPreferences.idealProjectDuration.unit` | `string` (enum) | No | Time unit for project duration. One of: `days`, `weeks`, `months`, `years`. | `"months"` | Anchors the min/max values to a concrete time unit. |
| `engagementPreferences.noticePeriod` | `string` | No | How much notice the candidate needs before starting a new engagement. | `"2 weeks"` | Tells agents when the candidate can realistically begin. A role that needs someone tomorrow should deprioritize candidates with a 3-month notice period. |
| `engagementPreferences.teamSizePreference` | `string` (enum) | No | Preferred team size. One of: `solo`, `small`, `medium`, `large`, `any`. | `"small"` | Cultural fit signal. `solo` means the candidate works best alone. `small` typically means 2-10 people. `medium` means 10-50. `large` means 50+. `any` means no preference. |
| `engagementPreferences.dealBreakers` | `array` of `string` | No | Conditions that would cause the candidate to reject an opportunity outright. | `["Onsite-only requirement", "Micromanagement culture"]` | Hard exclusion filters expressed in natural language. Agents should compare these against known attributes of the role and flag conflicts. |

---

## `portfolio`

Links to the candidate's public work and contributions.

| Path | Type | Required | Description | Example | Rationale |
|------|------|----------|-------------|---------|-----------|
| `portfolio.github.url` | `string` (URI) | No | GitHub profile URL. | `"https://github.com/danielrosehill"` | Entry point for agents to crawl public repositories and verify technical contributions. |
| `portfolio.github.publicRepos` | `number` | No | Approximate number of public repositories. | `400` | Quick signal of activity level. Agents can use this as a proxy for open-source engagement. |
| `portfolio.github.contributions` | `string` | No | Free-text description of GitHub contribution patterns. | `"Active daily contributor across AI tooling..."` | Adds qualitative context that raw repo counts cannot convey. |
| `portfolio.huggingFace.url` | `string` (URI) | No | Hugging Face profile URL. | `"https://huggingface.co/danielrosehill"` | Entry point for AI/ML portfolio verification. |
| `portfolio.huggingFace.datasets` | `number` | No | Approximate number of published datasets. | `50` | Signals data-centric AI expertise. |
| `portfolio.huggingFace.spaces` | `number` | No | Approximate number of Hugging Face Spaces (demos/apps). | `10` | Signals hands-on deployment experience with ML models. |
| `portfolio.huggingFace.models` | `number` | No | Approximate number of published models. | `2` | Signals model training and publishing experience. |
| `portfolio.npm.url` | `string` (URI) | No | npm profile URL. | `"https://www.npmjs.com/~danielrosehill"` | Entry point for JavaScript/Node.js package verification. |
| `portfolio.npm.packages` | `number` | No | Approximate number of published npm packages. | `5` | Signals JavaScript ecosystem contributions. |
| `portfolio.publications` | `array` of `object` | No | Notable publications, open-source projects, articles, or other public works. | See below. | Provides verifiable evidence of expertise and thought leadership. |
| `portfolio.publications[].title` | `string` | Yes (within array) | Title of the publication or project. | `"Awesome LLM Second Brains"` | Human-readable identifier for the work. |
| `portfolio.publications[].url` | `string` (URI) | Yes (within array) | URL where the publication can be accessed. | `"https://github.com/danielrosehill/awesome-llm-second-brains"` | Verifiable link to the actual work. |
| `portfolio.publications[].type` | `string` (enum) | No | Type of publication. Suggested values: `open_source`, `article`, `paper`, `book`, `talk`, `video`, `podcast`, `other`. | `"open_source"` | Enables agents to filter by publication type (e.g., show only research papers, or only open-source projects). |
| `portfolio.speakingEngagements` | `array` of `object` | No | Public speaking engagements, conference talks, and presentations. | See below. | Evidence of thought leadership and communication skills. |
| `portfolio.speakingEngagements[].event` | `string` | Yes (within array) | Name of the event or conference. | `"AI Dev Summit 2025"` | Identifies the venue and its prestige. |
| `portfolio.speakingEngagements[].topic` | `string` | Yes (within array) | Topic or title of the talk. | `"Building Multi-Agent Systems with MCP"` | Signals domain expertise in the specific topic area. |

---

## `languages`

Human languages the candidate speaks.

| Path | Type | Required | Description | Example | Rationale |
|------|------|----------|-------------|---------|-----------|
| `languages[].language` | `string` | Yes (within array) | Name of the language. | `"English"` | Human-readable language name. |
| `languages[].proficiency` | `string` (enum) | Yes (within array) | Proficiency level per the ILR scale. One of: `elementary`, `limited_working`, `professional_working`, `full_professional`, `native`. | `"native"` | Standard proficiency scale that agents can compare against role requirements. `elementary` = basic phrases. `limited_working` = routine social demands. `professional_working` = practical needs. `full_professional` = fluent in professional contexts. `native` = native or bilingual. |
| `languages[].iso639` | `string` (ISO 639-1) | No | Two-letter ISO 639-1 language code. | `"en"` | Machine-parseable language identifier. Avoids ambiguity from language name variations. |

---

## `matchCriteria`

Explicit signals about what the candidate is and is not looking for. This is the most agent-actionable section in the entire document.

| Path | Type | Required | Description | Example | Rationale |
|------|------|----------|-------------|---------|-----------|
| `matchCriteria.lookingFor` | `array` of `string` | No | Free-text descriptions of the types of work, roles, or projects the candidate actively wants. | `["AI agent development and orchestration projects", "Technical documentation for AI/ML products"]` | Positive matching targets. Agents should compare these against job descriptions using semantic similarity. High overlap = strong match. |
| `matchCriteria.notLookingFor` | `array` of `string` | No | Free-text descriptions of what the candidate explicitly does not want. | `["Pure frontend web development", "Entry-level positions"]` | Negative filters. Agents should check for overlap between these and the role description. Any match should reduce the score or exclude the candidate. |
| `matchCriteria.idealEngagement` | `string` | No | A free-text description of the candidate's ideal engagement in narrative form. | `"A remote consulting or fractional role focused on AI agent orchestration..."` | The richest signal for semantic matching. Agents can compute embedding similarity between this paragraph and a job description to produce a fit score. |
| `matchCriteria.keywords` | `array` of `string` | No | Keywords and phrases relevant to the candidate's expertise and job search. | `["AI agents", "LLM", "MCP", "prompt engineering"]` | Enables keyword-based matching for systems that do not support semantic search. Also useful for index construction and search engine optimization. |

---

## `communication`

How and when to reach the candidate.

| Path | Type | Required | Description | Example | Rationale |
|------|------|----------|-------------|---------|-----------|
| `communication.preferredMethod` | `string` (enum) | No | The candidate's preferred communication method. One of: `email`, `linkedin`, `phone`, `slack`, `calendar`, `other`. | `"email"` | First-contact protocol. Agents should use this method for initial outreach to maximize response rates. |
| `communication.responseTimeSLA` | `string` | No | How quickly the candidate typically responds to messages. | `"24-48 hours"` | Sets expectations for recruiters. Also useful for agents scheduling follow-ups. |
| `communication.bestTimeToReach.start` | `string` (HH:MM) | No | Start of the best window to reach the candidate. | `"09:00"` | Avoids contacting the candidate at inconvenient times. |
| `communication.bestTimeToReach.end` | `string` (HH:MM) | No | End of the best window to reach the candidate. | `"17:00"` | Paired with `start` to define the contact window. |
| `communication.bestTimeToReach.timezone` | `string` (IANA) | No | Timezone for the best-time-to-reach window. | `"Asia/Jerusalem"` | Anchors the time window to a specific timezone. |
| `communication.calendarBookingUrl` | `string` (URI) or `null` | No | A URL where recruiters can book time on the candidate's calendar directly. | `"https://cal.com/danielrosehill"` | Eliminates scheduling back-and-forth. If present, agents can include this link in outreach messages. `null` means no calendar booking is available. |

---

## `values`

| Path | Type | Required | Description | Example | Rationale |
|------|------|----------|-------------|---------|-----------|
| `values` | `array` of `string` | No | A list of professional values, principles, or cultural priorities the candidate holds. | `["Open source", "Async-first communication", "Transparency and directness"]` | Enables cultural fit matching. Agents can compare these against a company's stated values or culture page to produce an alignment score. Also useful for candidates who want to signal what matters to them beyond technical skills. |

---

## `skills`

Structured skills data organized by category.

| Path | Type | Required | Description | Example | Rationale |
|------|------|----------|-------------|---------|-----------|
| `skills.primary` | `array` of `string` | Yes | The candidate's core, highest-confidence skills. These are the skills the candidate leads with. | `["AI Agent Orchestration", "Workflow Automation", "Prompt Engineering"]` | Primary matching vector. Agents should weight these heavily when scoring candidates against job requirements. |
| `skills.secondary` | `array` of `string` | No | Supporting skills that complement the primary skill set. | `["Linux Systems", "Python", "Cloud & Docker"]` | Secondary matching vector. These skills add value but are not the candidate's primary selling point. |
| `skills.tools` | `object` of `string` arrays | No | Specific tools, platforms, and technologies the candidate uses, grouped by category. The keys are category names and the values are arrays of tool names. | `{ "aiMl": ["Claude Code", "Anthropic API"], "cloud": ["Docker", "Kubernetes"] }` | Enables precise tool-level matching. If a job requires experience with "n8n" or "LangChain," agents can check this object directly. The category grouping aids human readability. |

---

## `experience`

A chronological (or reverse-chronological) list of professional experience entries.

| Path | Type | Required | Description | Example | Rationale |
|------|------|----------|-------------|---------|-----------|
| `experience[].role` | `string` | Yes (within array) | Job title or role name. | `"Independent Consultant"` | Primary identifier for the position. Enables role-title matching against job requirements. |
| `experience[].organization` | `string` | Yes (within array) | Name of the employer or client organization. | `"DSR Holdings"` | Identifies the organization. Agents can cross-reference for company reputation or industry. |
| `experience[].url` | `string` (URI) | No | URL of the organization's website. | `"https://dsrholdings.cloud"` | Allows agents or recruiters to verify the organization exists and understand its domain. |
| `experience[].startDate` | `string` (ISO 8601 date) | Yes (within array) | Start date of the position. | `"2015-01-01"` | Enables tenure calculation and timeline construction. |
| `experience[].endDate` | `string` (ISO 8601 date) or `null` | No | End date of the position. `null` for current positions. | `"2026-02-01"` or `null` | Paired with `startDate` for duration calculation. `null` signals a current position. |
| `experience[].current` | `boolean` | No | Whether this is the candidate's current position. | `true` | Explicit current-position flag. Redundant with `endDate: null` but simpler for agents to check. |
| `experience[].location` | `string` | No | Location where the work was performed. | `"Jerusalem, Israel"` | Provides geographic context for the role. |
| `experience[].type` | `string` (enum) | No | The employment type. Suggested values: `full_time`, `part_time`, `contract`, `freelance`, `self_employed`, `internship`, `full_time_then_contract`, `other`. | `"self_employed"` | Enables agents to understand the nature of the engagement. A candidate with ten years of `self_employed` experience is different from one with ten years of `full_time` experience at a single company. |
| `experience[].summary` | `string` | No | Free-text description of responsibilities and accomplishments. | `"Technical communications and AI consulting for businesses..."` | The richest field per experience entry. Agents can use semantic matching against job descriptions. |
| `experience[].tags` | `array` of `string` | No | Machine-friendly tags describing the work. | `["ai", "consulting", "technical_writing"]` | Enables keyword-based filtering across experience entries. Faster than parsing `summary` text for every query. |

---

## `education`

Academic qualifications.

| Path | Type | Required | Description | Example | Rationale |
|------|------|----------|-------------|---------|-----------|
| `education[].degree` | `string` | Yes (within array) | The degree title or abbreviation. | `"MA"` | Identifies the qualification level. |
| `education[].field` | `string` | Yes (within array) | The field of study or major. | `"Political Journalism"` | Identifies the academic discipline. Agents can match against education requirements in job postings. |
| `education[].institution` | `string` | Yes (within array) | Name of the educational institution. | `"City University London"` | Identifies the institution for verification and prestige assessment. |
| `education[].location` | `string` | No | Location of the institution. | `"London, UK"` | Provides geographic context for the education. |

---

## `agentInstructions`

Meta-instructions for AI agents consuming this document. This section exists specifically to communicate with automated systems about how to handle the data responsibly.

| Path | Type | Required | Description | Example | Rationale |
|------|------|----------|-------------|---------|-----------|
| `agentInstructions.purpose` | `string` | No | A statement explaining what this document is for, aimed at AI agents. | `"This document is a machine-readable resume designed for AI agents performing talent search..."` | Provides explicit context for LLM-based agents that might otherwise be uncertain about the document's intent. Helps agents understand they should treat this as authoritative career data. |
| `agentInstructions.confidenceNotes` | `string` | No | Notes about the reliability of different data points in the document. | `"Experience dates and skills are accurate. Compensation ranges are estimates..."` | Tells agents which fields to trust absolutely and which to treat as approximate. Portfolio statistics change frequently; compensation is negotiable; but experience dates are firm. |
| `agentInstructions.updateFrequency` | `string` (enum) | No | How often the candidate plans to update this document. One of: `weekly`, `monthly`, `quarterly`, `annually`. | `"monthly"` | Lets agents decide how aggressively to re-fetch and re-index this document. A `weekly` update frequency justifies more frequent polling than `annually`. |
| `agentInstructions.humanVerification` | `string` | No | Instructions for what actions require human confirmation before an agent takes them. | `"Contact the candidate via email before making commitments, scheduling interviews, or sharing personal information with third parties."` | The most important ethical guardrail in the entire schema. Agents must respect this field. It defines the boundary between automated processing and actions that require explicit human consent. |
