---
name: ui-design-review
description: Review UI design from a running app, screenshots, or source code. Use for design audits, visual QA, design polish, and spotting generic AI-looking interfaces. Report prioritized findings; edit and verify fixes only when requested.
compatibility: Pi with read, bash, edit, write, and subagent tools. Live inspection requires available browser tooling; screenshots and source reviews do not.
---

Use **set-voice** Skill tool for communication.

# UI design review

Review with a designer's eye and support each finding with evidence. Prioritize visual quality and usability, check consistency with the project, and include obvious accessibility problems. This is not a complete accessibility audit.

This skill is self-contained. Resolve bundled references relative to this skill directory, not the reviewed project's working directory. Use the tools exposed by the current pi session; no named reviewer agents, other skills, additional model CLIs, or external services are required.

## 1. Establish scope and authority

Read the request and available inputs before asking questions. Inspect project design guidance such as `DESIGN.md`, design tokens, and relevant components when source is available.

- Identify the screen or flow, its audience and purpose, and whether it is marketing, app UI, or hybrid.
- For a precise request, use that scope. For an app-wide request, propose a bounded sample of key screens, states, and flows, then wait for agreement. Do not silently substitute a sample for a whole-app audit.
- If asked to review a diff, determine the actual comparison base and map changes to affected screens. Branch names alone do not trigger diff review. Claim before/after differences only when both versions have evidence.
- Default to review and recommendations. An explicit request to fix authorizes edits within that scope; a review request does not.
- Preserve the current design language. Keep aesthetic objections visible even when they conflict with project conventions, but label that conflict. Offer a broader design review when a specific change could substantially improve the result, and wait before expanding scope.

Normal navigation is allowed. Get permission before submitting forms, changing application data, or performing destructive actions, including through an authenticated browser. Check for autosave before interacting with editable fields. A fix request authorizes source edits, not writes to live application data.

Leave unrelated work intact. A dirty working tree does not block review. Git mutations, dependency installation, test-framework or CI setup, and backlog edits require separate requests. Use existing sessions for authentication only within the agreed scope; ask the user to sign in when needed, rather than reading or importing credentials and cookies.

**Done when:** the target, review boundary, input types, and permission to edit are known.

## 2. Obtain usable evidence

Choose the best available evidence, combining inputs where useful:

| Input | What to do | What it cannot establish alone |
|---|---|---|
| Running app | Use available browser tools to navigate, capture screens, inspect rendered styles, and exercise permitted interactions. | Untested screens, states, devices, or production conditions. |
| Screenshots | Read each relevant image. Identify its screen, viewport if known, and visible state. Request a crop or clearer capture when details are unreadable. | Keyboard behavior, hidden states, motion, DOM semantics, or load performance. |
| Source code | Read relevant components, styles, tokens, and existing tests. Cite file and line for implementation findings. | Actual rendered appearance or runtime behavior. |

For live review, inspect the exposed browser tool descriptions or an installed tool's help before using it. Reuse the project's existing browser/test tooling where suitable. A URL fetch is not a rendered browser. Do not invent tool names or install a browser dependency automatically.

If browser access is unavailable, ask for screenshots or offer a code-only review. If source was the requested input, proceed directly in code-only mode. If images cannot be inspected, say so and use available source or ask for accessible evidence; do not describe unseen images. Ask before starting a local server unless the request already authorizes it, and use the project's documented command.

Keep captures in a unique temporary directory unless the user supplied a destination. Read captured images before citing them. Record the route, viewport, state, and relevant capture path or attachment identifier. Capture a viewport-sized image or detail crop as well as a full-page image when needed for legibility. Keep private data out of unnecessary captures and reports; never publish artifacts.

Label the review as live, screenshot-based, code-only, or mixed. Separate observed facts, source-supported concerns, and unverified questions. Never infer an exact contrast ratio, target size, or performance measurement from appearance alone.

**Done when:** each in-scope target has inspectable evidence, or a specific access limitation is recorded. If nothing can be inspected, request the missing input instead of fabricating a review.

## 3. Review the UI

Read [references/design-criteria.md](references/design-criteria.md) completely before evaluating. It contains the review checklist, the aesthetic blacklist, and the marketing/app design preferences.

1. For rendered evidence, record the first impression before detailed analysis: what the screen communicates, what draws attention, and whether the primary action is clear. Skip visual first impressions in code-only mode.
2. Identify the actual design system from rendered styles and/or source. Distinguish observed values from inferred conventions. Compare typography, color, spacing, and component behavior across the available screens.
3. Apply every relevant checklist category. Record unavailable checks as unverified, not passed. Separate aesthetic objections from usability, accessibility, consistency, and performance defects.
4. With browser access, inspect representative narrow and wide layouts, intermediate widths where layout changes, and relevant hover, focus, loading, empty, error, and success states. Use existing fixtures or permitted interactions. Note exact viewports and states tested; screenshots of one size do not prove responsiveness.
5. Walk the agreed flows within the action permissions. Check response, feedback, transitions, and form behavior. Observe reduced-motion behavior with the preference enabled when tooling allows, rather than merely detecting that the preference exists.
6. Compare shared components and patterns across screens. When prior captures or reports are supplied, compare equivalent states and viewports, explaining any mismatch that limits conclusions.

