# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Shinsekai plugin that provides ASR (Automatic Speech Recognition) by connecting to a self-hosted FunASR WebSocket service. It registers the `funasr_wss` ASR provider slug with Shinsekai's plugin system.

**Plugin entry**: `funasr_wss.plugin:FunASRWssPlugin`
**Adapter class**: `funasr_wss.adapter.FunASRWSSAdapter`

## Setup

```bash
pip install -r requirements.txt
```

## Running FunASR Server Locally

```bash
docker compose -f docker-compose.funasr.yml up -d
```

Wait for log message: `asr model init finished. listen on port:10095`

## Architecture

### Plugin registration (`plugin.py`)

`FunASRWssPlugin` extends `PluginBase`. In `initialize()`, it calls `register.register_asr_adapter("funasr_wss", FunASRWSSAdapter)` to register the adapter with Shinsekai's capability registry.

### ASR Adapter (`adapter.py`)

`FunASRWSSAdapter` extends `ASRAdapter` from the SDK. It maintains two daemon threads:

- **`_sender_loop`**: Reads PCM audio from PyAudio input stream and sends binary frames to the FunASR WebSocket server
- **`_receiver_loop`**: Receives JSON messages from the server and dispatches to `_handle_server_message()`

The adapter supports three FunASR modes: `2pass` (default), `online`, and `offline`.

### Message flow

1. On `start()`, connects WebSocket and starts sender/receiver threads
2. Sends a JSON start payload with config (mode, chunk_size, language, etc.)
3. Sends binary PCM audio frames while `is_speaking=True`
4. Receives transcriptions via `_handle_server_message()` and calls `callback(text, is_partial)`
5. On `stop()`, sends `{"is_speaking": False}` and joins threads

### Config schema

`get_config_schema()` exposes: `host`, `port`, `use_ssl`, `mode`. Other parameters (chunk_size, chunk_interval, audio_fs, etc.) are hardcoded defaults in `__init__`.