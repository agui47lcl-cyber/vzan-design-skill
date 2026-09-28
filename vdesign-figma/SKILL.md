---
name: vdesign-figma
description: Generate or repair VDesign Web System admin UI in Figma with reference-led styling, non-destructive versioning, real component instances, text styles, variables, and compact changed-subtree validation. Use for ordinary VDesign screens, dialogs, drawers, panels, tables, and focused repairs; use vdesign-figma-audit only for explicit full, formal, certification-level, or 100% audits.
metadata:
  short-description: Generate VDesign UI with a low-call workflow
---

<!--
[INPUT]: 依赖 Figma MCP 的 use_figma、按需 search_design_system 与 get_screenshot，依赖 references/vdesign-assets.json 的按意图资产缓存
[OUTPUT]: 对外提供参考稿优先、默认不覆盖、运行时安全的低调用 VDesign 生成与局部修复流程，并完成本轮子树聚合验证
[POS]: vdesign-figma 的轻量生成入口；认证级全量审计由独立的 vdesign-figma-audit 承担
[PROTOCOL]: 变更时更新此头部，然后检查 CLAUDE.md
-->

# VDesign Figma

Generate visible, editable VDesign admin UI quickly without substituting hardcoded lookalikes for real design-system assets.

## Source precedence and version intent

When the user supplies several sources, assign each source one role before writing:

1. The current request overrides every source.
2. A named design reference controls visual treatment: density, alignment, upload patterns, action styling, typography roles, and component presentation.
3. A prototype controls content, fields, states, and interaction intent only. Do not copy prototype styling when a design reference exists.
4. The destination frame controls the required canvas width, shell, and layout boundaries unless the user asks to replace them.

Inspect only the named reference regions needed for the requested elements. Record the source role in the working plan so that content from a prototype cannot silently become the visual style.

Treat version intent explicitly:

- “修复、调整当前版本” with an exact target edits that target in place.
- “重新生成、重做、再出一版” creates a new sibling version when an earlier result exists, unless the user explicitly authorizes overwrite.
- “不要覆盖” is absolute: never remove, rename, or reuse an earlier version's wrapper.
- Use one stable wrapper name for the active version so a retry reuses only that version; it must not reuse an older version merely because the base name matches.

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

1. **Compact inspect, only when needed.** Resolve the exact mutation root and its ancestor `PAGE`, inspect the named reference regions, and return only direct structure, reusable instance/style/variable keys, dimensions, actual resolved spacing values, and one font-availability result when text will change. Do not dump every paint or descendant.
2. **Write + bind + validate.** In one `use_figma` script, create or update the deterministic wrapper, apply bindings, then validate only nodes changed by that script. Return aggregate counts and failure records; do not return successful node details.
3. **One final screenshot.** Take it when visual QA is needed. Do not perform another full structural read merely to produce a report.

If the reference already exposes sufficient reusable assets, combine inspection with the write and finish in two calls. One scoped search may raise the target to four calls. More than four calls is a performance regression unless a reported tool failure or user-requested strict audit explains it.

- First mutation must happen by call 2.
- Retry the same failed operation once at most. After any thrown mutation or timeout, read back the exact target IDs inside the active wrapper before retrying; distinguish committed, rolled back, and partial state instead of guessing.
- Stop after two consecutive connection or transport failures.
- Do not take intermediate screenshots.

## One-script generation contract

Derive one exact wrapper name for the active version and reuse only a direct child with that exact name on retry. A deliberate new version uses a new sibling wrapper.

Within the main mutation script:

1. Resolve the target root first, walk its ancestors until `PAGE`, and call `await figma.setCurrentPageAsync(page)` before selection or page-bound operations. A section is not a page.
2. Preflight every node that may be replaced or removed, all required imports, exact component properties, and fonts before the first destructive mutation.
3. Build the Auto Layout structure directly inside the active-version wrapper.
4. Reuse/import real VDesign instances and set only inspected property keys.
5. Bind semantic colors, radii, and every changed non-zero gap or padding field.
6. Materialize text last, then audit the changed node IDs before returning.

Return only root dimensions, used asset keys, five coverage pairs, and failure records. Do not serialize full nodes, successful descendant records, complete fills, or complete `componentProperties` after the needed property keys are known.

Keep non-essential UI work out of the mutation transaction. Take the final screenshot in the separate screenshot call. Selection and viewport changes are optional, must occur only after switching to the resolved page, and must not be allowed to invalidate a successful write. Do not call `setPluginData`; it is unsupported in the Figma host used by this workflow and is unnecessary for idempotency.

## Mutation safety and recovery

- Mutate or delete only exact IDs proven to be descendants of the active wrapper. Never clean up with a page-wide name query.
- If a failed call may have left a temporary node, locate it read-only, confirm its exact ID and parent, then remove only that node in a later scoped repair.
- Before replacing several nodes, verify all targets and parents first; do not discover a missing later target after earlier deletions have started.
- After a failure, compare exact old IDs and expected new IDs. A missing old ID plus present new ID means committed; unchanged old IDs means rolled back.
- A screenshot or selection failure does not by itself mean the mutation failed. Read back the wrapper before repairing.

## Binding invariants

### Components

- Component-mappable controls must remain `INSTANCE` nodes whose main component or component set has a VDesign published key.
- Inspect actual component properties before calling `setProperties()`. Never guess generated property suffixes.
- Preserve existing compatible instances instead of re-importing or replacing them.
- Variant component keys can differ from the cached component-set key. Validate provenance through the instance's main component and its parent component set; do not report a false failure from a direct key comparison alone.
- For inputs and text areas, keep the component's own default placeholder/copy. Never add a sibling or overlay frame that repeats text already owned by the instance.
- Determine text ownership by walking ancestors to an `INSTANCE`, not by matching strings such as “输入文案”. Rich-text editor hints and unrelated copy must not be counted as component overlays.
- Override component copy only through an inspected exposed property or an existing editable text descendant. If a checkbox/radio variant intentionally contains no label, pair the instance with a sibling label in a horizontal Auto Layout.
- Never leave an empty button instance behind another text layer. If no compatible labeled text-action variant exists after the single exact search, use one visible, style-bound blue text action and report the component incompatibility; solid buttons still require a real compatible instance.

### Text

- Visible semantic text must use a published VDesign `TextStyle`.
- Keep three states separate: the style is importable, the style is bound to the node, and the style's font is available in the current execution host. None proves the others.
- Preflight the exact font once before mutation. If `PingFang SC` cannot load, do not repeat the failing load throughout the script.
- When PingFang is unavailable, materialize new copy with an available Chinese fallback such as `Noto Sans SC`, apply the VDesign text style last, and read back both `textStyleId` and effective typography. Report `pending-font-runtime` rather than passing typography if the effective font or metrics still differ from the resolved style.
- For existing text, load every actual range font before changing characters or sizing. Do not mutate unavailable-font text in place merely to force the write through.
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
- standard controls: compatible VDesign instances, structural ownership of default copy, and no empty-instance/text-overlay pair;
- business-semantic colors and primary surface radii: VDesign variables;
- all changed non-zero gaps and padding: VDesign `GAP` variables;
- no spacer-only layers or duplicate wrapper;
- changed subtree reads back successfully.

Also verify each explicit user correction separately, such as reference-led upload styling, red required markers, blue text actions, preserved versions, or component-owned placeholder copy. Do not infer these from aggregate coverage.

Report five `passed/eligible` counts, explicit-request checks, failing node IDs only, root dimensions, asset keys used, call count, typography runtime state, and whether the final screenshot was actually inspected. Label this as **focused changed-subtree validation**, never full-canvas certification.
