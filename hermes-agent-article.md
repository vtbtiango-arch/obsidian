### MacOS安装

官方源生支持，有mac无脑选mac，这个和OpenClaw是一样的。智能体时代有一台mac很重要。

直接执行官方一键安装：

```bash
# 执行官方脚本
curl -fsSL https://raw.githubusercontent.com/NousResearch/hermes-agent/main/scripts/install.sh | bash
# 安装完成后，会进行初始化，按照提示一步步输入api key，还会询问你是否导入OpenClaw的配置文件和skill

# 手动执行初始化命令：
hermes setup

# 后续在命令行中启动：
hermes
# 启动网页UI（目前网页UI还没有AI对话功能，只有各种配置项目，未来应该会加入AI对话窗口，就像OpenClaw一样）
hermes dashboard
```

安装过程会自动：安装 Python / Node / 依赖、创建虚拟环境、初始化配置、启动 setup wizard。

#### 网络问题

如果你所在地区，需要开代理访问外网，则需要在安装前，先解决Terminal的网络代理设置。 Terminal默认不走系统代理，所以要么把代理开启TUN模式（虚拟网卡模式），要么执行以下命令（更推荐）：

```bash
echo 'export http_proxy="http://127.0.0.1:7897"' >> ~/.zshrc
echo 'export https_proxy="http://127.0.0.1:7897"' >> ~/.zshrc
echo 'export all_proxy="socks5://127.0.0.1:7897"' >> ~/.zshrc
source ~/.zshrc
```

安装脚本会下载来自多个源的内容（npm, GitHub, python, playwright），单纯替换国内npm镜像不足以解决问题，所以还是用网络工具来解决。

### Windows安装

**Hermes 官方明确表示不支持 Windows 原生环境。** 由于 Hermes 具备极深的底层系统控制权（包含复杂的沙盒、Docker集成、终端穿透）， **在 Windows 上必须使用 WSL2** 。

在windows中打开PowerShell，然后开始下面的安装步骤：

1. 安装WSL2
```powershell
wsl --install
```
2. 安装完成后，设置用户名密码，然后启动wsl2
```powershell
wsl
```
3. **如果有网络问题** ：方法一走代理，那么在wsl2里执行：
```powershell
# 1. 定义配置块，增加唯一标识符防止重复写入
PROXY_BLOCK_START="# >>> WSL PROXY CONFIG >>>"
PROXY_BLOCK_END="# <<< WSL PROXY CONFIG <<<"

# 2. 清理旧配置，防止多次执行导致 .bashrc 变得臃肿
sed -i "/$PROXY_BLOCK_START/,/$PROXY_BLOCK_END/d" ~/.bashrc

# 3. 写入新配置（使用更加健壮的 IP 获取方式）
cat << EOF >> ~/.bashrc
$PROXY_BLOCK_START
# 动态获取宿主机IP：先尝试路由表，如果失败则尝试 resolv.conf
export hostip=\$(ip route show | grep default | head -n 1 | awk '{print \$3}')
if [ -z "\$hostip" ]; then
    export hostip=\$(grep nameserver /etc/resolv.conf | awk '{print \$2}')
fi

export PROXY_PORT=7897
export http_proxy="http://\$hostip:\$PROXY_PORT"
export https_proxy="http://\$hostip:\$PROXY_PORT"
export all_proxy="socks5://\$hostip:\$PROXY_PORT"
# 必须配置 NO_PROXY，防止本地开发流量走代理
export NO_PROXY="localhost,127.0.0.1,::1,*.local,192.168.*,10.*,172.16.*,172.17.*,172.18.*,172.19.*,172.20.*"
$PROXY_BLOCK_END
EOF

# 4. 生效并提示
source ~/.bashrc

# 5. 配置 npm（若已安装）  
if command -v npm >/dev/null 2>&1; then  
npm config set registry https://registry.npmmirror.com  
npm config set proxy "http://$hostip:$PROXY_PORT"  
npm config set https-proxy "http://$hostip:$PROXY_PORT"  
npm config set fetch-retries 5  
npm config set fetch-timeout 600000  
fi

echo "WSL2 网络代理已永久配置完成。"
echo "当前 Windows 宿主机 IP: $hostip"
echo "本地开发白名单 (NO_PROXY) 已自动配置。"
```

测试是否生效：

