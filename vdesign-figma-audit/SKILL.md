---
name: vdesign-figma-audit
description: Run read-only or post-generation certification-level audits of VDesign Figma roots. Use only when the user explicitly requests a full audit, formal acceptance, all-canvas inspection, certification, or 100% coverage for components, text styles, colors, radii, and spacing; do not use for ordinary changed-subtree checks.
metadata:
  short-description: Certify VDesign Figma binding coverage
---

<!--
[INPUT]: 依赖 Figma MCP 的 use_figma 与 get_screenshot，依赖兄弟 Skill 的严格审计合同和 VDesign 库身份
[OUTPUT]: 对外提供显式认证级的五类绑定覆盖率、失败节点清单、排除证据和只读或生成后验收结论
[POS]: vdesign-figma-audit 的严格审计入口，与低调用的 vdesign-figma 普通生成路径隔离
[PROTOCOL]: 变更时更新此头部，然后检查 CLAUDE.md
-->

# VDesign Figma Audit

Certify an exact Figma root only when the user explicitly asks for full, formal, all-canvas, certification-level, or 100% VDesign coverage.

## Required context

- VDesign file: `jjmsk6tyXH3mAEaGyR6FhL`
- Library: `VDesign Web System`
- Library key: `lk-764e6a23379883e834f1c7c139437927a42648079d028568f2f5b91578f1f1953f1fe80865e92bdb79267b6f9468ca25afa970bb065678685eb35dde47be29bb`
- Load `figma-use` before every `use_figma` call.
- Read [audit-contract.md](references/audit-contract.md) before auditing or repairing.
- Do not load `figma-generate-design` for a read-only audit.
- Do not call `get_design_context`; inspect native Figma state directly.

## Boundary

- **audit:** read-only. Never import, create, edit, select, or attach temporary nodes.
- **repair:** audit first, then change only the recorded failures. Re-audit the affected root.
- A normal request to generate UI and “check coverage” belongs to `vdesign-figma` focused validation, not this Skill.

## Compact execution

Use one `use_figma` traversal for the entire exact root. Resolve referenced styles, variables, and main components with async APIs and cache each resolved ID in memory for the script.

Return aggregate counts plus failures and exclusions only. Never return successful node-by-node records, full paints, complete component properties, or serialized Figma nodes.

Take one screenshot only when the user requested visual certification or the structural findings require visual confirmation. A screenshot never substitutes for binding evidence.

Stop after two consecutive connection failures and retry the same failed operation once at most.

## Required result

Report:

- exact root name and ID;
- mode and Figma call count;
- text styles, component instances, colors, radii, and spacing as `passed/eligible (percentage)`;
- failing node IDs with one concise reason;
- allowed exclusions with evidence;
- structure and screenshot validation status.

Certification passes only when every eligible category is 100%. Use `0/0 (N/A)` for an empty denominator. Never describe a focused or incomplete traversal as full certification.
