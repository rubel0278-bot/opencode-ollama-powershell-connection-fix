# Fix OpenCode + Ollama PowerShell Tool-Calling Problems on Windows

A practical troubleshooting guide for cases where OpenCode runs with a local Ollama model but the model does not execute PowerShell tools correctly.

**Repository purpose:** document symptoms, safe diagnostic steps, and evidence-based fixes so other Windows users can investigate the same class of problem.

## Symptoms this guide covers

- OpenCode works with a cloud model, but a local Ollama model fails to use tools.
- A local model may print tool-call-looking text such as `<function=shell>` instead of triggering the actual tool.
- Errors may include `llm_call_failed` or `Operation not allowed`.
- OpenCode may still be able to start Windows PowerShell, so the issue is not necessarily that PowerShell is missing.

These symptoms can have different causes. Do not assume they all have the same fix.

## Case notes: what was observed

In the reported Windows setup:

- OpenCode Desktop/CLI was installed under the user's Windows profile on drive C:.
- Ollama and local models were installed on drive D:.
- The OpenCode default model was `ollama/gpt-oss:20b`.
- Local models tested included GPT-OSS 20B and Qwen-family models.
- OpenCode logs showed that Windows PowerShell could be launched for simple commands such as `Get-Date` and `Get-ChildItem D:\`.
- The reported failure was that a local model could output literal tool-call syntax rather than causing OpenCode to execute the tool.
- Cloud-model tool use reportedly worked, suggesting that the local model/provider/tool-call path deserved investigation.

**Evidence limitation:** the available troubleshooting notes do not record a confirmed final configuration change or a successful end-to-end retest after a specific fix. This guide therefore does not claim that a particular setting has been proven to fix the case. Please contribute the exact successful change and verification output when available.

## Safe troubleshooting steps

### 1. Confirm PowerShell itself works

In Windows PowerShell, run:

```powershell
$PSVersionTable.PSVersion
Get-Date
Get-ChildItem D:\ -ErrorAction Stop | Select-Object -First 5
```

If these commands work in PowerShell but OpenCode does not invoke its shell tool, investigate the OpenCode tool-call path as well as PowerShell.

### 2. Confirm Ollama responds

```powershell
Invoke-RestMethod -Uri 'http://localhost:11434/api/tags' -Method Get
```

This checks the local Ollama API and lists available models. It does not prove that OpenCode tool calling works.

### 3. Check OpenCode's selected model and provider

Review the active OpenCode configuration and confirm the model name matches a model available in Ollama. In this case, the configured model was:

```text
ollama/gpt-oss:20b
```

Do not replace the user's default model or rewrite unrelated settings without evidence and a backup.

### 4. Compare tool behavior, not just text output

Ask OpenCode to run one harmless command, for example:

```powershell
Get-Date
```

A successful test means OpenCode actually invokes the shell tool and returns the command's result. Text that merely resembles a tool call is not a successful execution.

Compare behavior with a cloud model only as a diagnostic control. If the cloud model works and the local model does not, that narrows the investigation but does not by itself prove the root cause.

### 5. Inspect logs for the actual failure

OpenCode logs may be under:

```text
%USERPROFILE%\.local\share\opencode\log\
```

Search for the relevant timestamp, selected model/provider, tool-call attempts, and errors. Avoid publishing API keys, tokens, private prompts, or other secrets from logs.

### 6. Change one thing at a time

Before changing configuration:
1. Copy the original file to a dated backup.
2. Identify the setting tied to the observed failure.
3. Make the smallest justified change.
4. Restart only the affected process if needed.
5. Repeat the exact harmless tool test.
6. Record the before/after output and the changed setting.

Do not reinstall Ollama or OpenCode, delete models, or change unrelated ComfyUI/MCP settings unless evidence shows they are involved.

## What counts as a verified fix?

Record all of the following before marking a fix confirmed:

- The exact error or symptom before the change.
- The configuration key or component changed (never include secrets).
- The command or test used.
- Evidence that OpenCode actually invoked PowerShell, rather than printing tool-call text.
- The resulting command output and any remaining limitations.
- The OpenCode, Ollama, and model versions involved.

## Distinguish the installed Qwen models

The case notes list these separate Ollama models:

- `gpt-oss:20b` (about 13 GB)
- `qwen3-powershell:latest` (about 18 GB)

**Do not confuse `qwen3-powershell:latest` with a separate Qwen3 30B model.** The available inventory/report identifies the former, not the latter. A model being installed or selectable does not by itself prove its PowerShell tool calling works.

For `qwen3-powershell:latest`, verify by selecting that exact model in OpenCode and running a harmless test such as `Get-Date`. Count it as working only if OpenCode actually invokes PowerShell and returns the command output—not if the model merely prints tool-call-looking text. Record the model name, versions, redacted error (if any), and successful output.

**Qwen3 30B status:** no verified PowerShell test for a separate Qwen3 30B model is recorded in these case notes. Do not claim it is connected or fixed until that exact model is identified and tested.

## Related issue: ComfyUI MCP is separate

A ComfyUI MCP bridge problem is a separate integration path from OpenCode's PowerShell tool calling. Do not change ComfyUI's MCP configuration to address a PowerShell tool-call failure unless logs directly connect the two.

## Contributing

If you solved this exact problem, please open an issue or pull request with:
- the redacted error,
- the minimal configuration change,
- the versions used,
- and the successful verification output.

Please do not post credentials, tokens, private logs, or personal paths containing sensitive information.

---

**Status:** Initial case notes and safe diagnostics published. The exact final fix still needs a recorded, reproducible verification result; this repository intentionally avoids presenting an unverified guess as a solution.
