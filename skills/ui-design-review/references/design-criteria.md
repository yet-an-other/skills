# Design criteria

Use this reference for every UI design review. Evaluate only what the available evidence can establish. Keep missing checks visible without turning the report into a checklist dump.

## Judgment and precedence

Preserve the project's current design language unless broader work is approved. Read these criteria as two distinct kinds of judgment:

- **Defects:** evidenced problems with usability, accessibility, consistency, responsiveness, or performance. Explain their effect on users.
- **Aesthetic objections:** the blacklist and composition preferences below. Flag them directly, even if the project deliberately uses the pattern. Explain why the result feels generic or weak, acknowledge any project convention, and label the recommendation as broader scope when it would change that convention.

Keep the aesthetic objections; do not silently waive them because they are subjective. Equally, do not describe a font choice or decorative pattern as an accessibility failure or evidence of how the site was generated. "AI slop" describes a visual pattern here, not its provenance.

Treat prescribed layouts, backgrounds, and motion counts as review preferences rather than unconditional requirements. Flag departures, then explain whether the preference fits this screen. Accessibility and task needs take precedence. A static screenshot cannot establish the absence of animation or focus behavior.

## First impression and classification

For rendered evidence, identify what the first screen communicates, the elements that attract attention first, and whether the product and next action are clear. Note specific strengths as well as problems.

Classify each screen or section:

- **Marketing/landing:** brand-forward, hero-driven, conversion-focused.
- **App UI:** task-focused, workspace-driven, often data-dense.
- **Hybrid:** apply marketing preferences to marketing sections and app preferences to functional sections.

## Review checklist

### 1. Visual hierarchy and composition

- Clear focal point, identifiable primary action, and purposeful reading order suited to the content and reading direction.
- Purpose apparent in the first viewport; hierarchy survives a squint test.
- Competing CTAs, visual noise, accidental overlap, or unclear layering.
- Information density appropriate to the task; whitespace groups related content rather than filling a template.
- Each section has a distinct job. Headings alone should convey the page's structure.

### 2. Typography

- Coherent type scale and consistent roles. Ratios around 1.25 or 1.333 are useful starting points, not conformance tests.
- Flag more than three font families; inspect whether the extra families have a purpose.
- Readable line height and measure. Typical starting points are 1.5 for body line height, 1.15 to 1.25 for headings, and 45 to 75 characters per body line.
- Body text around 16px or larger and captions around 12px or larger are review targets, not universal accessibility thresholds. Test readability and zoom where possible.
- Heading semantics follow content structure; flag skipped levels when they misrepresent it. Use purposeful weight contrast.
- Balanced heading wraps where useful, appropriate punctuation, and ellipsis handling. Prefer typographic quotes and ellipsis glyphs in polished UI copy.
- Align numeric columns with tabular figures. Flag unnecessary letterspacing in lowercase text.
- Preserve the font objections: Papyrus, Comic Sans, Lobster, Impact, and Jokerman; generic primary use of Inter, Roboto, Open Sans, and Poppins; default Arial/system stacks. Report these as aesthetic choices, not usability defects by themselves.

### 3. Color and contrast

- Coherent palette, consistent semantic roles, and understandable states without relying on color alone. Flag more than twelve non-gray colors for review rather than treating the count as a failure.
- Measure contrast when foreground and background values are known, including composited backgrounds. WCAG AA text targets are 4.5:1 for normal text and 3:1 for large text, at least 24 CSS px regular or approximately 18.67 CSS px bold. Relevant non-text UI boundaries and states generally need 3:1, subject to the criterion's applicability and exceptions.
- Gradients, images, transparency, and screenshots can make exact contrast uncertain. Report that uncertainty rather than inventing ratios.
- Dark mode has intentional surface hierarchy rather than a simple inversion. Review off-white text and slightly desaturated accents as preferences, while checking actual contrast.
- Check the document's `color-scheme` when native controls need to match the active theme.
- Avoid red/green-only encoding. Prefer a consistent warm or cool neutral palette; flag unintended mixing.

