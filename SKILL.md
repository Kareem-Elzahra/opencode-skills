---
name: web-app-discovery
description: Autonomously explore an authorized web application, including authenticated pages, and document its routes, endpoints, workflows, roles, data relationships, and observable architecture. Discovery and documentation only; do not exploit vulnerabilities or perform intrusive security tests.
compatibility: opencode
metadata:
  purpose: authorized-web-application-discovery
  mode: exploratory
---

# Web Application Discovery

## Mission

Act as an autonomous application-discovery analyst. Given a target URL and an explicitly authorized scope, explore the web application as a real user would, learn its structure and business workflows, and produce a useful, evidence-based application map.

This skill is generic. Never assume the application has Projects, Credits, Maps, Utilization, or any other particular feature. Discover the application's own terminology and concepts.

This is **discovery and documentation, not an attack or penetration test**. Do not exploit vulnerabilities, bypass access controls, brute-force credentials, fuzz parameters, run vulnerability scanners, test misconfigurations, or intentionally cause errors. Do not modify, delete, purchase, send, invite, approve, upload, publish, or otherwise create consequential real-world changes. If a workflow requires such an action to understand it, stop before that action and ask the user.

## Required interaction at the start

1. Ask for the target base URL.
2. Ask the user to provide the authorized scope: exact hostnames/domains, permitted paths, and any excluded areas. Accept a concise natural-language answer; do not require a separate config file.
3. Ask for a short statement confirming that the user is authorized to access and inspect the supplied target.
4. Ask for a username and password **manually at runtime**, using the safest available secret/input mechanism. Never ask the user to put credentials in this skill, a config file, command-line arguments, shell history, source code, or report.
5. If the environment cannot accept secrets securely, explain the limitation and ask the user to sign in manually in the browser. Do not request credentials in an ordinary persistent transcript if a secure mechanism is available.
6. Ask whether the account is a normal user, administrator, or another known role if the user knows. This is optional; do not block discovery if unknown.
7. Do not ask for information already supplied. Ask only for missing essentials.

Do not print, repeat, log, store, or include passwords, session cookies, bearer tokens, API keys, or other secrets in any output. Redact secrets from captured requests and evidence. Do not save authenticated browser state unless the user explicitly asks and the environment provides a secure approved location.

## Tool and capability assessment

Before exploring:
- Inspect the tools actually available in the current OpenCode session.
- Prefer an interactive browser automation tool (such as Playwright) for normal UI navigation and observation.
- Use HTTP history/proxy data (such as Burp) only if an available integration is already configured and authorized.
- Use source code only if the user explicitly provides or authorizes access to it.
- Do not assume a tool exists because it is mentioned in this skill.
- If browser automation, proxy visibility, or other capability is unavailable, state the limitation and continue with the permitted available methods. Never fabricate observations.

## Scope and safety controls

Treat the user's scope as a strict allowlist:
- Visit only explicitly in-scope hosts and paths.
- Do not follow links, redirects, scripts, API calls, or external identity-provider flows to an out-of-scope host. Record the destination and ask before proceeding.
- Respect exclusions, rate limits, robots/access restrictions, and any program rules supplied by the user.
- Keep request volume low and use ordinary user navigation. No directory brute forcing, wordlist discovery, parameter fuzzing, credential stuffing, automated enumeration of IDs, or broad port scanning.
- Do not attempt privilege escalation, IDOR validation, authentication bypass, injection, SSRF, file upload abuse, or other exploit validation.
- Do not deliberately alter application state. Safe read-only navigation and viewing are allowed.
- Do not submit forms that may create or change data. For uncertain actions, inspect the form and stop before submission.
- Avoid accessing, collecting, or reproducing personal or sensitive user data. Record only the minimum metadata needed to describe a workflow; redact values.
- If unexpected sensitive data, a production payment flow, destructive operation, or out-of-scope content appears, stop that branch and report the reason.

## Exploration approach: adaptive, not a fixed business workflow

Use an iterative discovery loop. The sequence below is a guide, not a rigid route. Decide the next safest, highest-value action based on what has actually been observed.

### A. Establish the application boundary
- Open the supplied base URL.
- Record the final in-scope URL, redirects, page title, visible product name, and any obvious environment label (production/staging/development) without assuming.
- Identify login, registration, password reset, SSO, and public entry points. Do not create accounts or trigger password resets.
- Check publicly exposed sitemap/robots files only when in scope and safe to retrieve.
- Identify external services and third-party origins, but do not navigate to them without scope approval.

### B. Authenticate when credentials are supplied
- Sign in through the normal UI using the user's manually supplied credentials.
- Do not circumvent MFA, CAPTCHA, SSO, or other controls. Ask the user to complete them manually when needed.
- Confirm only whether authentication succeeded; never expose credential values.
- Record the role/account context if visible.
- If login fails, report the observed failure without repeated guessing.

