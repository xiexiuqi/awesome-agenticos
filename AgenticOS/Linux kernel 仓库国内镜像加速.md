# Linux / Linux Kernel 仓库国内镜像加速

国内访问 kernel.org 官方 Git 仓库速度较慢，以下汇总了常用的国内镜像源。

---

## Linux Kernel 主仓库 (torvalds/linux)

| 镜像源 | 地址 | 推荐 |
|--------|------|------|
| 官方 | `https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git` | |
| 中科大 | `https://mirrors.ustc.edu.cn/linux.git` | ⭐ 推荐，速度快 |
| 清华大学 | `https://mirrors.tuna.tsinghua.edu.cn/git/linux.git` | |

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
| 清华大学 | `https://mirrors.tuna.tsinghua.edu.cn/git/linux-stable.git` |

---

## Linux Next（集成测试分支）

| 镜像源 | 地址 |
|--------|------|
| 官方 | `https://git.kernel.org/pub/scm/linux/kernel/git/next/linux-next.git` |
| 清华大学 | `https://mirrors.tuna.tsinghua.edu.cn/git/linux-next.git` |

---

## 配置 Git 全局镜像替换

可通过 Git 配置自动替换官方地址为镜像地址：

```bash
# 中科大镜像（仅主仓库）
git config --global url."https://mirrors.ustc.edu.cn/".insteadOf "https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/"

# 清华大学镜像（覆盖更多仓库）
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