```powershell
curl -s ipinfo.io
```
4. 方法二替换国内npm镜像和apt源，没有网络代理稳，部分组件有可能安装部完全。在wsl2里执行：
```powershell
set -e

echo "检测 Ubuntu 版本..."
CODENAME=$(lsb_release -cs)

echo "备份原始源..."
sudo cp /etc/apt/sources.list /etc/apt/sources.list.bak 2>/dev/null || true
sudo cp /etc/apt/sources.list.d/ubuntu.sources /etc/apt/sources.list.d/ubuntu.sources.bak 2>/dev/null || true

echo "配置 APT 清华源..."

if [ -f /etc/apt/sources.list.d/ubuntu.sources ]; then
sudo tee /etc/apt/sources.list.d/ubuntu.sources > /dev/null <<EOF
Types: deb
URIs: https://mirrors.tuna.tsinghua.edu.cn/ubuntu/
Suites: $CODENAME $CODENAME-updates $CODENAME-backports
Components: main restricted universe multiverse

Types: deb
URIs: https://mirrors.tuna.tsinghua.edu.cn/ubuntu/
Suites: $CODENAME-security
Components: main restricted universe multiverse
EOF
else
sudo tee /etc/apt/sources.list > /dev/null <<EOF
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ $CODENAME main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ $CODENAME-updates main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ $CODENAME-backports main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ $CODENAME-security main restricted universe multiverse
EOF
fi

echo "更新 apt..."
sudo apt update

echo "安装基础组件..."
sudo apt install -y curl git build-essential ffmpeg ripgrep

echo "配置 npm 镜像..."
npm config set registry https://registry.npmmirror.com

echo "配置 Playwright 镜像..."
npm config set ELECTRON_MIRROR https://npmmirror.com/mirrors/electron/
npm config set PLAYWRIGHT_DOWNLOAD_HOST https://npmmirror.com/mirrors/playwright

echo "当前 npm registry："
npm config get registry

echo "完成。"
```
5. 执行官方安装命令
```powershell
curl -fsSL https://raw.githubusercontent.com/NousResearch/hermes-agent/main/scripts/install.sh | bash
```

### 相关配置与核心技巧

#### 常用命令

```bash
hermes setup        # 一键初始化（推荐）
hermes config set   # 设置单个配置项
hermes model        # 切模型 / provider
hermes tools        # 启用/关闭工具
hermes              # 启动命令行 CLI 交互
hermes gateway      # 启动消息网关（Telegram等）
hermes doctor       # 诊断问题
hermes update       # 更新版本
```

**诊断与修复** 遇到问题可以用doctor来进行检查和修复，这与OpenCLaw是一样的。

```bash
# 诊断
hermes doctor

# 诊断并修复
hermes doctor --fix
```

**导入OpenClaw配置文件** ：官方明确支持导入OpenClaw配置文件：

```bash
hermes claw migrate
```

安装的时候也会自动检测并提示你是否导入。导入内容：memory、skills、config等等。

#### 连接消息软件

在命令行中，执行命令： `hermes gateway setup` ，进入配置向导。 飞书、微信等工具都可以通过扫描二维码来连接。

**微信** ：扫码后走的是微信ClawBot，也就是微信与OpenClaw的链接方式。 **飞书** ：扫码后自动创建机器人，你也可以选择手动创建。

#### SOUL.md (系统身份与全局约束)

**路径**: `~/.hermes/SOUL.md` **修改方式**: **纯手动修改** 。 **作用与机制**: 系统提示词，决定了 Hermes 的绝对行事准则。它具有最高优先级。 **举例**:

```markdown
- 保持简洁
- 针对技术术语要进行解释
- 主动提供最佳实践建议
- 在执行任何可能修改本地文件系统的终端命令前，必须先使用 \`ls\` 或 \`cat\` 探测环境。
- 在输出代码样例时，默认使用 TypeScript 并强制包含完整的代码注释。
```

#### MEMORY.md (事实记忆) 与 USER.md (用户画像)

**路径**: 位于 `~/.hermes/memories/` 目录下。 **修改方式**: **通过对话框让 AI 自动管理** 。 Hermes 自带一个原生的 `memory` 工具（包含 `add` 、 `replace` 、 `remove` 动作）。 当你在对话中说“记住我所有的前端开发都用 TypeScript”，Agent 会自动调用该工具，使用特殊的 `§` 符号作为分隔符将内容写入。并在空间不足时，自动触发 LLM Summarization（摘要压缩）来合并旧记忆。

#### AGENTS.md (项目级上下文)

**路径**: 存放在你具体的项目根目录下（例如 `~/projects/personal-site/AGENTS.md` ）。 **配置策略**: 将特定项目的上下文与全局隔离。比如在该文件中写明：本项目是一个个人网站，技术栈为 Next.js + Tailwind CSS，所有 API 调用必须封装在 `src/services/` 下。

#### 记忆机制

