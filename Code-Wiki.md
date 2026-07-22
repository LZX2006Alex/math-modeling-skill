# Code Wiki — Math Modeling Skill

> 本文档是 [math-modeling-skill](https://github.com/XiaoMaColtAI/math-modeling-skill) 的结构化代码百科，覆盖项目整体架构、模块职责、关键类/函数、依赖关系与运行方式。
> 版本：`1.0.0`（对应仓库 VERSION 文件）

---

## 1. 项目概述

**Math Modeling Skill** 是一套面向数学建模竞赛（CUMCM、MCM/ICM、APMCM、MathorCup 等）与建模项目的 **Agent Skill 工作流系统**。它将完整建模任务拆分为 **建模分析 → 代码实现 → 论文撰写** 三阶段，支持三角色（建模手、编程手、论文手）顺序执行或独立运行。

核心设计理念：
- **渐进式加载**：按当前阶段只读取必要的角色规范、算法资料和工具说明，降低单次上下文负担。
- **路径隔离**：`SKILL_ROOT`（只读）与 `PROJECT_ROOT`（只写）严格分离，禁止覆盖 Skill 自身文件。
- **可追溯与可复现**：记录随机种子、输入文件 SHA-256、运行时依赖版本和唯一复现命令。
- **双引擎论文搜索**：OpenAlex + AnySearch 并行检索，按 DOI/题名交叉核验。
- **模板驱动论文生成**：优先继承官方模板的页面与样式，再填充正文、OMML 公式、三线表和参考文献。

---

## 2. 整体架构

```
┌─────────────────────────────────────────────────────────────────────┐
│                         Math Modeling Skill                         │
│                           (SKILL_ROOT, 只读)                         │
├─────────────────────────────────────────────────────────────────────┤
│  根入口层  │  SKILL.md (总路由)  ·  README.md (项目说明)  ·  CHANGELOG.md │
├─────────────────────────────────────────────────────────────────────┤
│  角色层   │  references/roles/                                           │
│           │    ├── 建模手/  (题目分析、模型设计、算法选择)                   │
│           │    ├── 编程手/  (Python/MATLAB 实现、可视化、复现清单)          │
│           │    └── 论文手/  (Word 论文生成、结构校验、渲染抽检)              │
├─────────────────────────────────────────────────────────────────────┤
│  算法资料层 │  assets/  (7 大类 60+ 算法说明文档)                          │
│           │    └── 算法索引.md (按问题特征路由到对应算法文件)                │
├─────────────────────────────────────────────────────────────────────┤
│  工具层   │  tools/                                                      │
│           │    ├── docx/   (Word 生成、OMML 公式、OOXML 校验)              │
│           │    ├── xlsx/   (Excel 读取、公式重算、错误检查)                │
│           │    ├── pdf/    (PDF 文本/表格/图片提取)                       │
│           │    └── paper_search/ (OpenAlex + AnySearch 双引擎搜索)       │
├─────────────────────────────────────────────────────────────────────┤
│  测试层   │  tests/  (回归测试：公式、DOCX、Excel、论文结构、复现清单等)      │
└─────────────────────────────────────────────────────────────────────┘
                              ↓
                    PROJECT_ROOT (用户项目目录，只写)
```

### 2.1 三阶段工作流与数据流

```
阶段① 建模手
    输入：竞赛题目 + 附件
    产物：题目分析报告.md、术语表格.md
           ↓
阶段② 编程手
    输入：题目分析报告.md + 术语表格.md + 附件
    产物：.py/.m 代码、results/*.csv、figures/*.{svg,png}、results/复现清单.json
           ↓
阶段③ 论文手
    输入：全部上游产物 + 搜索结果
    产物：完整论文.docx
```

阶段反馈闭环：
- 编程手发现模型不可实现 → 携带证据返回建模手修正。
- 论文手发现缺少真实结果/图表/文献 → 返回对应阶段补齐。
- 修正后从被阻断阶段继续，不重复已通过阶段。

---

## 3. 目录结构详解

```text
math-modeling-skill/
├── VERSION                         # 当前版本唯一准据 (1.0.0)
├── SKILL.md                        # 根入口：总路由、根目录契约、完成判定
├── README.md                       # 项目说明、快速开始、安装方式、验证命令
├── CHANGELOG.md                    # 版本历史与变更说明
├── .gitignore
│
├── assets/                         # 算法资料库（7 大类）
│   ├── 01-优化算法说明.md
│   ├── 02-预测类算法说明.md
│   ├── 03-评价类算法说明.md
│   ├── 04-图论与网络分析算法说明.md
│   ├── 05-统计分析与数据处理算法说明.md
│   ├── 06-综合类算法说明.md
│   ├── 07-机器学习算法说明.md
│   └── README.md
│
├── imgs/                           # README 展示用示例图
│
├── references/
│   ├── README.md                   # 渐进式加载导航
│   ├── 算法索引.md                  # 按问题特征路由到 assets/ 下具体算法文件
│   └── roles/                      # 三角色子 Skill
│       ├── 建模手/
│       │   ├── SKILL.md            # 角色入口：固定交付物、执行顺序、选择原则
│       │   └── references/
│       │       ├── 前置合同.md
│       │       ├── 工作流程.md
│       │       ├── 常见模式.md
│       │       ├── 建模设计理论.md
│       │       └── 质检清单.md
│       ├── 编程手/
│       │   ├── SKILL.md            # 角色入口：输入/产物、执行顺序、动态依赖检查
│       │   ├── references/
│       │   │   ├── MATLAB规范.md
│       │   │   ├── 可视化规范.md
│       │   │   ├── 可视化面板模板.html
│       │   │   ├── 工作流程.md
│       │   │   ├── 常见模式.md
│       │   │   └── 质检清单.md
│       │   └── scripts/
│       │       ├── check_env.py         # Python 动态依赖检查
│       │       ├── check_matlab_env.m   # MATLAB 动态依赖检查
│       │       └── repro_manifest.py    # 生成复现清单 JSON
│       └── 论文手/
│           ├── SKILL.md            # 角色入口：唯一交付物、官方规则优先级
│           └── references/
│               ├── 写作规范.md
│               ├── 工作流程.md
│               ├── 章节模板.md
│               ├── 自审框架.md
│               ├── 英文化工作流.md
│               ├── 论文格式规范.md
│               └── 论文模板.docx
│
├── tools/                          # 专业子 Skill（工具层）
│   ├── docx/                       # Word 文档工具
│   │   ├── SKILL.md
│   │   ├── LICENSE.txt
│   │   └── scripts/
│   │       ├── paper_format.py          # 论文格式构建主模块
│   │       ├── equations.py             # LaTeX → OMML 公式转换
│   │       ├── self_check.py            # DOCX 工具一键自检
│   │       ├── check_env.py             # DOCX 环境检查
│   │       ├── comment.py               # 批注增删
│   │       ├── accept_changes.py        # 接受修订
│   │       ├── inspect_template_format.py
│   │       ├── office/                  # OOXML 基础工具（DOCX/XLSX 共用）
│   │       │   ├── unpack.py
│   │       │   ├── pack.py
│   │       │   ├── validate.py
│   │       │   ├── soffice.py
│   │       │   ├── helpers/
│   │       │   │   ├── merge_runs.py
│   │       │   │   └── simplify_redlines.py
│   │       │   ├── validators/
│   │       │   │   ├── __init__.py
│   │       │   │   ├── base.py
│   │       │   │   ├── docx.py
│   │       │   │   ├── pptx.py
│   │       │   │   └── redlining.py
│   │       │   └── schemas/             # XSD 校验模式库
│   │       │       ├── ISO-IEC29500-4_2016/
│   │       │       ├── ecma/
│   │       │       ├── mce/
│   │       │       └── microsoft/
│   │       └── templates/               # XML 模板片段
│   │           ├── comments.xml
│   │           ├── commentsExtended.xml
│   │           └── ...
│   ├── xlsx/                       # Excel 工具
│   │   ├── SKILL.md
│   │   ├── LICENSE.txt
│   │   └── scripts/
│   │       ├── read_rows.py             # 无隐式表头推断读取
│   │       └── recalc.py                # LibreOffice 隔离重算 + 错误检查
│   ├── pdf/                        # PDF 工具
│   │   ├── SKILL.md
│   │   ├── LICENSE.txt
│   │   ├── forms.md
│   │   ├── reference.md
│   │   └── scripts/
│   │       ├── check_bounding_boxes.py
│   │       ├── check_fillable_fields.py
│   │       ├── convert_pdf_to_images.py
│   │       ├── create_validation_image.py
│   │       ├── extract_form_field_info.py
│   │       ├── extract_form_structure.py
│   │       ├── fill_fillable_fields.py
│   │       ├── fill_pdf_form_with_annotations.py
│   │       └── ...
│   └── paper_search/               # 双引擎论文搜索
│       ├── SKILL.md
│       └── scripts/
│           ├── hybrid_scholar.py        # OpenAlex + AnySearch 并行融合
│           ├── openalex_scholar.py      # OpenAlex 单引擎封装
│           └── anysearch_academic.py    # AnySearch 单引擎封装
│
└── tests/                          # 回归测试
    ├── test_docx_tools.py
    ├── test_equations.py
    ├── test_paper_format.py
    ├── test_paper_search.py
    ├── test_recalc.py
    ├── test_reproducibility.py
    ├── test_xlsx_read.py
    └── ...
```

---

## 4. 主要模块职责

### 4.1 根入口层 (`SKILL.md`, `README.md`)

| 文件 | 职责 |
|------|------|
| `SKILL.md` | 项目总路由：定义根目录契约（`SKILL_ROOT` 只读 / `PROJECT_ROOT` 只写）、三阶段路由表、固定交付物清单、阶段反馈闭环规则、建模约束、渐进式加载表、完成判定标准。 |
| `README.md` | 面向用户的项目说明：简介、核心能力、快速开始、安装方式（Git/npx/ZIP）、使用示例、工作目录约定、集成工具说明、算法资料索引、论文生成流程、验证命令。 |
| `CHANGELOG.md` | 版本历史与变更记录，采用语义化版本 `MAJOR.MINOR.PATCH`。 |

### 4.2 角色层 (`references/roles/`)

#### 4.2.1 建模手 (`references/roles/建模手/`)

**职责**：题目理解、数据检查、模型选择、算法设计、验证方案规划。

**固定交付物**：
- `PROJECT_ROOT/题目分析报告.md`
- `PROJECT_ROOT/术语表格.md`

**核心参考文件**：
- `references/工作流程.md`：分析步骤、数据检查清单、模型评估流程。
- `references/建模设计理论.md`：模型组合决策树、每子问题最多两个独立模型体系的约束说明。
- `references/常见模式.md`：11 种建模常见模式（优化/预测/评价/综合分类）。
- `references/质检清单.md`：交付前自检项。

**关键约束**：
- 先完整理解题目、附件、目标、约束和评价口径，再形成结论。
- 每道子问题最多使用两个独立模型体系；物理题中同一机理的不同近似/精度展开按一个模型族计数。

#### 4.2.2 编程手 (`references/roles/编程手/`)

**职责**：Python 或 MATLAB 代码实现、运行、结果输出、可视化、复现清单生成。

**固定交付物**：
- 可运行的 `.py` 或 `.m` 代码
- `results/` 中的表格（`.csv` / `.xlsx`）
- `figures/` 中的三类图：`raw_*`（原始数据图）、`process_*`（模型运行过程图）、`result_*`（最终结果图）
- `results/复现清单.json`

**核心参考文件**：
- `references/工作流程.md`：编码、运行、验证、出图、生成复现清单的流程。
- `references/可视化规范.md`：SCI/Nature 级可视化规范（无网格线、仅左+下 spines、配色体系、统计标注规则）。
- `references/MATLAB规范.md`：MATLAB 工具箱、出图、结果导出规范。
- `references/常见模式.md`：11 种图表常见模式（分组柱状图、趋势+CI、热力图、气泡图、箱线图等）。

**关键脚本**：
- `scripts/check_env.py`：按选中的模型功能动态检查 Python 依赖（见 5.1）。
- `scripts/check_matlab_env.m`：MATLAB 动态依赖检查。
- `scripts/repro_manifest.py`：生成可复现运行清单 JSON（见 5.1）。

#### 4.2.3 论文手 (`references/roles/论文手/`)

**职责**：基于真实运行结果构建论证并生成完整 Word 论文。

**唯一交付物**：`PROJECT_ROOT/完整论文.docx`

**核心参考文件**：
- `references/工作流程.md`：Claim-Evidence 映射核对、官方结构写作、DOCX 生成、校验与渲染抽检流程。
- `references/章节模板.md`：摘要六要素、引言五要素、结果证据阶梯、Discussion 架构。
- `references/写作规范.md`：去 AI 味写作指南、人称约束、叙述方式规范。
- `references/论文格式规范.md`：Word 格式、OMML 公式、三线表、篇幅与图文引用检查。
- `references/自审框架.md`：四轮自审（论证逻辑 → 章节结构 → 表述质量 → 格式规范）。
- `references/英文化工作流.md`：三阶段转换（理解 → 写作 → 校准）。

### 4.3 算法资料层 (`assets/`)

| 文件 | 覆盖方向 |
|------|----------|
| `01-优化算法说明.md` | 线性规划、整数规划、非线性规划、多目标优化、启发式算法（遗传/模拟退火/粒子群） |
| `02-预测类算法说明.md` | 灰色预测、时间序列（ARIMA）、回归预测、机器学习预测 |
| `03-评价类算法说明.md` | AHP、TOPSIS、熵权法、灰色关联分析、DEA |
| `04-图论与网络分析算法说明.md` | 最短路、网络流、最小生成树、匹配、复杂网络 |
| `05-统计分析与数据处理算法说明.md` | 假设检验、聚类、降维（PCA）、多元统计 |
| `06-综合类算法说明.md` | 蒙特卡洛模拟、排队论、博弈论、马尔科夫链、微分方程、元胞自动机 |
| `07-机器学习算法说明.md` | 随机森林、集成学习、神经网络、异常检测、分类与聚类 |

**路由入口**：`references/算法索引.md` —— 按问题特征（如"约束优化"→`01-优化算法说明.md`）引导加载。

### 4.4 工具层 (`tools/`)

#### 4.4.1 DOCX 工具 (`tools/docx/`)

**职责**：Word 论文生成、LaTeX→OMML 公式转换、OOXML 解包/校验/重打包、修订接受、批注管理。

**核心脚本**：
- `paper_format.py`：论文格式构建主模块，提供 `new_document()`、`title()`、`abstract_title()`、`body()`、`keywords()`、`equation()`、`three_line_table()`、`figure_caption()`、`save_document()`、`validate_paper_structure()` 等 API。
- `equations.py`：LaTeX 子集 → Word 原生 OMML 公式转换器，支持分式、上下标、根式、希腊字母、关系符号、反三角函数、矩阵等。
- `self_check.py`：一键自检（环境、公式转换、三线表、字数统计）。
- `office/validate.py`：OOXML 结构校验（XSD 模式验证 + 修订标记检查）。
- `office/unpack.py` / `office/pack.py`：DOCX 解包与重打包。
- `office/soffice.py`：LibreOffice 隔离调用封装（用于渲染抽检和公式重算）。

**关键数据类**：
```python
@dataclass(frozen=True)
class ContestProfile:
    name: str
    paper: str
    margins: tuple[float, float, float, float]
    required_markers: tuple[str, ...]
    rules_source: str
```

**内置竞赛配置**：`cumcm`（全国大学生数学建模竞赛）、`mcm-icm`。

#### 4.4.2 XLSX 工具 (`tools/xlsx/`)

**职责**：Excel 读取（无隐式表头推断）、公式重算、错误检查。

**核心脚本**：
- `read_rows.py`：`read_excel_rows()` —— 使用 `openpyxl` 按行读取，支持 `header=False` 和 `expected_rows` 断言，避免 `pandas` 默认 `header=0` 吞掉第一行数据。
- `recalc.py`：通过隔离的 LibreOffice 进程重算含公式的工作簿，并检查 `#VALUE!` / `#DIV/0!` / `#REF!` 等 7 种 Excel 错误。

#### 4.4.3 PDF 工具 (`tools/pdf/`)

**职责**：PDF 文本/表格/图片提取、合并/拆分/旋转、OCR、表单填充、水印添加。

**依赖库**：`pypdf`、`pdfplumber`、`reportlab`、`PyMuPDF`、`pytesseract`、`pdf2image`。

**核心脚本**：
- `extract_form_structure.py` / `extract_form_field_info.py`：提取 PDF 表单结构。
- `fill_fillable_fields.py` / `fill_pdf_form_with_annotations.py`：填充可填写字段。
- `convert_pdf_to_images.py`：PDF 转图片（用于 OCR 或预览）。

#### 4.4.4 双引擎论文搜索 (`tools/paper_search/`)

**职责**：并行调用 OpenAlex 与 AnySearch，结果去重、交叉验证、相关性优先排序。

**核心脚本**：
- `hybrid_scholar.py`：双引擎融合主入口，使用 `ThreadPoolExecutor` 并行调用，输出分三区（交叉验证 / OpenAlex 独有 / AnySearch 独有）。
- `openalex_scholar.py`：OpenAlex API 封装，支持领域过滤（`FIELD_CONCEPTS`）、引用量/年份过滤、多种排序。
- `anysearch_academic.py`：AnySearch Academic API 封装（JSON-RPC 2.0）。

**关键数据类**：
```python
@dataclass
class HybridPaper:
    title: str
    authors: List[str]
    year: Optional[int]
    citations: int
    doi: Optional[str]
    abstract: Optional[str]
    sources: List[str]          # ["openalex"], ["anysearch"], 或两者
    venue: Optional[str] = None
    # ...
    @property
    def cross_validated(self) -> bool:
        return len(self.sources) >= 2
```

---

## 5. 关键类与函数说明

### 5.1 编程手脚本 (`references/roles/编程手/scripts/`)

#### `check_env.py` —— Python 动态依赖检查

```python
FEATURES = {
    "data":               [("numpy", "numpy"), ("pandas", "pandas")],
    "visualization":      [("matplotlib", "matplotlib")],
    "excel":              [("openpyxl", "openpyxl")],
    "optimization":       [("scipy", "scipy")],
    "integer-optimization": [("pulp", "pulp|ortools")],
    "statistics":         [("scipy", "scipy"), ("statsmodels", "statsmodels")],
    "machine-learning":   [("sklearn", "scikit-learn")],
    "time-series":        [("statsmodels", "statsmodels")],
    "graph":              [("networkx", "networkx")],
    "image":              [("PIL", "pillow")],
}

def check_features(features: List[str]) -> dict:
    """
    按功能检查依赖。
    返回: {"ok": bool, "features": [...], "python": str,
           "installed": {pkg: version}, "missing": [...],
           "install_command": str|None}
    """
```

**CLI 用法**：
```bash
python check_env.py --features data visualization optimization
```

#### `repro_manifest.py` —— 可复现清单生成

```python
SKILL_ROOT: Path = Path(__file__).resolve().parents[4]

def sha256_file(path: Path) -> str:
    """计算文件 SHA-256 摘要。"""

def build_manifest(
    inputs,           # 输入文件路径列表
    seed,             # 随机种子
    parameters,       # 关键参数字典
    command,          # 唯一复现命令
    packages,         # Python 包名列表
    runtime_name="python",
    runtime_version=None,
    dependencies=None,
) -> dict:
    """
    构建复现清单字典，包含：
    - schema_version
    - created_at_utc
    - random_seed
    - input_files: [{path, sha256, bytes}]
    - runtime: {name, version, platform, dependencies}
    - key_parameters
    - reproduce_command
    """

def resolve_output(project_root: Path, relative_path: str) -> Path:
    """
    解析输出路径，并强制执行路径隔离：
    - 禁止 PROJECT_ROOT 位于 SKILL_ROOT 内部
    - 禁止输出路径指回 SKILL_ROOT
    """
```

#### `check_matlab_env.m` —— MATLAB 动态依赖检查

```matlab
report = check_matlab_env(["data", "visualization", "optimization"]);
% 检查指定功能所需的 MATLAB 工具箱和函数是否可用
```

### 5.2 DOCX 工具脚本 (`tools/docx/scripts/`)

#### `paper_format.py` —— 论文格式构建

```python
# 竞赛配置
CONTEST_PROFILES = {
    "cumcm": ContestProfile(...),
    "mcm-icm": ContestProfile(...),
}

def get_profile(contest="cumcm") -> ContestProfile:
    """获取竞赛配置，未知竞赛抛出 ValueError。"""

def setup_page(doc: Document, contest="cumcm"):
    """按竞赛配置设置页面大小和页边距。"""

def new_document(contest="cumcm", template_path=None, preserve_template_content=False) -> Document:
    """
    创建新文档。
    - template_path: 官方模板路径；继承其样式。
    - preserve_template_content: False 时清除示例正文，True 时保留固定摘要页等。
    """

def title(doc: Document, text: str): ...
def abstract_title(doc: Document): ...
def body(doc: Document, text: str): ...
def keywords(doc: Document, text: str): ...
def heading1(doc: Document, text: str): ...
def heading2(doc: Document, text: str): ...
def equation(doc: Document, latex: str): ...
def equation_placeholder(doc: Document, latex: str) -> Tuple[str, str]: ...
def three_line_table(doc: Document, rows: List[List[str]]): ...
def figure_caption(doc: Document, text: str): ...
def page_break(doc: Document): ...
def save_document(doc: Document, project_root: Path, contest="cumcm"): ...

def count_chinese_chars(doc: Document) -> int:
    """统计中文字符数，用于篇幅检查。"""

def validate_paper_structure(doc: Document, contest="cumcm", rendered_pages: int = None) -> List[str]:
    """
    校验论文结构，返回错误列表。检查项包括：
    - 官方前置结构（摘要、关键词等 required_markers）
    - 中英文混排篇幅
    - 可编辑公式数量
    - 图和表的数量
    - 图表编号连续性
    - 正文引用（检查孤儿图表、缺失编号、编号跳跃）
    - 参考文献与正文引用双向对应
    """
```

#### `equations.py` —— LaTeX → OMML 公式转换

```python
OMML_NS = "http://schemas.openxmlformats.org/officeDocument/2006/math"

LATEX_SYMBOLS = {
    "alpha": "α", "beta": "β", "gamma": "γ", ...,
    "mu": "μ", "nu": "ν", "approx": "≈",
    "arcsin": "arcsin", "arccos": "arccos", "arctan": "arctan",
    # ... 完整希腊字母、关系符号、函数名映射
}

def latex2omml(latex: str) -> str:
    """将 LaTeX 子集转换为 OMML XML 字符串。"""

def replace_placeholders_in_docx(input_path, output_path, replacements: dict):
    """将 DOCX 中的占位符替换为 OMML 公式。"""

def generate_docx_from_markdown(md_path, output_path, template_path=None):
    """从 Markdown 生成含公式的 DOCX（复杂公式优先走 Pandoc）。"""
```

**支持的 LaTeX 特性**：分式、上下标、根式、n 次根、常用希腊字母、关系符号、反三角函数、常见矩阵。

### 5.3 XLSX 工具脚本 (`tools/xlsx/scripts/`)

#### `read_rows.py`

```python
from openpyxl import load_workbook

def read_excel_rows(path, sheet=0, *, header=False, expected_rows=None):
    """
    无隐式表头推断地读取 XLSX。
    - header=False: 返回 List[Tuple]，不将第一行当作列名。
    - expected_rows: 若指定，行数不符时抛出 ValueError。
    """
```

#### `recalc.py`

```python
EXCEL_ERRORS = ("#VALUE!", "#DIV/0!", "#REF!", "#NAME?", "#NULL!", "#NUM!", "#N/A")

def recalc_workbook(path: Path, timeout_sec: int = 60) -> dict:
    """
    使用隔离的 LibreOffice 进程重算 XLSX 公式：
    1. 使用唯一临时 LibreOffice 配置；
    2. 把重算结果写入临时目录；
    3. 检查常见 Excel 错误；
    4. 仅在成功后原子替换目标文件；
    5. 超时、失败或未生成输出时保留原文件。
    返回: {"status": "success"|"errors_found", "total_errors": int,
           "total_formulas": int, "error_summary": {...}}
    """
```

### 5.4 论文搜索脚本 (`tools/paper_search/scripts/`)

#### `hybrid_scholar.py`

```python
@dataclass
class HybridPaper:
    title: str
    authors: List[str]
    year: Optional[int]
    citations: int
    doi: Optional[str]
    abstract: Optional[str]
    sources: List[str] = field(default_factory=list)
    venue: Optional[str] = None
    # ...
    @property
    def cross_validated(self) -> bool: ...
    def to_dict(self) -> Dict: ...

def search_hybrid(query: str, limit: int = 10, email: str = None, api_key: str = None) -> Dict:
    """
    并行调用 OpenAlex + AnySearch，融合结果：
    - DOI 相同 → 直接交叉验证合并；
    - 无 DOI → 标题高度相似且年份相容时合并；
    - 同一引擎内预印本与正式版折叠，优先保留元数据更完整的记录；
    - 按查询词覆盖率过滤和重排，相关性优先于引用量。
    返回: {"cross_validated": [...], "openalex_only": [...], "anysearch_only": [...]}
    """
```

#### `openalex_scholar.py`

```python
FIELD_CONCEPTS = {
    'mathematics': 'https://api.openalex.org/concepts/C33923547',
    'computer_science': 'https://api.openalex.org/concepts/C41008148',
    'engineering': 'https://api.openalex.org/concepts/C127413603',
    'physics': 'https://api.openalex.org/concepts/C185592680',
    'statistics': 'https://api.openalex.org/concepts/C162324750',
    'operations_research': 'https://api.openalex.org/concepts/C126322002',
    'economics': 'https://api.openalex.org/concepts/C162111547',
}

@dataclass
class Paper:
    title: str
    authors: List[str]
    publication_year: Optional[int]
    # ...

class OpenAlexScholar:
    def search(self, query: str, limit: int = 10, **filters) -> List[Paper]: ...
    def get_paper_by_doi(self, doi: str) -> Optional[Paper]: ...
```

### 5.5 OOXML 校验器 (`tools/docx/scripts/office/validators/`)

```python
class DOCXSchemaValidator:
    """基于 XSD 模式库校验 DOCX 内部 XML 结构。"""
    def __init__(self, unpacked_dir: Path, original_file: Path = None, verbose=False): ...
    def validate(self) -> bool: ...

class RedliningValidator:
    """校验修订标记，对比原始文件检测未接受的修订。"""
    def __init__(self, unpacked_dir: Path, original_file: Path, verbose=False, author="Claude"): ...
    def validate(self) -> bool: ...
```

---

## 6. 依赖关系

### 6.1 Python 依赖（按功能分区）

| 功能域 | 依赖包 | 用途 |
|--------|--------|------|
| 数据处理 | `numpy`, `pandas` | 数值计算、表格处理 |
| 可视化 | `matplotlib` | 科研绘图 |
| Excel | `openpyxl` | XLSX 读写 |
| 优化 | `scipy` | 科学计算与优化 |
| 整数优化 | `pulp` 或 `ortools` | 线性/整数规划 |
| 统计 | `scipy`, `statsmodels` | 统计检验、时间序列 |
| 机器学习 | `scikit-learn` | 机器学习模型 |
| 图论 | `networkx` | 网络分析 |
| 图像 | `pillow` | 图像处理 |
| Word 生成 | `python-docx`, `lxml` | DOCX 构建与 OMML 公式 |
| PDF 处理 | `pypdf`, `pdfplumber`, `reportlab`, `PyMuPDF` | PDF 读写、提取、创建 |
| OCR | `pytesseract`, `pdf2image` | 扫描 PDF 文字识别 |
| 论文搜索 | `urllib` (标准库) | OpenAlex API 调用 |

**动态检查机制**：编程手在编码前只检查当前模型实际需要的功能域，不要求安装全部包。

### 6.2 外部工具依赖

| 工具 | 用途 | 检查方式 |
|------|------|----------|
| LibreOffice | XLSX 公式重算、DOCX 渲染抽检 | `tools/docx/scripts/office/soffice.py` 自动查找 |
| Pandoc | 复杂 LaTeX → DOCX 转换（备选） | 在 `equations.py generate` 流程中调用 |
| `poppler-utils` (pdftotext/pdfimages) | PDF 文本/图片提取 | CLI 命令 |
| `qpdf` | PDF 合并/拆分/旋转 | CLI 命令 |
| MATLAB + 工具箱 | MATLAB 代码运行 | `check_matlab_env.m` 动态检查 |

### 6.3 模块间依赖关系图

```
SKILL.md (根路由)
    ├── references/roles/建模手/SKILL.md
    │       ├── references/建模设计理论.md
    │       ├── references/常见模式.md
    │       └── ../../../references/算法索引.md → assets/*.md
    │       └── ../../../tools/paper_search/SKILL.md
    │               └── scripts/hybrid_scholar.py
    │                       ├── openalex_scholar.py
    │                       └── anysearch_academic.py
    │
    ├── references/roles/编程手/SKILL.md
    │       ├── references/可视化规范.md
    │       ├── references/MATLAB规范.md
    │       ├── references/常见模式.md
    │       ├── scripts/check_env.py
    │       ├── scripts/check_matlab_env.m
    │       ├── scripts/repro_manifest.py
    │       └── ../../../tools/xlsx/SKILL.md
    │               └── scripts/read_rows.py
    │               └── scripts/recalc.py → ../docx/scripts/office/soffice.py
    │
    └── references/roles/论文手/SKILL.md
            ├── references/章节模板.md
            ├── references/写作规范.md
            ├── references/论文格式规范.md
            ├── references/自审框架.md
            ├── references/英文化工作流.md
            └── ../../../tools/docx/SKILL.md
                    └── scripts/paper_format.py
                    └── scripts/equations.py
                    └── scripts/self_check.py
                    └── scripts/office/validate.py
                    └── scripts/office/unpack.py
                    └── scripts/office/pack.py
                    └── scripts/office/soffice.py
```

---

## 7. 项目运行方式

### 7.1 环境验证

```bash
# 1. 运行全部回归测试
python -m unittest discover -s tests -v

# 2. DOCX 工具自检
python tools/docx/scripts/self_check.py

# 3. Python 代码编译检查
python -m compileall -q tools references/roles/编程手/scripts
```

### 7.2 三阶段工作流运行示例

#### 完整流程（自动顺序执行）

```bash
# 用户指令示例（由 Agent 执行）：
# "使用数学建模 Skill 完成这道题，从题目分析一直生成完整 Word 论文。"

# 实际执行的是按 SKILL.md 路由依次读取角色文档并执行：
# 1. 读取 references/roles/建模手/SKILL.md → 产出 题目分析报告.md + 术语表格.md
# 2. 读取 references/roles/编程手/SKILL.md → 产出 代码 + 结果 + 图 + 复现清单.json
# 3. 读取 references/roles/论文手/SKILL.md → 产出 完整论文.docx
```

#### 单阶段执行

```bash
# 仅建模分析
# → 只加载 references/roles/建模手/SKILL.md 及其引用

# 仅代码实现
# → 加载 references/roles/编程手/SKILL.md
# → 读取已有 题目分析报告.md 和 术语表格.md

# 仅论文生成
# → 加载 references/roles/论文手/SKILL.md
# → 读取全部上游产物和真实运行结果
```

### 7.3 工具独立运行示例

#### 双引擎论文搜索

```bash
# 正式检索（默认双引擎）
python tools/paper_search/scripts/hybrid_scholar.py \
  --query "robust optimization vehicle routing" \
  --limit 10 \
  --json

# 单引擎诊断
python tools/paper_search/scripts/hybrid_scholar.py \
  --query "analytic hierarchy process" \
  --limit 8 \
  --openalex-only

# 配置 AnySearch 密钥（如需要）
export ANYSEARCH_API_KEY="<your-key>"
```

#### 动态依赖检查

```bash
# Python：按功能检查
python references/roles/编程手/scripts/check_env.py \
  --features data visualization optimization

# MATLAB
# addpath("references/roles/编程手/scripts");
# report = check_matlab_env(["data", "visualization", "optimization"]);
```

#### 复现清单生成

```bash
python references/roles/编程手/scripts/repro_manifest.py \
  --project-root ./my_project \
  --inputs ./my_project/data/input.csv \
  --seed 42 \
  --params '{"alpha": 0.1}' \
  --command "python 问题1_求解.py --seed 42" \
  --packages numpy pandas scipy matplotlib
```

#### DOCX 公式替换

```bash
# 占位符替换为 OMML 公式
python tools/docx/scripts/equations.py replace "输入.docx" \
  --replace "EQ_OBJECTIVE" "\min f(x)=\sum_{i=1}^{n}x_i^2" \
  --output "输出.docx"

# 从 Markdown 生成含公式 DOCX
python tools/docx/scripts/equations.py generate "论文.md" \
  --output "论文.docx" \
  --template "官方模板.docx"
```

#### XLSX 公式重算

```bash
python tools/xlsx/scripts/recalc.py "./results/结果.xlsx" 60
```

#### OOXML 校验

```bash
# 解包
python tools/docx/scripts/office/unpack.py "论文.docx" "./unpacked"

# 校验
python tools/docx/scripts/office/validate.py "论文.docx"

# 重打包
python tools/docx/scripts/office/pack.py "./unpacked" "论文_已修复.docx" --original "论文.docx"
```

---

## 8. 测试说明

### 8.1 测试目录结构

```text
tests/
├── test_docx_tools.py        # DOCX 工具测试（模板处理、样式继承、内容清除）
├── test_equations.py         # LaTeX → OMML 公式转换测试
├── test_paper_format.py      # 论文格式与结构校验测试
├── test_paper_search.py      # 双引擎搜索与去重逻辑测试
├── test_recalc.py            # Excel 公式重算测试
├── test_reproducibility.py   # 复现清单与动态依赖测试
└── test_xlsx_read.py         # Excel 无隐式表头读取测试
```

### 8.2 测试覆盖范围

| 测试文件 | 覆盖内容 |
|----------|----------|
| `test_reproducibility.py` | `check_env.check_features()` 只检查选中功能；`repro_manifest.build_manifest()` 正确记录种子、哈希、参数、命令；`resolve_output()` 阻止路径回退到 `SKILL_ROOT`；MATLAB 运行时记录。 |
| `test_paper_format.py` | `new_document()` 清除模板示例正文但保留样式；`preserve_template_content=True` 保留官方固定页；`validate_paper_structure()` 检测缺失摘要/关键词；完整 front matter 通过校验。 |
| `test_equations.py` | LaTeX 分式、上下标、根式、希腊字母、关系符号正确转换为 OMML XML；未知命令报错而非静默生成错误文本。 |
| `test_docx_tools.py` | `self_check.py` 一键通过；三线表生成；字数统计；页面包设置。 |
| `test_paper_search.py` | `HybridPaper.cross_validated` 正确标记；DOI 去重；标题模糊去重；查询词覆盖率过滤。 |
| `test_recalc.py` | LibreOffice 重算后正确检测 `#VALUE!` / `#DIV/0!`；超时保留原文件。 |
| `test_xlsx_read.py` | `read_excel_rows(header=False)` 不吞第一行；`expected_rows` 断言生效。 |

### 8.3 运行测试

```bash
# 全部测试
python -m unittest discover -s tests -v

# 单个测试文件
python -m unittest tests.test_paper_format -v
python -m unittest tests.test_reproducibility -v
```

---

## 9. 关键设计决策与约定

### 9.1 路径隔离

- `SKILL_ROOT` = 本仓库根目录，**只读**。所有角色规范、算法资料、脚本和模板从这里读取。
- `PROJECT_ROOT` = 用户题目所在目录，**只写**。所有运行产物（代码、结果、图、论文）只写入这里。
- 两个根目录必须不同；默认禁止覆盖 `SKILL_ROOT` 内任何文件。

### 9.2 渐进式加载

禁止一次性加载全部资料。按当前任务只读取必要的角色入口和引用文件：

| 当前任务 | 额外读取 |
|----------|----------|
| 选模型或查算法 | `references/算法索引.md` → 一个匹配的 `assets/*.md` |
| 搜索论文 | `tools/paper_search/SKILL.md` |
| 读取题目 PDF | `tools/pdf/SKILL.md` |
| 处理 Excel | `tools/xlsx/SKILL.md` |
| 生成 Word 论文 | `tools/docx/SKILL.md` |

### 9.3 模型选择约束

- 每道子问题 **最多两个独立模型体系**。
- 物理题中，**同一控制机理**在不同近似阶次、边界条件或精度下的展开，计为 **一个模型族**。
- 辅助的灵敏度分析、误差检验和可视化 **不计作独立模型**。

### 9.4 公式与可视化规范

- 论文公式必须为 **Word 原生 OMML**，确保在 Word 中可编辑、可渲染。
- 可视化 **禁用网格线**，仅保留左+下 spines；统计标注必须由代码计算；SVG + PNG 双格式输出，SVG 强制 `svg.fonttype='none'` 保持文本可编辑。

### 9.5 论文篇幅约束（CUMCM 示例）

- **质量目标**：约 15000 字词单位、约 20 页（可调整，非官方最低要求）。
- **硬约束**（以 2026 年官方规范为例）：摘要原则上不超过一页，**正文不超过 30 页**。
- 实际交付必须重新核对目标届次的官方文件。

---

## 10. 相关链接与资源

- **仓库地址**：https://github.com/XiaoMaColtAI/math-modeling-skill
- **版本文件**：[VERSION](VERSION)
- **主入口**：[SKILL.md](SKILL.md)
- **项目说明**：[README.md](README.md)
- **更新日志**：[CHANGELOG.md](CHANGELOG.md)
- **算法索引**：[references/算法索引.md](references/算法索引.md)
- **渐进式导航**：[references/README.md](references/README.md)
