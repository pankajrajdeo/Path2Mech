# What LangSmith Engine is

LangSmith Engine is an **opt-in AI agent built into LangSmith that improves other AI agents from their production telemetry**. It examines the traces already collected in a LangSmith project, groups recurring failures into issues, prioritizes them, diagnoses likely root causes, and recommends prompt or code changes. When a source repository is connected, it can prepare a GitHub pull request for review. It also proposes online evaluators and converts representative failing traces into offline dataset examples so the same failure can be detected during future testing. ([Docs by LangChain][1])

The intended loop is:

**Production traces → recurring issue → root-cause diagnosis → proposed fix or PR → evaluator → regression dataset → continued monitoring.**

Engine is currently presented by LangChain as a public-beta product and is designed to work on top of an existing LangSmith tracing setup rather than requiring a separate observability system. ([LangChain][2])

For self-hosted environments, LangChain’s current architecture documentation describes Engine orchestration operating in the customer’s VPC while model inference is reached through a private network connection to LangSmith Intelligence. On AWS, the documented path uses PrivateLink and models hosted through Bedrock, which corresponds closely to the architecture diagram in your second image. ([Docs by LangChain][3])

The following analysis is based on the 13 screenshots you supplied.

---

# Image 1 — Engine setup dialog

**10.31.41 AM**

## Screen purpose

This is the initial configuration experience for starting an Engine analysis. The interface is designed as a lightweight onboarding flow rather than a multi-step wizard: the user can configure optional context, select priorities, and initiate analysis from one screen.

## Layout and visual hierarchy

The screen uses a dark, presentation-style frame surrounding a large inner application card. The title, **“Engine — Find and fix your agent’s issues,”** sits prominently at the top with a small **Beta** badge. The typography creates a clear distinction between the product name and its value proposition.

The inner panel is divided into two major configuration sections:

1. **Repository connection**
2. **Analysis priorities**

A horizontal divider separates the sections, and a persistent-looking action footer sits at the bottom.

The composition follows a centered, single-column form layout. This is appropriate because the user is making a small number of sequential decisions and does not need side-by-side comparison.

## Repository-selection component

The first section is headed **“Connect your agent’s code repository”** and is explicitly labeled **Optional**. Supporting copy explains the benefit: the source code will be used to diagnose problems and generate fixes.

The repository control appears to be an asynchronous search/combobox:

* Search icon on the left
* Placeholder text for GitHub accounts and repositories
* Full-width input
* Secondary “Manage app access” link below

The link communicates an important dependency: the search can only return repositories that the GitHub application is authorized to access.

From a frontend perspective, this should be implemented as an accessible combobox rather than a conventional text field. It will need loading, no-results, authorization-error, selected-repository and revoked-access states.

## Priority-selection component

The second section asks **“What matters most to you?”** and is also marked optional.

The selectable chips include:

* Cost & Tokens
* Latency
* Reliability and Errors
* Output Quality
* Tool Call Failures
* Hallucinations
* Context and Memory
* Safety and Guardrails

The chips wrap over two rows, which makes the set scannable without introducing a dropdown. The custom **“Add something specific”** action lets users express priorities not represented by the predefined taxonomy.

The chip group should expose a clear selected state through more than color alone. In this screenshot, the chips appear visually neutral, so the selected/unselected behavior is not demonstrated. A check icon, pressed-state border, or `aria-pressed` semantics would improve clarity.

## Footer and primary action

The footer pairs an informational estimate—**“Analysis can take up to 20 minutes”**—with the blue **Start Analyzing** button. This is good expectation-setting immediately before the user initiates a relatively long-running operation.

The trial notice is placed beneath the main panel rather than inside the task flow. This appropriately lowers its visual priority while keeping commercial and terms information visible.

## UX assessment

The screen is strong because it uses progressive enhancement:

* Analysis can begin without connecting code.
* Priorities improve relevance but are not mandatory.
* There is only one primary action.
* Long-running behavior is disclosed before submission.

Potential improvements would be to explain the default behavior when no priority is selected, show the effect of connecting a repository more concretely, and display whether analysis can continue if the user closes the browser.

---

# Image 2 — AWS deployment architecture

**10.32.11 AM**

## Screen purpose