### 4. Spacing and layout

- Consistent grids, alignments, spacing tokens, and grouping. A 4px or 8px rhythm is a useful baseline; exceptions need a purpose.
- Appropriate content width, readable body measures, and no unintended horizontal overflow. Intentionally scrollable tables and canvases are not automatically defects.
- Radius hierarchy instead of the same bubbly radius everywhere. Check nested radii against the gap between surfaces.
- Safe-area handling where relevant, and layout that holds at narrow, intermediate, and wide widths. Useful probes are 375, 768, 1024, and 1440 CSS px, adapted to the actual layout and scope.
- Durable layout with CSS grid/flex where appropriate rather than fragile JavaScript measurements.
- Shareable filters, tabs, and pagination reflected in URLs when users need links or back/forward navigation.

### 5. Interaction states

- Recognizable hover, focus-visible, active, disabled, and loading states. Hover must not be the only way to reveal essential information.
- Visible keyboard focus, sensible focus order, keyboard-operable controls, accessible names, and modal focus management when testable.
- Disabled controls communicate their state; cursor treatment fits the platform. Flag ambiguous click affordances.
- Loading indicators match the operation. Skeletons should resemble the arriving content without shifting layout.
- Empty states explain the situation and a useful next action; visuals should help rather than decorate.
- Errors identify the problem and recovery step. Success feedback is clear and remains long enough to perceive; do not demand auto-dismissal of important information.
- Aim for comfortable touch targets around 44 by 44 CSS px. Smaller targets are not automatically WCAG AA failures: WCAG 2.2 AA uses a 24 by 24 CSS px size-or-spacing criterion with exceptions. Measure before making conformance claims.

### 6. Responsive usability

- Mobile composition serves the task rather than merely stacking desktop columns.
- Text stays readable, actions remain reachable, and navigation adapts without hiding essential tasks.
- Forms use suitable input types and avoid disruptive autofocus on mobile.
- Content tolerates zoom and reflow. Flag viewport settings that suppress zoom.
- Responsive images have suitable containment or `srcset`/`sizes`; content remains legible in landscape and at intermediate widths when tested.
- Check mobile touch targets and unintended overflow in the actual rendered state.

### 7. Motion

- Every animation explains a change, guides attention, or supports an intentional atmosphere. Flag ornament that delays or distracts from tasks.
- Easing and duration fit the action. Roughly 50 to 700ms covers many UI transitions; judge perceived latency rather than enforcing the range blindly.
- Prefer explicit transitioned properties and efficient transform/opacity animation. Flag `transition: all` and layout-affecting animation for inspection, not an unmeasured performance verdict.
- Enable reduced motion and observe the result when possible. Essential state feedback must remain understandable with nonessential motion removed.

### 8. Content and microcopy

- Specific action labels and product language instead of vague "Continue" or "Submit" where the action is unclear.
- Clear empty, loading, error, and success copy. Errors explain what happened and what to do next.
- Active voice, concise sentences, and no production placeholders.
- Truncation and wrapping preserve necessary information; tooltips must not be the only accessible source of essential text.
- Destructive actions have a suitable confirmation or recovery mechanism. Inspect safely; permission to review is not permission to delete data.

### 9. AI slop blacklist

Retain all ten objections. Identify the actual screen region and pattern, not just a generic accusation:

1. Purple/violet/indigo gradient backgrounds or blue-to-purple color schemes.
2. The symmetrical three-column feature grid: icon in a colored circle, bold title, and two-line description repeated three times.
3. Icons in colored circles used as section decoration.
4. Centered everything: headings, descriptions, cards, and long passages all centered.
5. The same large, bubbly border radius on every element.
6. Decorative blobs, floating circles, and wavy SVG dividers filling space that needs better content.
7. Emoji used as design elements, such as rockets in headings or emoji bullets.
8. Colored left-border accents on cards.
9. Generic hero copy such as "Welcome to [X]", "Unlock the power of...", or "Your all-in-one solution for...".
10. Cookie-cutter section rhythm: hero, three features, testimonials, pricing, CTA, with every section given the same weight or height.

