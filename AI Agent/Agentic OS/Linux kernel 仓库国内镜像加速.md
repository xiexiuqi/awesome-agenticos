# Linux / Linux Kernel 仓库国内镜像加速

国内访问 kernel.org 官方 Git 仓库速度较慢，以下汇总了常用的国内镜像源，可直接替换原地址使用。

---

## Linux Kernel 主仓库

| 镜像源 | 地址 |
|--------|------|
| 官方 | `https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git` |
| 中科大 | `https://mirrors.ustc.edu.cn/linux.git` |
| 清华大学 | `https://mirrors.tuna.tsinghua.edu.cn/git/linux.git` |
| 阿里云 | `https://mirrors.aliyun.com/linux-kernel/linux.git` |

### 使用示例

```bash
# 克隆主仓库（中科大镜像）
git clone https://mirrors.ustc.edu.cn/linux.git

# 或添加为远程仓库
git remote add ustc https://mirrors.ustc.edu.cn/linux.git
```

---

## Linux Stable 稳定版仓库

| 镜像源 | 地址 |
|--------|------|
| 官方 | `https://git.kernel.org/pub/scm/linux/kernel/git/stable/linux-stable.git` |
| 中科大 | `https://mirrors.ustc.edu.cn/linux-stable.git` |
| 清华大学 | `https://mirrors.tuna.tsinghua.edu.cn/git/linux-stable.git` |

---

## 其他常用 Kernel 相关仓库

### Linux Next（集成测试分支）

| 镜像源 | 地址 |
|--------|------|
| 官方 | `https://git.kernel.org/pub/scm/linux/kernel/git/next/linux-next.git` |
| 中科大 | `https://mirrors.ustc.edu.cn/linux-next.git` |

### BPF 工具仓库 (libbpf)

| 镜像源 | 地址 |
|--------|------|
| GitHub 官方 | `https://github.com/libbpf/libbpf.git` |
| GitHub 镜像（中科大） | `https://github.com/mirrors.ustc.edu.cn/libbpf/libbpf.git` |
| GitHub 镜像（清华） | `https://mirrors.tuna.tsinghua.edu.cn/github/libbpf/libbpf.git` |

---

## GitHub 通用镜像加速

对于 GitHub 上的其他内核相关项目，可使用以下通用镜像：

| 镜像源 | 替换规则 |
|--------|---------|
| 中科大 | `https://github.com/mirrors.ustc.edu.cn/<user>/<repo>.git` |
| 清华大学 | `https://mirrors.tuna.tsinghua.edu.cn/github/<user>/<repo>.git` |
| 阿里云 | `https://mirrors.aliyun.com/github/<user>/<repo>.git` |
| fastgit | `https://hub.fastgit.xyz/<user>/<repo>.git` |

### 使用示例

```bash
# 原始地址
git clone https://github.com/torvalds/linux.git

# 替换为中科大镜像
git clone https://github.com/mirrors.ustc.edu.cn/torvalds/linux.git

# 或替换为清华大学镜像
git clone https://mirrors.tuna.tsinghua.edu.cn/github/torvalds/linux.git
```

---

## 配置 Git 全局镜像替换

可通过 Git 配置自动替换官方地址为镜像地址：

```bash
# 中科大镜像
git config --global url."https://mirrors.ustc.edu.cn/".insteadOf "https://git.kernel.org/pub/scm/linux/kernel/git/"

# 清华大学镜像
git config --global url."https://mirrors.tuna.tsinghua.edu.cn/git/".insteadOf "https://git.kernel.org/pub/scm/linux/kernel/git/"
```

查看已配置的替换规则：

```bash
git config --global --get-regexp url.*.insteadOf
```

---

## 镜像源状态检查

| 镜像源 | 状态页面 |
|--------|---------|
| 中科大 | https://mirrors.ustc.edu.cn/status |
| 清华大学 | https://mirrors.tuna.tsinghua.edu.cn/status |
| 阿里云 | https://mirrors.aliyun.com/status |

---

## 相关资源

- [[SWE-bench]] — 软件工程基准测试，涉及 Linux 内核代码分析
- [[AgentCgroup Understanding and Controlling OS Resources of AI Agents]] — 基于 eBPF 的 OS 资源控制，与 Linux Kernel 密切相关