This is not an operational product screen. It is a system architecture diagram intended to explain how LangSmith Engine can operate in an AWS-oriented, private-network deployment.

It should be assessed as an information-visualization interface rather than as an application form.

## Major architectural regions

The diagram establishes three principal domains.

### LangChain’s cloud

The upper-left container is labeled **LangChain’s Cloud**. It contains:

* Billing
* Monitoring Stack
* LangSmith Intelligence

A Private Link symbol sits at its boundary.

### AWS-managed inference

The upper-right area contains the AWS branding and a **Bedrock** service icon. An arrow labeled **Model inference** connects LangSmith Intelligence to Bedrock.

### Customer VPC

The lower container is labeled **Your VPC**. It includes:

* LangSmith UI entry point
* Network Load Balancer
* Client Apps
* EKS Cluster
* LangSmith Services
* Monitoring Stack
* Engine APIs
* Storage services:

  * S3
  * RDS
  * ElastiCache

A second private-link path connects the customer-side Engine services to LangSmith Intelligence.

## Visual grammar

The diagram uses several consistent visual conventions:

* Solid rounded containers for major trust or deployment zones
* Dashed containers for internal service groups
* Dotted perimeter to suggest a broader AWS or protected-network boundary
* Directional arrows for traffic or dependency flow
* Cloud-link icons for private connectivity
* Brand-colored service icons to distinguish infrastructure providers

The dark navy canvas and cyan line work make the network boundaries visually prominent. Monospace labels are used for infrastructure terminology such as Private Link, Model inference, Engine APIs and storage components.

## UX and communication strengths

The strongest feature is the nested-boundary model. A reader can quickly understand which services live:

* In LangChain’s environment
* In the customer VPC
* In AWS’s model-serving layer

The diagram also separates operational storage from Engine APIs, making it clear that the application workload and persistence layer are different architectural concerns.

## Communication weaknesses

Some arrow directions are difficult to parse, especially around the central Private Link path. The diagram does not include a legend explaining whether an arrow indicates request direction, data flow, control flow or logical dependency.

The AWS logo is visually dominant relative to some of the system labels, while critical text such as **Engine APIs** is comparatively small. At presentation distance, smaller monospace labels may be hard to read.

## Frontend or diagram implementation recommendations

This should ideally be delivered as SVG rather than as a raster image. SVG would allow:

* Crisp rendering at different zoom levels
* Accessible text labels
* Hover explanations
* Highlighting of a complete request path
* Optional toggles for network, storage and inference layers
* Responsive reflow for narrow screens

A high-quality interactive version could allow the viewer to select a flow such as “UI request,” “trace analysis” or “model inference” and highlight only the relevant path.

---

# Image 3 — Analysis in progress modal

**10.33.18 AM**

## Screen purpose

This modal represents the asynchronous processing state after the user selects **Start Analyzing**.

Its primary UX responsibility is to reassure users that work is progressing during a task that may take approximately 20 minutes.

## Modal structure

The application behind the modal is heavily blurred, placing the user in a focused task state. The modal is large enough to provide meaningful progress detail but does not occupy the full viewport.

The top contains a pale-blue informational banner with:

* Clock icon
* **Estimated time: ~20 minutes**
* Explanatory message that traces are being analyzed

Below it is a vertical stepper.

## Progress stepper

The visible stages are:

1. Spinning up a new environment — complete
2. Querying traces — complete
3. Exploring errors and failures — active
4. Reviewing user interactions — pending
5. Clustering issues — pending

Completed steps use green checkmarks. The active step is shown with a spinner and a blue-highlighted row. Pending steps are reduced in contrast.

This is a strong state hierarchy because shape, iconography, text contrast and background are all used together. Progress is not communicated by color alone.

## Notification setup

The bottom section invites the user to **“Get notified the moment an issue is found.”** It includes supporting copy and an **Add webhook** secondary action.

This is smart task design: the system offers an escape from the waiting experience while the analysis is already running. Rather than forcing the user to remain on the page, it introduces a notification channel at the moment its value is most obvious.

## UX risks

A stepper can imply deterministic progress even if analysis phases are iterative or variable. The backend should drive these states truthfully rather than advancing them on a simulated timer.

The interface should also clarify:

