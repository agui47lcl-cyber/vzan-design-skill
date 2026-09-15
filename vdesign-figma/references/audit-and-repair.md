<!--
[INPUT]: 依赖 SKILL.md 的四模式工作流、vdesign-assets.json 的机器可读缓存和 vdesign-assets.md 的资产语义说明
[OUTPUT]: 对外提供快速生成与认证级审计两套分母、五类绑定覆盖率、文本真实生效与自适应尺寸检测、局部修复顺序和报告模板
[POS]: references 的分级质量门禁，让普通生成证明本轮关键节点和布局可靠性，让审计与修复继续提供全量绑定证明
[PROTOCOL]: 变更时更新此头部，然后检查 CLAUDE.md
-->

# Audit and repair contract

## Audit root

Require an exact Figma `fileKey` and root `nodeId`. Resolve the root by ID and restrict traversal to that subtree. Audit mode is read-only: do not import assets, create temporary instances, change selection, or mutate plugin data. Use `getStyleByIdAsync`, `getVariableByIdAsync`, and `getMainComponentAsync` to resolve existing references.

For `generate-fast`, the audit root is the wrapper created or reused by the current run, and eligibility is restricted to nodes created or modified by that run. For `generate-strict`, `audit`, and completed `repair`, eligibility covers the entire requested root.

## Mode-specific denominator

| Category | `generate-fast` | `generate-strict`, `audit`, completed `repair` |
| --- | --- | --- |
| Text | All visible changed semantic text | All visible semantic text in the root |
| Components | Changed standard controls with a known VDesign intent | Every component-mappable control in the root |
| Colors | Changed business-semantic paints: text, action, brand, status, validation, and primary borders | Every semantic solid fill and stroke, counted per paint |
| Radius | Changed primary cards, panels, menus, dialogs, and standard controls | Every non-zero semantic UI corner |
| Spacing | Every changed non-zero Auto Layout gap and padding field | Every non-zero Auto Layout gap and padding field in the root |
| Structure | Changed wrapper and requested states must read back | Entire root must read back |

In `generate-fast`, generic layout-container backgrounds, decorative dividers, spacer frames, image geometry, and untouched pre-existing nodes do not enter the denominator. This is a scoped delivery gate, not a claim that the full canvas is certified.

## Strict semantic denominator

Count visible UI nodes that communicate content, accept input, trigger actions, or define reusable interface surfaces.

Include:

- visible text used as headings, labels, body copy, values, hints, validation, navigation, table content, or actions;
- visible solid fills and strokes on controls, panels, cards, menus, tables, dividers, badges, status surfaces, and layout containers;
- non-zero corners on those UI surfaces;
- non-zero `itemSpacing` and padding fields on Auto Layout containers;
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

Apply these tests only to nodes eligible under the selected mode's denominator. A node excluded from `generate-fast` may still be eligible and fail in a later strict audit.

### Text styles

A semantic text node passes only when:

1. `textStyleId` is a non-empty string rather than `figma.mixed`;
2. `getStyleByIdAsync(textStyleId)` resolves;
3. the resolved style is a published VDesign text-style key from the live library or verified snapshot;
4. the node's actual `fontName`, `fontSize`, `lineHeight`, and `letterSpacing` equal the resolved style rather than a fallback or local override;
5. visible non-empty text has `width > 1`, `height > 1`, and a height consistent with at least one resolved line;
6. a fixed-width or wrapped text uses `textAutoResize = "HEIGHT"`, while an unconstrained single-line label normally uses `WIDTH_AND_HEIGHT`.

Raw typography equality without the published style ID does not pass. The style ID without matching actual typography does not pass either.

### Radius variables

A semantic corner passes only when its relevant field exists in `boundVariables` and resolves to a VDesign variable with `CORNER_RADIUS` scope. For a uniform radius, accept `cornerRadius`; for an individual-corner node, require aliases for every non-zero corner field. Raw number equality does not pass.

### Color variables

For every semantic `SOLID` paint in `fills` and `strokes`, require `paint.boundVariables.color`. Resolve the alias and confirm that its published key belongs to VDesign. Count paints, not paint-bearing nodes, so a fill and stroke are independently visible failures.

