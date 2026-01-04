---
description: Initialize project with Conductor context-driven development
---

<!-- Note: This is the Claude Code version. Gemini CLI uses colon-form commands like `/conductor:setup` -->

<!-- 
SYSTEM DIRECTIVE: You are an AI agent. Follow these instructions precisely.
CRITICAL: Validate every tool call. If any fails, halt and announce the failure.
CRITICAL: When determining model complexity, ALWAYS select the "flash" model.
-->

# Conductor Setup

Initialize this project with context-driven development. Follow this workflow precisely and sequentially.

---

## 1.0 RESUME CHECK

**PROTOCOL: Before starting setup, determine the project's state using the state file.**

1. **Read State File:** Check for `conductor/setup_state.json`
   - If it does NOT exist, this is a new project. Proceed to Section 1.1.
   - If it exists, read its content.

2. **Resume Based on State:** Let `last_successful_step` be `STEP`:
   - If `STEP` is `"2.1_product_guide"`: Announce "Resuming: Product Guide complete. Next: Product Guidelines." → Proceed to **Section 2.2**
   - If `STEP` is `"2.2_product_guidelines"`: Announce "Resuming: Guidelines complete. Next: Tech Stack." → Proceed to **Section 2.3**
   - If `STEP` is `"2.3_tech_stack"`: Announce "Resuming: Tech Stack complete. Next: Code Styleguides." → Proceed to **Section 2.4**
   - If `STEP` is `"2.4_code_styleguides"`: Announce "Resuming: Styleguides complete. Next: Workflow." → Proceed to **Section 2.5**
   - If `STEP` is `"2.5_workflow"`: Announce "Resuming: Scaffolding complete. Next: Initial Track." → Proceed to **Phase 2 (Section 3.0)**
   - If `STEP` is `"2.7_beads_integration"`: Announce "Resuming: Beads integration complete. Next: Initial Track." → Proceed to **Phase 2 (Section 3.0)**
   - If `STEP` is `"3.3_initial_track_generated"` or `"complete"`:
     - Announce: "Project already initialized. Use `/conductor-newtrack` or `/conductor-implement`."
     - **HALT** the setup process.
   - If `STEP` is unrecognized: Announce error and halt.

---

## 1.1 PRE-INITIALIZATION OVERVIEW

Present to user:
> "Welcome to Conductor. I will guide you through:
> 1. **Project Discovery:** Analyze if this is new or existing project
> 2. **Product Definition:** Define vision, guidelines, and tech stack
> 3. **Configuration:** Select code style guides and workflow
> 4. **Track Generation:** Create the initial development track
>
> Let's get started!"

---

## 2.0 PHASE 1: PROJECT SETUP

### 2.0 Project Inception - Brownfield/Greenfield Detection

1. **Classify Project Maturity:**

   **Brownfield Indicators (ANY match = Brownfield):**
   - Version control: `.git`, `.svn`, `.hg` directories exist
   - Dirty git repo: `git status --porcelain` returns non-empty output
   - Dependency manifests: `package.json`, `pom.xml`, `requirements.txt`, `go.mod`, `Cargo.toml`
   - Source directories: `src/`, `app/`, `lib/` containing code files

   **Greenfield Condition:**
   - NONE of above indicators found
   - Directory is empty or contains only generic docs (e.g., single `README.md`)

