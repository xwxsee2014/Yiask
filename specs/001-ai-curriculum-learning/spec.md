# Feature Specification: Yiask AI智能课程练习平台

**Feature Branch**: `001-ai-curriculum-learning`
**Created**: 2025-11-01
**Status**: Draft
**Input**: 用户描述：创建基于AI的课程同步学习平台，集成内容生成、语音朗读和图文结合问题功能

## Clarifications

### Session 2025-11-03

- Q: What is the target scale for this platform in terms of concurrent users and data volume? → A: Small scale (<100 concurrent users)
- Q: Which AI services do you prefer to use for content generation and text-to-speech? → A: Dify平台代理（调用硅基流动提供的DeepSeek/Qwen3）+ Xinference部署的CosyVoice2-0.5b和Whisper模型
- Q: How should the system provide images for the "image+text" exercises? → A: 让用户自行上传图片
- Q: What is your preferred tech stack for this platform? → A: Python/FastAPI后端 + React前端 + PostgreSQL数据库 + Docker部署
- Q: What are your security and privacy requirements for user data? → A: 简单JWT认证 + HTTPS传输 + 对密码字段进行加密
- Q: What user roles should the system distinguish? → A: 学习者 和 内容提供者
- Q: Should learners be able to input answers via speech? → A: 是的，学习者需要通过语音输入进行练习
- Q: Should exercise content be generated offline or in real-time? → A: 生成的练习内容应该离线生成，需要有内容提供者进行审核；需要实时处理的是学习者的语音输入部分
- Q: Should the system evaluate learner answers in real-time and provide recommended answers? → A: 是的，系统需要对学习者回答进行实时质量评价并提供推荐回答，推荐回答在离线生成时提供并由内容提供者审核；评价和推荐回答均需要语音朗读
- Q: What level of detail should the data model include? → A: 完全跳过数据模型定义，所有结构在实施时确定
- Q: What API design and system architecture approach should be used? → A: RESTful API
- Q: What are the overall system performance benchmarks? → A: 普通API响应时间<500ms (p95)，AI生成接口首token延迟<1min，支持100并发用户
- Q: What deployment architecture strategy should be used? → A: 多容器编排部署方案
- Q: What monitoring and logging strategy should be implemented? → A: 结构化JSON日志 + 基础指标监控

### Session 2025-11-04

- Q: What Python package management and virtual environment tool should be used for backend development? → A: uv - Use uv for virtual environment isolation and package dependency management
- Q: What dependency specification format should be used with uv? → A: pyproject.toml - Use pyproject.toml instead of requirements.txt for all Python dependencies
- Q: Which orchestration approach do you prefer for multi-container deployment? → A: Docker Compose (docker-compose.yaml) - Simple, YAML-based orchestration perfect for this scale
- Q: How should the container services be structured for the multi-container setup? → A: Separate containers: Backend (FastAPI), Frontend (React), Database (PostgreSQL), with AI services (Dify, Xinference) deployed independently
- Q: How should environment variables and configuration be managed for the multi-container setup? → A: Root .env file + service-specific env files (.env.backend, .env.frontend) for configuration
- Q: How should the PostgreSQL database container be configured? → A: Named volume for data persistence + init scripts (.sql/.sh) for database schema creation
- Q: How should service dependencies and build strategy be configured? → A: Local Docker builds (./backend/Dockerfile, ./frontend/Dockerfile) + depends_on with health checks for startup order
- Q: If users upload PDFs, what component should be used for content recognition and image extraction? → A: MinerU - for PDF content recognition including OCR and image extraction
- Q: How should offline content generation be implemented since the review process is simple? → A: FastAPI background tasks + database status tracking (simple polling for progress, no queue system)
- Q: Should AI text generation and evaluation services use Dify agents or direct API calls? → A: Use Dify as LLMOps platform to create agents for calling (not direct model API calls)
- Q: Alembic 配置文件应该如何设置和管理？ → A: 集成在 backend/src/ 目录下（backend/src/alembic.ini）
- Q: 数据库迁移应该在何时自动触发运行？ → A: 在 .env.backend 中添加一个 ENABLE_MIGRATION 参数进行触发
- Q: 初始数据库设置如何处理？是否需要保留 init 脚本？ → A: 完全使用 Alembic，包括初始数据库创建
- Q: 迁移失败时如何处理和回滚？ → A: 自动回滚 + 日志记录
- Q: Alembic 的数据库连接配置应该如何管理？ → A: DATABASE_URL

## User Scenarios & Testing

### User Roles

