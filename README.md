# CLIProxyAPI — Compaction-Optimized Fork

基于 [router-for-me/CLIProxyAPI](https://github.com/router-for-me/CLIProxyAPI) 的优化分支，聚焦于 **Codex CLI 多账号代理场景下的 token 消耗优化**。

> 完整配置说明请参考原项目 README 及 `config.example.yaml`。

---

## 改动说明

### 1. Compaction 透传 (`allow-compaction-replay`)

**问题**：Codex CLI 会本地压缩长对话（compaction），但代理只对 `provider == "codex"` 的账号放行 compaction 条目，其他上游一律剥离并用完整历史替代，导致 token 浪费。

**解决**：新增配置开关：

```yaml
routing:
  allow-compaction-replay: true  # 默认 false
```

开启后 compaction 条目直接透传上游，不再限制 provider。适用于 GPT 等原生支持 compaction 的上游。

### 2. `previous_response_id` 条件保留

**问题**：代理无条件删除 `previous_response_id`，导致 Codex CLI 的增量请求退化为全量，token 重复消耗。

**解决**：改为根据 auth 健康度决定：

- **额度健康** → 保留 `previous_response_id` → GPT 增量处理 → 省 token
- **额度用完/不可用** → 删除 → 准备 failover 到新账号 → 不报错

健康度检查函数 `codexAuthHealthy()` 覆盖：`auth.Disabled`、`auth.Unavailable`、`auth.Quota.Exceeded`、`ModelState` 级别的不可用和额度超限。

### 3. Session Affinity

已在 `config.yaml` 中开启：

```yaml
routing:
  session-affinity: true   # 同一会话钉同一账号，保持上游缓存热
```

---

## 配置示例

```yaml
routing:
  strategy: round-robin
  session-affinity: true
  allow-compaction-replay: true
```

其它配置项与原项目完全兼容，详见 `config.example.yaml` 及原项目文档。
