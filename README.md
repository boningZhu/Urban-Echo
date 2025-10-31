# Urban-Echo
太好啦—下面给你一份开发规格书（前端 / 后端要做什么 + 接口/数据/流程），照着做就能把 Urban Echo（MindScape Lite）从雏形跑到能演示的 MVP，再留出升级空间。

⸻

✅ 总览（MVP 目标）
	•	匿名提交“一句话心情”+ 选择区域（NW/NE/SW/SE）
	•	后端做情绪分类（正/中/负），返回“情绪标签 + 鼓励语 + 各区统计 + 着色”
	•	前端把地图着色、显示结果与简单统计

⸻

🧩 前端要做什么（Web）

1) 页面结构（Pages & Sections）
	•	首页 /
	•	顶部栏：Logo/项目名 + 简介一句话
	•	提交表单：区域选择（下拉框）、心情输入（单行文本，≤100 字）、提交按钮
	•	反馈区：显示“情绪分析（正/中/负）+ AI 鼓励语”
	•	地图区：Calgary 地图 + 4 区图层（NW/NE/SW/SE）
	•	统计区（可选）：今天的情绪占比饼图或柱状图
	•	页脚：隐私说明（匿名、无 PII）、Hackathon 标识

2) 组件/模块（建议）
	•	FormPanel：区域选择 + 文本输入 + 提交按钮（含校验与 loading）
	•	MapView：Leaflet/Mapbox 地图，支持更新图层颜色
	•	ResultCard：展示情绪结果与鼓励语
	•	StatsPanel（可选）：Chart.js 饼图/条形图
	•	Toast/Alert（可选）：错误提示

3) 状态管理（最简）
	•	ui.loading: 是否提交中
	•	form.region: “NW” | “NE” | “SW” | “SE”
	•	form.text: 用户输入
	•	result.label: “pos” | “neu” | “neg”
	•	result.reply: AI 鼓励语
	•	colors: {NW:"#xx", NE:"#xx", SW:"#xx", SE:"#xx"}
	•	stats（可选）：{NW:{pos,neu,neg}, ...}

4) 交互流程（前端）
	1.	用户选择区域 + 输入文本 → 点击提交
	2.	POST /submit 发送 { region, text }
	3.	收到响应：{ label, reply, colors, store }
	4.	更新：
	•	ResultCard 显示 label + reply
	•	MapView 用 colors 重新着色各区
	•	StatsPanel 用 store 重绘图表
	5.	清空输入框，光标回到输入框（便于连测）

5) 输入校验 & UX 细节
	•	文本必填（空则阻止提交 + 显示“请输入一句话心情”）
	•	文本长度 ≤ 100，过滤纯空白
	•	提交按钮在 loading 时禁用
	•	失败弹 toast：“网络异常，请稍后重试”
	•	地图初始缩放到 Calgary（z=10），四区有浅色填充
	•	色彩对照：绿色=正向、黄色=中性、红色=负向（可在图例处标注）

6) 可访问性（A11y）与可用性
	•	所有表单元素有 <label for=…>
	•	按钮可键盘触达（Enter 提交）
	•	颜色同时配合图例/文字说明（避免仅靠颜色表达信息）

7) 前端目录建议

/templates/index.html       # 页面骨架
/static/style.css           # 统一配色/布局
/static/script.js           # 取数、地图、事件绑定

已在我给你的 starter 包里生成，可直接按此扩展。

⸻

🧠 后端要做什么（Flask）

1) 路由（API 设计）
	•	GET /
返回 index.html
	•	POST /submit
请求体（JSON）：

{ "region": "NW", "text": "今天有点紧张但也期待" }

响应体（JSON）：

{
  "label": "pos|neu|neg",
  "reply": "鼓励语（中文短句）",
  "store": { "NW": {"pos": 3, "neu": 1, "neg": 2}, ... },
  "colors": { "NW": "#2ecc71", "NE": "#f1c40f", "SW": "#e74c3c", "SE": "#2ecc71" }
}



2) 数据模型（MVP）
	•	内存计数（dict/defaultdict）：

