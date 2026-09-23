<!--
[INPUT]: 依赖 vdesign-figma-audit/SKILL.md 的显式认证边界和 VDesign Web System 发布资产身份
[OUTPUT]: 对外提供严格审计分母、五类绑定判定、紧凑遍历、排除规则和修复顺序
[POS]: vdesign-figma-audit/references 的认证合同，只在全量审计或修复时加载
[PROTOCOL]: 变更时更新此头部，然后检查 CLAUDE.md
-->

# Strict audit contract

## Exact root and denominator

Require an exact `fileKey` and root `nodeId`. Restrict traversal to that root.

Include visible semantic UI that communicates content, accepts input, triggers actions, or defines reusable surfaces:

- all visible semantic text;
- every component-mappable control;
- every semantic solid fill and stroke, counted per paint;
- every non-zero semantic UI corner;
- every non-zero Auto Layout `itemSpacing` and padding field.

List but exclude hidden nodes, bitmap/image fills, vector artwork and icon paths, circular masks, geometry-derived pills, prototype notes/redlines, and third-party platform chrome outside VDesign ownership.

Library-instance internals are owned by the published main component. Audit their visible text and semantic paints, but exclude their internal Auto Layout fields from consumer-file spacing coverage unless the instance root itself owns the changed field. State this exclusion once as a class, not once per descendant.

## Compliance tests

### Text

A text node passes only when its non-empty style ID resolves to a remote VDesign `TextStyle`, its actual `fontName`, `fontSize`, `lineHeight`, and `letterSpacing` equal the style, its width and height exceed 1px, and its auto-resize mode suits single-line or wrapped content. A style ID alone does not pass.

### Components

A component-mappable control passes only when it is an `INSTANCE` and its remote main component or owning component set has the intended VDesign published key. Same-name frames, detached instances, and unrelated library instances fail.

### Colors

Each semantic `SOLID` fill or stroke requires `paint.boundVariables.color`. The alias must resolve to a remote VDesign color variable. Count paints, not nodes.

### Radius

Each semantic non-zero corner requires a remote VDesign variable with `CORNER_RADIUS` scope. Accept `cornerRadius` for uniform fields or require every active individual-corner field. Raw numeric equality fails.

### Spacing

Each eligible non-zero `itemSpacing`, `paddingTop`, `paddingRight`, `paddingBottom`, and `paddingLeft` requires a remote VDesign `FLOAT` variable with `GAP` scope. Count fields, not containers. Spacer-only layers are structural failures.

## Compact traversal

In one read-only script:

1. collect eligible nodes;
2. cache styles by style ID, variables by alias ID, and main components by instance ID;
3. increment category denominators and pass counts;
4. append only failures and allowed exclusions;
5. return aggregate counts, failures, exclusions, root dimensions, and node count.

Do not serialize successful nodes, full `fills`, full `strokes`, style objects, component properties, or variable objects.

## Repair order

When repair is authorized:

1. replace unambiguous manual controls with compatible VDesign instances;
2. remove spacer layers and bind owning Auto Layout fields;
3. repair text sizing, then apply the text style as the final typography write;
4. bind radii and colors;
5. re-run one strict traversal on the exact root.

Preserve position, dimensions, content, hierarchy, and already compliant nodes. Do not detach instances or invent a replacement token. If matching remains ambiguous, leave the node unresolved and report it.

## Report shape

```text
VDesign library: <name> <library key>
Root: <name> <id>
Mode: <audit | repair>
Figma calls: <count>

Text styles: <passed>/<eligible> (<percentage>)
Components: <passed>/<eligible> (<percentage>)
Colors: <passed>/<eligible paints> (<percentage>)
Radius: <passed>/<eligible> (<percentage>)
Spacing: <passed>/<eligible fields> (<percentage>)

Failures: <only failing node IDs and reasons>
Exclusions: <class or node evidence>
Validation: <structure and screenshot status>
```