### Spacing variables

For every eligible non-zero `itemSpacing`, `paddingTop`, `paddingRight`, `paddingBottom`, and `paddingLeft`, require the corresponding field in `boundVariables`. Resolve the alias and confirm that it belongs to VDesign and exposes `GAP` scope. Count fields, not containers.

An empty frame, rectangle, or shape used only to create visual distance is a structural failure even if its width or height equals a VDesign token. Report spacer layers separately and replace them with the parent Auto Layout gap or padding binding.

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

If more than one semantic choice remains, report the node as unresolved instead of inventing a style. Import the chosen style, load its `fontName`, preserve characters, configure content-driven sizing, and apply the style as the final typography write. Do not set raw typography properties afterward.

For wrapped text, use `textAutoResize = "HEIGHT"` and preserve the intended width. For unconstrained labels, use `WIDTH_AND_HEIGHT`. Never preserve an accidental height of `1`; any visible non-empty text with width or height less than or equal to `1` must be repaired before completion.

After the write, resolve the style again and compare the actual font family/style, size, line height, and letter spacing. If PingFang cannot be loaded, report `typography-pending` instead of accepting a fallback. Do not rely on a manual detach-and-undo cycle to materialize the intended style.

### Radius variables

For repairs that should preserve appearance, match the current numeric radius to the resolved VDesign token value and bind that token. If no exact token exists, choose a new token only when the requested component or source specification determines it; otherwise leave the node unresolved.

Use `setBoundVariable("cornerRadius", variable)` for a uniform corner field. When the node exposes individual corners, bind `topLeftRadius`, `topRightRadius`, `bottomLeftRadius`, and `bottomRightRadius` separately. Verify the resulting `boundVariables` after the write.

### Color variables

First match an existing paint value to an exact semantic VDesign variable. When several variables resolve to the same color, select by node role: text, background, border, interaction, or status. Import the variable, call `setBoundVariableForPaint`, capture the returned paint, and reassign the fills or strokes array.

### Spacing variables

Remove spacer-only layers and move their value to the owning Auto Layout container. Import the matching VDesign `GAP` variable and bind the exact `itemSpacing` or padding field with `setBoundVariable()`. Preserve the resulting layout dimensions and verify the alias after the write.

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
- Retain the run wrapper ID and deterministic exact name defined in `SKILL.md`; retries must resolve that direct child and continue only missing sections. `setPluginData()` is not supported by `use_figma` and must not be used.
- After a timed-out mutation, read back before the single allowed retry because the original call may have committed.
- Re-read the affected manifest after every repair batch. Reserve full-root traversal for final strict validation.

## Required report

Return a compact report with this structure:

```text
VDesign library: <name> <library key>
Root: <node name> <node id>
Mode: <generate-fast | generate-strict | audit | repair>
Figma calls: <used>/<target or N/A>

Text styles: <passed>/<eligible> (<percentage>)
Radius variables: <passed>/<eligible> (<percentage>)
Color variables: <passed>/<eligible paints> (<percentage>)
Components: <passed>/<component-mappable> (<percentage>)
Spacing variables: <passed>/<eligible fields> (<percentage>)

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

Completion requires every category in the selected mode's denominator to reach 100% after exclusions. `generate-fast` must label the result as scoped validation; it must not claim full-canvas compliance. If a category has no eligible nodes, report `0/0 (N/A)` rather than 100%.

## Fast-generation performance regression

For a simple screen or flow with up to four visual states, the default `generate-fast` acceptance target is:

| Metric | Target |
| --- | ---: |
| Total Figma MCP calls | <= 6 |
| Calls before first canvas mutation | <= 3 |
| Repeated searches for the same intent | 0 |
| Retry attempts for one failed operation | <= 1 |
| Final screenshots | 1 |
| Duplicate run wrappers | 0 |
| Visible text nodes with width or height <= 1 | 0 |
| Spacer-only layers | 0 |
| Unbound changed spacing fields | 0 |
| Context compaction | 0 |

Record the call count and any budget violation in the final report. A visually successful result that exceeds the call target passes design validation but fails the performance regression.

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