* Whether the job continues when the modal is closed
* Whether closing the tab cancels anything
* Whether the user will receive an in-product notification
* What happens if one stage fails
* Whether partial findings are retained

## Frontend architecture

This screen should be powered by a persistent analysis-job entity with a stable job ID. Status updates could be delivered through polling, server-sent events or a websocket.

Accessibility requirements include:

* `aria-live` updates when the active stage changes
* Focus trapping within the modal
* A keyboard-accessible close or minimize control
* Textual status independent of animation
* Reduced-motion handling for the spinner

---

# Image 4 — Main issue-triage workspace

**10.33.29 AM**

## Screen purpose

This is the core Engine workspace. It combines issue discovery, issue triage, supporting evidence and recommended remediation in a master-detail layout.

## Global application shell

The top navigation contains project-level areas such as:

* Runs
* Threads
* Evaluators
* Automations
* Insights
* Issues

**Issues** is selected. The surrounding shell also contains project utilities such as retention, dashboard access and creation actions.

A thin global icon rail appears at the far left, while a wider issue-navigation panel sits beside it.

## Issue list

The left panel displays a vertically scrollable collection of discovered issues. Each item includes several pieces of metadata:

* Issue title
* Category label
* Number of associated traces
* Recency
* Severity-colored indicator

Examples visible in the list include tool-call loops, failed recovery, hallucination, incorrect tool arguments, cost/performance, guardrail bypass and missing-tool issues.

This is a high-density triage list. It behaves similarly to an inbox, incident queue or code-review list.

The active issue is highlighted with a subtle background. The selected item is **“Agent stuck in retriever tool-call loop, hitting max_iterations.”**

## Issue-detail header

The main panel begins with the issue title and a linked-item icon. Beneath it are:

* High-severity control
* Tool Call Loop category chip
* Resolve action
* Ignore action
* Last-updated timestamp

The right side contains the primary remediation actions:

* Create Evaluator
* Copy Fix Context
* Open PR

This creates a logical split between issue-management actions on the left and engineering-response actions on the right.

## Diagnosis summary

A natural-language description explains the observed behavior: the agent repeatedly calls `search_docs` with near-identical query reformulations, reaches `max_iterations`, or consumes its token budget without producing an answer.

Inline code styling is used for tool and variable names. This helps technical users distinguish system identifiers from explanatory prose.

## Trend visualization

A blue bar chart shows occurrence volume over several dates. The chart is intentionally compact and sits directly below the diagnosis, helping the user understand whether the issue is isolated, recurring or increasing.

The screenshot suggests day-level aggregation. The y-axis is visible but minimally labeled.

## Linked traces

The **Linked Traces** section shows representative evidence. Each row includes:

* Agent or run name
* Linkable ID
* Brief failure explanation
* Relative timestamp
* Duration

The duration values are visually emphasized on the right. One particularly slow trace is highlighted in a warmer warning color, making performance outliers easy to spot.

## Proposed fix

The lower section introduces a natural-language recommendation followed by a prompt diff. Removed content appears in red and new content in green.

This gives the user both:

* A plain-language explanation of the intended change
* An inspectable implementation artifact

## UX model

The page successfully compresses the entire incident response workflow into one surface:

**Understand → inspect evidence → evaluate frequency → review fix → take action.**

The principal risk is density. On smaller screens, the issue list, chart, evidence table, code diff and action toolbar could compete for attention. A responsive layout would likely need to collapse the left list into a drawer below a desktop breakpoint.

---

# Image 5 — Issue detail with chart tooltip

**10.33.41 AM**

## Screen purpose

This image is a tighter view of the selected issue. It emphasizes the diagnosis, trend chart, linked traces and beginning of the proposed fix.

## Chart interaction

A cursor is positioned over one bar, showing the tooltip:

**“Sun May 3: 3 traces.”**

This confirms that the chart supports direct inspection rather than functioning only as decoration.

The tooltip uses a white floating surface with a subtle shadow. It is visually separated from the blue bars and does not obscure the entire series.

## UX value of the chart

The visualization answers several practical questions:

* When did the issue begin?
* Is it becoming more frequent?
* Was there a single spike?
* Is the problem still occurring?
* Which days should be investigated in the trace list?

The chart’s location immediately after the diagnosis makes frequency part of the issue narrative rather than a separate analytics workflow.

