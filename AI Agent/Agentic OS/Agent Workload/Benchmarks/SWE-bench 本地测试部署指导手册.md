#SWEBench #AgentticOS #Benchmark
## 一、环境要求

### 1.1 硬件要求

| 资源 | 最低要求 | 推荐配置 |
|------|---------|---------|
| CPU | 4 核 | 8 核及以上 |
| 内存 | 8 GB | 16 GB 及以上 |
| 磁盘空间 | 50 GB | 100 GB 及以上（完整仓库较大） |
| 网络 | 可访问互联网 | 稳定访问 GitHub/HuggingFace |

### 1.2 软件要求

- **操作系统**: Linux (Ubuntu 20.04+ 推荐) 或 WSL2
- **Python**: 3.10 及以上
- **Docker**: 20.10 及以上，需支持 buildx
- **Git**: 2.30 及以上

### 1.3 国内环境特殊说明

在中国大陆环境下，由于网络限制，访问 GitHub、HuggingFace、PyPI、Anaconda 等源存在困难。本手册针对国内环境提供了完整的解决方案。

---

## 二、基础环境安装

### 2.1 安装 Docker

```bash
# Ubuntu/Debian
sudo apt update
sudo apt install -y docker.io docker-compose

# 将当前用户加入 docker 组（免 sudo 使用 docker）
sudo usermod -aG docker $USER
newgrp docker

# 验证安装
docker --version
```

### 2.2 配置 Docker 国内镜像

创建或编辑 Docker 配置文件：

```bash
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json << 'EOF'
{
  "registry-mirrors": [
    "https://registry.cn-hangzhou.aliyuncs.com",
    "https://ccr.ccs.tencentyun.com",
    "https://swr.cn-north-4.myhuaweicloud.com"
  ]
}
EOF

# 重启 Docker 服务
sudo systemctl daemon-reload
sudo systemctl restart docker

# 验证镜像配置
sudo docker info | grep -A 5 "Registry Mirrors"
```

### 2.3 安装 Python 和基础工具

```bash
# Ubuntu/Debian
sudo apt update
sudo apt install -y python3 python3-venv python3-pip git curl wget

# 验证版本
python3 --version  # 需 >= 3.10
pip3 --version
```

---

## 三、SWE-bench 部署

### 3.1 克隆仓库

```bash
# 创建工作目录
mkdir -p ~/workspace/swebench
cd ~/workspace/swebench

# 克隆 SWE-bench 仓库
git clone https://github.com/princeton-nlp/SWE-bench.git
cd SWE-bench
```

**注意**: 如果 GitHub 访问不稳定，可以尝试以下方法：
- 使用代理工具（Clash、V2Ray 等）并配置 git 代理
- 使用 GitHub 镜像站（如 ghproxy.com，但稳定性不保证）
- 分时段多次尝试

### 3.2 创建 Python 虚拟环境

```bash
# 创建虚拟环境
python3 -m venv venv

# 激活环境
source venv/bin/activate

# 验证已在虚拟环境中
which python  # 应显示 venv 路径
```

### 3.3 安装 swebench 包

```bash
# 使用清华 PyPI 镜像安装（国内必需）
pip install --index-url https://pypi.tuna.tsinghua.edu.cn/simple -e .

# 验证安装
python -c "import swebench; print(swebench.__version__)"
```

### 3.4 配置 HuggingFace 镜像（**必须设置 Token**）

```bash
# 设置环境变量（使用国内 hf-mirror）
export HF_ENDPOINT=https://hf-mirror.com

# HuggingFace Token（必须设置，否则会被限速到基本不可用）
export HF_TOKEN="your_token_here"
```

**⚠️ 重要：HF_TOKEN 必须设置**

- 不设置 Token 时，HuggingFace（包括 hf-mirror）会施加严格的速率限制
- 表现为数据集加载极慢（几十 KB/s）或频繁超时/断开
- 基本无法完成 300 条记录的 SWE-bench_Lite 数据集下载
- Token 可在 https://huggingface.co/settings/tokens 免费申请

建议将上述环境变量添加到 `~/.bashrc` 或 `~/.zshrc`：

```bash
echo 'export HF_ENDPOINT=https://hf-mirror.com' >> ~/.bashrc
echo 'export HF_TOKEN="your_token_here"' >> ~/.bashrc
source ~/.bashrc
```