| 记忆模块分类 | 机制特性与容量限制及配置路径 | 用户/系统触发逻辑 |
| --- | --- | --- |
| **个性化用户画像** | 记录身份与偏好。限 **1375 字符** （约 **500 Tokens** ）。   配置文件：USER.md | **显式指令** ：告知“称呼我为杰森”。   **隐式推断** ：Agent 自动观察你的技术栈偏好，静默调用 memory 工具更新。 |
| **事实性长期记忆** | 存储客观事实与项目经验。限 **2200 字符** （约 **800 Tokens** ）。   配置文件：MEMORY.md | **显式指令** ：要求“记住这个 API 地址”。   **后台刷写** ：闲置超时前系统主动提炼写入。（需开启新会话生效） |
| **程序化技能库** | 封装高频工作流。按需加载（Lazy Load），支持自我进化。   配置文件：SKILL.md | **自主进化** ：完成复杂任务后自动生成。   **显式封装** ：下令“将刚才的操作存为技能”，后续通过 /skills 调用。 |
| **跨会话历史检索** | 处理历史对话。利用本地 SQLite 与 FTS5 引擎。   配置文件：state.db | **按需触发** ：当内存无对应信息时，Agent 自动调用 session\_search 工具检索本地数据库并重组答案。 |
| **项目局部上下文** | 针对特定项目规范，兼容.cursorrules。   配置文件：项目根目录或子目录下的 AGENTS.md | **系统探测** ：启动会话或通过终端 cd 进入子目录时，动态拦截并临时注入该目录下的规则。 |

#### 配置辅助模型

在模型配置界面，可以配置多个模型，包括本地模型。 然后，分别针对下面的任务，分配不同的模型。

| 任务名称 | 任务功能说明 | 建议模型选型 | CLI 配置命令示例 |
| --- | --- | --- | --- |
| **`compression`** | **上下文压缩** ：会话Token逼近上限时，压缩并总结历史记录。 | **超长上下文 + 极低成本模型**   *(注：吞吐量极大，推荐 Gemini 3 Flash 或 Kimi K2.5，支持百万级上下文且 API 成本极低。)* | `hermes config set auxiliary.compression.model gemini-3-flash` |
| **`web_extract`** | **网页内容提取** ：清洗HTML DOM树并提炼为纯文本Markdown。 | **高吞吐 + 零成本本地模型**   *(注：这是典型的脏活。推荐本地 Ollama 运行 Qwen 3.5:14b 或 DeepSeek-R1-Distill，速度快且数据不出网。)* | `hermes config set auxiliary.web_extract.model qwen3.5:14b` |
| **`flush_memories`** | **记忆提炼与持久化** ：后台扫描对话，提取关键实体写入记忆库。 | **极强信息抽取 + 高性价比API**   *(注：推荐 DeepSeek-V3 (deepseek-chat)，其结构化抽取能力对标顶流，但 API 价格低。)* | `hermes config set auxiliary.flush_memories.model deepseek-chat` |
| **`vision`** | **视觉与多模态解析** ：处理图片输入、前端UI截图还原、图表解析。 | **原生多模态视觉模型**   *(注：用于你的前端开发UI还原，Kimi K2.5 或 Gemini 3 Pro，目前在空间理解和组件拆解上处于第一梯队。)* | `hermes config set auxiliary.vision.model gemini-3-pro` |
| **`skills_hub`** | **技能中枢** ：静默评估操作链路，自动编写并优化自动化脚本。 | **顶级代码推理模型**   *(注：它是在后台替你写代码和重构 Skill，绝不能省钱。强制要求使用 GPT-5.4 或 Claude 4 级别算力。)* | `hermes config set auxiliary.skills_hub.model gpt-5.4-turbo` |
| **`mcp`** | **MCP 协议交互** ：处理与外部MCP Server间的工具握手与参数转换。 | **极强Tool-calling模型**   *(注：必须严格遵守JSON Schema且不能产生幻觉。)* | `hermes config set auxiliary.mcp.model gpt-5.4-turbo` |
| **`approval`** | **危险命令审计** ：审查即将执行的Shell脚本是否存在高危破坏风险。 | **高安全对齐 + 低延迟本地模型**   *(注：响应要快，否则会卡住你的终端执行流程。本地部署 Llama-4:8b-Instruct。)* | `hermes config set auxiliary.approval.model llama-4:8b` |
| **`session_search`** | **历史会话检索** ：基于SQLite进行FTS5搜索，融合检索出的历史碎片。 | **极速响应 + 基础推理模型**   *(注：典型的RAG文本组装任务，DeepSeek-V3 或 Gemini 3 Flash 即可胜任。)* | `hermes config set auxiliary.session_search.model deepseek-chat` |

如果你配置了多个provider + model，那么：

