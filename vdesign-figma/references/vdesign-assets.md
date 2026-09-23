<!--
[INPUT]: 依赖 VDesign Web System 发布库及规范文件 jjmsk6tyXH3mAEaGyR6FhL 的只读盘点结果
[OUTPUT]: 对外解释稳定库身份、圆角与真实解析间距、中文文本样式和主要组件的 published key 语义及已知例外
[POS]: references 的人工可读资产说明，与 vdesign-assets.json 同源；JSON 以 resolvedPx 负责按需命中，本文只在歧义、异常或维护缓存时加载
[PROTOCOL]: 变更时更新此头部，然后检查 CLAUDE.md
-->

# VDesign asset snapshot

Verified against the published source and the dialog read-back on 2026-09-23.

## Library identity

| Field | Value |
| --- | --- |
| File key | `jjmsk6tyXH3mAEaGyR6FhL` |
| Library | `VDesign Web System` |
| Library key | `lk-764e6a23379883e834f1c7c139437927a42648079d028568f2f5b91578f1f1953f1fe80865e92bdb79267b6f9468ca25afa970bb065678685eb35dde47be29bb` |
| Primary variable collection | `vzan` |
| Primary font | `PingFang SC` |

Use [vdesign-assets.json](vdesign-assets.json) for routine lookup and batch import. Do not revalidate a cached key merely because a new generation run started. Search only this library when a stored key cannot be imported, no cached intent matches, or the user explicitly requests a refresh. Do not silently search community libraries.

## Cache policy

- A successful cached import is sufficient for `generate-fast`; record it as `cached` and continue.
- One failed cached import permits one exact scoped search for that intent. Store the result in the working table, but do not edit the snapshot unless the user asked to refresh it.
- `vdesign-figma-audit` verifies identity through existing published keys during explicit certification; it does not require a redundant pre-generation search.
- Keep this Markdown and the JSON cache aligned whenever a verified key is intentionally updated.

## Spacing variables

VDesign spacing names are scale labels, not literal pixels. Select by `resolvedPx`; using the number in the name doubles the intended layout value and causes avoidable generate-read-repair calls.

| Resolved px | Variable name | Scope | Published key |
| ---: | --- | --- | --- |
| 2 | `padding/padding 1` | `GAP` | `3a400f14f3ea3c4c7b1f9060a26e067a381e1a03` |
| 4 | `padding/padding 2` | `GAP` | `d5c076053f5ff7f456320c607cf842f9e53b9fc0` |
| 6 | `padding/padding 3` | `GAP` | `6fa1c8cdca4a6da89ddfcc2f77c99fa7f7bef8a9` |
| 8 | `padding/padding 4` | `GAP` | `f10aa609c8a9342ede37216d2412dcdec8c35067` |
| 10 | `padding/padding 5` | `GAP` | `330342c9d23f59eb87545da707023643f3157b8d` |
| 12 | `padding/padding 6` | `GAP` | `d727bebe46b6e69d9307a4ab5b11c0bee71da0eb` |
| 16 | `padding/padding 8` | `GAP` | `4718f622ed5831b35031f908772d9e295721db42` |
| 20 | `padding/padding 10` | `GAP` | `2ec4a1093abe71ea4ff9c79d3d46ee067c56992b` |
| 24 | `padding/padding 12` | `GAP` | `93e576637fba54e172893a1df7963150d42c2857` |
| 32 | `padding/padding 16` | `GAP` | `5a0bb5cfe4b1e6af9afa98e3b6385c6acf82358a` |
| 40 | `padding/padding 20` | `GAP` | `0e2470addc030e13d8c901941463db5f152d3c19` |

Never create an empty frame named `间距/16`; bind the owning Auto Layout field. When a required `resolvedPx` is absent, run one exact scoped search and verify the imported variable's actual value before writing.

## Radius variables

The names are historical labels. Use the resolved value and published key.

| Resolved px | Variable name | Published key |
| ---: | --- | --- |
| 0 | `radius/None` | `fc6b9e9fa338aa0ec0cd4ffaacbd474978e14126` |
| 2 | `radius/radius 1` | `b5942515521e7d4796163238114282a897001965` |
| 4 | `radius/radius 2` | `d58f4e5f5f776b9b5a034f5b5afca5354cc74fe9` |
| 6 | `radius/radius 3` | `f60c8ddeae2fddd19696f04af1ee481d66c27362` |
| 8 | `radius/radius 4` | `8b50b57d307ad2e2800e3ae9051c60602ec2fdaa` |
| 10 | `radius/radius 5` | `e6d9262c10cddce0d6afb5d3005f4cfb68475bc9` |
| 12 | `radius/radius 6` | `a01b6ca1102eb7c1ff7b06e08166d65be68c3542` |
| 16 | `radius/radius 8` | `3522580b10f64cbe6864cf9be3ccfc8dd969d204` |
| 20 | `radius/radius 10` | `9feea3831b96ad790c94f09ae0ce5fcfb4f2a39c` |

