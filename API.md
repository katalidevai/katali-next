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
.\katali-next-server.exe C:\models\katali-next-qwen35-122b-a10b 8080
```

Health check:

```http
GET /health HTTP/1.1
Host: 127.0.0.1:8080
```

Response:

```json
{"status":"ok"}
```

Generation request:

```http
POST /generate HTTP/1.1
Host: 127.0.0.1:8080
Content-Type: application/json

{"tokens":[9419,494]}
```

Generation response:

```json
{"results":[{"input":9419,"next":40719,"position":0,"ms":2658.58,"tok_s":0.376141},{"input":494,"next":79506,"position":1,"ms":2561.22,"tok_s":0.390439}]}
```

The server is intentionally local-only and binds to `127.0.0.1`. It keeps sequence state in the running process, so send tokens for one conversation to the same process in order.

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
