# Support operations automation portfolio

I build tools that reduce repetitive work in support operations, from generating documents to coordinating incidents and helping agents find answers. I also build systems to evaluate changes to AI-assisted support. This portfolio includes live automations, working pilots, and projects still in development or planning.

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
| [AI support experiment system](#ai-support-experiment-system) | In use | Run controlled workflow tests and connect variants to decision-ready outcomes |
| [Knowledge base assistant](#knowledge-base-assistant) | Internal pilot | Answer operational questions with cited sources and case context |
| [Product team finder](#product-team-finder) | Pilot | Route support questions to the responsible product team using cited evidence |
| [Shift swap validation](#shift-swap-validation) | Working pilot | Validate and apply shift swaps with manager review |
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

## Pilots

### AI support experiment system

I wanted a dependable way to evaluate changes to AI-assisted support flows. A workflow change can improve one operational measure while increasing customer effort, reopens, or other unwanted outcomes. The system keeps the assignment, outcome analysis, data-quality checks, and final decision in one experiment record.

| Before | After |
| --- | --- |
| Aggregate reports showed overall performance, but did not provide the conversation-level comparison needed to isolate a workflow change. Reporting also included spreadsheet-based steps. | Each eligible conversation or user receives a stable variant. The assignment is connected to later outcomes, and a shared experiment structure records preparation, live monitoring, analysis, and the rollout decision. |

#### How I built it

I created a reusable experiment lifecycle with four stages: preparation, run, analysis, and decision. Preparation records the hypothesis, eligible population, assignment unit, primary measure, customer guardrails, observation window, and the rule for scaling or stopping the change.

Control and treatment paths are configured in the support workflow. A deterministic assignment rule uses the experiment identifier with a conversation or user identifier, depending on the behavior being tested. The original assignment is saved before routing and reused if the workflow is entered again. Invalid assignments follow the existing safe path and remain outside the comparison.

Assignments are joined to replicated conversation outcomes in Snowflake. Streamlit readouts show allocation, sample coverage, metric definitions, arm results, effect differences, confidence intervals, statistical tests, and data-quality checks. Where an existing experimentation platform is a better fit, I use it for the main result and keep Snowflake for supporting diagnostics.

My work covers experiment design, workflow implementation, assignment logic, quality assurance, SQL analysis, dashboard development, and the decision record. Statistical review is included before results are used for rollout decisions.

Tools: Intercom Fin, Snowflake, SQL, Streamlit.

#### Experiment portfolio

Status as of November 2026:

| Experiment area | Status | Focus |
| --- | --- | --- |
| AI response context | Active | Improve answer relevance |
| Escalation flow | Review pending | Reduce avoidable handoffs |
| Handoff quality | Completed; not scaled | Improve information available to agents |
| Workflow timing | Review pending | Reduce avoidable delays |
| Feedback collection | In preparation | Improve response coverage |
| Conversation closure | Planned | Reduce resolution time |

The experiments use different assignment units when the behavior requires it. Conversation-level tests keep each conversation in one arm. User-level tests keep the same person in one arm across later conversations. Each design records safe fallback behavior so an assignment or connector failure does not expose the customer to an incomplete treatment.

#### Measurement and validation

Each experiment has one primary outcome and a small set of customer or operational guardrails. The exact measures depend on the workflow, but they can include resolution, handling time, customer inputs, reopens, repeat contacts, customer effort, and customer happiness.

The official comparison keeps each valid unit in its original assigned group. Treatment delivery, workflow incidents, missing outcomes, source freshness, and duplicate records are checked separately so they do not silently change the comparison. Diagnostic views can explain a result, but they do not replace the original control-versus-treatment analysis.

One completed workflow experiment did not demonstrate the intended improvement and raised customer-experience concerns. The treatment was stopped instead of being scaled. The same lifecycle is now used for several experiments at different stages.

#### Current direction

- Automate more checks for assignment balance, missing outcomes, source freshness, and treatment delivery.
- Reuse the same lifecycle and reporting structure while keeping each experiment's population and measures explicit.
- Preserve workflow versions, assignments, incidents, results, and decisions after a test ends.

### Knowledge base assistant

The idea was to let support staff ask operational questions in plain language and receive an answer they could check against the original articles.

| Before | After in the internal pilot |
| --- | --- |
| Agents searched articles manually or interrupted a colleague for guidance. Reviewing a case and finding the relevant documentation were separate tasks. | A Slack assistant returns step-by-step guidance with cited sources, supports follow-up questions, and can include referenced conversation or ticket context in configured channels. |

#### How I built it

An Apps Script webhook receives questions from configured Slack channels. The OpenAI API expands the search query, and the system retrieves content from indexed internal articles, selected Confluence spaces, and public help articles. The answer uses that content with the Slack thread history and, where applicable, a referenced Intercom conversation or ticket.

The article index uses text chunks, scheduled refreshes, and change detection. Retrieval rules prioritize the current authoritative source when the same procedure appears in more than one system. Alias expansion helps match internal shorthand to the terms used in documentation. A separate reference lookup supports questions about conversation tags.

The response appears in the Slack thread with source links. Google Sheets records interactions, retrieval diagnostics, and feedback from reactions. Duplicate-message checks prevent repeated processing, and failed case lookups can fall back to a documentation-only answer.

Tools: Google Apps Script, Slack API, OpenAI API, Intercom API, Confluence API, Google Sheets.

The Slack version is in internal use with support staff, team leads, and onboarding colleagues. A wider rollout needs a more suitable hosted environment than the current pilot. The assistant provides guidance; it does not change cases or send customer responses.

### Product team finder

The idea was to help senior support agents identify which product team owns an issue without relying on memory or broad access to internal systems.

| Before | After in the pilot |
| --- | --- |
| Finding the right owner meant searching several pages or asking colleagues. Similar issues could be routed differently depending on who handled them. | A Slack assistant checks the support ownership map and related evidence, then suggests the responsible product team and its public internal channel. It gives a confidence level and asks for clarification when the evidence is incomplete. |

#### How I built it

I built a dedicated hosted Slack agent with the Cursor Agent SDK and a scoped, read-only Atlassian connection. Its instructions require it to check the ownership map first, follow the affected product flow, cite the source used for the recommendation, and avoid guessing when the available evidence is insufficient.

Follow-up questions stay in the same Slack thread. The assistant can read the approved ownership and documentation sources, but it cannot edit Atlassian content, submit reports, or route incidents automatically.

Tools: Cursor Agent SDK, Slack, Jira, Confluence.

This is a working pilot. Broader rollout and usage measurement are still in progress.

## In development

### Shift swap validation

The current version is a working pilot for workforce managers. I separated it from a broader workforce portal so the shift-swap process could be tested without waiting for the infrastructure needed for a larger product.

| Before | After in the pilot |
| --- | --- |
| Managers checked requests manually against scheduling rules, updated schedules themselves, and followed up with both agents. | Requests enter a shared queue with rule checks and a schedule preview. A manager makes the final decision, and an accepted swap is applied through the scheduling API with a saved snapshot for recovery if part of the update fails. |

#### How I built it

A Slack workflow records the request in Google Sheets. A domain-restricted Apps Script web app gives workforce managers a queue, history, search, rule results, and schedule previews.

The tool maps each agent to their email, operational level, Slack identity, and scheduling profile. It checks same-level eligibility, consecutive working days, minimum rest, overnight leftovers, monthly freeze rules, and ramp-up status. Planned leave that fully covers customer time is excluded from working-day and rest calculations. Managers can still accept a request when a rule produces a warning; the tool shows the warning instead of making the decision for them.

For an accepted request, the tool saves the existing schedules, applies the swap through the Assembled API, and restores the snapshot if the update fails. Both agents receive the decision by Slack direct message. Blocked requests remain visible in the manager queue instead of disappearing from the tool.

The remaining gap is reliable start-date data for the three-month ramp-up check. A bulk update and a new-hire process are needed so a missing start date cannot be treated as a passed check. The broader agent self-service portal remains outside the current pilot.

Tools: Slack Workflow Builder, Google Sheets, Google Apps Script, HTML Service, Assembled API, Slack API.

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
