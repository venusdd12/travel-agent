# 智能旅行助手 🌍✈️

集成高德地图MCP服务,提供个性化的旅行计划生成。

## ✨ 功能特点

- 🤖 **AI驱动的旅行规划**: SimpleAgent,智能生成详细的多日旅程
- 🗺️ **高德地图集成**: 通过MCP协议接入高德地图服务,支持景点搜索、路线规划、天气查询
- 🧠 **智能工具调用**: Agent自动调用高德地图MCP工具,获取实时POI、路线和天气信息
- 🔎 **行程质量审查**: 独立审查Agent对生成结果进行评分,检查节奏、天气覆盖、位置数据和预算一致性
- 🎨 **现代化前端**: Vue3 + TypeScript + Vite,响应式设计,流畅的用户体验
- 📱 **完整功能**: 包含住宿、交通、餐饮和景点游览时间推荐

## 🏗️ 技术栈

### 后端
- **框架**:  基于SimpleAgent
- **API**: FastAPI
- **MCP工具**: amap-mcp-server (高德地图)
- **LLM**: 支持多种LLM提供商(OpenAI, DeepSeek等)

### 前端
- **框架**: Vue 3 + TypeScript
- **构建工具**: Vite
- **UI组件库**: Ant Design Vue
- **地图服务**: 高德地图 JavaScript API
- **HTTP客户端**: Axios

## 📁 项目结构

```
helloagents-trip-planner/
├── backend/                    # 后端服务
│   ├── app/
│   │   ├── agents/            # Agent实现
│   │   │   └── trip_planner_agent.py
│   │   ├── api/               # FastAPI路由
│   │   │   ├── main.py
│   │   │   └── routes/
│   │   │       ├── trip.py
│   │   │       └── map.py
│   │   ├── services/          # 服务层
│   │   │   ├── amap_service.py
│   │   │   └── llm_service.py
│   │   ├── models/            # 数据模型
│   │   │   └── schemas.py
│   │   └── config.py          # 配置管理
│   ├── requirements.txt
│   ├── .env.example
│   └── .gitignore
├── frontend/                   # 前端应用
│   ├── src/
│   │   ├── components/        # Vue组件
│   │   ├── services/          # API服务
│   │   ├── types/             # TypeScript类型
│   │   └── views/             # 页面视图
│   ├── package.json
│   └── vite.config.ts
└── README.md
```

## 🚀 快速开始

### 前提条件

- Python 3.10+
- Node.js 16+
- 高德地图API密钥 (Web服务API和Web端(JS API))
- LLM API密钥 (OpenAI/DeepSeek等)

### 后端安装

1. 进入后端目录
```bash
cd backend
```

2. 创建虚拟环境
```bash
python -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate
```

3. 安装依赖
```bash
pip install -r requirements.txt
```

4. 配置环境变量
```bash
cp .env.example .env
# 编辑.env文件,填入你的API密钥
```

5. 启动后端服务
```bash
uvicorn app.api.main:app --reload --host 0.0.0.0 --port 8000
```

### 前端安装

1. 进入前端目录
```bash
cd frontend
```

2. 安装依赖
```bash
npm install
```

3. 配置环境变量
```bash
# 创建.env文件, 填入高德地图 Web 端 JavaScript API Key 和安全密钥
cp .env.example .env
```

前端地图使用高德地图 JavaScript API 2.0，需要同时配置：

- `VITE_AMAP_WEB_JS_KEY`：高德地图 Web 端 JavaScript API Key
- `VITE_AMAP_WEB_JS_SECURITY_CODE`：高德地图 Web 端 JavaScript API 安全密钥（`securityJsCode`）

后端 MCP 服务使用 `backend/.env` 中的 `AMAP_MAPS_API_KEY`，必须是高德控制台中“Web服务”类型的 Key；前端的 `VITE_AMAP_WEB_JS_KEY` 是“Web端（JS API）”类型，两者不能混用。旧变量 `AMAP_API_KEY` 仅作为兼容回退。前后端密钥均通过环境变量读取，请勿提交真实 `.env` 文件。

4. 启动开发服务器
```bash
npm run dev
```

5. 打开浏览器访问 `http://localhost:5173`

## 📝 使用指南

1. 在首页填写旅行信息:
   - 目的地城市
   - 旅行日期和天数
   - 交通方式偏好
   - 住宿偏好
   - 旅行风格标签

2. 点击"生成旅行计划"按钮

3. 系统将:
   - 调用HelloAgents Agent生成初步计划
   - Agent自动调用高德地图MCP工具搜索景点
   - Agent获取天气信息和路线规划
   - 整合所有信息生成完整行程

4. 查看结果:
   - 每日详细行程
   - 景点信息与地图标记
   - 交通路线规划
   - 天气预报
   - 餐饮推荐

## 🔧 核心实现

### HelloAgents Agent集成

