# Shinsekai FunASR WSS

[English](./README.md) | 中文版

Shinsekai 的语音识别插件，通过 WebSocket 连接将麦克风音频流发送到自托管的 FunASR 服务。

## 功能说明

- 注册新的 ASR provider slug：`funasr_wss`
- 将麦克风 PCM 音频发送至自托管的 FunASR WebSocket 服务器
- 将 FunASR 流式识别结果映射到 Shinsekai 现有的 `callback(text, is_partial)` 接口
- 支持 `2pass` 模式下的 partial 和 final 转写回调

## 注册信息

本仓库计划通过 `Shinsekai-Plugin-Registry` 发布，条目如下：

```json
{
  "name": "funasr_wss",
  "author": "umikok7",
  "repo": "umikok7/shinsekai-funasr-wss",
  "description": "FunASR WebSocket 语音识别插件，支持自托管实时 STT。",
  "entry": "funasr_wss.plugin:FunASRWssPlugin"
}
```

## 主机兼容性说明

本插件依赖 Shinsekai 主机端的一个小修复，用于支持外部 ASR provider slug。

具体来说，Shinsekai 需要在解析 `system_config.asr_provider` 时保留插件的 ASR slug（如 `funasr_wss`），而不是将未知值强制回退到内置 provider。

如果主机端修复尚未合入，插件可能安装成功但无法被选为当前 ASR 后端。

## 插件入口

插件市场入口路径：

```text
funasr_wss.plugin:FunASRWssPlugin
```

## 运行时依赖

将依赖安装到 Shinsekai 所在的 Python 环境：

```bash
pip install -r requirements.txt
```

当前运行时依赖：

- `websocket-client`

## 推荐初始配置

安装完成后，在 Shinsekai 设置中选择 `funasr_wss`，推荐参数：

- `host`：`127.0.0.1`
- `port`：`10096`
- `use_ssl`：`false`
- `mode`：`2pass`

## 本地 FunASR 服务

使用 Docker 启动本地服务：

```bash
docker compose -f docker-compose.funasr.yml up -d
docker compose -f docker-compose.funasr.yml logs -f
```

当日志中出现以下内容时表示服务已就绪：

```text
asr model init finished. listen on port:10095
```

## 开发备注

- 插件假定 Shinsekai 负责麦克风采集的生命周期管理
- 本版本 FunASR 服务器采用自托管部署方式
- 热词（hotword）调优等高级参数可后续添加，不影响基础 provider 合约