2. **Execute Based on Maturity:**

   **IF BROWNFIELD:**
   - Announce: "Existing project detected."
   - If uncommitted changes detected: "WARNING: You have uncommitted changes. Please commit or stash before proceeding."
   - **Request Permission:**
     > "I've detected an existing project. May I perform a deep analysis to understand it?"
     > A) Yes - Full analysis (recommended)
     > B) Quick scan only
     > C) No - I'll provide context manually
     >
     > Please respond with A, B, or C.
   - If C, halt and await instructions.
   - **Deep Code Analysis (Option A):**
     - Respect `.gitignore` and `.geminiignore` patterns
     - **Use gkg repo_map** to understand project structure:
       ```
       mcp__gkg__repo_map with relative_paths: ["src", "app", "lib"] 
       to get architecture overview
       ```
     - **Use gkg search_codebase_definitions** to find key patterns:
       - Main entry points, routers, controllers
       - Database models/schemas
       - Authentication/authorization patterns
     - **Use Oracle** to synthesize analysis:
       > "Analyze this codebase structure and identify:
       > 1. Architecture pattern (MVC, Clean, Hexagonal, etc.)
       > 2. Key abstractions and conventions
       > 3. Tech stack with versions
       > 4. Potential areas of complexity"
     - Present Oracle's analysis to user for confirmation
   - **Quick Scan (Option B):**
     - Analyze README.md, package.json, directory structure only
     - Extract: Programming Language, Frameworks, Database Drivers
     - Infer: Architecture type (Monorepo, Microservices, MVC)
   - Proceed to **Section 2.1**

   **IF GREENFIELD:**
   - Announce: "New project will be initialized."
   - Initialize git if `.git` doesn't exist: `git init`
   - **Ask:** "What do you want to build?"
   - **CRITICAL:** Wait for user response before any tool calls.
   - Upon response:
     - Execute: `mkdir -p conductor`
     - Create `conductor/setup_state.json`: `{"last_successful_step": ""}`
     - Write response to `conductor/product.md` under `# Initial Concept`
   - **Deep Research Phase:**
     - **Use exa-code** to research best practices:
       > Query: "<user's project idea> best practices architecture"
       > Example: "real-time chat app best practices architecture"
     - **Use exa-code** for tech stack recommendations:
       > Query: "<project type> recommended tech stack 2024"
     - **Use Oracle** to synthesize research:
       > "Based on the user's idea '<user input>' and the research findings,
       > recommend:
       > 1. Optimal tech stack with rationale
       > 2. Architecture pattern that fits
       > 3. Key libraries/frameworks to consider
       > 4. Potential challenges to plan for"
     - Present Oracle's recommendations to user
     - Store recommendations for use in subsequent Q&A sections
   - Proceed to **Section 2.1**

---

### 2.1 Generate Product Guide (Interactive)

1. **Announce:** "Now let's create `product.md`."

2. **Leverage Deep Research:**
   - **For Greenfield:** Use Oracle's recommendations to inform question options
   - **For Brownfield:** Use gkg analysis to pre-populate known facts
   - Questions should validate/refine research findings, not start from scratch

3. **Ask Questions Sequentially (max 5):**
   - **Question Classification:** Before each question, classify as:
     - **Additive:** For brainstorming (users, goals, features) - add "(Select all that apply)"
     - **Exclusive Choice:** For singular decisions - do NOT add multi-select
   - **Format:** Vertical list with options:
     ```
     A) [Option A] ← informed by research
     B) [Option B] ← informed by research  
     C) [Option C] ← informed by research
     D) Type your own answer
     E) Autogenerate and review product.md
     ```
   - For Brownfield: Ask context-aware questions based on gkg/Oracle analysis
   - For Greenfield: Options should reflect exa-code/Oracle research
   - **AUTO-GENERATE:** If user selects E, stop questions and generate based on context + research

3. **Draft Document:** Generate `product.md` using ONLY user's selected answers. Ignore unselected options.

4. **User Confirmation Loop:**
   > "I've drafted the product guide. Please review:"
   > ```markdown
   > [Drafted content]
   > ```
   > A) **Approve** - Proceed
   > B) **Suggest Changes** - Tell me what to modify
   - Loop until approved.

5. **Write File:** Append to `conductor/product.md`, preserving `# Initial Concept`.

6. **Commit State:** Write `conductor/setup_state.json`:
   ```json
   {"last_successful_step": "2.1_product_guide"}
   ```

7. **Continue:** Proceed to Section 2.2.

---

### 2.2 Generate Product Guidelines (Interactive)