```bash
# 指定使用openrouter提供的claude
hermes config set auxiliary.skills_hub.provider openrouter
# 模型名称必须带有厂商前缀（OpenRouter 的 slug 规范）
hermes config set auxiliary.skills_hub.model anthropic/claude-4

# 指定使用Anthropic官方的claude
hermes config set auxiliary.skills_hub.provider anthropic
# 注意：直连官方 API 时，模型名称必须是原生名称，绝不能带 anthropic/ 前缀
hermes config set auxiliary.skills_hub.model claude-sonnet-4-6
```

#### 聚焦主题压缩 /compress

当对话变长、响应变慢时，输入 `/compress [聚焦主题]` 。这会强制执行摘要压缩，清除冗余信息，并引导智能体优先保留与“聚焦主题”相关的核心语义

**致命机制** ：如果你给 `compression` 配置了一个上下文窗口比你的主力模型 **还要小** 的模型（例如某个最大仅支持 8K 或 32K 的开源模型），压缩任务会直接爆 Context Length Error（上下文超载报错）。 **Hermes 遇到这种报错时，会为了保证系统不崩溃，直接跳过总结，静默丢弃掉你中间所有的对话记录。**

**规避操作：** 强制分配长上下文窗口模型给 `compression` 任务。有1M~2M 超大上下文窗口，即可彻底杜绝你的大段对话内容在后台压缩时被系统静默截断吞掉的风险。

#### 安装Skills

1. 命令行安装：
```bash
hermes skills install <skill-name>
```
2. 直接与AI对话安装
3. 手动复制文件：在 `~/.hermes/skills/` 下新建一个文件夹（例如 `my-skill` ）。

官方skills市场： [https://hermes-agent.nousresearch.com/docs/skills/](https://hermes-agent.nousresearch.com/docs/skills/)

#### Karpathy Skill

Hermes Agent官方已经集成了Karpathy的LLM Wiki Skill。GitHub地址： `NousResearch/hermes-agent/blob/main/skills/research/llm-wiki/SKILL.md`

使用方法：

- **变量挂载** ：在 ~/.hermes/.env 中配置 WIKI\_PATH，将其直接指向你的 **Obsidian Vault 目录** 。
```bash
WIKI="${WIKI_PATH:-$HOME/wiki}"
```
- **录入资料** ：/llm-wiki ingest ~/ob-vault/raw/paper.pdf
- **查询知识** ：/llm-wiki query Cursor 和 OpenClaw 的对比是什么？
- **自动体检** ：/llm-wiki lint

也可以使用自然语言向智能体发送指令。

目录结构：

```markdown
wiki/
├── SCHEMA.md           # 规范、结构规则、领域配置
├── index.md            # 分块的内容目录，带有单行摘要
├── log.md              # 按时间排序的操作日志（仅追加，每年轮转）
├── raw/                # 第一层：不可变的原始素材
│   ├── articles/       # 网页文章、剪报
│   ├── papers/         # PDF文件、arXiv论文
│   ├── transcripts/    # 会议记录、访谈逐字稿
│   └── assets/         # 原始素材引用的图片、图表
├── entities/           # 第二层：实体页面（人物、组织、产品、模型）
├── concepts/           # 第二层：概念/主题页面
├── comparisons/        # 第二层：横向对比分析页面
└── queries/            # 第二层：值得保留的查询结果归档
```

#### 目前免费AI模型供应商

1. Google AI Studio 提供的免费额度，其中Gemma 4 26b/31b比较合适，每分钟15次，每天总请求次数1500次。（26b和31b两个加一起就是30次/3000次）
- **Model Name (模型 ID)**: gemma-4-31b-it
- **API URL**: [https://generativelanguage.googleapis.com/v1beta/models/gemma-4-31b-it:generateContent](https://generativelanguage.googleapis.com/v1beta/models/gemma-4-31b-it:generateContent)

测试：

```bash
curl "https://generativelanguage.googleapis.com/v1beta/models/gemma-4-31b-it:generateContent" \
  -H "x-goog-api-key: $你的apikey" \
  -H 'Content-Type: application/json' \
  -X POST \
  -d '{
    "contents": [{
      "parts": [{"text": "请简述Gemma 4相比Gemma 3的核心提升。"}]
    }],
    "generationConfig": {
      "temperature": 0.7,
      "maxOutputTokens": 2048
    }
  }'
```
1. [英伟达AI服务](https://build.nvidia.com/models) ，目前提供MiniMax M2.7免费使用，每分钟40次。还有其他免费模型。
```markdown
URL: https://integrate.api.nvidia.com/v1
Model: "minimaxai/minimax-m2.7"
```
1. [OpenRouter](https://openrouter.ai/) ，一直有免费模型可用。