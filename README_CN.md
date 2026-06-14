# CLIProxyAPI — previous_response_id 优化分支

基于 [router-for-me/CLIProxyAPI](https://github.com/router-for-me/CLIProxyAPI) 的优化分支。

> 完整配置说明请参考原项目 README 及 `config.example.yaml`。

---

## 改动说明

### `previous_response_id` 条件保留

Codex CLI 发送增量请求时会带 `previous_response_id`，但代理无条件删除，导致增量变全量、token 重复消耗。

改为根据 auth 健康度条件删除：

- **额度健康** → 保留 → GPT 增量处理 → ✅ 省 token
- **额度用完/不可用** → 删除 → 安全 failover → ✅ 不报错

健康度检查 `codexAuthHealthy()` 覆盖：`Disabled`、`Unavailable`、`Quota.Exceeded`、`ModelState` 级别异常。

改动文件：`internal/runtime/executor/codex_executor.go`
