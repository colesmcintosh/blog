---
layout: post
title: "the codex harness"
date: 2026-06-19 09:00:00 -0500
categories: update
---

The most interesting part of Codex is not the model call. It is the harness around the model call.

That is the thing people underrate about coding agents. A good model matters, obviously. But a useful coding agent is not just a chat completion pointed at a repo. It is a runtime system. It has to start sessions, accept user input, build context, stream model output, expose tools, route tool calls, ask for approvals, enforce sandboxing, persist history, recover from interruptions, and keep frontends in sync while all of that is happening.

The Codex harness in `codex-rs` is interesting because it treats that runtime as the product. The mental model is not "one wrapper object around the model." It is a set of cooperating layers around `codex-core`.

At the outside edge, frontends and API transports submit protocol `Op` values. `ThreadManager` owns the live thread registry. `CodexThread` is the public per-thread handle. `Session` owns the mutable runtime state, services, queues, persistence, and async submission loop. `run_turn` performs the model/tool loop until the turn reaches a terminal state. `ToolRouter`, `ToolCallRuntime`, and `ToolOrchestrator` decide what tools exist, dispatch calls, handle approval, select sandboxes, and retry when policy allows it.

That layering is the important design choice. It makes the harness less like a CLI script and more like an operating environment for agent work.

## Threads are the unit of continuity

The thread lifecycle starts before the first model request. `ThreadManager` creates process-wide services that should be shared across sessions: auth, model management, environments, skills, plugins, MCP, extensions, persistent thread storage, analytics, state DB, and installation identity.

Starting or resuming a thread flows through `ThreadManager::start_thread_with_options`, then `Codex::spawn`, then `Session::new`. Each layer has a narrow job. The manager validates environments and session source. `Codex::spawn` creates the submission and event channels, resolves model defaults, builds `SessionConfiguration`, and starts the submission loop. `Session::new` wires the actual runtime bundle: model client, hooks, rollout tracing, live thread persistence, MCP connection manager, network proxy, exec policy, unified exec process manager, code-mode service, extension data stores, and turn environment snapshots.

The key detail is that `SessionConfigured` is emitted before later startup events. That sounds mundane, but it is the kind of ordering guarantee that makes clients stable. A UI can attach its state before MCP and hook messages start arriving.

This is how you get a frontend that can reconnect, resume, fork, replay history, or attach to an already-running thread without treating the agent like a black box.

## Turns are loops, not requests

The center of the harness is `run_turn` in `core/src/session/turn.rs`. A turn is not one request and one response. It is a loop.

The loop has to do a surprising amount of work before the model even samples. It may compact context if the window is too large. It records context updates and user input. It chooses skill, plugin, and extension injections for the turn. It runs input hooks. Only then does it build model-visible input from `ContextManager` history and `TurnContext`.

Once the model starts streaming, the harness converts response events into protocol events and persisted history. If the model asks for tools, those calls are routed, executed, and appended as tool output items to the next model request. The turn ends only when the model no longer needs a follow-up, the turn is interrupted, a fatal error occurs, or a hook blocks or stops execution.

That distinction matters. A serious coding agent spends a lot of time outside the model response. It is sampling, acting, observing, sampling again, and preserving enough structure that the whole chain can be inspected later.

It also explains why mid-turn user input is carefully drained only after the fresh input has been sampled. Steering messages should not overtake the user message that started the turn. These ordering details are easy to miss until you build a real agent UI and discover that "just send another message" creates race conditions.

## The protocol boundary is clean

Core talks to clients through `codex_protocol::protocol`.

Client-to-core messages are `Op` values: `UserInput`, `ThreadSettings`, `Interrupt`, `ExecApproval`, `PatchApproval`, `UserInputAnswer`, `RequestPermissionsResponse`, `Compact`, `Review`, and `Shutdown`.

Core-to-client messages are `EventMsg` values: `SessionConfigured`, `TurnStarted`, `ItemStarted`, `AgentMessageContentDelta`, `ExecApprovalRequest`, `ApplyPatchApprovalRequest`, `RawResponseItem`, `TurnDiff`, `TurnComplete`, `TurnAborted`, and `Error`.

That boundary gives the runtime a stable contract. Frontends do not need to know how the model client streams SSE, how tool calls are represented internally, or how rollout persistence is implemented. They submit operations and render events.

It is also why app-server and TUI can be separate. The modern TUI can talk through app-server request processors, while `codex-core` remains the place where agent semantics live.

## Tools are planned separately from tools being run

The harness separates tool exposure from tool execution.

