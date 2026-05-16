# Shinsekai FunASR WSS

Plugin-marketplace ASR plugin for Shinsekai, connecting the built-in microphone flow to a self-hosted FunASR WebSocket service.

## What it does

- registers a new ASR provider slug: `funasr_wss`
- sends microphone PCM audio to a self-hosted FunASR WSS server
- maps FunASR streaming results onto Shinsekai's existing `callback(text, is_partial)` contract
- supports partial and final transcript callbacks in `2pass` mode

## Registry entry

This repository is intended to be published through `Shinsekai-Plugin-Registry` with the following entry:

```json
{
  "name": "funasr_wss",
  "author": "umikok7",
  "repo": "umikok7/shinsekai-funasr-wss",
  "description": "FunASR WebSocket ASR plugin for Shinsekai with self-hosted real-time STT support.",
  "entry": "funasr_wss.plugin:FunASRWssPlugin"
}
```

## Host compatibility

This plugin depends on a small Shinsekai host-side compatibility fix for external ASR provider slugs.

In practice, Shinsekai needs to preserve plugin ASR slugs such as `funasr_wss` when resolving `system_config.asr_provider`, instead of forcing unknown values back to built-in providers.

If the host-side fix is not present yet, the plugin may install successfully but still fail to be selected as the active ASR backend.

## Plugin entry

Plugin marketplace entry path:

```text
funasr_wss.plugin:FunASRWssPlugin
```

## Runtime dependencies

Install dependencies into the same Python environment used by Shinsekai:

```bash
pip install -r requirements.txt
```

Current runtime dependencies:

- `websocket-client`

## Recommended initial configuration

After installation, choose `funasr_wss` in Shinsekai Settings and start with:

- `host`: `127.0.0.1`
- `port`: `10096`
- `use_ssl`: `false`
- `mode`: `2pass`

## Local FunASR service

Example local Docker command:

```bash
docker compose -f docker-compose.funasr.yml up -d
docker compose -f docker-compose.funasr.yml logs -f
```

The service is ready once logs show:

```text
asr model init finished. listen on port:10095
```

## Development notes

- the plugin assumes Shinsekai handles microphone capture lifecycle
- FunASR server deployment is intentionally kept self-hosted in this first version
- advanced parameters such as hotword tuning can be added later without changing the basic provider contract