Explain the objection and suggest a concrete alternative. Preserve project conventions for in-scope fixes; offer broader work when removing the pattern would change the design language.

### 10. Performance as design

- Observe response delays, layout shifts, image loading, and font changes where live inspection permits. Source can identify a risk, not prove a runtime measurement.
- The original review's aspirational LCP targets are under 2s for apps and under 1.5s for informational pages, with CLS under 0.1. Treat these as design goals, not universal pass/fail standards. Record the route, environment, and measurement method; a local run does not establish field performance.
- Images reserve their space, use suitable formats/sizes, and load at the right time. Lazy-load offscreen images, not the critical hero/LCP image.
- Fonts use a deliberate loading strategy. Review `font-display`, fallbacks, and selective preloads/preconnects for visible swaps or blocked text; adding preload everywhere is not a fix.
- Loading feedback appears promptly and resembles the final layout. Decorative shimmer is optional and must respect reduced motion.

## Strong composition objections

Preserve these as prominent design concerns, not automatic failing grades:

- Generic SaaS card grid as the first impression.
- Beautiful image with weak brand identity.
- Strong headline with no clear action.
- Busy imagery behind text.
- Sections repeating the same mood statement.
- Carousel with no narrative purpose.
- App UI made of stacked cards instead of a considered workspace layout.

Use these questions to test the composition where evidence permits:

- Is the brand/product unmistakable in the first screen?
- Is there a strong visual anchor?
- Can a user understand the page by scanning headings?
- Does each section have one job?
- Are cards necessary for grouping or interaction?
- Does motion improve hierarchy or atmosphere?
- Would the design still feel considered if decorative shadows disappeared?

## Marketing and landing preferences

- One composition in the first viewport, not a dashboard. Prefer a poster-like first screen.
- Brand-first hierarchy: brand, headline, body, CTA. Expressive typography, with two purposeful typefaces at most rather than default stacks.
- Prefer gradients, imagery, or subtle patterns over a flat single-color background. The purple-gradient objection still applies.
- Prefer a full-bleed, edge-to-edge hero rather than inset, tiled, or rounded variants.
- Hero content budget: brand, one headline, one supporting sentence, one CTA group, and one image. Flag competing content and decorative hero cards.
- Prefer cards only when the card itself is the interaction, rather than packaging every paragraph into a box.
- One purpose, headline, and short supporting thought per section. Cut repetitive copy; ask whether deleting 30 percent would improve it.
- Prefer two or three intentional motion moments, such as entrance, scroll, or hover/reveal, when useful. Flag an unconsidered static experience as an aesthetic departure, but never impose motion over reduced-motion needs, legibility, or task speed.
- A defined color system, usually one accent. Avoid purple-on-white defaults.

## App UI preferences

- Calm surface hierarchy, strong typography, few colors, and dense but readable content with minimal chrome.
- Distinct primary workspace, navigation, and secondary context, usually with one accent.
- Object to dashboard-card mosaics, thick borders, decorative gradients, and ornamental icons.
- Prefer cards when the card is an interaction, not as a substitute for layout.
- Utility copy describes orientation, status, and action. Section headings say what an area contains or what the user can do.
- Keep marketing mood statements and decorative motion out of task-focused areas.

## Shared design preferences

Use semantic color tokens or CSS variables, purposeful typography rather than default stacks, one job per section, concise copy, and cards that earn their existence. Check consistency across navigation, footers, buttons, forms, spacing, responsive behavior, and tone.

The checklist and aesthetic preferences are bundled here so the review needs no web lookup. They retain the original skill's design stance, informed by [OpenAI's frontend design guidance](https://developers.openai.com/blog/designing-delightful-frontends-with-gpt-5-4). The link is background reading, not a runtime dependency.