## Accessibility considerations

Hover-only behavior would be insufficient. The bars should also be keyboard-focusable, with the same date and count exposed in an accessible label.

The chart would benefit from:

* A title or short axis description
* A textual total
* A nonvisual summary
* Focus indicators on individual bars
* A larger click target than the visible bar itself

## Linked-trace hierarchy

The visible evidence rows show a useful progression:

* Maximum-iteration failure
* Repeated calls with high query similarity
* Multiple nearly identical retriever calls
* A long-running trace

This demonstrates that Engine is not showing an abstract diagnosis alone; it links the issue to concrete runtime evidence.

## Design observation

This crop gives the main content more breathing room than the complete workspace. It reveals that the central issue panel has a strong information hierarchy even though the full application can feel dense.

---

# Image 6 — Complete desktop issues workspace

**10.34.03 AM**

## Screen purpose

This is the clearest full-width view of the Engine desktop application and its overall information architecture.

## Three-level navigation model

The interface contains three distinct navigation layers:

1. **Global icon rail**
   Provides access to broader LangSmith product areas.

2. **Project navigation bar**
   Contains Runs, Threads, Evaluators, Automations, Insights and Issues.

3. **Issue inbox**
   Displays the Engine-generated issues for the active tracing project.

This layered structure is appropriate for an enterprise developer tool, but it requires disciplined responsive behavior because the combined navigation consumes significant horizontal space.

## Issue queue behavior

The issue column shows the queue count and a **Scanning traces…** state near the heading. Filter, sort and settings controls are positioned at the top-right of the panel.

Issue cards use a repeatable structure:

* Small severity glyph
* Truncated title
* Category badge
* Trace count
* Last-seen time

Truncation is unavoidable at this width, so full titles should be available through a tooltip and screen-reader label.

## Main-panel hierarchy

The issue detail uses a conventional vertical document flow:

1. Header and actions
2. Diagnosis
3. Frequency chart
4. Linked traces
5. Proposed fix
6. Diff
7. Evaluation-related actions further below

This makes the panel usable as a review document while preserving actionable controls near the top.

## Density and spacing

The application uses compact spacing, small metadata typography and restrained borders. It resembles an IDE, issue tracker and observability dashboard more than a consumer interface.

That is appropriate for the target audience, but the system should support:

* Adjustable issue-panel width
* Collapsible navigation
* Full-screen diff view
* Persistent scroll position per issue
* Keyboard shortcuts for next/previous issue
* Deep links to individual sections

## Frontend component model

A sensible decomposition would be:

* `AppShell`
* `ProjectNavigation`
* `IssueInbox`
* `IssueListItem`
* `IssueDetailHeader`
* `DiagnosisSummary`
* `OccurrenceChart`
* `LinkedTraceTable`
* `ProposedFix`
* `PromptDiff`
* `CodeDiff`
* `EvaluatorSuggestion`

The issue list and detail panel should manage scroll independently so changing issues does not reset the queue position.

---

# Image 7 — “Open PR” guided-action spotlight

**10.34.13 AM**

## Screen purpose

This image appears to be part of a guided demonstration, onboarding tour or feature highlight. The rest of the interface is blurred while the remediation action group is enlarged and elevated.

## Spotlighted actions

The floating action strip contains:

* Create Evaluator
* Copy Fix Context
* Open PR

**Open PR** is the visual primary action:

* Solid blue fill
* White text
* Branch/pull-request icon
* Large hand cursor
* Rightmost placement

The hierarchy communicates that the intended high-value outcome is moving from diagnosis into an actionable source-code change.

## UX behavior

The spotlight isolates a small decision set from a complex screen. This is useful in product education because the underlying issue page contains many competing elements.

The three actions also represent different levels of integration:

* **Create Evaluator:** keep monitoring the behavior
* **Copy Fix Context:** transfer remediation context manually
* **Open PR:** allow the connected workflow to prepare a repository change

## Interaction concerns

If this is an actual onboarding overlay, it should include:

* A clear explanation of what will happen
* A way to dismiss the tour
* Escape-key handling
* Focus trapping or guided focus
* A “do not show again” preference
* Correct anchoring when the page scrolls or resizes

Duplicating live controls inside a floating spotlight can create state synchronization problems. A better implementation is often to spotlight the existing toolbar through a mask rather than render a second copy of the buttons.