**验证 Token 是否生效**：

```bash
# 方法 1：快速验证
python3 -c "
import os
from datasets import load_dataset

if not os.environ.get('HF_TOKEN'):
    print('❌ 未设置 HF_TOKEN')
    exit(1)

# 尝试加载数据集（streaming 模式）
ds = load_dataset('princeton-nlp/SWE-bench_Lite', split='test', streaming=True)
for i, item in enumerate(ds):
    if i >= 3:
        break
print('✓ Token 有效，数据集可正常访问')
"

# 方法 2：使用附录 D 的完整验证脚本
bash verify_hf_token.sh
```

---

## 四、关键修改（适配国内环境）

由于国内网络环境的特殊性，需要对 SWE-bench 源码进行以下修改。这些修改解决 Docker 容器内网络不通、conda/pip 下载超时、git clone 失败等问题。

### 4.1 修改 1：Docker 使用 Host 网络模式

**修改文件**: `swebench/harness/docker_build.py`

**修改原因**: Docker 容器默认使用 bridge 网络，在国内环境下无法稳定访问外部网络（GitHub、conda 源等）。使用 host 网络模式让容器共享宿主机的网络栈。

**修改位置 1** - 在 `build_image` 函数中（约第 127 行）：

找到以下代码：

```python
response = client.api.build(
    path=str(build_dir),
    tag=image_name,
    rm=True,
    forcerm=True,
    decode=True,
    platform=platform,
    nocache=nocache,
)
```

修改为：

```python
response = client.api.build(
    path=str(build_dir),
    tag=image_name,
    rm=True,
    forcerm=True,
    decode=True,
    platform=platform,
    nocache=nocache,
    network_mode="host",  # 添加 host 网络模式
)
```

**修改位置 2** - 在 `build_container` 函数中（约第 517 行）：

找到以下代码：

```python
container = client.containers.create(
    image=test_spec.instance_image_key,
    name=test_spec.get_instance_container_name(run_id),
    user=DOCKER_USER,
    detach=True,
    command="tail -f /dev/null",
    platform=test_spec.platform,
    cap_add=cap_add,
)
```

修改为：

```python
container = client.containers.create(
    image=test_spec.instance_image_key,
    name=test_spec.get_instance_container_name(run_id),
    user=DOCKER_USER,
    detach=True,
    command="tail -f /dev/null",
    platform=test_spec.platform,
    cap_add=cap_add,
    network_mode="host",  # 添加 host 网络模式
)
```

### 4.2 修改 2：conda 使用清华镜像

**修改文件**: `swebench/harness/test_spec/python.py`

**修改原因**: conda 默认使用 `repo.anaconda.com`，国内访问超时。需要配置清华 Anaconda 镜像。

**修改位置 1** - 在 `make_env_script_list_py_from_conda` 函数中（约第 324 行）：

找到以下代码：

```python
reqs_commands = [
    "source /opt/miniconda3/bin/activate",
    f"cat <<'{HEREDOC_DELIMITER}' > /root/environment.yml\n{cached_environment_yml}\n{HEREDOC_DELIMITER}",
    "conda env create -f /root/environment.yml",
    f"conda activate {env_name}",
]
```

修改为：

```python
reqs_commands = [
    "source /opt/miniconda3/bin/activate",
    # 配置 conda 使用清华镜像
    "conda config --remove channels defaults || true",
    "conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main/ || true",
    "conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/ || true",
    "conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/conda-forge/ || true",
    "conda config --set show_channel_urls yes || true",
    "conda config --set channel_priority strict || true",
    # 配置 pip 使用清华镜像
    "export PIP_INDEX_URL=https://pypi.tuna.tsinghua.edu.cn/simple",
    "export PIP_TRUSTED_HOST=pypi.tuna.tsinghua.edu.cn",
    f"cat <<'{HEREDOC_DELIMITER}' > /root/environment.yml\n{cached_environment_yml}\n{HEREDOC_DELIMITER}",
    "conda env create -f /root/environment.yml",
    f"conda activate {env_name}",
]
```

**修改位置 2** - 在 `make_env_script_list_py` 函数中（约第 350 行）：

找到以下代码：

```python
reqs_commands = [
    "source /opt/miniconda3/bin/activate",
]
```

修改为：

