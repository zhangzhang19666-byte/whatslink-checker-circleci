# ed2k Link Checker — CircleCI 部署版

> 从 whatslink.info 校验大量 ed2k 链接的有效性，支持断点续传和自动重试。
> **公共仓库存代码 | 私有仓库存数据 | CircleCI 自动运行**

---

## 架构概览

```
┌─────────────────┐       ┌──────────────────────┐       ┌─────────────────┐
│  公共仓库        │       │  CircleCI             │       │  私有数据仓库   │
│  (代码)          │       │  (运行环境)            │       │  (数据)         │
│                 │       │                       │       │                 │
│  check.py       │──────→│  1. checkout 公共仓库  │       │  data/*.txt     │
│  .circleci/*    │       │  2. clone 私有仓库     │←──────│  work/          │
│  requirements   │       │  3. 运行 check.py      │       │                 │
│                 │       │  4. push work/ 回私有   │──────→│                 │
│                 │       │  5. store_artifacts    │       │                 │
│                 │       │  6. auto-trigger 续跑   │       │                 │
└─────────────────┘       └──────────────────────┘       └─────────────────┘
```

## 你需要准备 3 样东西

### ① 私有数据仓库

已创建: **`zhangzhang19666-byte/ed2k-private-data`**

把 ed2k 链接的 `.txt` 文件放进去:

```bash
git clone https://github.com/zhangzhang19666-byte/ed2k-private-data.git
cd ed2k-private-data
# 把你的 .txt 文件放到这里
# 每行一条 ed2k 链接，支持 # 注释
git add *.txt
git commit -m "add ed2k links"
git push
```

文件格式：
```
ed2k://|file|example.avi|734003200|ABCDEF1234567890ABCDEF1234567890|/
ed2k://|file|movie.mkv|1468006400|1234567890ABCDEF1234567890ABCDEF|/
```

### ② GitHub PAT

用于 CI 读写上面的私有数据仓库。

1. 打开 https://github.com/settings/tokens → **Fine-grained tokens**
2. **Token name**: `circleci-ed2k-checker`
3. **Repository access**: 选 `zhangzhang19666-byte/ed2k-private-data`
4. **Permissions**: `Contents: Read and write`
5. 创建 → 复制 token 值

### ③ CircleCI API Token

用于 auto-trigger（链式续跑）。

1. 打开 https://app.circleci.com/settings/user → **Personal API Tokens**
2. 创建 → 复制 token 值

---

## 设置环境变量

| 变量 | 值 | 位置 |
|------|-----|------|
| `GH_PAT` | GitHub PAT（上一步创建的） | CircleCI Project Settings → Environment Variables |
| `DATA_REPO` | `zhangzhang19666-byte/ed2k-private-data` | 同上 |
| `CIRCLECI_API_TOKEN` | CircleCI API Token | 同上 |

## 部署到 CircleCI

1. 打开 https://app.circleci.com/projects/
2. **Add Project** → 选择本仓库
3. 设置上面 3 个环境变量
4. **Trigger Pipeline** → main 分支

## 查看结果

运行完成后，在 Job 详情页 **Artifacts** 标签:

- `summary.html` → HTML 汇总报告
- `work-results/all_success_ed2k.txt` → 全部有效链接下载

私有仓库的 `work/` 目录也会自动更新，下次运行自动续接。

## 关于安全性

- **公共仓库只有代码**，没有 data/ 和 work/
- **数据在私有仓库**，只有你有权限
- **GH_PAT** 存在 CircleCI 环境变量里，加密存储，别人看不到
- 别人 PR 公共仓库 → 只能改代码，碰不到你的数据

## 本地运行

```bash
pip install -r requirements.txt
# 从私有仓库拉 data/ 和 work/
DATA_DIR=./data WORK_DIR=./work python check.py
python check.py --status
```