`core/src/tools/spec_plan.rs` decides which tools are visible to the model for the current turn. That includes core tools, MCP tools, dynamic tools, extension tools, hosted tools, and code-mode tools. Some handlers may be dispatch-only and hidden from the model.

Execution then flows through `ToolRouter`, `ToolCallRuntime`, `ToolRegistry`, and `ToolOrchestrator`. The router converts response items into executable tool calls. The runtime manages parallelism and cancellation. The registry wraps handlers with telemetry, extension lifecycle notifications, pre-tool hooks, post-tool hooks, input rewrites, and final output formatting. The orchestrator handles the dangerous part: approval, sandbox selection, network policy, and retry.

This split is one of the most important parts of the design. "Can the model see this tool?" and "Can this tool execute right now under this policy?" are different questions. Treating them as different questions keeps the system safer and more flexible.

## Approval is not bolted on

Shell, unified exec, and apply-patch runtimes implement shared traits for approval and sandboxing. `ToolOrchestrator` drives the sequence.

It determines the approval requirement from exec policy, approval policy, sandbox policy, command prefix rules, additional permissions, and tool metadata. Permission hooks get a chance to answer first. The request may go through automated guardian review or become a user-facing approval event. Then the sandbox is selected, execution happens, and sandbox denials can be final, retried with approval, or converted into network approval prompts depending on policy.

The subtle part: escalation does not silently remove denied-read restrictions. That is a real security boundary. If a file was denied before approval, approval to retry does not automatically mean the process gets to read everything.

The approval flow is asynchronous. For command approvals, the session inserts a oneshot sender into active turn state, emits `EventMsg::ExecApprovalRequest`, and waits until the frontend submits `Op::ExecApproval`. Patch approval follows the same shape with `ApplyPatchApprovalRequest` and `PatchApproval`.

This is the difference between a coding agent that can safely run in a production developer environment and one that is only comfortable in a toy sandbox.

## Persistence is part of the runtime

The harness records both model-visible history and protocol events.

`ContextManager` stores the response items and context-window state the model needs. `record_conversation_items` writes response items into history, persists rollout response items, and emits raw response item events. `send_event_raw` persists protocol events as `RolloutItem::EventMsg`, records rollout traces, and sends events to clients. `ThreadStore` owns persisted thread metadata and history for resume, fork, archive, delete, and cold-thread operations. `StateDbHandle` backs process-wide indexed metadata.

That means the event stream is not just UI decoration. It is part of the durable execution record. That matters for debugging, replay, resume, and trust.

## The model client is split by lifetime

`ModelClient` is session-scoped. `ModelClientSession` is turn-scoped.

The session-scoped client owns stable provider and auth state, websocket fallback state, attestation, installation ID, compression, beta headers, and timing metrics. The turn-scoped session owns response connection reuse, last request and response state, sticky `x-codex-turn-state` routing, and request metadata.

That is a good separation. Provider configuration belongs to the session. Routing and request state belong to the turn. When a turn contains multiple follow-up model requests because tools were called, the turn-scoped client can preserve exactly the state it needs without leaking that state into unrelated turns.

## The architecture says what Codex is trying to become

The practical reading order tells the story:

- `app-server/src/request_processors/turn_processor.rs` shows how client requests become `Op::UserInput`.
- `core/src/codex_thread.rs` shows the frontend-facing thread API.
- `core/src/session/handlers.rs` shows operation dispatch.
- `core/src/session/turn.rs` shows the sampling and tool follow-up loop.
- `core/src/tools/spec_plan.rs` and `core/src/tools/router.rs` show tool exposure and call conversion.
- `core/src/tools/registry.rs`, `core/src/tools/parallel.rs`, and `core/src/tools/orchestrator.rs` show execution, hooks, approvals, and sandbox retry.
- `core/src/session/mod.rs` shows persistence, event delivery, service wiring, and startup or shutdown behavior.

That is a lot of machinery. But it is not accidental complexity. Coding agents need machinery because useful autonomy is mostly about controlled side effects.

The model decides what to do. The harness decides what is possible, what is visible, what is allowed, what is recorded, what can be retried, what the user must approve, and how the outside world learns what happened.

The industry conversation still over-indexes on model comparisons. Those comparisons matter, but they are not enough. Agent quality is increasingly a systems problem. The winning tools will be the ones that combine strong models with disciplined runtime architecture: clean protocol boundaries, durable state, resumable threads, secure tool execution, policy-aware approvals, rich extension points, and UI event streams that make long-running work understandable.

Codex is interesting because the harness takes that seriously.