```python
reqs_commands = [
    "source /opt/miniconda3/bin/activate",
    # 配置 conda 使用清华镜像
    "conda config --remove channels defaults || true",
    "conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main/ || true",
    "conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/ || true",
    "conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/conda-forge/ || true",
    "conda config --set show_channel_urls yes || true",
    "conda config --set channel_priority strict || true",
    # 配置 pip 使用清华镜像
    "export PIP_INDEX_URL=https://pypi.tuna.tsinghua.edu.cn/simple",
    "export PIP_TRUSTED_HOST=pypi.tuna.tsinghua.edu.cn",
]
```

### 4.3 修改 3：预下载仓库到本地

**修改原因**: Docker 容器内 `git clone` GitHub 反复超时。解决方案是在宿主机上预先下载完整仓库，然后在 Docker build 时从本地复制。

#### 步骤 1：预下载仓库

在宿主机上创建目录并克隆仓库：

```bash
# 创建仓库目录
mkdir -p ~/workspace/swebench/repos
cd ~/workspace/swebench/repos

# 完整克隆仓库（不要使用 --depth 1，需要完整历史）
git clone https://github.com/sympy/sympy.git
git clone https://github.com/django/django.git
git clone https://github.com/matplotlib/matplotlib.git
git clone https://github.com/scikit-learn/scikit-learn.git
git clone https://github.com/pytest-dev/pytest.git

# 验证仓库大小
du -sh */
```

**重要**: 必须使用完整克隆（不带 `--depth 1`），因为 SWE-bench 需要在特定的历史 commit 上运行测试。

#### 步骤 2：修改 Dockerfile 复制本地仓库

**修改文件**: `swebench/harness/dockerfiles/python.py`

找到以下代码（约第 48 行）：

```python
_DOCKERFILE_INSTANCE_PY = r"""FROM --platform={platform} {env_image_name}

COPY ./setup_repo.sh /root/
RUN sed -i -e 's/\r$//' /root/setup_repo.sh
RUN /bin/bash /root/setup_repo.sh

WORKDIR /testbed/
"""
```

修改为：

```python
_DOCKERFILE_INSTANCE_PY = r"""FROM --platform={platform} {env_image_name}

# 从宿主机复制预下载的仓库
COPY ./repo /testbed
COPY ./setup_repo.sh /root/
RUN sed -i -e 's/\r$//' /root/setup_repo.sh
RUN /bin/bash /root/setup_repo.sh

WORKDIR /testbed/
"""
```

#### 步骤 3：修改 build_instance_image 复制仓库到 build context

**修改文件**: `swebench/harness/docker_build.py`

在 `build_instance_image` 函数中（约第 452 行），在 `build_image(...)` 调用之前添加以下代码：

```python
# 复制预下载的仓库到 build context
import shutil
repo_name = test_spec.repo.split("/")[-1]
local_repo_path = f"{os.path.expanduser('~')}/workspace/swebench/repos/{repo_name}"
build_repo_path = build_dir / "repo"
if build_repo_path.exists():
    shutil.rmtree(build_repo_path)
if Path(local_repo_path).exists():
    shutil.copytree(local_repo_path, build_repo_path)
    logger.info(f"Copied local repo {local_repo_path} to build context")
else:
    logger.warning(f"Local repo not found at {local_repo_path}, will rely on git clone in setup script")
```

**注意**: 请根据实际路径修改 `local_repo_path`。上述示例使用 `~/workspace/swebench/repos/`，如果你的路径不同，请相应调整。

#### 步骤 4：修改 setup 脚本使用本地仓库

**修改文件**: `swebench/harness/test_spec/python.py`

在 `make_repo_script_list_py` 函数中（约第 273 行），找到 setup_commands 列表：

将原来的：

```python
setup_commands = [
    f"git clone -o origin {branch} --single-branch https://github.com/{repo} {repo_directory}",
    f"chmod -R 777 {repo_directory}",
    f"cd {repo_directory}",
    f"git reset --hard {base_commit}",
    # ... 后续命令
]
```

修改为：

