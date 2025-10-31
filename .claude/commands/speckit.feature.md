# 生成新增特性的说明文档


## 任务要求

结合现有工程的代码和文档，生成新增特性的说明文档。

请确保说明文档包含以下内容：
1. 功能特性名称、功能英文标题
2. 功能描述
3. 特性相关的代码文件（如果有），从 `.qoder/repowiki/zh/` 目录下查找
4. 特性相关的文档文件（如果有），从 `.qoder/repowiki/zh/` 目录下查找
5. 相关的UI设计图、figma链接（如果有提供）

## 文档示例

`````markdown
# 实现题目解析页面

功能特性名称：题目解析页
功能英文标题：quiz-analyze-page

## 功能概述

实现一个页面，入参包括：

- resId: 题目资源ID，例如：4e61e5e0-0597-4713-b451-08dd9a6c6d06
- containerId: 内容库ID，例如：2373f050-d82b-4656-a3bf-4a8fc1a16d78

通过入参可以获取题目解析的数据来源：`https://bdcs-file-1.ykt.cbern.com.cn/zxx/api_static/questions/{containerId}_{resId}/data.json`
响应数据示例：`../data/quiz_analyze_data.json`

## 数据结构说明

API响应数据的主要结构如下：

### 根对象
- `id`: 题目资源ID (String)
- `container_id`: 内容库ID (String) 
- `type`: 题目类型码 (Int) - 例如：206表示问答题
- `qt_info`: 题型信息对象
- `content`: 题目内容对象

### 题型信息 (qt_info)
```json
{
  "type": 206,
  "name": "问答题",
  "code": "ESSAY_QUESTION",
  "is_objective": false
}
```

### 题目内容 (content)
- `description`: 题干HTML内容 (String)
- `title`: 题目标题 (String)
- `responses`: 参考答案数组
- `answer_steps`: 解析步骤数组 (可选)

### 参考答案结构 (responses[])
```json
{
  "identifier": "RESPONSE_1-1",
  "corrects": ["<div>参考答案HTML内容</div>"]
}
```

### 解析步骤结构 (answer_steps[])
```json
{
  "step_order": "1",
  "hint": "步骤提示HTML",
  "content": "步骤详细内容HTML",
  "hint_explanation": "步骤说明HTML"
}
```

### 数据获取策略
- **题型**: 从 `qt_info.name` 获取，如果为空则显示"未分类题型"
- **题干**: 从 `content.description` 获取，支持HTML富文本渲染
- **参考答案**: 从 `content.responses[0].corrects` 获取，支持多个答案
- **步骤解析**: 从 `content.answer_steps` 按 `step_order` 排序展示

### 容错处理
- 缺少题型时显示默认值"未分类题型"
- 缺少参考答案时显示"暂无参考答案" 
- 缺少解析步骤时显示"暂无解析"
- HTML内容需要安全渲染，防止XSS攻击

## 页面结构

页面包含：

1. 题型+题目标题：注意题型显示为标签与标题内容混排显示
2. 题干
3. 步骤解析：
    - 步骤1
        - 步骤详述
        - 解题过程

    - 步骤2
    - ...

### 设计稿(figgma)

页面实现时参考figma设计，使用 figma mcp 获取数据并分析设计元素后实现：

- 题型+题目： https://www.figma.com/design/vwGUuK62rc4M0hdj5gNduq/-%E5%A4%9A%E7%AB%AF--%E6%99%BA%E8%83%BD%E5%AE%A2%E6%9C%8D%E5%8A%A9%E6%89%8B-UE--2.0%E7%89%88%E6%9C%AC--Copy-?node-id=7561-43198&t=nIKG9r2I70WniXue-4
- 步骤解析：  https://www.figma.com/design/vwGUuK62rc4M0hdj5gNduq/-%E5%A4%9A%E7%AB%AF--%E6%99%BA%E8%83%BD%E5%AE%A2%E6%9C%8D%E5%8A%A9%E6%89%8B-UE--2.0%E7%89%88%E6%9C%AC--Copy-?node-id=7561-43550&t=nIKG9r2I70WniXue-4

### 业务规则

#### 步骤显示规则

1. 步骤x：后面跟着解题思路，进入点击即显示第一个步骤的解题思路
2. 底部显示步骤X对应的视频时间点
3. 右侧显示详解引导，单击播放快速翻转动效，动效效果根据UI效果确认；翻转后显示步骤详述及解题过程
4. 步骤详述：标签默认显示，内容假的加载中跑马灯效果出现
5. 解题过程：默认不显示，隐藏，用户单击跑马灯显示
6. 默认仅显示第一步，剩余步骤点击一次出一个
7. 最后一步完是显示答案按钮，单击显示
8. 查看完整解析则一次性将所有内容平铺展开显示（注意此时如果高度很高，纵向滚动，希望步骤解析的视频区域始终置顶在屏幕范围内，直到滚出步骤解析）

鼓励提示语：
1. 继续解题前跟随鼓励提示语
2. 鼓励随机显示如下：
    - 你太棒了！全程独立思考呢~
    - 再接再厉！
    - 每推进一步，答案就更近一分。
    - 离答案越来越近了，想想下一步思路吧！
    - 一步一步来不着急。
    - 把小问题解决，答案自然就出来了。
`````
