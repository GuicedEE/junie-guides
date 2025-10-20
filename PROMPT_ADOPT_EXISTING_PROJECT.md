# 🔄 Starter Prompt — Adopt RulesRepository in an Existing Project

Paste this prompt into your AI tool to migrate an existing repository to the RulesRepository methodology. The AI will analyze the repo, add the RulesRepository submodule, establish Pact → Rules → Guides → Implementation, and refactor docs to the modular, forward-only model.

Supported: JetBrains AI (Junie), GitHub Copilot Chat, Cursor, ChatGPT, Claude.

---

## 0) Provide Inputs
Fill before running.

- Repository URL / local path: <REPO_URL_OR_PATH>
- Org and project name: <ORG_NAME> / <PROJECT_NAME>
- Short description: <ONE_LINE_DESCRIPTION>
- License (if missing or to change): <LICENSE>
- Primary languages: <LANGUAGES>
- Detected/Chosen tech topics (tick):
  - Backend: [ ] Vert.x 5  [ ] Hibernate Reactive 7  [ ] DDD  [ ] MapStruct  [ ] Lombok  [ ] Logging
  - Frontend: [ ] Angular 20  [ ] Web Components  [ ] JWebMP  [ ] WebAwesome
  - Infra/CI: [ ] GitHub Actions  [ ] Terraform  [ ] GCP Cloud Run
  - Database: [ ] PostgreSQL  [ ] MySQL  [ ] Other: <DB_OTHER>
- Architecture: [ ] Monolith  [ ] Microservices  [ ] Micro Frontends
- AI engine used: [ ] JetBrains Junie  [ ] GitHub Copilot  [ ] Cursor  [ ] ChatGPT  [ ] Claude
- Level of change: [x] Forward-only (default)  [ ] Conservative (only if explicitly required)

Policies (must honor):
- Use Markdown for docs. Follow RULES.md sections: 4 (Behavioral), 5 (Technical), Document Modularity Policy, 6 (Forward-Only Change Policy).
- Do NOT place project-specific docs inside the submodule directory.

---

## 1) Self‑Configure the AI Engine
- Pin ./RULES.md anchors (sections above). Operate in forward-only mode: remove/replace legacy docs as needed; update all references.
- For Copilot/Cursor: create a workspace note or .cursor/rules.md summarizing these constraints.

---

## 2) Migration Plan (AI must draft first)
The AI should begin by producing a short plan with:
- Inventory: existing docs (README, RULES, GUIDES, architecture docs), CI, env files, components.
- Gaps: missing Pact/Rules/Guides/Implementation, outdated monolithic docs, absent indexes.
- Actions: submodule add; create/relocate docs; refactor to modular; update links; CI/env alignment.
- Risk notes: any breaking removals per forward-only policy.

When approved, execute the plan as one change set.

---

## 3) Required Changes
1. Add RulesRepository submodule (rules/ or docs/rules-repository) and document usage in README.
2. Create PACT.md (root or docs/) based on rules/creative/pact.md. Fill project details and cross-links.
3. Create/Update project RULES.md (outside submodule):
   - Declare scope, chosen stacks, and any project-specific conventions.
   - Link to relevant topic indexes:
     - rules/generative/webcomponents/README.md
     - rules/generative/hibernate/README.md
     - rules/generative/jwebmp/webawesome/README.md
4. Create/Update GUIDES.md with links to chosen modular entries (e.g., Hibernate transactions, CRUD; Web Components custom-elements/shadow-dom; Angular producing/consuming; WebAwesome Button/Input rules).
5. Create/Update IMPLEMENTATION.md explaining current modules, code layout, and back-links to guides.
6. Refactor legacy docs to modular model where feasible:
   - Split overly large monoliths into focused docs or replace with links to the submodule’s modular entries (preferred).
   - Remove deprecated/duplicate documents per Forward-Only policy; update all inbound links.
7. Environment alignment
   - Create or update .env.example using rules/implementation/env-variables.md as the source of truth.
8. CI alignments
   - Add/update minimal GitHub Actions workflows for build/test and document required secrets.
9. README updates
   - State adoption of RulesRepository, link submodule path, and link PACT/RULES/GUIDES/IMPLEMENTATION.

---

## 4) Output Checklist
- [ ] Submodule added and referenced in README
- [ ] PACT.md present and linked
- [ ] Project RULES.md present, linking to enterprise RULES and topic indexes
- [ ] GUIDES.md and IMPLEMENTATION.md present with back/forward links
- [ ] Monolithic/legacy docs removed or replaced; all references updated
- [ ] .env.example aligned to env-variables.md
- [ ] CI updated/added
- [ ] All links resolve; no project files placed inside the submodule

---

## 5) Guardrails
- Apply Forward-Only Change Policy fully; do not keep legacy stubs/anchors for compatibility.
- Document any risky removals briefly in MIGRATION.md (create if necessary).
- Close loops between artifacts (traceability in both directions).

---

## 6) AI Response Format
1) Proposed migration plan with inventory and diffs
2) Questions (if any)
3) Final diffs/files content
4) Post-migration validation notes (broken links check summary)

End of prompt.