#### 学习者 (Learner)
- 使用AI生成的练习内容进行学习和练习（含语音朗读）
- 通过语音输入回答问题并获得实时质量评价
- 查看推荐回答（支持文本和语音朗读）
- 查看练习进度和结果

#### 内容提供者 (Content Provider)
- 创建和管理课程内容
- 上传课程材料（PDF、文本、图片）
- 上传与练习相关的场景图片
- 触发离线生成练习内容
- 审核系统生成的练习内容和推荐回答（通过/修改/驳回）
- 设置课程难度和主题
- 管理练习生成的参数

### User Story 1 - 课程同步内容生成与练习 (Priority: P1)

作为英语学习者，我希望系统能够基于我当前学习的正规课程内容，经过审核后的相关问答和对话练习，以便我在课后进行针对性复习和练习。

**Why this priority**: 这是平台的核心价值主张 - 提供与正规课程紧密结合的高质量练习内容。没有这个功能，整个平台就失去了意义。

**Independent Test**: 可以通过以下方式独立测试：内容提供者上传课程材料 → 触发离线生成 → 审核通过后发布 → 学习者选择课程单元并完成练习 → 验证内容与课程的相关性和难度匹配度。整个流程可以独立运行并交付价值。

**Acceptance Scenarios**:

1. **Given** 内容提供者创建了"基础英语对话"课程单元并触发生成，**When** 内容提供者审核通过生成的练习，**Then** 学习者可以选择该课程单元并获得5-10个与主题相关的问答和对话练习
2. **Given** 内容提供者创建了"商务英语写作"课程单元并触发生成，**When** 内容提供者审核通过生成的练习，**Then** 学习者可以获得包含商务场景的对话和书面表达练习
3. **Given** 内容提供者创建了"托福听力训练"课程单元并触发生成，**When** 内容提供者审核通过生成的练习，**Then** 学习者可以获得听力理解相关的问答，涵盖该单元的词汇和语法点
4. **Given** 内容提供者创建了"雅思口语"课程单元并触发生成，**When** 内容提供者审核通过生成的练习，**Then** 学习者可以获得口语对话场景问题和回答示例

---

### User Story 2 - AI语音朗读功能 (Priority: P2)

作为英语学习者，我希望每个生成的练习题目都配备清晰的AI语音朗读，帮助我熟悉英语的语音语调，提高听力和口语能力。

**Why this priority**: 语音朗读是英语学习的关键环节，特别是对于听力和口语技能的培养。这个功能将文本学习转化为多感官学习体验，显著提升学习效果。

**Independent Test**: 可以通过以下方式独立测试：生成包含问题的练习 → 点击语音播放按钮 → 验证语音清晰度、发音准确性和语调自然度。功能可独立于图片功能运行。

**Acceptance Scenarios**:

1. **Given** 练习中包含问题"What is your favorite hobby?"，**When** 学习者点击语音播放按钮，**Then** 系统播放清晰、自然的英语语音朗读
2. **Given** 练习中包含较长的对话文本，**When** 学习者请求语音朗读，**Then** 系统能够流畅地朗读整个对话，语调自然有感情
3. **Given** 学习者对某个词汇的发音不确定，**When** 学习者选择该词汇并点击朗读，**Then** 系统单独朗读该词汇，发音清晰标准
4. **Given** 学习者正在移动设备上使用，**When** 学习者点击语音播放，**Then** 语音可以通过设备扬声器或耳机正常播放

---

### User Story 2.2 - 语音输入练习功能 (Priority: P2)

作为英语学习者，我希望能够通过语音直接回答练习问题，并获得实时的质量评价和推荐回答，这样我可以改进我的表达并学习更好的说法。

**Why this priority**: 语音输入是语言学习的核心技能，实时反馈能帮助学习者立即纠正错误并学习更地道的表达。相比键盘输入，语音输入配合实时评价更贴近真实交流场景，能有效提升学习者 confidence 和 fluency。

**Independent Test**: 可以通过以下方式独立测试：系统展示练习问题 → 学习者通过语音回答 → 系统识别并转换文字 → 系统评价回答质量 → 提供推荐回答和评价反馈 → 验证功能完整性。功能可独立于图片功能运行。

**Acceptance Scenarios**:

1. **Given** 练习问题为"What is your favorite hobby?"，**When** 学习者通过语音回答"I enjoy reading books"，**Then** 系统能够准确识别并转换为文字，并给出质量评价"很好，语法正确，表达自然"
2. **Given** 学习者发音不够清晰，**When** 学习者多次尝试语音输入，**Then** 系统能够容忍一定程度的发音偏差并进行准确识别，并给出改进建议
3. **Given** 学习者在嘈杂环境中，**When** 学习者进行语音输入，**Then** 系统能够过滤噪音并准确识别语音，并提供清晰的质量评价
4. **Given** 学习者回答质量不高，**When** 系统进行评价后，**Then** 系统能够提供推荐回答并通过语音朗读推荐回答和评价内容

