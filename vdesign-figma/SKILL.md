---
name: vdesign-figma
description: Generate, audit, and repair Figma web-admin designs with VDesign Web System components, variables, and text styles. Use when creating or modifying VDesign back-office UI in Figma, checking VDesign token/style/component bindings, or correcting hand-built controls that should be library instances.
metadata:
  short-description: Align Figma admin designs with VDesign
---

<!--
[INPUT]: 依赖 Figma MCP 的 get_libraries、search_design_system、use_figma、get_metadata 与 get_screenshot，依赖 references 中的机器可读资产缓存和分级审计契约
[OUTPUT]: 对外提供 generate-fast、generate-strict、audit、repair 四种工作模式，以及文本真实生效、布局自适应和间距变量绑定验收
[POS]: vdesign-figma 的工作流入口，在快速可见交付与认证级绑定证明之间分流，并阻止样式 ID 假绑定、1px 文本和占位间距进入交付
[PROTOCOL]: 变更时更新此头部，然后检查 CLAUDE.md
-->

# VDesign Figma

Create, inspect, and correct VDesign web-admin interfaces in Figma. Default generation optimizes for a fast, usable first result without falling back to hardcoded substitutes; certification-level coverage is explicit.

## Required context

- Design-system file: `jjmsk6tyXH3mAEaGyR6FhL`
- Library name: `VDesign Web System`
- Library key: `lk-764e6a23379883e834f1c7c139437927a42648079d028568f2f5b91578f1f1953f1fe80865e92bdb79267b6f9468ca25afa970bb065678685eb35dde47be29bb`
- Read [references/vdesign-assets.json](references/vdesign-assets.json) as the primary key cache. Read [references/vdesign-assets.md](references/vdesign-assets.md) only for caveats, ambiguous matches, or cache maintenance.
- Read [references/audit-and-repair.md](references/audit-and-repair.md) for audit or repair work, and use its mode-specific validation table when completing generated work.
- Load `figma-use` before every `use_figma` call. Load `figma-generate-design` as well when building or updating a composed screen or view. Follow both skills' tool-call rules.
- Do not load `figma-generate-library` unless the user explicitly asks to create or maintain a reusable component library.
- Do not call `get_design_context` for native Figma design generation. It belongs to design-to-code workflows.

Treat published keys as identity and names only as search hints. Import cached keys directly. Use live discovery only when the cache has no suitable entry, a cached key fails to import, or the user explicitly asks to refresh the design system. Scope every search to the exact VDesign library key; never search community libraries silently.

## Choose the mode

- **generate-fast** (default): the user asks to create or update a VDesign screen, view, modal, drawer, panel, table, or admin flow. Deliver the visible design first, then validate only the changed subtree against the focused generation gate.
- **generate-strict**: the user explicitly asks for complete audit, formal delivery, certification, or 100% binding coverage. Generate, then run the full strict audit gate.
- **audit**: the user asks to inspect compliance or requests a report. Do not mutate the file.
- **repair**: the user asks to fix an existing design. Audit first, then change only failing nodes.

General phrases such as "use VDesign" or "follow the design system" do not imply strict mode. Use `generate-strict` only for explicit certification-level intent. Never turn an audit-only request into a repair.

## Resolve assets progressively

Before the first canvas mutation, resolve only the page skeleton, visible type roles, and primary controls. Keep one working asset-resolution table and extend it region by region:

| Element intent | VDesign asset | Published key | Required properties | Text override path | Status |
| --- | --- | --- | --- | --- | --- |

For each control or pattern:

1. Look up the intent in `vdesign-assets.json` and batch-import cached keys.
2. Search the scoped VDesign library once only when the cache misses or import fails.
3. Inspect the imported component's actual property definitions before setting variant, Boolean, text, or instance-swap properties.
4. Mark the element `cached`, `resolved-live`, `controlled-manual`, or `blocked`.
5. Continue drawing all resolved regions. A local unknown blocks only that region, not the rest of the screen.

