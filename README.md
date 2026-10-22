# Support operations automation portfolio

I build tools that reduce repetitive work in support operations, from generating documents to coordinating incidents and helping agents find answers. This portfolio includes live automations, a working AI assistant pilot, and projects still in development or planning.

These are project write-ups. Company identifiers, private links, operational data, and deployment configuration have been omitted. Source code is not included.

## Projects

| Project | Status | Focus |
| --- | --- | --- |
| [Certificate generation](#certificate-generation) | Live | Generate consistent PDF certificates from form submissions |
| [Product update monitoring](#product-update-monitoring) | Live | Capture relevant Slack discussions and summarize updates |
| [Data access request tracking](#data-access-request-tracking) | Live | Coordinate case ownership, deadlines, and follow-up |
| [Service credit conversion](#service-credit-conversion) | Live | Calculate transfers between different hourly rates |
| [Outage coordination](#outage-coordination) | Live | Standardize incident notifications and resolution updates |
| [Emergency case handover](#emergency-case-handover) | Live | Reassign open work when an agent becomes unavailable |
| [Early outage signal detection](#early-outage-signal-detection) | Live | Flag possible incident patterns in support discussions |
| [Knowledge base assistant](#knowledge-base-assistant) | Pilot | Answer operational questions with cited sources |
| [Shift swap validation](#shift-swap-validation) | Pilot | Check requests before manager review |
| [Conversation workflow shortcuts](#conversation-workflow-shortcuts) | Proposed | Make routine actions easier to access |
| [Connected bug escalation](#connected-bug-escalation) | Proposed | Keep support cases and engineering discussions connected |
| [Shared case notes](#shared-case-notes) | Idea | Make relevant context available across conversations and teams |

## Live projects

### Certificate generation

The idea was to turn certificate preparation into a form submission, with a fixed template controlling the design.

| Before | After |
| --- | --- |
| Each certificate was edited manually in a design tool. Template changes could introduce formatting errors and require extra review. | A form submission produces a personalized PDF in a consistent format and saves it to a designated folder. |

#### How I built it

A Google Apps Script trigger reads the submitted recipient and completion details, copies a Google Slides template, and replaces its placeholders. The script applies the required formatting, exports the result as a PDF, saves it in Google Drive, and deletes the temporary Slides copy. A separate scheduled script handles folder cleanup.

Tools: Google Forms, Google Apps Script, Google Slides, Google Drive.

### Product update monitoring

The idea was to collect useful product discussions in one place and make them easier to review for a recurring team update.

| Before | After |
| --- | --- |
| Reviewing product updates meant searching Slack channels and reading mentions individually. Relevant replies inside older threads were easy to miss. | Mentions are collected in a searchable sheet, with summaries for discussions that contain meaningful product updates or clarifications. |

#### How I built it

A scheduled Apps Script scans selected Slack channels for group mentions and text references. It checks new messages and a limited set of recent threads, then retrieves the conversation context for each match.

The OpenAI API classifies whether the discussion contains a relevant product update. Relevant threads receive a summary of the change, expected behavior, and support guidance. Google Sheets stores the mention, timestamp, permalink, and summary. Saved checkpoints and duplicate checks limit repeat processing.

Tools: Slack API, Google Apps Script, Google Sheets, OpenAI API.

### Data access request tracking

The idea was to give data access requests a shared case record so ownership and follow-up would survive a handover between people.

| Before | After |
| --- | --- |
| Case context was spread across conversations, with ownership and deadlines tracked manually. Taking over a case required reconstructing its history. | Requests enter a central register with a unique ID, an owner, a due date, and a dedicated coordination thread. Status changes and reminders are recorded. |

#### How I built it

I connected email intake and an internal web form to a Google Sheets case register. Apps Script creates case IDs, calculates due dates, and opens a Slack thread for each request. Start and resolve actions record the responsible person and timestamp, while scheduled checks send reminders for unowned or unresolved cases.

The workflow includes signed action links, state-transition checks, script locking, and duplicate-ingestion protection. The sheet holds the case record; Slack supports coordination. Case review and customer responses remain with people.

Tools: Google Apps Script, HTML Service, Google Sheets, Gmail, Slack API.

### Service credit conversion

The idea was to replace repeated manual arithmetic with a small calculator for transferring unused service hours between different rates.

| Before | After |
| --- | --- |
| Agents calculated the value of unused hours, the new allocation, and any remaining balance by hand. | A web form returns the transferable hours, remaining credit, and original total value using the same calculation rules each time. |

#### How I built it

I built an HTML interface backed by Google Apps Script. It accepts the original hourly rate, the new rate, and the hours to transfer. The calculator checks that inputs are positive, calculates the original value, converts it into whole hours at the new rate, and returns the residual balance. Monetary outputs use consistent rounding.

Tools: Google Apps Script, HTML.

### Outage coordination

The idea was to use one structured submission to create the incident record and send the appropriate updates to each audience.

| Before | After |
| --- | --- |
| Incident announcements were written and posted separately. Agents could miss updates, and resolution communication varied between incidents. | One submission creates a tracked outage, sends channel updates and direct notifications, and provides a consistent resolution process. |

#### How I built it

A Slack workflow writes intake data to a staging sheet. Apps Script validates the entry, normalizes timestamps to UTC, generates an outage ID, and saves the main record in Google Sheets.

The Slack API posts detailed instructions and audience-specific summaries, then sends direct notifications in batches. A web form records resolution details, updates the case, and sends the closing notifications. Duplicate resolution attempts are blocked.

The OpenAI API produces short summaries. Fallback logic allows the notification workflow to continue if summary generation fails.

Tools: Slack Workflow Builder, Slack API, Google Sheets, Google Apps Script, HTML Service, OpenAI API.

### Emergency case handover

The idea was to let an agent request a handover quickly while allowing time for a brief disruption to resolve before moving their work.

| Before | After |
| --- | --- |
| Another person had to notice the request, change the agent's availability, and reassign open work manually. Brief interruptions could trigger unnecessary redistribution. | The agent is marked unavailable immediately. After a configurable grace period, the system reassigns open work only if the agent is still unavailable. |

#### How I built it

Zapier passes a Slack trigger to an Apps Script webhook. The script validates the request, skips duplicate triggers, and uses a mapping sheet to resolve the agent's Intercom identity and destination queues.

It sets the agent to Away and schedules a later availability check. If the agent has returned, the handover is skipped. Otherwise, open conversations and back-office tickets move to the mapped queues. The outcome is logged, with optional Slack confirmations.

Credentials are stored in Script Properties and excluded from logged payloads. A configuration switch can disable Intercom changes.

Tools: Slack, Zapier, Google Apps Script, Google Sheets, Intercom API, Slack API.

### Early outage signal detection

The idea was to catch informal reports of recurring issues and bring available agents into the discussion sooner.

| Before | After |
| --- | --- |
| Possible incident patterns depended on someone noticing similar reports and escalating them manually. | The automation flags possible signals and asks available agents to review them in the original Slack thread. |

#### How I built it

A scheduled Apps Script reads new messages from a selected Slack channel. The OpenAI API classifies whether each message suggests an outage or a pattern of related issues.

For a flagged message, the script checks availability in Intercom, matches eligible agents to their Slack identities, and posts a thread reply mentioning them. Saved checkpoints limit repeat processing, and a lock prevents overlapping runs. Missing mappings are skipped.

The classifier supplies a signal for review. The automation sends the notification; people determine whether an incident needs escalation.

Tools: Slack API, Google Apps Script, Google Sheets, OpenAI API, Intercom API.

## Pilot

### Knowledge base assistant

The idea was to let support staff ask operational questions in plain language and receive an answer they could check against the original articles.

| Before | After in the pilot |
| --- | --- |
| Agents searched articles manually or interrupted a colleague for guidance. Reviewing a case and finding the relevant documentation were separate tasks. | A Slack assistant returns guidance with cited sources, supports follow-up questions, and can include case context in configured channels. |

#### How I built it

An Apps Script webhook receives questions from configured Slack channels. The OpenAI API expands the search query, and the system retrieves content from an indexed knowledge base and public help articles. The answer uses that content together with the thread history and, where applicable, a referenced Intercom conversation or ticket.

The article index uses text chunks and change detection. Retrieval rules prioritize authoritative articles, while alias expansion helps match shorthand to the terms used in documentation. A separate reference lookup supports questions about conversation tags.

The response appears in the Slack thread with source links. Google Sheets records interactions, retrieval diagnostics, and feedback from reactions. Duplicate-message checks prevent repeated processing, and failed case lookups can fall back to a documentation-only answer.

Tools: Google Apps Script, Slack API, OpenAI API, Intercom API, Google Sheets.

The Slack version is a working pilot. Wider rollout and an assistant embedded in the support inbox remain planned. The assistant provides guidance; it does not change cases or send customer responses.

## In development

### Shift swap validation

The current focus is a standalone request and validation flow. An interface demo exists, but the feature is still in progress. The broader workforce portal remains in planning.

#### Before

Managers checked swap requests manually against scheduling rules. Availability information and announcements were spread across separate forms, sheets, and messages.

#### Idea

Check a swap request before it reaches a manager, collect the other participant's acceptance, and keep the final decision with the manager. Validation would cover eligibility, date ranges, and scheduling constraints.

#### Intended outcome

Managers would review requests that have passed the initial checks and have been accepted by both participants. Applicants would receive status updates, and each request would have a shared history. A wider portal could later bring availability and announcements into the same place.

## Proposed projects and ideas

### Conversation workflow shortcuts

Status: Proposed.

#### Before

Frequently used actions require repeated navigation inside the support inbox. Agents may perform the same steps manually when shortcuts are hard to find.

#### Idea

Put common actions in a small sidebar panel beside the active conversation, with the available actions tailored to the user's role.

#### Intended outcome

Agents would complete routine actions with fewer clicks, see confirmation in the same panel, and have an action log for reviewing usage. The scope of each shortcut still needs to be checked against the platform's supported capabilities.

### Connected bug escalation

Status: Proposed.

#### Before

Reporting a bug requires copying context from a support conversation into an engineering discussion. Follow-up questions and status updates then move between the two systems manually.

#### Idea

Connect the support case to its engineering discussion, with a report that an agent reviews before submission and a visible history of earlier reports.

#### Intended outcome

Reports would follow a consistent format, follow-up questions would return to the support case, and agents could see previous reports without searching old conversations. Cases awaiting action could be routed to an available agent when the current owner is away. Any AI-generated draft would require human review before posting.

### Shared case notes

Status: Idea.

#### Before

Useful context sits inside individual conversations or separate team tools. Understanding a returning customer's history requires searching across those records.

#### Idea

Create a shared notes space linked to an internal record identifier, accessible from the support inbox and through a separate interface for other authorized teams.

#### Intended outcome

Teams would find relevant context across conversations without maintaining separate copies. Access permissions and retention requirements need to be agreed before development.
