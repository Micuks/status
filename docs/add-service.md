# 手动添加监控项

以后要往 `status.micuks.click` 加新服务，按这个顺序做。

## 1. 改配置源

编辑：

- `/root/.openclaw/workspace/status-new/.upptimerc.yml`

在 `sites:` 下新增一项，例如：

```yml
- name: 俄罗斯方块
  url: https://tetris.micuks.click
```

建议：

- 前端页面监控主页 URL
- 后端接口监控健康检查 URL 或稳定 API
- 名称用中文，和页面展示保持一致

## 2. 本地检查 YAML

至少检查缩进和格式，避免 GitHub Actions 直接炸掉。

简单检查方式：

```bash
sed -n '1,120p' .upptimerc.yml
```

如果机器上有 `python3`，也可以这样粗验 YAML 是否能读：

```bash
python3 - <<'PY'
import yaml
from pathlib import Path
print(yaml.safe_load(Path('.upptimerc.yml').read_text())['sites'][-1])
PY
```

## 3. 同步 README

编辑：

- `/root/.openclaw/workspace/status-new/README.md`

把新增的服务补到“监控的服务”列表里，避免文档和配置脱节。

## 4. 提交并推送

在 `status-new` 仓库里：

```bash
git add .upptimerc.yml README.md docs/add-service.md
git commit -m "chore: add tetris to status monitor"
git push
```

## 5. 等 GitHub Actions 生成页面

这个仓库用 Upptime，实际页面和 `api/`、`history/` 下的产物由 GitHub Actions 生成。

关键工作流：

- `.github/workflows/uptime.yml`
- `.github/workflows/response-time.yml`
- `.github/workflows/graphs.yml`
- `.github/workflows/static-site.yml`

如果只是改了 `.upptimerc.yml`，本地的 `api/status.json` 和 `history/summary.json` 不一定会立刻变化，这正常。推送后等 Actions 跑完即可。

## 6. 验证

先看 GitHub Actions：

- `https://github.com/Micuks/status/actions`

再看线上页面：

- `https://status.micuks.click`

如果新服务还没出现，重点查：

- `.upptimerc.yml` 缩进是否正确
- GitHub Actions 是否失败
- 被监控 URL 是否能从公网访问
- 是否误填了本机内网地址

## 7. Tunnel 类服务的额外注意事项

如果新服务是走 Cloudflare Tunnel 暴露的：

- 先确认公网域名已经能访问
- 再把域名加到 Upptime
- `cloudflared` 侧尽量用 `127.0.0.1:端口`，不要用 `localhost:端口`

这台机器之前出现过：

```text
dial tcp [::1]:5000: connect: connection refused
```

原因是 `localhost` 可能被优先解析到 IPv6，而本地服务只监听 IPv4。