```python
setup_commands = [
    # 仓库已从宿主机复制，直接设置
    f"chmod -R 777 {repo_directory}",
    f"cd {repo_directory}",
    "git config --global user.email setup@swebench.config",
    "git config --global user.name SWE-bench",
    # 重置并切换到目标 commit
    "git reset --hard HEAD",
    "git clean -fd",
    f"git checkout {base_commit}",
    # 移除远程，防止 agent 看到新提交
    "git remote remove origin || true",
    # 清理标签和 reflog
    "git tag -l | xargs -r git tag -d || true",
    "git reflog expire --expire=now --all",
    "git gc --prune=now --aggressive",
    # 激活 conda 环境
    "source /opt/miniconda3/bin/activate",
    f"conda activate {env_name}",
    'echo "Current environment: $CONDA_DEFAULT_ENV"',
]
```

---

## 五、准备数据集

### 5.1 下载数据集

```bash
source venv/bin/activate
export HF_ENDPOINT=https://hf-mirror.com
export HF_TOKEN="your_token_here"  # 必须设置！

python3 << 'PYEOF'
import os
from datasets import load_dataset
import json

# 检查 Token 是否设置
if not os.environ.get('HF_TOKEN'):
    print("警告：未设置 HF_TOKEN，下载可能被限速")
    print("申请地址：https://huggingface.co/settings/tokens")

# 加载完整数据集
ds = load_dataset('princeton-nlp/SWE-bench', split='test', streaming=True)

# 收集代表性用例（每个仓库取 2 个）
instances = []
repo_counts = {}
target_repos = ['sympy', 'django', 'matplotlib', 'scikit-learn', 'pytest-dev']

for item in ds:
    repo = item['instance_id'].split('__')[0]
    if repo in target_repos:
        if repo_counts.get(repo, 0) < 2:
            instances.append(dict(item))
            repo_counts[repo] = repo_counts.get(repo, 0) + 1
    if all(repo_counts.get(r, 0) >= 2 for r in target_repos):
        break

# 保存到本地
with open('full_dataset_selected.json', 'w') as f:
    json.dump(instances, f, indent=2, default=str)

print(f"Saved {len(instances)} instances from {len(repo_counts)} repos")
for repo, count in repo_counts.items():
    print(f"  {repo}: {count}")
PYEOF
```

### 5.2 验证数据集

```bash
# 查看保存的用例
python3 -c "
import json
with open('full_dataset_selected.json') as f:
    data = json.load(f)
for inst in data:
    print(f\"{inst['instance_id']} | {inst['repo']} | python={inst.get('version', 'N/A')}\")
"
```

---

## 六、运行评估

### 6.1 运行单个用例

```bash
# 激活环境
source venv/bin/activate
export HF_ENDPOINT=https://hf-mirror.com
export HF_TOKEN="your_token_here"  # 必须设置

# 运行单个用例评估
python -m swebench.harness.run_evaluation \
    --dataset_name ./full_dataset_selected.json \
    --predictions_path gold \
    --max_workers 1 \
    --instance_ids sympy__sympy-11232 \
    --run_id validate-gold-sympy \
    --namespace ''
```

### 6.2 运行多个用例

```bash
export HF_ENDPOINT=https://hf-mirror.com
export HF_TOKEN="your_token_here"  # 必须设置

python -m swebench.harness.run_evaluation \
    --dataset_name ./full_dataset_selected.json \
    --predictions_path gold \
    --max_workers 2 \
    --instance_ids sympy__sympy-11232 django__django-10087 \
    --run_id validate-gold-batch \
    --namespace ''
```

### 6.3 参数说明

| 参数 | 说明 | 示例 |
|------|------|------|
| `--dataset_name` | 数据集路径 | `./full_dataset_selected.json` |
| `--predictions_path` | 预测文件路径或 `gold` | `gold` |
| `--max_workers` | 并行 worker 数量 | `1` 或 `2` |
| `--instance_ids` | 指定用例 ID（空格分隔） | `sympy__sympy-11232` |
| `--run_id` | 运行标识符 | `validate-gold-sympy` |
| `--namespace` | 镜像命名空间，`''` 表示本地构建 | `''` |

### 6.4 查看结果

```bash
# 查看生成的报告
cat gold.<run_id>.json | python3 -m json.tool

# 示例输出解读
{
    "total_instances": 1,
    "completed_instances": 1,    // 成功完成
    "resolved_instances": 1,     // 成功修复（gold patch 通过测试）
    "error_instances": 0,        // 出错数量
    "resolved_ids": ["sympy__sympy-11232"]
}
```

---