1. **Announce:** "Now let's create `product-guidelines.md`."

2. **Ask Questions Sequentially (max 5):**
   - Topics: Prose style, brand messaging, visual identity
   - Same A/B/C/D/E format as Section 2.1
   - For each option, provide brief rationale and highlight recommendation

3. **Draft Document:** Generate using ONLY user's selected answers.

4. **User Confirmation Loop:** Same as Section 2.1.

5. **Write File:** Write to `conductor/product-guidelines.md`.

6. **Commit State:**
   ```json
   {"last_successful_step": "2.2_product_guidelines"}
   ```

7. **Continue:** Proceed to Section 2.3.

---

### 2.3 Generate Tech Stack (Interactive)

1. **Announce:** "Now let's define the technology stack."

2. **Leverage Deep Research:**
   - **For Greenfield:** Present Oracle's tech stack recommendations as primary options
   - **For Brownfield:** gkg analysis already identified the stack - confirm it
   - Include rationale from research for each recommendation

3. **Ask Questions Sequentially (max 5):**
   - Topics: Programming languages, frameworks, databases, tools
   - Same A/B/C/D/E format with research-informed options

   **FOR BROWNFIELD:**
   - **CRITICAL:** Document EXISTING stack from gkg analysis, don't propose changes
   - Present gkg/Oracle findings and ask:
     > "Based on my analysis, your tech stack is:
     > - Language: [detected]
     > - Framework: [detected]
     > - Database: [detected]
     > - Key libraries: [detected]
     >
     > A) Yes, this is correct
     > B) No, I need to provide corrections"
   
   **FOR GREENFIELD:**
   - Present Oracle's recommendations with rationale:
     > "Based on research for your project type, I recommend:
     > A) [Recommended stack] - [rationale from Oracle]
     > B) [Alternative stack] - [rationale]
     > C) [Minimal stack] - for quick prototyping
     > D) Type your own tech stack
     > E) Autogenerate based on recommendations"

3. **Draft Document:** Generate using ONLY user's selected answers.

4. **User Confirmation Loop:** Same as Section 2.1.

5. **Write File:** Write to `conductor/tech-stack.md`.

6. **Commit State:**
   ```json
   {"last_successful_step": "2.3_tech_stack"}
   ```

7. **Continue:** Proceed to Section 2.4.

---

### 2.4 Select Code Styleguides (Interactive)

1. **List Available Guides:** Check `templates/code_styleguides/` directory.

2. **For Greenfield:**
   - Recommend guides based on tech stack with explanation
   - Ask:
     > A) Include recommended style guides
     > B) Edit the selected set

3. **For Brownfield:**
   - Announce inferred guides based on tech stack
   - Ask:
     > A) Yes, proceed with suggested guides
     > B) No, I want to add more guides

4. **Copy Files:** `mkdir -p conductor/code_styleguides && cp [selected guides]`

5. **Commit State:**
   ```json
   {"last_successful_step": "2.4_code_styleguides"}
   ```

---

### 2.5 Select Workflow (Interactive)

1. **Copy Initial Workflow:** Copy `templates/workflow.md` to `conductor/workflow.md`

2. **Ask:**
   > "Use default workflow or customize?"
   > Default includes: 80% test coverage, commit after each task, Git Notes for summaries
   > A) Default
   > B) Customize

3. **If Customize:**
   - **Q1:** "Default coverage is >80%. Change it?"
     - A) No (Keep 80%)
     - B) Yes (Enter new percentage)
   - **Q2:** "Commit after each task or each phase?"
     - A) After each task (Recommended)
     - B) After each phase
   - **Q3:** "Use git notes or commit message for task summary?"
     - A) Git Notes (Recommended)
     - B) Commit Message
   - Update `conductor/workflow.md` based on responses

4. **Commit State:**
   ```json
   {"last_successful_step": "2.5_workflow"}
   ```

---

### 2.6 Finalization

1. **Summarize:** List all files created/copied.
2. **Transition:** Announce proceeding to initial track generation.

