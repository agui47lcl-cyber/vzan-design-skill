---
name: vdesign-figma
description: Generate, audit, and repair Figma web-admin designs with VDesign Web System components, variables, and text styles. Use when creating or modifying VDesign back-office UI in Figma, checking VDesign token/style/component bindings, or correcting hand-built controls that should be library instances.
metadata:
  short-description: Align Figma admin designs with VDesign
---

<!--
[INPUT]: 依赖 Figma MCP 的 get_libraries、search_design_system、use_figma、get_metadata 与 get_screenshot，依赖 references 中的 VDesign 资产和审计契约
[OUTPUT]: 对外提供 VDesign Figma 设计稿 generate、audit、repair 三种工作模式及结构化规范验收报告
[POS]: vdesign-figma 的工作流入口，把通用 Figma 写入能力约束到 VDesign Web System 的真实组件、变量和文本样式
[PROTOCOL]: 变更时更新此头部，然后检查 CLAUDE.md
-->

# VDesign Figma

Create, inspect, and correct VDesign web-admin interfaces in Figma. A visually similar value is not compliant unless the node remains linked to the published VDesign asset.

## Required context

- Design-system file: `jjmsk6tyXH3mAEaGyR6FhL`
- Library name: `VDesign Web System`
- Library key: `lk-764e6a23379883e834f1c7c139437927a42648079d028568f2f5b91578f1f1953f1fe80865e92bdb79267b6f9468ca25afa970bb065678685eb35dde47be29bb`
- Read [references/vdesign-assets.md](references/vdesign-assets.md) before resolving assets.
- Read [references/audit-and-repair.md](references/audit-and-repair.md) for audit or repair work and before final validation of generated work.
- Load `figma-use` before every `use_figma` call. Load `figma-generate-design` as well when building or updating a composed screen or view. Follow both skills' tool-call rules.

The asset reference is a verified snapshot, not an excuse to skip live discovery. Confirm access with `get_libraries`; use the exact library key to scope `search_design_system`. Treat published keys as identity and names only as search hints.

## Choose the mode

- **generate**: the user asks to create or update a VDesign screen, view, modal, drawer, panel, table, or admin flow.
- **audit**: the user asks to inspect compliance or requests a report. Do not mutate the file.
- **repair**: the user asks to fix an existing design. Audit first, then change only failing nodes.

When a request contains both creation and compliance language, use `generate` and run the complete audit gate afterward. Never turn an audit-only request into a repair.

## Resolve assets before writing

Build an asset-resolution table for every semantic element before the first canvas mutation. Record:

| Element intent | VDesign asset | Published key | Required properties | Text override path | Status |
| --- | --- | --- | --- | --- | --- |

For each control or pattern:

1. Check the verified asset snapshot.
2. Search the scoped VDesign library for the exact component intent when the snapshot has no entry or the entry cannot be imported.
3. Import the component or component set by published key and inspect its actual property definitions and nested instances.
4. Select the variant using source requirements and component defaults. Never guess property names or values.
5. Mark the element `resolved`, `controlled-manual`, or `blocked`. Do not start drawing while a component-mappable element remains unresolved.

For variables and styles, search the linked library as well as local assets. Empty local-variable or local-style results do not prove the library lacks them.

## Asset priority

1. Reuse a VDesign component instance and set its real variant, Boolean, text, and instance-swap properties.
2. Compose uncovered business structures from VDesign variables, text styles, effect styles, and auto layout.
3. Use controlled manual construction only after an exact component search finds no compatible asset. Record the query, candidates, and incompatibility. If the custom structure repeats or maps to a reusable source concept, create one local component and place instances.

Do not create replacement VDesign variables, text styles, or library-like components. Do not detach imported instances.

## Binding rules

### Components

- Component-mappable controls must be `INSTANCE` nodes whose main component or component-set published key belongs to VDesign.
- A frame named `Button`, `按钮`, `输入框`, or another known control is not compliant.
- Inspect `componentProperties` before overriding content. Use `setProperties()` for exposed `TEXT`, `BOOLEAN`, `VARIANT`, and `INSTANCE_SWAP` properties.
- If the component does not expose a text property, load every font returned by the target text node's styled segments, then edit only the intended instance text override.

### Text

- Every visible semantic UI text node must use a published VDesign `TextStyle` key.
- Import the selected style with `importStyleByKeyAsync()` and apply it with `setTextStyleIdAsync()`.
- Prefer an exact typography-signature match during repair. Use semantic role to break ties. Do not assign by display name alone because the library contains historical spelling and duplicate-name inconsistencies.
- Loading a font successfully does not count as applying a text style.

### Colors

- Every semantic UI `SOLID` fill or stroke must have a paint `boundVariables.color` alias to a VDesign color variable.
- Import variables by published key. Bind paints with `setBoundVariableForPaint()` and reassign the returned paint.
- Prefer semantic variables that match the node role over equal-valued primitives.

### Radius and layout values

- Every semantic UI corner must bind to a VDesign `CORNER_RADIUS` variable.
- Bind `cornerRadius` when the node exposes a uniform corner field. For nodes with individual corner fields, bind all four radius fields to the selected variable unless the design intentionally uses different corner tokens.
- Use the VDesign spacing variables for padding and gaps when a matching token exists.
- A raw numeric value equal to a token is still a failure until its variable alias is present.

## Generate

1. Inspect the target file, requested source, neighboring screens, and linked libraries.
2. Build the complete asset-resolution table.
3. Create the page structure with auto layout and VDesign instances. Build incrementally by section and return every created or mutated node ID.
4. Apply variable and style bindings at creation time; do not postpone tokenization to a cleanup pass.
5. Validate each section visually and structurally before continuing.
6. Run the audit gate. Fix every non-exempt failure before reporting completion.

## Audit

Run the read-only procedure in [references/audit-and-repair.md](references/audit-and-repair.md). Report all four coverage categories separately: text styles, radius variables, color variables, and component instances. Include node IDs and reasons for every failure and exclusion.

## Repair

1. Audit and keep the node-level findings as the repair manifest.
2. Preserve already compliant nodes.
3. Apply changes in this order: component replacement, text-style binding, radius binding, color binding, repeated custom-pattern componentization.
4. Preserve position, dimensions, content, auto-layout relationships, and intended state. Replace a manual control only when component intent and variant selection are unambiguous.
5. Re-audit the same root after each repair batch and finish with structural read-back plus screenshots.

## Completion gate

Completion requires 100% coverage among semantic UI nodes in every category. Exclusions do not enter the denominator but must be listed with evidence. A screenshot alone cannot pass the gate.

Return:

- resolved library identity and asset keys used;
- per-category numerator, denominator, percentage, and failing node IDs;
- controlled-manual decisions and search evidence;
- exclusions with reasons;
- structural and visual validation results.