## 七、常见问题及解决方案

### 问题 1：GitHub 克隆超时

**现象**:
```
fatal: unable to access 'https://github.com/.../': 
Failed to connect to github.com port 443: Connection timed out
```

**解决方案**:
1. 使用代理工具（推荐）
2. 预下载仓库到本地（本手册方案）
3. 分时段多次尝试

### 问题 2：HuggingFace 数据集加载失败/极慢

**现象**:
```
httpx.ReadTimeout: The read operation timed out
# 或下载速度极慢（<100KB/s），长时间无进度
```

**原因**: 未设置 `HF_TOKEN`，被 HuggingFace 限速

**解决方案**:
```bash
# 1. 申请 Token（免费）
# 访问 https://huggingface.co/settings/tokens 申请

# 2. 设置环境变量
export HF_ENDPOINT=https://hf-mirror.com
export HF_TOKEN="your_token_here"

# 3. 验证 Token 是否生效
python3 -c "
import os
from datasets import load_dataset
print('Token set:', bool(os.environ.get('HF_TOKEN')))
ds = load_dataset('princeton-nlp/SWE-bench_Lite', split='test', streaming=True)
print('Dataset loaded successfully')
"
```

### 问题 3：pip 安装超时

**现象**:
```
ERROR: Could not find a version that satisfies the requirement xxx
```

**解决方案**:
```bash
pip install --index-url https://pypi.tuna.tsinghua.edu.cn/simple -e .
```

### 问题 4：conda 创建环境失败

**现象**:
```
CondaHTTPError: HTTP 000 CONNECTION FAILED for url <https://repo.anaconda.com/...>
```

**解决方案**:
按照 4.2 节修改 `python.py`，配置清华 Anaconda 镜像。

### 问题 5：Docker 容器内网络不通

**现象**:
```
Network is unreachable
```

**解决方案**:
按照 4.1 节修改 `docker_build.py`，使用 `network_mode="host"`。

### 问题 6：git checkout 失败（本地修改冲突）

**现象**:
```
error: Your local changes to the following files would be overwritten by checkout
```

**解决方案**:
按照 4.3 节修改 `python.py`，在 `git checkout` 前执行 `git reset --hard HEAD && git clean -fd`。

### 问题 7：环境镜像构建成功但实例镜像失败

**现象**:
```
Error building image sweb.eval.x86_64.xxx:latest
```

**排查步骤**:
1. 检查日志：`logs/build_images/instances/.../build_image.log`
2. 确认本地仓库路径正确
3. 确认仓库是完整克隆（不是 `--depth 1`）
4. 确认目标 commit 存在于本地仓库中

---

## 八、维护命令

### 8.1 清理 Docker 镜像

```bash
# 查看所有 swebench 相关镜像
docker images | grep sweb

# 删除所有 swebench 镜像
docker images | grep sweb | awk '{print $1":"$2}' | xargs -r docker rmi -f

# 清理所有未使用的 Docker 资源
docker system prune -f
```

### 8.2 查看构建日志

```bash
# 环境镜像构建日志
ls logs/build_images/env/
cat logs/build_images/env/<image_name>/build_image.log

# 实例镜像构建日志
ls logs/build_images/instances/
cat logs/build_images/instances/<image_name>/build_image.log

# 运行实例日志
ls logs/run_evaluation/
cat logs/run_evaluation/<run_id>/gold/<instance_id>/run_instance.log
```

### 8.3 验证仓库完整性

```bash
cd ~/workspace/swebench/repos/sympy
git log --oneline -5  # 查看最新 5 个 commit
git cat-file -t <commit_hash>  # 验证特定 commit 是否存在
```

---

## 九、典型用例运行时间参考

| 阶段 | 耗时 | 说明 |
|------|------|------|
| Base 镜像构建 | ~1 分钟 | 首次运行，后续复用 |
| Env 镜像构建 | ~5-7 分钟 | conda 环境 + 依赖安装 |
| Instance 镜像构建 | ~1-2 分钟 | 复制仓库 + checkout commit |
| 测试执行 | ~1-2 分钟 | 应用 patch + 运行测试 |
| **总计** | **~8-12 分钟/用例** | 首次运行 |

**注意**: 并行运行多个用例时，总时间取决于最慢的用例。建议 `max_workers` 不超过 CPU 核心数的 75%。