---

### User Story 3 - 图文结合场景练习 (Priority: P3)

作为英语学习者，我希望针对"看图回答"等视觉题型，系统能够自动匹配或生成相关的场景图片，帮助我在真实语境中理解和使用英语。

**Why this priority**: 图文结合能够提供丰富的语境信息，帮助学习者更好地理解和记忆英语表达。这是提升英语实际应用能力的重要方式。

**Independent Test**: 可以通过以下方式独立测试：选择图文题型 → 系统展示场景图片 → 学习者根据图片内容回答问题 → 验证图片与问题的匹配度和相关性。

**Acceptance Scenarios**:

1. **Given** 练习类型为"看图回答"，**When** 系统加载练习内容，**Then** 自动展示与问题内容相关的场景图片
2. **Given** 图片问题场景为"餐厅点餐"，**When** 学习者看到图片后回答问题，**Then** 图片内容清晰显示餐厅环境、菜单和顾客，服务于问题理解
3. **Given** 图片问题场景为"机场问路"，**When** 学习者查看图片，**Then** 图片准确展示机场大厅、指示牌等场景元素
4. **Given** 学习者完成图文练习后，**When** 学习者查看答案和解析，**Then** 解析内容与图片场景紧密结合，提供有针对性的学习指导

---

### Edge Cases

- 当课程内容过少或过于简单时，系统如何调整生成内容的难度？
- 当AI语音服务暂时不可用时，如何保证学习者仍能完成练习？
- 当无法找到合适的场景图片时，系统如何处理？
- 当生成的练习内容与课程主题关联度较低时，如何识别和纠正？
- 学习者在离线状态下如何使用已下载的内容和语音？

## Requirements

### Functional Requirements

#### 系统核心功能

- **FR-001**: 学习者 MUST 能够浏览和选择系统中已发布的课程单元
- **FR-002**: 内容提供者 MUST 能够触发离线生成练习内容，系统生成后需要内容提供者审核通过才能发布给学习者
- **FR-003**: 生成的练习内容 MUST 与课程主题保持一致，难度 MUST 与课程水平匹配
- **FR-004**: 系统 MUST 为每个练习问题配备AI语音朗读功能
- **FR-005**: 语音朗读 MUST 发音清晰、语调自然，语速适合学习者水平
- **FR-006**: 系统 MUST 支持"看图回答"等图文结合题型
- **FR-007**: 内容提供者 MUST 能够为图文问题上传相关的场景图片
- **FR-008**: 学习者 MUST 能够通过语音输入答案（语音转文字）
- **FR-009**: 系统 MUST 能够对学习者回答进行实时质量评价（准确性、语法、流利度等维度）
- **FR-010**: 系统 MUST 能够为质量不高的回答提供推荐回答（推荐回答在离线生成时提供）
- **FR-011**: 学习者 MUST 能够重播语音内容（包括评价和推荐回答）
- **FR-012**: 系统 MUST 记录学习者的练习进度和结果
- **FR-013**: 系统 MUST 提供练习答案和解析，帮助学习者理解错误

#### 内容管理功能

- **FR-014**: 内容提供者 MUST 能够创建和管理课程单元（包含主题、词汇表、语法点、难度等级）
- **FR-015**: 内容提供者 MUST 能够上传课程材料（PDF文件、文本内容、图片）
- **FR-016**: 内容提供者 MUST 能够触发离线生成练习内容并审核（通过/修改/驳回）
- **FR-017**: 内容提供者 MUST 能够审核推荐回答（通过/修改/驳回）
- **FR-018**: 内容提供者 MUST 能够设置练习生成参数（题目数量、题型偏好、难度范围）
- **FR-019**: 内容提供者 MUST 能够管理自己创建的课程内容（编辑、删除、发布/下线）
- **FR-020**: 系统 MUST 支持两种课程内容输入方式：1）上传PDF文件并使用MinerU组件自动解析提取课程内容（包含OCR文字识别能力和图片提取）2）手动录入课程内容（包括文本和图片）

#### 系统管理功能

- **FR-021**: 系统 MUST 根据内容提供者提供的材料和设置的参数生成相应的练习内容
- **FR-022**: 系统 MUST 实现基于角色的权限控制，确保学习者和内容提供者只能访问授权的功能

