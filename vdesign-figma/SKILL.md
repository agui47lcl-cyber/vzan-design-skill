---
name: vdesign-figma
description: Generate or repair VDesign Web System admin UI in Figma with real component instances, text styles, variables, and compact changed-subtree validation. Use for ordinary VDesign screens, dialogs, drawers, panels, tables, and focused repairs; use vdesign-figma-audit only for explicit full, formal, certification-level, or 100% audits.
metadata:
  short-description: Generate VDesign UI with a low-call workflow
---

<!--
[INPUT]: 依赖 Figma MCP 的 use_figma、按需 search_design_system 与 get_screenshot，依赖 references/vdesign-assets.json 的按意图资产缓存
[OUTPUT]: 对外提供低调用的 VDesign 生成与局部修复流程，在同一次写入中完成绑定和本轮子树聚合验证
[POS]: vdesign-figma 的轻量生成入口；认证级全量审计由独立的 vdesign-figma-audit 承担
[PROTOCOL]: 变更时更新此头部，然后检查 CLAUDE.md
-->

# VDesign Figma

Generate visible, editable VDesign admin UI quickly without substituting hardcoded lookalikes for real design-system assets.

## Library and routing

- VDesign file: `jjmsk6tyXH3mAEaGyR6FhL`
- Library: `VDesign Web System`
- Library key: `lk-764e6a23379883e834f1c7c139437927a42648079d028568f2f5b91578f1f1953f1fe80865e92bdb79267b6f9468ca25afa970bb065678685eb35dde47be29bb`
- Primary font: `PingFang SC`
- Load `figma-use` before every `use_figma` call. Load `figma-generate-design` only when its own trigger applies.
- Do not load `figma-generate-library` unless the user asks to build or maintain a reusable library.
- Do not call `get_design_context` for native Figma generation.
- Ordinary requests such as “使用 VDesign” or “完成后检查覆盖率” stay in this skill's focused validation. Route to `vdesign-figma-audit` only when the user explicitly asks for a full audit, formal acceptance, certification, all-canvas coverage, or 100% coverage.

## Retrieve only needed assets

Do not load the whole asset catalog into context. Query only the entries needed for the current design from [references/vdesign-assets.json](references/vdesign-assets.json), for example with `jq` by component intent, text role, resolved radius, or `resolvedPx` spacing.

Use this order:

1. Reuse a compatible VDesign instance already present in the target or reference node.
2. Import the exact cached published key.
3. Search the scoped VDesign library once only when both reuse and cache miss, an import fails, or the user asks to refresh the library.
4. Use controlled manual composition only when one exact search finds no compatible asset. Record only the query and incompatibility.

Never rediscover a cached key, repeat a query, search community libraries, detach an imported instance, or create replacement VDesign tokens.

## Fast call budget

For a simple screen, dialog, drawer, panel, or up to four states, target **three Figma calls**:

1. **Compact inspect, only when needed.** Read the target/reference and return only direct structure, reusable instance/style/variable keys, dimensions, and actual resolved spacing values. Do not dump every paint or descendant.
2. **Write + bind + validate.** In one `use_figma` script, create or update the deterministic wrapper, apply bindings, then validate only nodes changed by that script. Return aggregate counts and failure records; do not return successful node details.
3. **One final screenshot.** Take it when visual QA is needed. Do not perform another full structural read merely to produce a report.

If the reference already exposes sufficient reusable assets, combine inspection with the write and finish in two calls. One scoped search may raise the target to four calls. More than four calls is a performance regression unless a reported tool failure or user-requested strict audit explains it.

- First mutation must happen by call 2.
- Retry the same failed operation once at most. After a timeout, check the deterministic wrapper before retrying because the write may have committed.
- Stop after two consecutive connection or transport failures.
- Do not take intermediate screenshots.

## One-script generation contract

Derive one exact wrapper name and reuse only a direct child with that name. Never create a second wrapper on retry.

Within the main mutation script:

1. Build the Auto Layout structure directly inside the wrapper.
2. Reuse/import real VDesign instances and set only inspected property keys.
3. Bind semantic colors, radii, and every changed non-zero gap or padding field.
4. Materialize text last.
5. Audit the changed node IDs before returning.

Return only root dimensions, used asset keys, five coverage pairs, and failure records. Do not serialize full nodes, successful descendant records, complete fills, or complete `componentProperties` after the needed property keys are known.

## Binding invariants

### Components

- Component-mappable controls must remain `INSTANCE` nodes whose main component or component set has a VDesign published key.
- Inspect actual component properties before calling `setProperties()`. Never guess generated property suffixes.
- Preserve existing compatible instances instead of re-importing or replacing them.

### Text

- Visible semantic text must use a published VDesign `TextStyle`.
- Load the node's current fonts and the imported style font before editing.
- Set content, width, wrapping, and `textAutoResize` first; call `setTextStyleIdAsync()` last. Do not write raw typography afterward.
- Single-line labels normally use `WIDTH_AND_HEIGHT`; fixed-width or wrapped copy uses `HEIGHT`.
- Validation must compare actual `fontName`, `fontSize`, `lineHeight`, and `letterSpacing` with the resolved style. A style ID alone does not pass.
- Visible text with width or height `<= 1` fails. Never use a guessed 1px text height.

### Colors and radius

- Semantic solid paints require a VDesign color-variable alias. Capture and reassign the paint returned by `setBoundVariableForPaint()`.
- Semantic corners require VDesign `CORNER_RADIUS` aliases; raw numeric equality does not pass.

### Spacing and layout

- Use Auto Layout gap and padding fields; never spacer-only layers such as `间距/16`.
- Select spacing by `resolvedPx` from the cache, **not by the number embedded in the variable name**. VDesign spacing names are scale labels: for example `padding/padding 8` resolves to 16px and `padding/padding 16` resolves to 32px.
- Bind every changed non-zero `itemSpacing` and padding field to a VDesign `GAP` variable and verify the resulting alias within the same mutation call.
- Content containers that should follow text use vertical `HUG`; fixed-height product surfaces are allowed only when intentional.

## Focused completion gate

Validate only nodes created or changed in this run:

- visible text: published style, actual typography match, and content-driven size;
- standard controls: compatible VDesign instances;
- business-semantic colors and primary surface radii: VDesign variables;
- all changed non-zero gaps and padding: VDesign `GAP` variables;
- no spacer-only layers or duplicate wrapper;
- changed subtree reads back successfully.

Report five `passed/eligible` counts, failing node IDs only, root dimensions, asset keys used, call count, and whether the final screenshot was visually checked. Label this as **focused changed-subtree validation**, never full-canvas certification.