---

## 十、修改文件清单

| 文件 | 修改内容 | 行数变化 |
|------|---------|---------|
| `swebench/harness/docker_build.py` | Docker host 网络 + 本地仓库复制 | +14 行 |
| `swebench/harness/dockerfiles/python.py` | Dockerfile COPY 本地仓库 | +2 行 |
| `swebench/harness/test_spec/python.py` | conda/pip 镜像 + git 脚本 | +33/-11 行 |

---

## 十一、附录

### A. 完整运行脚本示例

```bash
#!/bin/bash
set -e

# 配置环境（HF_TOKEN 必须设置）
export HF_ENDPOINT=https://hf-mirror.com
export HF_TOKEN="your_token_here"  # 必须！否则会被限速到基本不可用

# 验证 Token 已设置
if [ -z "$HF_TOKEN" ] || [ "$HF_TOKEN" = "your_token_here" ]; then
    echo "错误：请设置有效的 HF_TOKEN"
    echo "申请地址：https://huggingface.co/settings/tokens"
    exit 1
fi

source venv/bin/activate

# 运行评估
python -m swebench.harness.run_evaluation \
    --dataset_name ./full_dataset_selected.json \
    --predictions_path gold \
    --max_workers 2 \
    --instance_ids sympy__sympy-11232 django__django-10087 \
    --run_id my-evaluation-run \
    --namespace ''

# 查看结果
echo "=== Evaluation Results ==="
cat gold.my-evaluation-run.json | python3 -m json.tool
```

### B. 预下载所有仓库脚本

```bash
#!/bin/bash
set -e

REPOS_DIR="$HOME/workspace/swebench/repos"
mkdir -p "$REPOS_DIR"
cd "$REPOS_DIR"

repos=(
    "https://github.com/sympy/sympy.git"
    "https://github.com/django/django.git"
    "https://github.com/matplotlib/matplotlib.git"
    "https://github.com/scikit-learn/scikit-learn.git"
    "https://github.com/pytest-dev/pytest.git"
)

for url in "${repos[@]}"; do
    name=$(basename "$url" .git)
    if [ -d "$name" ]; then
        echo "Updating $name..."
        cd "$name" && git pull && cd ..
    else
        echo "Cloning $name..."
        git clone "$url" "$name"
    fi
done

echo "All repos ready!"
ls -la
```

### C. 网络诊断命令

```bash
# 测试 GitHub 连通性
curl -sI --connect-timeout 10 https://github.com

# 测试 HuggingFace 连通性
curl -sI --connect-timeout 10 https://hf-mirror.com

# 测试 Docker 网络（从容器内）
docker run --rm --network host alpine ping -c 3 github.com

# 测试 conda 镜像
curl -sI https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main/linux-64/repodata.json
```

### D. HuggingFace Token 验证脚本

```bash
#!/bin/bash
# verify_hf_token.sh - 验证 HF_TOKEN 是否有效

set -e

export HF_ENDPOINT=https://hf-mirror.com

# 检查 Token 是否设置
if [ -z "$HF_TOKEN" ]; then
    echo "❌ 错误：未设置 HF_TOKEN 环境变量"
    echo ""
    echo "请执行以下步骤："
    echo "1. 访问 https://huggingface.co/settings/tokens 申请 Token"
    echo "2. 设置环境变量：export HF_TOKEN='your_token_here'"
    echo "3. 建议添加到 ~/.bashrc 使其持久化"
    exit 1
fi

if [ "$HF_TOKEN" = "your_token_here" ]; then
    echo "❌ 错误：HF_TOKEN 未替换为实际值"
    exit 1
fi

echo "✓ HF_TOKEN 已设置"
echo "Token 前缀: ${HF_TOKEN:0:8}..."

# 使用 Python 验证 Token 有效性
python3 -c "
import os
from huggingface_hub import HfApi

token = os.environ.get('HF_TOKEN')
print('Token length:', len(token))

try:
    api = HfApi(token=token)
    user_info = api.whoami()
    print(f'✓ Token 有效，用户: {user_info}')
except Exception as e:
    print(f'✗ Token 验证失败: {e}')
    exit(1)
"

# 测试数据集下载速度
echo ""
echo "测试数据集下载速度..."
python3 -c "
import time
import os
from datasets import load_dataset

start = time.time()
try:
    ds = load_dataset('princeton-nlp/SWE-bench_Lite', split='test', streaming=True)
    # 读取前 5 条记录
    for i, item in enumerate(ds):
        if i >= 5:
            break
    elapsed = time.time() - start
    print(f'✓ 数据集加载成功，5条记录耗时: {elapsed:.2f}s')
    if elapsed > 30:
        print('⚠ 警告：加载较慢，Token 可能未生效或网络不稳定')
except Exception as e:
    print(f'✗ 数据集加载失败: {e}')
    exit(1)
"

echo ""
echo "✓ HuggingFace 环境验证通过"
```

