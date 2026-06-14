# API Reference

## Base URL

All endpoints are served from `http://<host>:<port>`. Default: `http://0.0.0.0:3000`.

## Projects

### Create a Project

```
POST /projects
```

```json
{
  "name": "my-app",
  "repo_url": "https://github.com/you/my-app.git",
  "local_path": "/home/ubuntu/my-app",
  "agent_profile_id": "optional-uuid",
  "deploy_config": "optional-json-string"
}
```

Returns `201` with the created project.

### List Projects

```
GET /projects
```

Returns `200` with an array of projects.

## Agent Profiles

### Create an Agent Profile

```
POST /agent-profiles
```

```json
{
  "name": "claude-dev",
  "agent_type": "claude",
  "cli_path": "/usr/local/bin/claude",
  "credentials_encrypted": "optional-encrypted-key"
}
```

`agent_type` must be one of: `claude`, `opencode`, `aider`, `cline`, `copilot`.

Returns `201` with the created profile.

### List Agent Profiles

```
GET /agent-profiles
```

Returns `200` with an array of profiles.

## Tasks

### Dispatch a Task

```
POST /tasks
```

```json
{
  "project_id": "uuid",
  "agent_profile_id": "optional-uuid",
  "description": "Natural language task description"
}
```

Triggers preflight check. Returns `201` with the task. If complex, the task stays `pending` until confirmed.

### List Tasks

```
GET /tasks
GET /tasks?project_id=<uuid>
```

Returns `200` with an array of tasks.

### Get Task Detail

```
GET /tasks/:id
```

Returns `200` with the task object, or `404`.

### Stream Task Logs (SSE)

```
GET /tasks/:id/logs
```

Returns a `text/event-stream` with these event types:

```
event: log
data: {"id":1,"task_id":"...","role":"agent","chunk":"Analyzing...","timestamp":"..."}

event: message
data: {"type":"text","content":"Found the relevant files","timestamp":"..."}

event: done
data: {"taskId":"..."}

event: error
data: {"message":"..."}
```

### Confirm a Complex Task

```
POST /tasks/:id/confirm
```

Marks a complex task as confirmed and dispatches it. Returns `200`.

### Reply to a Running Task

```
POST /tasks/:id/reply
```

```json
{
  "message": "Add rate limiting too"
}
```

Resumes the agent session via `session_id`. Returns `200`.

### Delete / Kill a Task

```
DELETE /tasks/:id
```

Kills the running agent, cleans up the worktree, marks as failed. Returns `204`.

## Tokens

### Usage Summary

```
GET /tokens
```

Returns `200`:

```json
{
  "total_input": 15000,
  "total_output": 32000,
  "total_tokens": 47000,
  "task_count": 3
}
```

## Task Lifecycle

```
pending ──► running ──► pr_ready ──► merged
                  │                    │
                  └──► failed          └──► failed
```

| Status | Description |
|---|---|
| `pending` | Created, preflight running or waiting for confirmation |
| `running` | Agent actively working in an isolated worktree |
| `pr_ready` | Agent finished, PR created |
| `merged` | PR merged (manual, via GitHub) |
| `failed` | Agent errored or was killed |
