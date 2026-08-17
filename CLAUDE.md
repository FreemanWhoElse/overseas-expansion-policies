# 国家政策整理指令（以日本为范本）

当用户说「整理 XX 国家的政策」或「按日本标准做 XX 国」时，按本规范执行。目标是让每个国家在 `index.html` 中达到与日本相同的完整度。

---

## 一、先备份现状

修改 `index.html` 前，先备份：
```bash
cp index.html "index.backup-YYYY-MM-DD.html"
```

---

## 二、数据采集

- 用 tavily-search（`mcp__tavily__tavily_search` / `tavily_extract`）搜索
- 优先官方来源（该国投资局、劳工部、税务局、移民局、法律库官网）
- 数据时效以 2026 年为最新，关键数字标注年份/生效日
- 多个独立维度可用子 Agent 并行采集，主对话只收结果

---

## 三、内容维度（每个国家都要补 4 个决策刚需维度）

新增到 `DATA[key].categories`，插在 `contract` 之前：

| key | 标题 | 内容要点 |
|---|---|---|
| `marketentry` | 市场准入与外资审查 | 外资准入限制/负面清单、股比上限、须审批行业、敏感行业审查 |
| `companysetup` | 公司设立 | 公司类型、最低资本、注册流程与耗时、外资100%持股条件 |
| `workvisa` | 签证与在留资格 | 工作签证/投资签证/高管签证、配额、门槛金额、永居路径 |
| `consumptiontax` | 税收全景 | 增值税/消费税标准税率、预提税、关税、双边税收协定 |

> 新加坡已有 `workpass` 维度，不再重复加 `workvisa`。

**每条内容格式**（3 条/维度）：
```js
{text:"<中文 80-150 字，具体到数字/比例/法条号>", source:"官方URL", sourceLabel:"机构 · 内容"}
```

---

## 四、用工成本测算器

每个国家加一个测算函数，复用日本 `jp-calc-*` 的 CSS 类。

**结构**：`Xx_CALC_CITIES` 常量 + `Xx_CALC` 费率常量 + `xxCalcCompute/Render/Update/Breakdown/Compare/Init` 函数。

**必含**：
- 3 个中国企业出海主要城市 tabs（城邦国家如新加坡改用「本地/外籍」切换）
- 输入薪资 + 成本明细 + 横向对比 + 最低工资提示 + 缴费上限封顶提示
- 免责声明（费率依据、年份、来源）

**诚实标注城市差异**（重要）：
- 费率全国统一的国家：对比卡片如实标注「三城成本相同」，差异只在最低工资/生活成本
- 真有城市/州差异的国家（美/墨/印/日/阿联酋/沙特/泰）：突出州工薪税、最低工资、外籍劳工税、医保等真实变量
- 禁止硬造差异误导企业

**挂载**：在 `goDetail` 的 `CALC_SECTION` / `CALC_RENDER` / `CALC_INIT` 三个映射表里各加一行。

**新国家必须同步改 `REGIONS` 数组**（`const REGIONS` 在 DATA 上方）：把新国家的 key 加进对应分组的 `countries` 列表，否则主页不显示该国家。同时新增的特色维度 key（如 `qatarization`）要加进 `CATEGORY_ORDER` 数组。

---

## 五、验证

改完后必须：
```bash
node -e "const fs=require('fs');const s=fs.readFileSync('index.html','utf8');const m=s.match(/<script>([\s\S]*?)<\/script>/g);let js=m.map(x=>x.replace(/<\/?script>/g,'')).join('\n');try{new Function(js);console.log('JS OK');}catch(e){console.log('JS ERR:',e.message);}"
```
再 `open index.html` 打开预览。

---

## 六、关键约定

- 中文内容，全角引号，JS 内嵌套引号用 `\"` 转义
- `sourceLabel` 格式：`机构名 · 内容`
- **每条来源必须标明官方/非官方，且优先找官方链接**：
  - **优先级铁律**：先搜官方链接（该国政府部委官网、法律库如 lawphil/legislation.gov.uk、官方投资局/税务局/移民局、官方公报/Official Gazette、国际组织如 ILO/NATLEX/UNCTAD）；只有确认官方无对应来源（如流程性/综述性内容、法律未强制规定项）时，才降级用非官方
  - 官方来源 → `机构名（官网）· 内容`
  - 第三方专业机构（PwC/KPMG/EY/Deloitte、律所、Deel/Skuad/Emerhub/Truein/Boundless 等汇总站）→ `机构名（非官方）· 内容`（如内容援引法条，补 `，援引<法条>口径`）
  - 内容来自非官方二手转述但需明确非官方性质 → `参考资料（非官方，援引<法条/政策>口径）`
  - 无法确认官方性时，宁可标"非官方"也不留空
- 费率/最低工资/封顶都要 2026 最新值，附来源
- 完成后主动提醒上下文用量，超 80% 建议 /clear
