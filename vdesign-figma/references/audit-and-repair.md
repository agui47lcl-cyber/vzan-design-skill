<!--
[INPUT]: 依赖 SKILL.md 的三模式工作流和 vdesign-assets.md 的已验证 VDesign published key
[OUTPUT]: 对外提供语义节点审计分母、四类覆盖率检测、局部修复顺序、受控手绘规则和报告模板
[POS]: references 的质量门禁，把视觉一致性转化为可回读的变量、样式与组件关联证明
[PROTOCOL]: 变更时更新此头部，然后检查 CLAUDE.md
-->

# Audit and repair contract

## Audit root

Require an exact Figma `fileKey` and root `nodeId`. Resolve the root by ID and restrict traversal to that subtree. Audit mode is read-only: do not import assets, create temporary instances, change selection, or mutate plugin data. Use `getStyleByIdAsync`, `getVariableByIdAsync`, and `getMainComponentAsync` to resolve existing references.

## Semantic denominator

Count visible UI nodes that communicate content, accept input, trigger actions, or define reusable interface surfaces.

Include:

- visible text used as headings, labels, body copy, values, hints, validation, navigation, table content, or actions;
- visible solid fills and strokes on controls, panels, cards, menus, tables, dividers, badges, status surfaces, and layout containers;
- non-zero corners on those UI surfaces;
- nodes whose names, structure, or role match a known VDesign component intent.

Exclude from the denominator, but list separately:

- hidden nodes;
- bitmap/image fills and image-crop containers;
- vector artwork, logos, freeform illustrations, and icon paths;
- circles, circular avatar masks, and geometry-derived pills where the radius is half the shortest side;
- prototype notes, redlines, measurement labels, and reference annotations outside the product screen;
- third-party platform chrome that the requested VDesign scope does not own.

Do not exclude a node merely because it is difficult to repair. Each exclusion needs node ID, name, type, and one reason from the list above.

## Compliance tests

### Text styles

A semantic text node passes only when:

1. `textStyleId` is a non-empty string rather than `figma.mixed`;
2. `getStyleByIdAsync(textStyleId)` resolves;
3. the resolved style is a published VDesign text-style key from the live library or verified snapshot.

Raw `fontName`, `fontSize`, or `lineHeight` equality does not pass.

### Radius variables

A semantic corner passes only when its relevant field exists in `boundVariables` and resolves to a VDesign variable with `CORNER_RADIUS` scope. For a uniform radius, accept `cornerRadius`; for an individual-corner node, require aliases for every non-zero corner field. Raw number equality does not pass.

### Color variables

For every semantic `SOLID` paint in `fills` and `strokes`, require `paint.boundVariables.color`. Resolve the alias and confirm that its published key belongs to VDesign. Count paints, not paint-bearing nodes, so a fill and stroke are independently visible failures.

### Components

A component-mappable control passes only when the node is an `INSTANCE` and its main component or owning component-set key matches the chosen VDesign asset. A same-name frame, local lookalike, detached instance, or unrelated library instance fails.

When auditing an existing VDesign instance, preserve it even when a descendant override fails another category; repair the descendant binding without replacing the instance.

## Repair matching

### Components first

Replace a manual control only when intent, state, size, and content map unambiguously to a VDesign component. Preserve its parent index, absolute position when applicable, auto-layout sizing, dimensions where the component permits resizing, visible text, and state. Remove the old frame only after the new instance has been validated.

Inspect the imported component's live property definitions. Use exact returned property keys with `setProperties()`. Never guess generated suffixes. Never detach the replacement.

### Text styles

Choose in this order:

1. an exact canonical style key already specified by the source design;
2. a unique exact match on font family, font style, size, line height, and letter spacing;
3. a semantic role match from the canonical VDesign type ramp.

If more than one semantic choice remains, report the node as unresolved instead of inventing a style. Import the chosen style and apply its ID. Preserve characters and text sizing behavior.

### Radius variables

For repairs that should preserve appearance, match the current numeric radius to the resolved VDesign token value and bind that token. If no exact token exists, choose a new token only when the requested component or source specification determines it; otherwise leave the node unresolved.

Use `setBoundVariable("cornerRadius", variable)` for a uniform corner field. When the node exposes individual corners, bind `topLeftRadius`, `topRightRadius`, `bottomLeftRadius`, and `bottomRightRadius` separately. Verify the resulting `boundVariables` after the write.

### Color variables

First match an existing paint value to an exact semantic VDesign variable. When several variables resolve to the same color, select by node role: text, background, border, interaction, or status. Import the variable, call `setBoundVariableForPaint`, capture the returned paint, and reassign the fills or strokes array.

## Controlled manual construction

Controlled manual construction is allowed only after recording:

- the exact scoped search query;
- all plausible VDesign candidates;
- why each candidate cannot represent the required behavior or structure;
- the variables and text/effect styles used by the manual result;
- whether the pattern repeats.

One-off business composition may remain a tokenized frame. A repeated pattern or reusable source concept must become one local component with instances. Do not create a local component whose purpose duplicates a VDesign asset.

## Mutation discipline

- Keep each `use_figma` call small and return every created or mutated node ID.
- Load actual fonts before editing text content or traversing text properties that require loaded fonts.
- Batch independent imports with `Promise.all`.
- Reassign immutable fill and stroke arrays.
- Use auto layout for structural child relationships.
- Do not mutate nodes that already pass.
- Re-read the affected subtree after every repair batch before continuing.

## Required report

Return a compact report with this structure:

```text
VDesign library: <name> <library key>
Root: <node name> <node id>

Text styles: <passed>/<eligible> (<percentage>)
Radius variables: <passed>/<eligible> (<percentage>)
Color variables: <passed>/<eligible paints> (<percentage>)
Components: <passed>/<component-mappable> (<percentage>)

Failures:
- <category> <node id> <node name>: <reason>; expected <asset key>

Controlled manual:
- <node or intent>: <search evidence and decision>

Exclusions:
- <node id> <node name>: <allowed exclusion reason>

Validation:
- structure: <result>
- screenshots: <result>
```

Completion requires every category to reach 100% after exclusions. If a category has no eligible nodes, report `0/0 (N/A)` rather than 100%.

## Regression fixture

Use this known sample for read-only regression checks:

- file: `qim2RjyYi833JXyFeIJd88`
- root: `3912:125405`
- root name: `客服反馈优化二期｜侧栏售后协同方案`

Verified baseline on 2026-09-12:

| Metric | Baseline |
| --- | ---: |
| Text nodes | 104 |
| Text nodes with a style ID | 88 |
| Non-zero corner nodes | 48 |
| Non-zero corner nodes with a radius alias | 31 |
| Solid paints | 201 |
| Solid paints with a color alias | 162 |
| Instances | 14 |

The fixture contains three hand-built button frames: `3905:220`, `3905:222`, and `3905:367`. An audit must flag them as component failures and resolve `Button` to component-set key `aa98f5b34d9385d5e938c3c9750ea312b80a1d97`. An audit must not flag the 88 already styled text nodes or the 31 already bound non-zero corners as missing their respective bindings.
