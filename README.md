# katali-next

`katali-next` is a native Windows inference runtime for models packaged in the katali-next format.

This public release contains compiled binaries only. No source code is included.

## Model

Download the model package from Hugging Face:

**[katali-next-qwen35-122b-a10b](https://huggingface.co/katalidevai/katali-next-qwen35-122b-a10b)**

The model package contains the model shards and tokenizer metadata. Keep the model on a fast local SSD when possible.

## Requirements

- Windows 10 or newer, 64-bit
- A CPU with AVX2 support is recommended
- System RAM and free disk space sufficient for the selected model
- The model directory must remain complete; do not rename or remove shard files

## Command-line API

The runtime accepts token IDs and returns the next predicted token ID. This keeps the executable independent of a specific application or tokenizer library.

```powershell
.\katali-next.exe C:\models\katali-next-qwen35-122b-a10b 9419 494
```

Each supplied token is processed in order. The output reports the input token, predicted next token, position, elapsed time, and measured tokens per second.

Use the tokenizer files included with the model package to convert user text into token IDs. To turn generated token IDs back into text, decode them with the same tokenizer.

## Integrating from another application

Run `katali-next.exe` as a child process and pass:

```text
katali-next.exe <model-directory> <token-id> [token-id ...]
```

Read standard output line by line. A typical result is:

```text
step=0 input=9419 next=40719 pos=0 ms=2606.1 tok_s=0.384
```

The `next` field is the generated token ID. Keep one process alive for a sequence so the runtime can preserve its decoding state between tokens.

## Minimal application flow

1. Download the model package from the link above.
2. Extract it to a local directory such as `C:\models\katali-next-qwen35-122b-a10b`.
3. Encode the prompt with the included tokenizer.
4. Start `katali-next.exe` with the model directory and encoded token IDs.
5. Append each returned `next` token to the sequence and continue decoding.
6. Decode the returned token IDs with the same tokenizer.

## Performance

The reference validation run used CPU, system RAM, and SSD storage without GPU acceleration. Performance depends heavily on CPU, storage, memory bandwidth, and prompt length. The published executable is intended as an experimental native runtime and should be benchmarked on the target machine before production deployment.

## Distribution

This repository intentionally publishes compiled Windows binaries and documentation only. Model weights are distributed separately through Hugging Face.