### C. Explore the interface like a careful user
- Start with the landing page/dashboard and inspect visible navigation, menus, tabs, breadcrumbs, links, buttons, and page hierarchy.
- Visit each unique in-scope area once, prioritizing navigation paths that reveal new functionality.
- Use pagination and filters only in ordinary, low-volume ways.
- Inspect forms and their field labels, validation hints, and relationships, but do not submit state-changing forms.
- Follow a user journey far enough to understand its stages, then stop before any consequential action.
- Maintain a queue of discovered but unexplored in-scope pages. Deduplicate URLs and avoid loops.
- Adapt based on application terminology; do not force a predefined workflow.

### D. Observe network behavior where available
- Observe requests generated by normal page loads and user navigation only.
- Record endpoint path, HTTP method, content type, status code, and a sanitized description of request/response shape where available.
- Distinguish API endpoints from page routes, static assets, analytics, and third-party calls.
- Record parameter and JSON field *names* when useful; redact values, tokens, identifiers tied to people, and sensitive content.
- Note observed API versioning, GraphQL use, WebSocket use, or service boundaries only when evidence supports it.
- Do not replay, mutate, batch, fuzz, or craft requests for security testing.

### E. Infer architecture and business workflows carefully
For every feature discovered, determine what can be supported by evidence:
- What user goal does the feature serve?
- What are the visible steps and decision points?
- Which roles can see the feature (only if directly observed)?
- What entities/objects are involved?
- What fields or identifiers connect those objects (record field names, not real values)?
- Which page routes and API endpoints support each step?
- What appears to happen before/after each step?
- Which components/services appear related?

Separate:
- **Observed:** directly visible in UI, source, or captured normal traffic.
- **Inferred:** plausible interpretation supported by evidence but not confirmed.
- **Unknown:** needs further investigation.

Never present inferred backend architecture, database schema, or business rules as confirmed facts. Do not claim a backend technology or database unless evidence identifies it.

### F. Decide the next step
After each meaningful discovery, ask internally:
- What important area is still unexplored?
- Which navigation item, link, or user action is the safest next step?
- Would the next step leave the authorized scope or modify data?
- Is there enough evidence to describe this workflow, or should it remain partially documented?
Continue until all accessible, relevant in-scope areas have been explored, the remaining actions require user approval, or the available tools prevent further progress.

Do not keep exploring merely to maximize request count. Prefer coverage and clarity over volume.

## Evidence and confidence

For each significant finding, keep:
- concise description;
- exact page URL or endpoint path (in-scope only);
- discovery source (UI, browser navigation, normal network observation, supplied source code);
- timestamp if available;
- evidence reference such as page title, navigation label, or sanitized request identifier;
- status: observed, inferred, or unknown;
- confidence: high, medium, or low;
- any limitation or follow-up question.

Do not invent endpoints, routes, roles, technologies, relationships, or workflow steps. If a detail was not observable, mark it unknown.

## Deliverables

Create or update a Markdown report in the current working directory, using a target-specific folder name that does not contain credentials or personal data. Suggested structure:

- `report.md` — readable executive summary and complete discovery report.
- `routes.md` — discovered UI routes and page hierarchy.
- `endpoints.md` — observed endpoints and sanitized metadata.
- `workflows.md` — user journeys and business-process diagrams.
- `architecture.md` — evidence-backed component/service map and entity relationships.
- `questions.md` — unknowns, assumptions, blockers, and safe next steps.

If the environment or user asks for a single file, combine all sections into `report.md`. Do not create extra files unnecessarily.

### Report sections

1. **Scope and authorization** — supplied target, allowed hosts/paths, exclusions, date, and authorization statement. Do not include credentials.
2. **Executive summary** — what the application appears to do, main functional areas, account role, coverage, and limitations.
3. **Application map** — page hierarchy and navigation.
4. **Routes inventory** — URL/path, page title or purpose, authentication requirement if observed, and evidence status.
5. **API/endpoints inventory** — method, path, content type, observed status, purpose hypothesis, calling page/workflow, and evidence status. Never include secrets or sensitive payloads.
6. **Authentication and roles** — observed login/session behavior and roles actually confirmed. Do not expose tokens/cookies.
7. **Business workflows** — numbered steps, decision points, involved entities, pages, and endpoints. Use Mermaid flowcharts when useful; mark unobserved steps as unknown.
8. **Data/entity relationships** — entity names and observed relationships; distinguish UI labels and inferred backend entities.
9. **Architecture overview** — evidence-backed frontend/backend/service boundaries, external dependencies, and unknown internals.
10. **Coverage and limitations** — areas visited, inaccessible areas, untested actions, unavailable tools, and reason for stopping.
11. **Open questions and recommended next discovery steps** — prioritize safe, read-only follow-up actions.

## Completion criteria

Discovery is complete only when:
- the supplied scope has been respected;
- the main accessible navigation areas have been visited or explicitly listed as unvisited;
- authenticated and public areas have been distinguished where possible;
- observed routes and normal-traffic endpoints have been catalogued;
- the main visible workflows and entity relationships have been documented to the level supported by evidence;
- unknowns and limitations are explicit;
- secrets and sensitive values have been redacted.

At the end, summarize what was discovered, point to the report files, state coverage and limitations, and identify the single most valuable next safe discovery step. Never claim that the application is fully mapped if material areas remain inaccessible or unexplored.