Keep an evidence-backed list as you work. Favor useful findings over quotas. For each candidate record:

- Screen/component and category.
- Concrete observation and evidence, with screenshot region, rendered measurement, or source `file:line` as appropriate.
- User impact, or the design rationale for an aesthetic objection.
- A specific fix that fits the current scope, or a clearly marked broader-design option.

**Done when:** every agreed screen/flow has been checked against the applicable criteria, and skipped checks have explicit reasons.

## 4. Get independent checks

For a nontrivial review, launch two read-only subagents concurrently using the session's subagent tool. Skip delegation for a trivial review such as one isolated spacing adjustment and state that in the coverage note. Use the current configured model unless the user requests otherwise; do not invoke additional model CLIs or require a workflow engine.

Give each reviewer the agreed scope, product context, project design constraints, evidence mode, absolute paths to relevant captures/source and this skill's criteria, and the limits on permitted actions. Include the finding fields above. If evidence exists only as attachments, supply it through the tool if supported; otherwise explain that the subagent cannot independently inspect those images. Do not treat your description as independent visual confirmation.

- **Usability/accessibility reviewer:** independently check hierarchy, task clarity, interaction feedback, responsive usability, and obvious accessibility problems. Distinguish observed defects from checks that require missing evidence. Do not claim comprehensive accessibility compliance.
- **Consistency reviewer:** independently compare typography, spacing, colors, component patterns, and responsive conventions. Apply the aesthetic objections and the relevant marketing/app preferences. Distinguish deliberate project choices from accidental inconsistency.

Reviewers inspect evidence and return findings, not edits or report files. They do not control the shared browser, submit forms, launch further agents, or expand scope. The main agent owns live interactions and captures.

Wait for both results before final synthesis. If a reviewer fails, state the gap and continue with available evidence. Merge duplicates, resolve conflicting observations against the actual UI or source, and verify each retained finding yourself. Agreement between agents is not proof. Keep unverifiable suggestions out of the confirmed findings.

**Done when:** both reviewers have returned or failed, and each retained finding has been checked against evidence.

## 5. Report and offer export

Present the report in chat. Use this structure, omitting empty optional sections:

1. **Scope and evidence:** reviewed screens/flows, input mode, viewports/states tested, delegation coverage, and limitations.
2. **Overall assessment:** a short first impression when rendered evidence exists, what works, and the main problems. No numerical or letter grades.
3. **Prioritized findings:** stable IDs such as `UI-01`, impact, category, location, observation, evidence, rationale, and concrete recommendation. Mark aesthetic objections explicitly. Rank high for blocked tasks, unreadable content, or severely misleading hierarchy; medium for meaningful friction or inconsistency; polish for refinements. An aesthetic preference alone is not a high-impact usability defect.
4. **Where to start:** the smallest worthwhile changes, drawn from the findings. Avoid unsupported time estimates or padding to a fixed count.
5. **Broader design opportunity:** include only when there is a concrete case for substantial improvement. Explain the likely benefit and tradeoff, then ask whether the user wants that expanded scope.
6. **Unverified checks:** missing evidence and what would be needed to assess it.

End by asking whether the user wants a Markdown export. If they already requested an export, honor that request without asking again. On acceptance, use their destination or propose one and confirm it. Save the report and any referenced captures together, using relative links; avoid overwriting existing files. A report must not depend on temporary capture paths that will disappear. Note evidence that cannot be included, and exclude secrets or unnecessary personal data. Do not create persistent reports, baselines, design-system files, or project indexes without a request.

**Done when:** the report is delivered and export has been offered, or the requested export has been saved with working evidence links. An unanswered export offer does not authorize writing files.

## 6. Fix and verify, only when requested

If the initial request includes fixes, perform this step after triage and include its results in the report. Otherwise wait for the user to request fixes and agree on which findings to address.

- Inspect the current diff and relevant source before editing. Preserve pre-existing changes; stop for guidance when a fix overlaps work whose intent is unclear.
- Make the smallest change that resolves the finding, using existing tokens and components. Prefer styling changes where appropriate, but do not assume CSS changes are risk-free. Broader design changes still need approval.
- Use existing relevant tests and checks. For behavior changes, add a focused regression test within the existing test setup when feasible. If no suitable test setup exists, record the verification gap rather than bootstrapping one.
- Recheck the affected screen, state, and responsive layouts. Capture comparable before/after evidence when browser access exists. Check for new console errors and regressions in adjacent components.
- Mark each result **visually verified**, **code/test-checked only**, **unverified**, or **deferred**, with what actually ran. Passing tests alone do not establish visual improvement. If a change regresses, stop and explain; correct only your own edits within the authorized scope, leaving unrelated changes intact.
- Report changed files, check results, remaining limitations, and finding status. No automatic commits or Git rollback. If an earlier report was exported, ask before updating it unless ongoing updates were already requested.

**Done when:** each requested fix has a status and supporting verification evidence or an explicit limitation.