Pills, circles, and image masks may use geometry-derived radii and are audit exclusions when they are not semantic controls.

## Canonical Chinese text styles

Names contain existing spelling such as `Reagular`. Preserve those names in reports but identify and import styles by key.

| Role | Style name | Size / line height | Published key |
| --- | --- | --- | --- |
| Body | `正文/12/12·Regular` | 12 / 20 | `a76c6f52e42b9b27e5061a7c95ea949248bb518f` |
| Body | `正文/12/12·Medium` | 12 / 20 | `c8fee2c5fda22010f36d7cdff76cd9e7e72e3fe3` |
| Body | `正文/12/12·bold` | 12 / 20 | `6727571298ca7ada1d150e673922bbf77c1eb8cb` |
| Body | `正文/13/13·Regular` | 13 / 20 | `9b08f6624ab91df5d8627ce05cdfe47fe684ae4a` |
| Body | `正文/13/13·Medium` | 13 / 20 | `6214b9fd577ddef99fdb66e7d80c90221fc1e44c` |
| Body | `正文/13/13·bold` | 13 / 20 | `e90ce005e2ea53d109fee368fca2dcc213824fee` |
| Body | `正文/14/14·Regular` | 14 / 22 | `1ece1ed6bc5d0ba4465fbe86b0889fae4b396153` |
| Body | `正文/14/14·Medium` | 14 / 22 | `526be3d4b7918921f39af7664994d35b80e1dd98` |
| Body | `正文/14/14·bold` | 14 / 22 | `592efb59abd65917f50ee5a7498da274a5e144b0` |
| Label | `标签/10/10·Reagular` | 10 / 18 | `cd1dcf6a3217db82b1d924ac4c8c5562d41df6c2` |
| Label | `标签/10/10·Medium` | 10 / 18 | `b2d3e0838b2c2103b9cb4eb423c6e7a9a1d8b5f3` |
| Label | `标签/10/10·bold` | 10 / 18 | `fe66ea7c18628c226615574aa590979bbb170a9a` |
| Label | `标签/11/11·Reagular` | 11 / 18 | `b33f8984e6a74baf0a97ae496a16be0c5e4a0031` |
| Label | `标签/11/11·Medium` | 11 / 18 | `e0346a26904d9f9db7697b8f33f7e51493eedb53` |
| Label | `标签/11/11·bold` | 11 / 18 | `e05e3ed82fae1c89590b73c9135c76b19517c737` |
| Heading | `标题/16/16·Medium` | 16 / 24 | `73f188c4ae1b75522276bf3b35f08844b68471b2` |
| Heading | `标题/16/16·bold` | 16 / 24 | `585fdb0317a76c1e06c13c8f7c1466f10a1dee4a` |
| Heading | `标题/18/18·Reagular` | 18 / 26 | `541dc58c9ce440a99f56c18c9ba93f0635f2a621` |
| Heading | `标题/18/18·Medium` | 18 / 26 | `b2963576e37afed8f17a5adedbb4715210ea9edc` |
| Heading | `标题/20/20·Medium` | 20 / 28 | `b901cd16a6ac6038d726fc6264dbe7bd8b3f78d7` |
| Heading | `标题/20/20·bold` | 20 / 28 | `491ed76e3bdd15425c675aaea9feac3d2048ae0b` |
| Heading | `标题/24/24·Medium` | 24 / 32 | `2b0ac8e5bb78991931ecea0bce2e362fd6c43dcf` |
| Heading | `标题/24/24·bold` | 24 / 32 | `beaeb44cd171fcc8a0e3aa81a0fb82711d4ab4fa` |
| Heading | `标题/28/28·bold` | 28 / 36 | `c22dfdb9d71e3b40dae4f1db4f75cc136d2fe57d` |
| Heading | `标题/32/32·Medium` | 32 / 40 | `9e0c3b5d9152760b634306e7ffcbd0a017a7e6a0` |
| Heading | `标题/32/32·bold` | 32 / 40 | `be3da65184703da4ed8014cca21f15a86a7fb813` |
| Heading | `标题/36/36·Medium` | 36 / 44 | `4d62bc4137fe90561bf6546682d4a376de069a74` |
| Heading | `标题/36/36·bold` | 36 / 44 | `53c6225d5dad03e6aee7be8956cc3cbde34960b2` |
| Heading | `标题/40/40·Medium` | 40 / 48 | `fd10ea0c0b6c0a5ea19015a78aea45f02ac457fd` |
| Heading | `标题/40/40·bold` | 40 / 48 | `a797d643213363d6891a4230945d9e57a1423a42` |
| Heading | `标题/48/48·Medium` | 48 / 56 | `5a0f5c561a8c284b856801b0f752ceb25675f594` |
| Heading | `标题/48/48·bold` | 48 / 56 | `d2a6d74a5311fede21578a3b56d1a7bfeadb5891` |
| Heading | `标题/56/56·Medium` | 56 / 66 | `5eed44d1d33a28091db644ac30722813aa165cea` |
| Heading | `标题/56/56·bold` | 56 / 64 | `457f1d38c81656f20829655d23d5fefbaef56035` |