Do not repeat the same search intent or query in one run. Empty local-variable or local-style results do not prove the linked library lacks the asset; consult the cache, then use the single scoped live search allowance.

## Figma call budget

For `generate-fast`, count every Figma MCP read, search, script, and screenshot call.

- Make the first canvas mutation no later than call 4. Before it, use at most three calls to inspect the target and resolve skeleton assets.
- Search each distinct asset intent at most once. Reuse the working resolution table across sections.
- Retry the same failed operation at most once. Before retrying a timed-out mutation, read back the intended wrapper because the first call may have committed.
- After two consecutive connection or transport failures, stop Figma work and report the last confirmed state. Do not enter an open-ended recovery loop.
- Take one final screenshot by default. Add another only when a structural read-back cannot diagnose a visible defect, and report why.
- A simple design with up to four states should finish within six total Figma calls. Exceeding this is a performance failure to report, even when the design itself succeeds.

Strict audit work may exceed the six-call target, but the per-intent search limit, retry limit, and two-failure stop condition still apply.

## Asset priority

1. Reuse a VDesign component instance and set its real variant, Boolean, text, and instance-swap properties.
2. Compose uncovered business structures from VDesign variables, text styles, effect styles, and auto layout.
3. Use controlled manual construction only after the cache misses and the single exact component search finds no compatible asset. Record the query, candidates, and incompatibility. If the custom structure repeats in the requested design, create one local component and place instances.

Do not create replacement VDesign variables, text styles, or library-like components. Do not detach imported instances.

## Binding rules

Apply these rules to nodes eligible under the selected mode's denominator. They define what passes; they do not expand `generate-fast` into a full-canvas audit.

### Components

- Component-mappable controls must be `INSTANCE` nodes whose main component or component-set published key belongs to VDesign.
- A frame named `Button`, `按钮`, `输入框`, or another known control is not compliant.
- Inspect `componentProperties` before overriding content. Use `setProperties()` for exposed `TEXT`, `BOOLEAN`, `VARIANT`, and `INSTANCE_SWAP` properties.
- If the component does not expose a text property, load every font returned by the target text node's styled segments, then edit only the intended instance text override.

### Text

- Every visible semantic UI text node must use a published VDesign `TextStyle` key.
- Import the selected style with `importStyleByKeyAsync()`. Load both the text node's current font and the imported style's `fontName` before changing characters or applying the style.
- Set characters, wrapping mode, and width first. Apply the imported style with `setTextStyleIdAsync()` as the final typography write. Do not write `fontName`, `fontSize`, `lineHeight`, `letterSpacing`, or paragraph properties afterward.
- A non-empty single-line label should normally use `WIDTH_AND_HEIGHT`. Wrapped or fixed-width copy must use `HEIGHT`, retain its intended width, and grow vertically from content. Never call `resize()` with a guessed height such as `1`.
- After binding, resolve the applied style and compare the node's actual `fontName`, `fontSize`, `lineHeight`, and `letterSpacing` with it. `textStyleId` alone does not pass. Every visible non-empty text node must have `width > 1`, `height > 1`, and a height consistent with at least one resolved line.
- If the imported PingFang font cannot be loaded or actual typography differs after one final style application, mark the node `typography-pending`; do not substitute a fallback font and report it as compliant.
- Prefer an exact typography-signature match during repair. Use semantic role to break ties. Do not assign by display name alone because the library contains historical spelling and duplicate-name inconsistencies.
- Loading a font successfully does not count as applying a text style.

### Colors

- Every semantic UI `SOLID` fill or stroke must have a paint `boundVariables.color` alias to a VDesign color variable.
- Import variables by published key. Bind paints with `setBoundVariableForPaint()` and reassign the returned paint.
- Prefer semantic variables that match the node role over equal-valued primitives.

### Radius

- Every semantic UI corner must bind to a VDesign `CORNER_RADIUS` variable.
- Bind `cornerRadius` when the node exposes a uniform corner field. For nodes with individual corner fields, bind all four radius fields to the selected variable unless the design intentionally uses different corner tokens.
- A raw numeric value equal to a token is still a failure until its variable alias is present.