### E. 一键环境检查脚本

```bash
#!/bin/bash
# check_env.sh - 一键检查所有环境配置

echo "=== SWE-bench 环境检查 ==="
echo ""

# 检查 Python
python3 --version 2>/dev/null || { echo "❌ Python3 未安装"; exit 1; }

# 检查 Docker
docker --version 2>/dev/null || { echo "❌ Docker 未安装"; exit 1; }
docker info >/dev/null 2>&1 || { echo "❌ Docker 服务未运行"; exit 1; }

# 检查虚拟环境
if [ -d "venv" ]; then
    echo "✓ 虚拟环境存在"
else
    echo "⚠ 虚拟环境不存在，请执行: python3 -m venv venv"
fi

# 检查 HF_TOKEN
if [ -z "$HF_TOKEN" ]; then
    echo "❌ HF_TOKEN 未设置（必须设置！）"
    echo "   申请地址: https://huggingface.co/settings/tokens"
else
    echo "✓ HF_TOKEN 已设置"
fi

# 检查 HF_ENDPOINT
if [ "$HF_ENDPOINT" = "https://hf-mirror.com" ]; then
    echo "✓ HF_ENDPOINT 已设置为国内镜像"
else
    echo "⚠ HF_ENDPOINT 未设置，建议: export HF_ENDPOINT=https://hf-mirror.com"
fi

# 检查 swebench 包
python3 -c "import swebench; print(f'✓ swebench 版本: {swebench.__version__}')" 2>/dev/null || echo "❌ swebench 未安装"

# 检查 datasets 包
python3 -c "import datasets; print(f'✓ datasets 版本: {datasets.__version__}')" 2>/dev/null || echo "❌ datasets 未安装"

echo ""
echo "=== 检查完成 ==="
```

---

## 十一、附录

### A. 完整运行脚本示例

```bash
#!/bin/bash
set -e

# 配置环境（HF_TOKEN 必须设置）
export HF_ENDPOINT=https://hf-mirror.com
export HF_TOKEN="your_token_here"  # 必须！否则会被限速到基本不可用

# 验证 Token 已设置
if [ -z "$HF_TOKEN" ] || [ "$HF_TOKEN" = "your_token_here" ]; then
    echo "错误：请设置有效的 HF_TOKEN"
    echo "申请地址：https://huggingface.co/settings/tokens"
    exit 1
fi

source venv/bin/activate

# 运行评估
python -m swebench.harness.run_evaluation \
    --dataset_name ./full_dataset_selected.json \
    --predictions_path gold \
    --max_workers 2 \
    --instance_ids sympy__sympy-11232 django__django-10087 \
    --run_id my-evaluation-run \
    --namespace ''

# 查看结果
echo "=== Evaluation Results ==="
cat gold.my-evaluation-run.json | python3 -m json.tool
```

### B. 预下载所有仓库脚本

```bash
#!/bin/bash
set -e

REPOS_DIR="$HOME/workspace/swebench/repos"
mkdir -p "$REPOS_DIR"
cd "$REPOS_DIR"

repos=(
    "https://github.com/sympy/sympy.git"
    "https://github.com/django/django.git"
    "https://github.com/matplotlib/matplotlib.git"
    "https://github.com/scikit-learn/scikit-learn.git"
    "https://github.com/pytest-dev/pytest.git"
)

for url in "${repos[@]}"; do
    name=$(basename "$url" .git)
    if [ -d "$name" ]; then
        echo "Updating $name..."
        cd "$name" && git pull && cd ..
    else
        echo "Cloning $name..."
        git clone "$url" "$name"
    fi
done

echo "All repos ready!"
ls -la
```

### C. 网络诊断命令

