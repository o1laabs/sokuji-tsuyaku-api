# Keeping This Fork in Sync with Upstream

> 面向本 fork(`o1laabs/sokuji-tsuyaku-api`)维护者的同步指南。
> 目标上游有两个,但本 fork 直接继承的是:
> - **直接上游**: `kizuna-ai-lab/sokuji-tsuyaku-api`(即時通訳 API 服务器,实时翻译 fork)
> - **上游的上游**: `speaches-ai/speaches`(本仓库 fork 改造的源,TTS/STT 服务器)

日常主要同步 `kizuna-ai-lab/sokuji-tsuyaku-api` 即可。

---

## 0. 认识 forks 关系

```
speaches-ai/speaches   (原始 TTS/STT 服务器, "Ollama but for TTS/STT")
   └─ fork/rewrite → kizuna-ai-lab/sokuji-tsuyaku-api   (改为聚焦实时同传翻译, MIT)
        └─ our fork  → o1laabs/sokuji-tsuyaku-api       (本仓库, 加了分析/索引等文档 + 后续可能的维护)
```

- 我 fork 是在上游 `kizuna-ai-lab` 的 `main`(commit `1fa7cca`)基础上做的。
- 我们在此之上推了自己的文档 commit(`4308bb1` + 本批 README/SYNC)。
- 若上游将来有更新,需要用下面方式把上游 commit **并进**本 fork 的 `main`。

---

## 1. 一次性:在可写的本地克隆里挂好 upstream remote

> 下面假设你有一个本 fork 的本地克隆(`origin` 指向 `o1laabs/sokuji-tsuyaku-api`)。
> 若没有,先:
> ```bash
> git clone git@github.com:o1laabs/sokuji-tsuyaku-api.git
> cd sokuji-tsuyaku-api
> ```

添加并拉取上游 remote(用 HTTPS 或 SSH,这里用 SSH 便于推送与 auth 干净):

```bash
git remote add upstream https://github.com/kizuna-ai-lab/sokuji-tsuyaku-api.git
git fetch upstream          # 拉取上游所有分支/tag
git remote -v               # 确认: origin=o1laabs, upstream=kizuna-ai-lab
```

若也想关注最上层 speaches 的变动(可选):

```bash
git remote add speaches https://github.com/speaches-ai/speaches.git
git fetch speaches
```

---

## 2. 日常同步(推荐:基于 main rebase,保持历史线性)

Step A — 先确保本地 `main` 与远端 fork 一致
```bash
git checkout main
git pull origin main
```

Step B — 把上游最新代码 rebase 到本地 main 之上
```bash
git fetch upstream
git rebase upstream/main
```

> 说明:
> - 本 fork 只加了 **docs** 层(README.md / xxx-analysis.md / UPSTREAM-SYNC.md),
>   与上游 `src/`、`pyproject.toml` 撞文件的概率低,rebase 通常顺利。
> - 真冲突时:逐文件解决 → `git add <file>` → `git rebase --continue`。
> - 若后悔可 `git rebase --abort` 回到 pull 后状态,无损。

Step C — 推送 rebase 过的分支回 fork
```bash
git push --force-with-lease origin main
```

> 用 `--force-with-lease`(安全强推)而非裸 `--force`,避免覆盖他人推到 fork 主干的提交。
> 因为 rebase 把本地历史线改写,才需 force;仅你自己在用 fork 主干时非常安全。

---

## 3. 同步后检查

```bash
# 对比落后/领先
git rev-list --left-right --count upstream/main...origin/main
# 左=upstream 独有(应=0), 右=fork 独有(应只剩我们的 docs commit)
git log upstream/main..origin/main --oneline   # 应只看到 docs commit(们),无 src 改动

# 确认本 fork 相关文档仍在
ls docs/README.md docs/sokuji-tsuyaku-api-analysis.md docs/UPSTREAM-SYNC.md
```

理想状态:右边只剩类似
```
4308bb1 docs: add sokuji-tsuyaku-api research analysis
XXXXXXX docs: add docs index + upstream sync guide
```

---

## 4. 说明与边界

- **不要**把 sokes + speaches 的两层上游随意双向 merge,保持单一方向:
  上游 `kizuna-ai-lab` 是主体。speaches 的变动一般已经体现在 `kizuna-ai-lab` 的同步里,极少需要你直接跟 speaches。
- docs 目录沿用了大量上游英文文档(见 `docs/README.md` 索引)。同步上游时会一并带入上游的 docs 改动,这没问题;
  我们新增的三份中文/索引文档只要不与上游新增文件名撞车即可(push 前 `git status` 看清楚)。
- 本仓库最终形态遵循 MIT(见根 LICENSE)。
- 若你想回馈功能(而非文档)给上游,请在 `kizuna-ai-lab/sokuji-tsuyaku-api` 开 PR,不要在 fork 主干混提交功能改动,方便长期同步。

---

## 快速清单(把上面浓缩成一串命令,验证于 2026-09-02,上游无落后、fork 领先自己 docs)

```bash
cd /path/to/o1laabs/sokuji-tsuyaku-api
git checkout main && git pull origin main
git fetch upstream
git rebase upstream/main         # 冲突则解决后 git rebase --continue
git push --force-with-lease origin main
git rev-list --left-right --count upstream/main...origin/main   # 左应0
```