### Key Entities

- **课程单元**: 包含课程主题、词汇表、语法点、难度等级等信息的教学单元
- **练习题目**: 由系统生成的问答或对话内容，包含问题文本、标准答案、推荐回答和相关解析
- **语音文件**: AI生成的题目朗读音频文件，支持重复播放
- **语音输入记录**: 学习者通过语音输入的答案记录，包含原始音频和转录文本
- **实时评价记录**: 系统对学习者回答的实时质量评价结果，包含评分、评价维度（准确性、语法、流利度）和建议
- **场景图片**: 与图文题型相关的图片内容，用于提供视觉语境
- **内容审核记录**: 内容提供者对生成练习内容的审核结果（通过/修改/驳回）和意见
- **学习记录**: 学习者完成练习的历史记录，包含答题情况、用时和错误分析
- **学习者档案**: 包含学习者当前课程、水平和个人偏好的用户信息
- **内容提供者档案**: 包含创建者信息、课程管理权限和内容管理功能的用户信息
- **课程内容库**: 由内容提供者创建和管理的一组课程材料和元数据

## Success Criteria

### Measurable Outcomes

- **SC-001**: 学习者能够在2分钟内获得基于当前课程的个性化练习内容
- **SC-002**: 生成的练习内容与课程主题的相关性达到90%以上
- **SC-003**: 95%的练习题目能够成功配备清晰的AI语音朗读
- **SC-004**: 80%的图文问题能够由内容提供者成功上传与场景高度相关的图片
- **SC-005**: 学习者完成一轮练习（10个问题）的平均时间不超过15分钟
- **SC-006**: 学习者对语音朗读清晰度和自然度的满意度评分达到4.0/5.0以上
- **SC-007**: 系统能够根据课程难度自动调整生成内容的复杂度
- **SC-008**: 90%以上的语音输入能够被准确识别和转换为文字
- **SC-009**: 系统能够在3秒内完成学习者回答的实时质量评价
- **SC-010**: 推荐回答的准确性和实用性评分达到4.0/5.0以上
- **SC-011**: 内容提供者能够在24小时内完成练习内容和推荐回答的审核
- **SC-012**: 学习者通过练习后，对相关课程内容的掌握率提升30%以上
- **SC-013**: 95%的学习者能够在首次使用后成功完成至少一轮完整练习

## Assumptions

- 学习者已拥有正式的英语课程教材或课程资源
- 学习者具备基本的设备访问权限（电脑或移动设备）
- 系统有稳定的网络连接以支持AI内容生成和语音合成
- 学习者愿意使用AI生成的练习内容进行学习
- 课程内容以数字化形式提供（文本、音频）
- 目标学习者群体为中小学到成人英语学习者
- 系统主要支持英语语言学习

## Technical Stack

- **后端**: Python/FastAPI
- **前端**: React
- **数据库**: PostgreSQL
- **部署**: Docker

## Security & Privacy

- **认证**: 简单JWT认证
- **传输安全**: HTTPS传输
- **数据保护**: 对密码字段进行加密存储
- **角色权限控制**: 基于角色的访问控制（RBAC）区分学习者和内容提供者
  - 学习者权限：查看和完成练习、语音输入和接收评价、查看推荐回答、查看个人学习记录
  - 内容提供者权限：创建和管理课程内容、上传课程材料、触发生成练习、审核练习内容和推荐回答、上传练习场景图片、设置课程参数
- **用户认证和个人档案管理系统** - 需要实现基础的用户认证机制

## Dependencies

- 需要PDF内容解析和OCR处理服务以支持课程材料上传 - 使用MinerU组件进行PDF内容识别和图片提取
- 需要Dify作为LLMOps平台以创建AI文本生成代理 - 用于练习内容生成（调用硅基流动提供的DeepSeek/Qwen3）
- 需要FastAPI后台任务机制以支持离线练习内容生成和简单审核流程 - 通过background tasks + 数据库状态跟踪实现
- 需要Dify作为LLMOps平台以创建实时文本分析代理 - 用于学习者回答质量评价（调用硅基流动提供的DeepSeek/Qwen3）
- 需要高质量的AI语音合成服务以生成朗读音频 - 使用Xinference部署的CosyVoice2-0.5b模型
- 需要语音识别服务以支持学习者语音输入 - 使用Xinference部署的Whisper模型
- 需要图片上传和存储服务（内容提供者上传练习场景图片）
- 需要课程内容数据库或课程资源管理系统 - PostgreSQL
- 需要用户认证和个人档案管理系统
- 需要练习进度跟踪和数据分析功能