---

### 2.7 BEADS INTEGRATION

**PROTOCOL: Set up Beads integration for persistent task memory.**

1. **Check for Beads CLI:**
   - Run `which bd` to detect if Beads is installed
   - **If NOT found:**
     > "⚠️ Beads CLI (`bd`) is not installed. Beads provides persistent task memory across sessions."
     > "A) Continue without Beads integration"
     > "B) Stop - I'll install Beads first"
     - If A: Set `beads_available = false`, skip to Section 3.0
     - If B: HALT and wait for user

2. **If Beads Available, Ask User:**
   > "Beads detected. Choose integration mode for persistent task memory:"
   > A) Full integration (commits .beads/ to repo)
   > B) Stealth mode - Local only (use `bd init --stealth`)
   >
   > Please respond with A or B.

3. **Initialize Beads:**
   - Run `bd init` (for A) or `bd init --stealth` (for B)
   - **If command fails:**
     > "⚠️ Beads command failed: <error message>"
     > "A) Continue without Beads integration"
     > "B) Retry the failed command"
     > "C) Stop - I'll fix the issue first"
     - If A: Set `beads_available = false`, skip to Section 3.0
     - If B: Retry the command
     - If C: HALT and wait for user
   - Create `conductor/beads.json`:
     ```json
     {
       "enabled": true,
       "mode": "normal|stealth",
       "sync": "bidirectional",
       "epicPrefix": "conductor",
       "autoCreateTasks": true,
       "autoSyncOnComplete": true,
       "compactOnArchive": true
     }
     ```
   - Announce: "Beads integration enabled in [normal/stealth] mode."

---

## 3.0 PHASE 2: INITIAL PLAN AND TRACK GENERATION

### 3.1 Generate Product Requirements (Greenfield Only)

1. **Transition:** "Initial setup complete. Now defining high-level requirements."
2. **Analyze:** Read `conductor/product.md`
3. **Ask Questions (max 5):** Same A/B/C/D/E format, topics: user stories, functional/non-functional requirements
4. **AUTO-GENERATE:** If E selected, infer remaining details

---

### 3.2 Propose Initial Track

1. **Announce:** "I will now propose an initial track."
2. **Generate Track Title:** Analyze project context and propose:
   - **Greenfield:** Usually MVP track
   - **Brownfield:** Maintenance or targeted enhancement
3. **User Confirmation:** If declined, ask for clarification.

---

### 3.3 Create Track Artifacts

1. **Announce:** "Creating artifacts for the initial track."

2. **Initialize Tracks File:** Create `conductor/tracks.md`:
   ```markdown
   # Project Tracks

   This file tracks all major tracks for the project.

   ---

   ## [ ] Track: <Track Description>
   *Link: [./conductor/tracks/<track_id>/](./conductor/tracks/<track_id>/)*
   ```

3. **Generate Track Artifacts:**
   - Generate unique Track ID: `shortname_YYYYMMDD`
   - Create directory: `conductor/tracks/<track_id>/`
   - Create `metadata.json`:
     ```json
     {
       "track_id": "<track_id>",
       "type": "feature",
       "status": "new",
       "created_at": "<timestamp>",
       "updated_at": "<timestamp>",
       "description": "<description>"
     }
     ```
   - Generate `spec.md` and `plan.md`
   - **CRITICAL: Inject Phase Completion Tasks** - For each Phase in `plan.md`, append:
     `- [ ] Task: Conductor - User Manual Verification '<Phase Name>' (Protocol in workflow.md)`

4. **Commit State:**
   ```json
   {"last_successful_step": "3.3_initial_track_generated"}
   ```

5. **Announce Progress:** "Track '<description>' created."

---

### 3.4 Final Announcement

1. **Announce Completion:** "Project setup and initial track generation complete."
2. **Commit Files:** `git add conductor && git commit -m "conductor(setup): Add conductor setup files"`
3. **Next Steps:** "Run `/conductor-implement` to begin work."
