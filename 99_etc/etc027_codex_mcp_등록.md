# Codex MCP 등록

터미널에서 codex 실행했더니 openaiDeveloperDocs MCP 연결하래서 해보았다.

명령어까지 친절하게 알려주었다.

```shell
codex mcp add openaiDeveloperDocs --url https://developers.openai.com/mcp.
```

그리고 다시 codex 시작 > 에러 메세지

```shell
⚠ MCP client for `openaiDeveloperDocs` failed to start: MCP startup failed: handshaking with MCP server failed: Send message error Transport
[codex_rmcp_client::event_notification_transport::EventNotificationTransport<rmcp::transport::worker::WorkerTransport<rmcp::transport::streamable_ht
tp_client::StreamableHttpClientWorker<codex_rmcp_client::http_client_adapter::StreamableHttpClientAdapter>>>] error: unexpected server response:
HTTP 404: Not found, when send initialize request

⚠ MCP startup incomplete (failed: openaiDeveloperDocs)
```

원인은, 등록할 때 URL 마지막에 `.` 이 존재하여..

## 수정하자

```shell
# 지우고
codex mcp remove openaiDeveloperDocs

# 정상 URL 로 등록
codex mcp add openaiDeveloperDocs --url https://developers.openai.com/mcp
```

OK

## 참고사항

codex 에 설정된 값은 MAC 기준, 루트 폴더의 .codex/config.toml 에서 확인할 수 있다.
