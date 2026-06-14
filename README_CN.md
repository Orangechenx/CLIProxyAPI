# CLIProxyAPI — Compaction 优化分支

基于 [router-for-me/CLIProxyAPI](https://github.com/router-for-me/CLIProxyAPI) 的优化分支，针对 **Codex CLI 多账号代理场景** 进行 token 消耗优化。

> 完整配置说明请参考原项目 README 及 `config.example.yaml`。

---

## 改动说明

### 1. Compaction 透传 (`allow-compaction-replay`)

**问题**：Codex CLI 会本地压缩长对话（compaction），但代理只放行 `provider == "codex"` 的账号，其他剥离 compaction 发完整历史，浪费大量 token。

**解决**：新增配置开关，开启后 compaction 条目直达上游：

```yaml
routing:
  allow-compaction-replay: true  # 默认 false
```

适合 GPT 等原生支持 compaction 的上游。

### 2. `previous_response_id` 条件保留

**问题**：代理无条件删除 `previous_response_id`，增量请求变全量，token 重复消耗。

**解决**：改为根据 auth 健康度条件删除：

- **额度健康** → 保留 → GPT 增量处理 → ✅ 省 token
- **额度用完** → 删除 → 安全 failover → ✅ 不报错

健康度检查 `codexAuthHealthy()` 覆盖：`Disabled`、`Unavailable`、`Quota.Exceeded`、`ModelState` 级别异常。

### 3. Session Affinity

```yaml
routing:
  session-affinity: true   # 同会话钉同账号，上游缓存热
```

---

## 配置示例

```yaml
routing:
  strategy: round-robin
  session-affinity: true
  allow-compaction-replay: true
```

其它配置项与原项目完全兼容，详见 `config.example.yaml`。
