# Smoke-test the streaming response in your deployment

An application can list models and call Chat Completions successfully while its streaming parser still fails. If the application uses streaming, make the next check from the deployment that will read the events. This Ailoot AI engineering note was prepared with AI assistance and checked against the API documentation.

## Run one representative request

Choose a model ID that is visible to the workspace and supports Chat Completions. Use a short synthetic prompt and a small output budget that leaves enough room for the expected reply. Set `stream: true` on the request to `POST /v1/chat/completions`. Keep the existing parser, proxy and timeout in the path so the test exercises the same configuration as the application. [Ailoot Chat Completions](https://ailoot.ai/docs/en/api/chat-completions/)

Check that the client successfully reads the Server-Sent Events response, consumes the reply and recognizes successful completion. If usage matters to the application, request it with `stream_options.include_usage` and verify the final usage event. Ailoot delivers billable text in one block after calculating the charge, so the check should accept that documented event pattern. [Streaming and request parameters](https://ailoot.ai/docs/en/api/chat-completions/)

## Record the point where the check fails

1. Record the requested model ID, HTTP status and request ID where available. Use an approved test credential and omit the authorization header from the record.
2. Record time to the first event and whether the parser completed the response. A proxy timeout, a parse failure and normal completion need distinct test outcomes.
3. Treat `finish_reason: "length"` as a completed request that reached its output budget. For a test that expects a complete short reply, give the fixture enough room and check whether the resulting text met its acceptance condition.

Exercise the parser's handling of a premature connection close with a local fixture. This lets you verify that the application records an incomplete response without sending more generations. [Ailoot reference for errors and limits](https://ailoot.ai/docs/en/errors/)
