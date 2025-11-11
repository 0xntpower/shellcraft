# ShellCraft

**Multi-provider AI tool for organizing IDA disassembly into shellcode-ready blocks.**

Supports Claude, OpenAI, Gemini, and local models (Ollama) • x86/x64 auto-detection

---

### Why Not Just Use a C to shellcode generator like pe_to_shellcode or donut?
#### Different Goals, Different Tools:

###### Automated converters (pe_to_shellcode, donut, sRDI):
* Purpose: Convert complete executables to shellcode
* User: Red teamers who need working payloads fast
* Learning value: None - complete black box
* Control: Minimal - you get what you get
* Use case: Production operations

###### ShellCraft:
* Purpose: Organize compiler output for hand-crafted shellcode
* User: students, security researchers, learners
* Control: Complete - you decide what to keep, optimize, modify
* Learning value: High - you see and control every instruction
* Use case: Education, custom development, shellcode research

---

## Workflow

```
1. Write C code
   ↓
2. Compile (x86/x64)
   ↓
3. Disassemble in IDA
   ↓
4. Export to text
   ↓
5. shellcraft.py exported_asm.txt --dry-run  ← Safety check
   ↓
6. shellcraft.py exported_asm.txt -v         ← Process
   ↓
7. Use clean,commented,organized blocks to add to your shellcode
```

---

## Quick Start

```bash
# Install
pip install cryptography anthropic  # Claude (or openai, google-generativeai)

# Save token (one-time)
python shellcraft.py --save-token claude sk-ant-xxxxx

# Process file
python shellcraft.py revshell_asm.txt --dry-run  # Preview first
python shellcraft.py revshell_asm.txt -v         # Process
```

## AI Providers

Prices are approximate - check with the provider for current pricing.

| Provider | Cost/File | Install | Token Required |
|----------|-----------|---------|----------------|
| Claude | ~$0.02 | `pip install anthropic` | Yes |
| OpenAI | ~$0.03 | `pip install openai` | Yes |
| Gemini | ~$0.01 | `pip install google-generativeai` | Yes |
| Ollama | FREE | `pip install requests` | No |

List available providers with `python shellcraft.py --list-providers`

### Usage

```bash
# Claude (default)
python shellcraft.py file.txt

# OpenAI
python shellcraft.py file.txt --provider openai

# Gemini (cheapest)
python shellcraft.py file.txt --provider gemini

# Ollama (free, local)
python shellcraft.py file.txt --provider ollama --model qwen2.5-coder:7b -v
```

## Features

- Multi-provider AI (Claude, OpenAI, Gemini, Ollama)
- Auto-detects x86/x64 architecture
- Dry-run mode to preview changes
- Removes MSVC debug code automatically (NOTE: MSVC-specific patterns only)
- Logs all removed lines for review

## Commands

```bash
# Safety
shellcraft.py file.txt --dry-run           # Preview changes
shellcraft.py file.txt --preserve-all      # No removal
shellcraft.py file.txt --save-removed-log  # Track removals

# Architecture
shellcraft.py file.txt --arch x64          # Force x64

# Providers
shellcraft.py file.txt --provider openai   # Use ChatGPT
shellcraft.py file.txt --provider ollama   # Use local model

# Output
shellcraft.py file.txt -o custom.txt       # Custom output path
```

## Architecture Support

Auto-detects x86 or x64 from register usage, or force with `--arch x86` / `--arch x64`

**x86:** Stack-based parameters, ESP/EBP frame pointers, cdecl/stdcall
**x64:** Register parameters (rcx, rdx, r8, r9), shadow space, Microsoft x64 calling convention

## Example Output

```c
" wsa_startup:                       "
"   push    rcx                       ;" # First parameter (x64 fastcall)
"   mov     edx, 0x202                ;" # Second parameter (version 2.2)
"   call    qword ptr [WSAStartup]    ;" # Initialize Winsock

" create_socket:                     "
"   xor     r9d, r9d                  ;" # Fourth param (dwFlags = 0)
"   mov     r8d, 6                    ;" # Third param (IPPROTO_TCP)
"   ...
```

## Using Ollama (Local & Free)

```bash
# Install Ollama
# Linux/Mac: curl https://ollama.ai/install.sh | sh
# Windows: winget install Ollama.Ollama

# Pull a model
ollama pull qwen2.5-coder:7b

# Use it
python shellcraft.py file.txt --provider ollama --model qwen2.5-coder:7b -v
```

## Token Storage

Tokens are encrypted with Fernet and stored in `~/.shellcraft/`. The encryption key is also on your machine, so this is obfuscation not real security - anyone with access to your account can decrypt them. Same as how AWS CLI, Docker, etc. work.

Good enough for personal use. Don't use on shared machines or for production secrets.

```bash
# Save tokens
python shellcraft.py --save-token claude sk-ant-xxxxx
python shellcraft.py --save-token openai sk-xxxxx
python shellcraft.py --save-token gemini xxxxx
```

## Troubleshooting

**"Provider not available"**
- Install: `pip install anthropic` (or openai, google-generativeai)

**"No token found"**
- Save: `python shellcraft.py --save-token <provider> <token>`

**"Too many lines removed"**
- Use: `python shellcraft.py file.txt --preserve-all`

**Ollama connection error**
- Start Ollama: `ollama serve`