```bash
# 测试 GitHub 连通性
curl -sI --connect-timeout 10 https://github.com

# 测试 HuggingFace 连通性
curl -sI --connect-timeout 10 https://hf-mirror.com

# 测试 Docker 网络（从容器内）
docker run --rm --network host alpine ping -c 3 github.com

# 测试 conda 镜像
curl -sI https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main/linux-64/repodata.json
```

### D. HuggingFace Token 验证脚本

```bash
#!/bin/bash
# verify_hf_token.sh - 验证 HF_TOKEN 是否有效

set -e

export HF_ENDPOINT=https://hf-mirror.com

# 检查 Token 是否设置
if [ -z "$HF_TOKEN" ]; then
    echo "❌ 错误：未设置 HF_TOKEN 环境变量"
    echo ""
    echo "请执行以下步骤："
    echo "1. 访问 https://huggingface.co/settings/tokens 申请 Token"
    echo "2. 设置环境变量：export HF_TOKEN='your_token_here'"
    echo "3. 建议添加到 ~/.bashrc 使其持久化"
    exit 1
fi

if [ "$HF_TOKEN" = "your_token_here" ]; then
    echo "❌ 错误：HF_TOKEN 未替换为实际值"
    exit 1
fi

echo "✓ HF_TOKEN 已设置"
echo "Token 前缀: ${HF_TOKEN:0:8}..."

# 使用 Python 验证 Token 有效性
python3 -c "
import os
from huggingface_hub import HfApi

token = os.environ.get('HF_TOKEN')
print('Token length:', len(token))

try:
    api = HfApi(token=token)
    user_info = api.whoami()
    print(f'✓ Token 有效，用户: {user_info}')
except Exception as e:
    print(f'✗ Token 验证失败: {e}')
    exit(1)
"

# 测试数据集下载速度
echo ""
echo "测试数据集下载速度..."
python3 -c "
import time
import os
from datasets import load_dataset

start = time.time()
try:
    ds = load_dataset('princeton-nlp/SWE-bench_Lite', split='test', streaming=True)
    # 读取前 5 条记录
    for i, item in enumerate(ds):
        if i >= 5:
            break
    elapsed = time.time() - start
    print(f'✓ 数据集加载成功，5条记录耗时: {elapsed:.2f}s')
    if elapsed > 30:
        print('⚠ 警告：加载较慢，Token 可能未生效或网络不稳定')
except Exception as e:
    print(f'✗ 数据集加载失败: {e}')
    exit(1)
"

echo ""
echo "✓ HuggingFace 环境验证通过"
```

### E. 一键环境检查脚本

```bash
#!/bin/bash
# check_env.sh - 一键检查所有环境配置

echo "=== SWE-bench 环境检查 ==="
echo ""

# 检查 Python
python3 --version 2>/dev/null || { echo "❌ Python3 未安装"; exit 1; }

# 检查 Docker
docker --version 2>/dev/null || { echo "❌ Docker 未安装"; exit 1; }
docker info >/dev/null 2>&1 || { echo "❌ Docker 服务未运行"; exit 1; }

# 检查虚拟环境
if [ -d "venv" ]; then
    echo "✓ 虚拟环境存在"
else
    echo "⚠ 虚拟环境不存在，请执行: python3 -m venv venv"
fi

# 检查 HF_TOKEN
if [ -z "$HF_TOKEN" ]; then
    echo "❌ HF_TOKEN 未设置（必须设置！）"
    echo "   申请地址: https://huggingface.co/settings/tokens"
else
    echo "✓ HF_TOKEN 已设置"
fi

# 检查 HF_ENDPOINT
if [ "$HF_ENDPOINT" = "https://hf-mirror.com" ]; then
    echo "✓ HF_ENDPOINT 已设置为国内镜像"
else
    echo "⚠ HF_ENDPOINT 未设置，建议: export HF_ENDPOINT=https://hf-mirror.com"
fi

# 检查 swebench 包
python3 -c "import swebench; print(f'✓ swebench 版本: {swebench.__version__}')" 2>/dev/null || echo "❌ swebench 未安装"

# 检查 datasets 包
python3 -c "import datasets; print(f'✓ datasets 版本: {datasets.__version__}')" 2>/dev/null || echo "❌ datasets 未安装"

echo ""
echo "=== 检查完成 ==="
```