```python
from hello_agents import SimpleAgent, HelloAgentsLLM
from hello_agents.tools import MCPTool

# 创建高德地图MCP工具
amap_tool = MCPTool(
    name="amap",
    server_command=["uvx", "amap-mcp-server"],
    env={"AMAP_MAPS_API_KEY": "your_api_key"},
    auto_expand=True
)

# 创建旅行规划Agent
agent = SimpleAgent(
    name="旅行规划助手",
    llm=HelloAgentsLLM(),
    system_prompt="你是一个专业的旅行规划助手..."
)

# 添加工具
agent.add_tool(amap_tool)
```

### 行程质量审查 Agent

旅行计划生成后,结果页可以运行独立的 `行程质量审查专家`。它接收结构化 `TripPlan`,先使用 LLM 检查行程是否合理,再合并确定性规则检查结果:

- 每日景点数量和预计游览时长是否过满
- 景点地址、经纬度是否缺失
- 天气是否覆盖所有行程日期
- 预算明细和总计是否一致
- 每日餐饮安排是否完整

审查接口为 `POST /api/trip/review`,即使 LLM 或高德服务不可用,也会使用规则审查返回评分和问题清单。这种“LLM 判断 + 规则兜底”的设计可以避免外部依赖异常时影响核心业务。

### MCP工具调用

Agent可以自动调用以下高德地图MCP工具:
- `maps_text_search`: 搜索景点POI
- `maps_weather`: 查询天气
- `maps_direction_walking_by_address`: 步行路线规划
- `maps_direction_driving_by_address`: 驾车路线规划
- `maps_direction_transit_integrated_by_address`: 公共交通路线规划

## 📄 API文档

启动后端服务后,访问 `http://localhost:8000/docs` 查看完整的API文档。

主要端点:
- `POST /api/trip/plan` - 生成旅行计划
- `POST /api/trip/review` - 使用独立Agent审查旅行计划
- `GET /api/map/poi` - 搜索POI
- `GET /api/map/weather` - 查询天气
- `POST /api/map/route` - 规划路线

## 🤝 贡献指南

欢迎提交Pull Request或Issue!

## 📜 开源协议

本项目是在 [Datawhale Hello-Agents](https://github.com/datawhalechina/hello-agents) 基础上进行二次开发的项目。

- 上游项目: **Hello-Agents: Building an AI Agent from Scratch**
- 上游作者: Sizhou Chen、Tao Sun、Shufan Jiang、Peilin Huang、Xinmin Zeng、Hao Hu、Xinzhong Zhu 及 Hello-Agents 贡献者
- 上游许可证: [CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/)
- 本项目新增和修改内容: 多 Agent 旅行规划业务、高德地图 MCP 集成、行程质量审查 Agent、Vue 前端和项目工程化配置

除特别说明外,本项目的新增和修改内容继续按照 CC BY-NC-SA 4.0 发布。使用或再分发本项目时,请保留本声明、上游项目署名和许可证信息,并注明本项目包含修改。

## 🙏 致谢

- [HelloAgents](https://github.com/datawhalechina/Hello-Agents) - 智能体教程
- [HelloAgents框架](https://github.com/jjyaoao/HelloAgents) - 智能体框架
- [高德地图开放平台](https://lbs.amap.com/) - 地图服务
- [amap-mcp-server](https://github.com/sugarforever/amap-mcp-server) - 高德地图MCP服务器

---

**HelloAgents智能旅行助手** - 让旅行计划变得简单而智能 🌈

## 🎯 面试项目说明

这是一个面向真实业务流程的多 Agent 旅行规划系统,不是简单的单轮聊天应用。系统把旅行规划拆分为多个职责清晰的 Agent,并使用结构化数据在 Agent 之间传递结果。

### 核心流程

```text
用户需求
  -> 景点搜索 Agent -> 天气查询 Agent -> 酒店推荐 Agent
  -> 行程规划 Agent -> 行程质量审查 Agent -> 可视化结果
```

### 项目亮点

- **多 Agent 分工**: 景点、天气、酒店、规划和质量审查各自负责独立任务。
- **工具调用**: 通过 MCP 接入高德地图,获取 POI、天气和路线数据。
- **质量控制**: 使用审查 Agent 加规则引擎检查行程节奏、数据完整性、天气覆盖和预算一致性。
- **可靠性设计**: LLM 或地图服务异常时保留规则审查和基础行程兜底,避免整个流程失败。
- **完整产品闭环**: Vue 前端表单、地图展示、行程编辑、预算、天气、图片和导出功能。

### 技术栈

`Vue 3` `TypeScript` `Vite` `FastAPI` `Pydantic` `HelloAgents` `MCP` `高德地图 JS API`

### 安全说明

真实密钥只放在本地 `backend/.env` 和 `frontend/.env`,不会提交到 GitHub。后端使用高德 Web 服务 Key (`AMAP_MAPS_API_KEY`),前端使用高德 JS API Key 和安全密钥,两者不能混用。
