# katali-next API tutorial

## Binary interface

```text
katali-next.exe MODEL_DIRECTORY TOKEN_ID [TOKEN_ID ...]
```

Example:

```powershell
$model = 'C:\models\katali-next-qwen35-122b-a10b'
& .\katali-next.exe $model 9419 494 10213 4115
```

The process writes one result line for each input token. The fields are:

| Field | Meaning |
|---|---|
| `step` | Zero-based input step |
| `input` | Token ID supplied to the runtime |
| `next` | Predicted next token ID |
| `pos` | Sequence position |
| `ms` | Time used for the step |
| `tok_s` | Measured tokens per second |

## PowerShell wrapper example

```powershell
$model = 'C:\models\katali-next-qwen35-122b-a10b'
$tokens = @(9419, 494)
$result = & .\katali-next.exe $model @tokens
$result | ForEach-Object { $_ }
```

## HTTP API

Start the local server:

```powershell
.\katali-next-server.exe C:\models\katali-next-qwen35-122b-a10b 8090
```

Health check:

```http
GET /health HTTP/1.1
Host: 127.0.0.1:8090
```

Response:

```json
{"status":"ok"}
```

Generation request:

```http
POST /generate HTTP/1.1
Host: 127.0.0.1:8090
Content-Type: application/json

{"tokens":[9419,494]}
```

Generation response:

```json
{"results":[{"input":9419,"next":40719,"position":0,"ms":2658.58,"tok_s":0.376141},{"input":494,"next":79506,"position":1,"ms":2561.22,"tok_s":0.390439}]}
```

The server is intentionally local-only and binds to `127.0.0.1`. Each request is an independent sequence; send the full prompt token sequence on every request. The server enforces a 4096-token combined prompt and generation limit.

Reset decoder state:

```http
POST /v1/reset HTTP/1.1
Host: 127.0.0.1:8090
```

## OpenAI-style chat endpoint

Model listing:

```http
GET /v1/models HTTP/1.1
Host: 127.0.0.1:8090
```

Chat completion request using token IDs:

```http
POST /v1/chat/completions HTTP/1.1
Host: 127.0.0.1:8090
Content-Type: application/json

{"model":"katali-next","token_ids":[9419,494],"max_tokens":4}
```

The response contains generated IDs in `choices[0].message.x_token_ids` and standard usage counts. The `content` field is empty because text decoding is left to the client tokenizer.

## Desktop GUI

Launch the native Windows application:

```powershell
.\katali-next-gui.exe
```

Enter a normal question or code request in the GUI's **Prompt** field and select **Generate**. The GUI uses the selected model's `tokenizer.json` to convert that text into token IDs before calling the native runtime. The **TOKEN IDS / ADVANCED** field remains available for manual token-level tests. Python 3 and the `tokenizers` package are required for prompt encoding.

The GUI runtime selector supports the Qwen2.5-Coder 14B adapter when `katali-next-coder.exe` is placed beside `katali-next-gui.exe`. The coder process caches its weights in system RAM during startup.

## C# process integration

```csharp
using System.Diagnostics;

var psi = new ProcessStartInfo {
    FileName = "katali-next.exe",
    Arguments = @"C:\models\katali-next-qwen35-122b-a10b 9419 494",
    RedirectStandardOutput = true,
    UseShellExecute = false,
    CreateNoWindow = true
};

using var process = Process.Start(psi)!;
while (!process.StandardOutput.EndOfStream)
    Console.WriteLine(await process.StandardOutput.ReadLineAsync());
process.WaitForExit();
```

## Important behavior

- Token IDs must come from the tokenizer shipped with the model.
- Keep a single process alive for a generation sequence.
- The runtime is a token-level API; applications are responsible for text encoding and decoding.
- The model directory is read-only input and must contain every shard.
- Do not mix tokenizer files from a different model.