### Layout and spacing

- Use Auto Layout for repeated vertical or horizontal relationships. Represent spacing with `itemSpacing` and padding fields, never with empty frames, rectangles, or layers named like `间距/16`.
- Import cached VDesign spacing variables such as `padding/padding 16` with `figma.variables.importVariableByKeyAsync()`; when a used value is not cached, search all exact required spacing values in one batched design-system call.
- Bind `itemSpacing`, `paddingTop`, `paddingRight`, `paddingBottom`, and `paddingLeft` with `setBoundVariable()` when the matching VDesign variable has `GAP` scope. A matching raw number does not pass.
- Use `layoutSizingVertical = "HUG"` for content containers that should follow child height. Fixed-height product surfaces are allowed only when the source requires them and their text descendants still auto-size correctly.
- Read back every changed Auto Layout container and verify both the numeric value and `boundVariables` alias. Spacing bindings are part of `generate-fast`, not deferred to strict audit.

## Generate: two passes

1. Inspect the target root and relevant neighboring screen conventions. Do not inventory the whole file unless the requested scope requires it.
2. Derive one deterministic wrapper name from the requested screen or flow name. Under the exact target root, reuse only a direct child with that exact name; otherwise create it once and retain its returned node ID for the run.
3. **Pass 1 — structure:** build Auto Layout structure, content, and primary VDesign component instances incrementally by section. Use real gap and padding fields rather than spacer layers. Return every created or mutated node ID.
4. After a timeout or retry, resolve the wrapper by its retained ID or exact direct-child name and continue missing sections. Never create a second wrapper with the same name.
5. **Pass 2 — bindings:** batch-apply semantic color variables, major surface radius variables, and all changed spacing fields. Materialize text last: load the imported style font, configure auto-resize and width, then apply the text style as the final typography write.
6. Read back the changed subtree once. Use one final screenshot for visual validation.
7. In `generate-fast`, run the focused generation gate only on nodes created or modified in this run. In `generate-strict`, run the full strict gate and fix every non-exempt failure.

## Audit

Run the strict read-only procedure in [references/audit-and-repair.md](references/audit-and-repair.md). Report all five binding categories separately: text styles, radius variables, color variables, component instances, and spacing variables. Include node IDs and reasons for every failure and exclusion.

## Repair

1. Audit and keep the node-level findings as the repair manifest.
2. Preserve already compliant nodes.
3. Apply changes in this order: component replacement, Auto Layout and spacer repair, content-driven text sizing plus final text-style binding, radius binding, color binding, repeated custom-pattern componentization.
4. Preserve position, dimensions, content, auto-layout relationships, and intended state. Replace a manual control only when component intent and variant selection are unambiguous.
5. Re-audit the affected manifest after each repair batch and finish with one structural read-back plus one screenshot. Run a full-root re-audit at completion.

## Completion gates

`generate-fast` passes when the changed subtree satisfies all of these focused checks:

- every visible text node has a published VDesign text style;
- standard controls such as Button, Input, Select, Checkbox, Radio, and Switch use compatible VDesign instances;
- business-semantic colors and primary card or panel radii use VDesign variables;
- every changed Auto Layout gap and padding value uses a VDesign `GAP` variable, with no spacer layers;
- every visible text node has content-driven height and actual typography matching its resolved text style;
- the changed subtree can be read back and the final screenshot shows the requested states without obvious breakage.

Generic layout containers, decorative dividers, image geometry, and untouched pre-existing nodes are outside the fast denominator. List unresolved changed nodes without expanding the audit to the full canvas.

`generate-strict`, `audit`, and completed `repair` require 100% coverage among all semantic UI nodes in the strict root. Exclusions do not enter the denominator but must be listed with evidence. A screenshot alone cannot pass either gate.

Return:

- resolved library identity and asset keys used;
- mode and Figma call count;
- per-category numerator, denominator, percentage, and failing node IDs for that mode's denominator;
- controlled-manual decisions and search evidence;
- exclusions with reasons;
- structural and visual validation results.