## Visual design

The elevated white surface, large blur and oversized cursor create a promotional or tutorial tone rather than a normal production interaction. That is effective for a demo but should be used sparingly inside the operational application.

---

# Image 8 — Proposed implementation and suggested evaluator

**10.34.27 AM**

## Screen purpose

The issue detail has been scrolled down to the implementation layer. This is where Engine transitions from explaining the problem to offering a concrete fix and a regression-prevention mechanism.

## Prompt-change diff

The upper part shows a prompt change with:

* A removed instruction in red
* Several new instructions in green
* Added and removed line counts
* View Prompt action
* Apply action

The proposed prompt encourages the agent to state what new information it is seeking and stop repeating near-duplicate searches.

The red/green convention is familiar to developers, but additions and deletions should also be marked with `+` and `−` symbols, as they are here, for users with color-vision deficiencies.

## Code-change diff

Below the prompt change is a larger implementation diff. The proposed code wraps tools with a deduplication guard. The visible code introduces concepts such as:

* Similarity threshold
* Maximum repeat count
* Previously seen queries
* Short-circuiting repeated calls
* Returning a cached or previous result

This is presented as an inspectable artifact rather than a black-box recommendation.

## Suggested evaluator

The lower card is labeled **Suggested Evaluator** and includes a named evaluator:

`tool_call_loop_under_3_searches`

A JavaScript function counts `search_docs` tool-start events and returns a pass/fail score based on the number of calls.

The **Create Evaluator** button is positioned at the top-right of the card, turning the suggestion into a direct workflow action.

## UX strength

This screen establishes a powerful cause-and-control relationship:

* Fix the immediate behavior through prompt or code changes.
* Add an evaluator that detects the same behavior in future runs.

The evaluator is contextual rather than generic. It is visibly derived from the particular failure mode.

## Frontend considerations

The diff viewer should support:

* Line-number alignment
* Syntax highlighting
* Copying selected code
* Expand/collapse for large files
* Side-by-side and unified views
* Keyboard navigation
* Virtualized rendering for large diffs

The evaluator card should display validation errors before creation and make the scoring contract explicit.

---

# Image 9 — Configure Evaluator modal

**10.34.39 AM**

## Screen purpose

This modal allows the user to review and configure Engine’s suggested evaluator before creating it.

## Modal anatomy

The modal is a tall, centered white surface over a blurred application background. The heading reads **Configure Evaluator**.

A secondary strip labeled **Evaluator** appears beneath the heading, functioning as a section label or step indicator.

The visible form fields are:

* **Name**
* **Application**
* Code editor

## Name field

The evaluator name is pre-populated as:

`tool_call_loop_under_3_searches`

Prefilling the name minimizes setup effort and creates consistency between the issue recommendation and the resulting evaluator.

The name is descriptive but implementation-oriented. Depending on the broader product taxonomy, a human-readable display name and a machine-safe key could be separated.

## Application field

The dropdown displays **No application**. It appears selectable but visually subdued.

The term “Application” may be ambiguous to users. Supporting text could explain whether this means:

* Tracing project
* Agent deployment
* Dataset
* Automation
* Workspace application

## Code editor

The editor shows a JavaScript `performEval(run)` function. It:

1. Reads events from the run.
2. Filters for `tool_start` events where the tool is `search_docs`.
3. Counts the events.
4. Returns a score of 1 when the call count is at or below the threshold.
5. Returns a comment describing the observed count.

Line numbers and syntax coloring make the evaluator inspectable.

## UX observations

The most important strength is reviewability: generated evaluation logic is not silently deployed. The user can inspect and presumably edit it.

The visible crop does not show a primary save/create action. If it exists below the viewport, the action footer should be sticky. Users should never need to search for the action that completes a modal workflow.

## Frontend architecture

A production implementation would likely use Monaco Editor or CodeMirror with:

* JavaScript syntax highlighting
* Linting
* Read-only/generated-state indicator
* Formatting
* Keyboard shortcuts
* Error gutter
* Accessible text-area fallback

The system should clearly distinguish code validation from execution validation.

---

# Image 10 — Evaluator configuration with sample-data tester

**10.34.47 AM**

## Screen purpose