mood_store = {
  "NW": {"pos": 0, "neu": 0, "neg": 0},
  "NE": {"pos": 0, "neu": 0, "neg": 0},
  "SW": {"pos": 0, "neu": 0, "neg": 0},
  "SE": {"pos": 0, "neu": 0, "neg": 0}
}



进阶：换 Firebase/SQLite，增加 timestamp、text_len、lang 字段做统计/趋势图。

3) 业务逻辑
	•	校验 region ∈ {NW, NE, SW, SE}，文本存在
	•	情绪分类（两种模式）
	•	简易规则模式（默认）：正/负关键词计数，决定 pos/neg/neu
	•	GPT 模式（可选）：设置 OPENAI_API_KEY 后，先“分类”再“生成鼓励语”
	•	更新 mood_store[region][label] += 1
	•	颜色计算（简单规则即可）
	•	neg_ratio > 0.5 → 红
	•	pos_ratio > 0.5 → 绿
	•	其他 → 黄
	•	返回 colors 供前端直接着色

4) 安全与隐私
	•	不记录 IP / UA / 原文（MVP 可只保存计数）；如需留原文，务必脱敏并写明用途
	•	后端限制文本长度（≤ 200 字）
	•	CORS：同源场景无需开启，若分离部署再加

5) 配置与环境变量
	•	OPENAI_API_KEY（可选）
	•	PORT（可选）
	•	FLASK_ENV=production（部署时）

6) 日志与错误处理
	•	对 /submit 做 try/except，失败返回：

{ "error": "service_unavailable" }


	•	控制台打印最简日志：时间、region、label

⸻

🔗 前后端对接契约（Contract）

请求

POST /submit
Content-Type: application/json

{ "region": "NE", "text": "Midterm 压力很大" }

响应（示例）

{
  "label": "neg",
  "reply": "辛苦了，先深呼吸一下 💛",
  "store": {
    "NW": {"pos":2,"neu":1,"neg":0},
    "NE": {"pos":0,"neu":1,"neg":3},
    "SW": {"pos":1,"neu":0,"neg":0},
    "SE": {"pos":0,"neu":0,"neg":0}
  },
  "colors": {
    "NW": "#2ecc71",
    "NE": "#e74c3c",
    "SW": "#2ecc71",
    "SE": "#f1c40f"
  }
}

前端只需：
	•	读 label/reply → 显示结果
	•	读 colors → 给四个 Layer setStyle({ fillColor: color, color: color })
	•	（可选）读 store → 重绘图表

⸻

🧪 测试用例（最少集）

正向文本："今天阳光很好，和朋友散步很开心" → 期望 label=pos
负向文本："作业好多，压力大又焦虑" → 期望 label=neg
中性文本："上课了一天，有点累但还行" → 期望 label=neu
空文本：应在前端拦截；后端返回错误或按 neu
非法区域："XX" → 正常落入默认 "NW"

⸻

🚀 部署与演示
	•	本地：python app.py → http://127.0.0.1:5000
	•	云端（可选）：
	•	Render / Railway / Fly.io（免费快速）
	•	或 Vercel（需用 Python Serverless 适配）
	•	演示脚本：
	1.	输入正向文本 → 地图变绿
	2.	输入负向文本 → NE 变红
	3.	展示统计面板（如果做了）
	4.	讲社会意义：早期预警、数字共情、匿名与包容

⸻

🧱 进阶功能（时间允许再上）
	•	真实 GeoJSON 分区（替换 4 个圆）
	•	Chart.js 饼图/折线（今日占比 / 最近 N 次）
	•	i18n（中/英切换）
	•	历史存储（Firebase）+ 时间维度热力图
	•	基于情绪阈值的“社区关怀提醒”弹窗/邮件（演示版即可）

⸻

你们已经有 starter 代码包（Flask + Leaflet + 简易情绪 + 颜色策略）。
直接按上面规格清单扩展即可。如果你愿意，我可以：
	1.	把四个圆换成 Calgary 的 GeoJSON 多边形版本
	2.	加一个 统计饼图 与 图例（颜色说明）
	3.	给你们出一页 Devpost 模板（中英对照）

你想先做GeoJSON 分区还是统计图？我这边现在就给你成品代码片段。