`标题/28/28·Medium` currently resolves to 18 / 26 despite its name. Do not select it by name-derived size. Legacy `正文·14/14·Medium` with key `48e022395306066e2fbcde49b22c97152a855fed` appears in existing work but is not the canonical published entry; new work should use `正文/14/14·Medium`.

## Primary published components

The property lists are lookup aids. Read the imported component's current definitions before calling `setProperties()`.

| Intent | Published component set | Key | Main properties |
| --- | --- | --- | --- |
| Button | `Button` | `aa98f5b34d9385d5e938c3c9750ea312b80a1d97` | 类型、种类、尺寸、状态、图标、替换图标 |
| Button group | `Button Group` | `d77614113ac557096770fa784e784cf73dc34c02` | 尺寸、数量 |
| Breadcrumb | `面包屑` | `4bf10c7a89a9af5f5db36aaad55a9e9bd7096f4f` | 层级 |
| Pagination | `多页数` | `fdb3090880c3ad9f6fd0aa85c2f890fb6d023509` | 跳转、显示总数、翻页 |
| Horizontal tabs | `.短蓝条` | `2feb616a6214259920d1d14bccfb100e02f7a479` | 样式、数量 |
| Horizontal tabs | `长蓝条` | `60a07e689d92eddeb04157a9ad87b48fd6581f5d` | 数量、图标 |
| Navigation menu | `.菜单` | `81abafd063101988ae8556742dd09196eef08d8e` | 样式 |
| Text input | `输入框` | `9ca0420a3f8ae10776e9ac5cc71556e93bbbe906` | 状态、图标、字数、前缀、下拉、注释 |
| Numeric input | `数字输入框` | `3301cfc44bc61a6dfda62a658243667effdcf5cc` | 状态、图标、字数、注释 |
| Search input | `搜索框` | `61e575425ffca091a3af2fe21ed69f6cc3efff52` | 状态、分类、下拉 |
| Text area | `文本框` | `2199b5e4d5ffba965e78c87ec8e2e5835699a358` | 状态 |
| Radio | `单选框` | `80bd0103c1373858c60daa0f0327c8471f083249` | 文案、选中、状态 |
| Checkbox | `多选框` | `c4f7cff011fffc8afc7ee61b336a3e0e9ad16587` | 文案、状态、鼠标移入、禁用 |
| Switch | `开关` | `afef94c279e0f17037788fbf0b66f59edb7dc656` | 尺寸、开关、状态 |
| Select | `下拉框` | `539d984bfbf58a418a1edd2adba5c454714ac8b6` | 图标、多选、状态 |
| Select group | `下拉框组` | `79dca4aca3d9a0d62bb9edcaf8d418264a5d4061` | 多选、下拉 |
| Confirmation dialog | `提示弹窗` | `247d6c7069662f333b1c72c2a7d9b0ab97641e04` | 图标、操作、类型、标题 |
| Custom dialog | `自定义弹窗` | `c09037d89cd8c89f0cbcc0c3641bcc15b10123bf` | 窗口大小 |
| Inline alert | `公告提示` | `e39e633c22c2d2b903987eb87b9bae2959e41ce4` | 关闭、操作、文案、样式 |
| Global alert | `全局提示` | `652e5c8561eadb057aec0c5b828f9b3d01dc2e71` | 关闭、操作、文案、样式 |
| Table header | `表头` | `b47135ef4bb21c7e5a149fcbb5087de02843ca55` | 多选框、提示、排序、种类 |
| Table cell | `表格单元` | `4324c30fa436fd58a7048d66923d9188516998c1` | 内容类型及显示属性 |
| Title bar | `标题栏` | `21af895f8d9dcb7d72a4a6a414ec48557479eaa6` | 后缩按钮 |

The Button set does not currently expose a TEXT property in its top-level definitions. Inspect the instance for nested text properties; if none exist, override the intended text descendant after loading its actual font. Dot-prefixed component sets are internal building blocks: prefer their public composite when one exists.

No component set was found on the current drawer page snapshot. Search the live VDesign library for `抽屉` before using controlled manual construction.

## Dynamic assets

Color and spacing inventories are larger and evolve more often. Reuse any keys already resolved in the current run or target file. When no cached or local linked asset matches, use the single live-search allowance against the same library:

- colors: search by the required semantic intent such as background, text, border, brand, success, warning, or danger;
- spacing: batch-search the exact uncached numeric spacing values used by the current design, prefer semantic `padding/*` variables, and require `GAP` scope;
- effects: search the exact shadow or effect intent and require an effect style.

Do not use equal raw values as identity. Save the selected published key in the working resolution table and final report.