The evaluator configuration experience has expanded into a two-panel testing workspace.

The left side contains the evaluator definition, while the right side provides sample data and a **Test** action.

## Split-panel layout

The left configuration panel retains:

* Name
* Application
* JavaScript editor

The right floating panel is labeled **Sample data** and explains that sample traces or dataset examples can be selected from the linked session or dataset.

A selector near the top shows **run**, indicating the current test-input type.

## JSON viewer

The right panel displays a structured JSON representation of a run. Visible fields include:

* Name
* Inputs
* Additional keyword arguments
* Parsed decision
* Content
* ID
* Invalid tool calls
* Response metadata
* Finish reason
* Model name
* Model provider
* Token usage

This gives advanced users direct access to the evaluator’s input schema.

## Test action

The **Test** button is positioned in the upper-right corner of the sample-data panel. This supports a rapid edit-test loop without leaving the modal.

A complete testing flow should expose:

* Running state
* Runtime duration
* Returned score
* Returned comment
* Console output
* Exceptions
* Schema mismatches
* Whether execution was sandboxed
* Reset-to-generated-code option

## UX strengths

This is an effective developer-tool pattern because code and fixture data are visible simultaneously. Users do not need to mentally infer the runtime object structure.

The sample-data explanation also anchors the evaluator in real production evidence rather than an abstract test object.

## UX risks

The combined interface is very wide and contains multiple independent scroll regions. On a laptop or narrow viewport, this will become difficult to use.

Recommended responsive modes include:

* Side-by-side above a large breakpoint
* Tabbed **Code / Sample / Result** interface at medium widths
* Full-screen editor on small widths
* Resizable split pane on desktop

## Security and execution design

Evaluator code should run in an isolated execution environment. The UI should never imply that arbitrary generated code executes directly inside the browser or application server without sandboxing and resource limits.

---

# Image 11 — Mid-scroll remediation workspace

**10.35.00 AM**

## Screen purpose

This is another full-width issue-detail state, positioned between the evidence section and the evaluator section.

It demonstrates how the issue page works as one continuous remediation document.

## Visible sequence

The top begins with **Linked Traces**, followed by the Engine-generated examples action.

The **Proposed Fix** section then contains:

* Natural-language remediation summary
* Prompt diff
* Code diff
* Suggested Evaluator heading at the bottom edge

This ordering reinforces an evidence-first approach: users see the traces supporting the diagnosis before reviewing the proposed modification.

## Diff presentation

The prompt and code changes are visually grouped but separated into distinct blocks. This is important because they represent different change surfaces:

* Prompt-behavior adjustment
* Runtime guardrail or tool-wrapper implementation

The code diff occupies most of the viewport, signaling that the suggested remediation is substantial enough to require careful engineering review.

## Scroll model

The left issue queue remains visible while the main panel scrolls. This supports rapid triage but introduces a possible nested-scroll challenge.

The application should ensure that:

* Mouse-wheel behavior affects the expected pane
* Keyboard Page Up/Page Down targets the focused pane
* Issue selection preserves or intentionally resets main-panel scroll
* Deep links can jump to “Fix,” “Evaluator” or “Examples”
* Sticky headers do not cover code lines

## UX observation

The system does not force users directly into the PR flow. They can read the explanation, inspect both prompt and code changes, and move into evaluator creation separately. This advisory posture is appropriate for generated engineering changes.

---

# Image 12 — Add as offline example modal, collapsed view

**10.35.22 AM**

## Screen purpose

This modal converts representative failing traces into offline evaluation examples.

It is the dataset-building part of the Engine workflow.

## Dialog structure

The modal title is **Add as offline example**. It contains three selectable trace cards.

The first card, `docs_agent`, is expanded. The other two `AnswerGeneration` entries remain collapsed.

The expanded card shows:

* Input
* Wrong Output
* Expected Output

This is a clear supervised-evaluation structure.

## Input and output treatment

The input appears as:

**“structed output for langraph”**

The wrong output is shown in red with strikethrough styling. The expected output is shown in green.

The visual language communicates transformation:

* What the user asked
* What the agent did incorrectly
* What correct behavior should look like

## Trace alternatives

The two collapsed examples summarize different manifestations of the same issue:

* Reaching `max_iterations=12` without producing an answer
* Six retriever calls with very high cosine similarity

This lets the user decide which traces are useful as canonical regression examples.

## Footer actions

The footer includes:

* Edit in annotation queue
* Cancel
* Add to Dataset

**Add to Dataset** is the primary blue action and includes a database icon.

The annotation-queue action offers a more involved review path, while Add to Dataset enables immediate acceptance.

## UX strengths

The design keeps human approval in the loop. Engine proposes examples, but the user chooses what becomes ground truth.

The expanded/collapsed card model also scales better than presenting all trace content simultaneously.

## Potential improvements

The dialog should clarify:

* Which dataset will receive the examples
* Whether a new dataset will be created
* How many examples are selected
* Whether expected outputs are editable
* Whether sensitive trace content is included
* Whether duplicates already exist

Checkboxes or explicit selection indicators would make multi-example behavior clearer.

---

# Image 13 — Add offline example, expanded expected criteria

**10.35.31 AM**

## Screen purpose

This is an expanded view of the first offline-example candidate, revealing the full expected-output specification.

## Expected-output design

Rather than presenting only a single ideal answer, the expected output defines several behavioral criteria:

### `tool_calls_under_4`

The agent should make no more than three calls to `search_docs` for the question.

### `no_query_repetition`

No two `search_docs` queries should be near-duplicates, using a cosine-similarity threshold.

### `answers_with_what_it_has`

When retrieval no longer returns useful information, the agent should answer with the available material and acknowledge the information gap instead of repeatedly searching.

This is significant from an evaluation-design perspective. The expected output describes **behavioral invariants**, not merely a preferred sentence.

## Visual treatment

The expected criteria are displayed in a code-like, monospaced format with blue keys and green explanatory values.

This presentation makes the data feel structured and suitable for machine evaluation, while remaining understandable to a technical reviewer.

The wrong output remains visually crossed out in red, creating a direct contrast between observed and expected behavior.

## UX implications

This screen bridges domain reasoning and evaluation engineering. A user can understand why the trace is defective and what future behavior should be enforced.

However, the criteria mix multiple concerns in one example:

* Call-count constraint
* Semantic query-diversity constraint
* Answer-completion behavior

The system should make clear whether these become:

* One composite evaluator
* Three separate feedback keys
* Dataset reference fields
* Natural-language expected output
* Structured evaluator metadata

## Editing experience

The **Edit in annotation queue** action is important because generated expected behavior may need refinement before it becomes durable ground truth.

Ideally, the user should be able to edit each criterion independently, preview the resulting dataset schema and see how the example will be evaluated in future experiments.

---

# Overall UI/UX architecture

Across the screenshots, Engine follows a coherent workflow architecture:

1. **Configure analysis**
2. **Run an asynchronous investigation**
3. **Present recurring failures as an issue inbox**
4. **Explain and quantify each issue**
5. **Link the diagnosis to supporting traces**
6. **Propose prompt and code changes**
7. **Prepare a repository pull request**
8. **Generate an online evaluator**
9. **Generate offline regression examples**

The experience combines patterns from several established developer tools:

* An incident-management inbox
* An observability dashboard
* A code-review diff viewer
* An evaluator IDE
* A dataset annotation interface

The most important frontend challenge will be preserving coherence across those modes. The UI succeeds when everything remains anchored to one issue. It will become confusing if PRs, evaluators and dataset examples feel like unrelated tools.

A strong frontend domain model would therefore center every artifact around a stable `Issue` object containing references to:

* Evidence traces
* Diagnosis
* Occurrence history
* Proposed prompt changes
* Proposed code changes
* Repository PR
* Suggested evaluators
* Suggested offline examples
* Resolution status
* Recurrence history

That shared model would allow the different screens to remain synchronized and would support reliable deep linking, permissions, audit logs and real-time job updates.

[1]: https://docs.langchain.com/langsmith/engine?utm_source=chatgpt.com "Find and fix your agent's issues with LangSmith Engine"
[2]: https://www.langchain.com/blog/introducing-langsmith-engine?utm_source=chatgpt.com "Introducing LangSmith Engine"
[3]: https://docs.langchain.com/langsmith/engine-self-hosted?utm_source=chatgpt.com "LangSmith Engine on self-hosted - Docs by LangChain"
