# Setup Qwen Code with Ollama
## Ollama
```
ollama pull qwen2.5-coder:7b
ollama list
ollama run qwen2.5-coder:7b

Check availability
curl http://localhost:11434/v1/models
```
## Qwen Code
```
powershell -Command "Invoke-WebRequest 'https://qwen-code-assets.oss-cn-hangzhou.aliyuncs.com/installation/install-qwen.bat' -OutFile (Join-Path $env:TEMP 'install-qwen.bat'); & (Join-Path $env:TEMP 'install-qwen.bat')"

Put settings.json as described below this section
Restart shell....

qwen.cmd
```
### C:\Users\Frodo\.qwen\settings.json
```
{
  "env": {
    "OLLAMA_API_KEY": "ollama",
    "VLLM_API_KEY": "not-needed",
    "LMSTUDIO_API_KEY": "lm-studio"
  },
  "modelProviders": {
    "openai": [
      {
        "id": "qwen2.5-coder:7b",
        "name": "Qwen2.5 7B (Ollama)",
        "envKey": "OLLAMA_API_KEY",
        "baseUrl": "http://localhost:11434/v1",
        "generationConfig": {
          "timeout": 300000,
          "maxRetries": 2,
          "contextWindowSize": 32768,
          "samplingParams": {
            "temperature": 0.2,
            "top_p": 0.9,
            "max_tokens": 8192
          }
        }
      }
    ]
  },
    "security": {
    "auth": {
      "selectedType": "openai"
    }
  },
  "model": {
    "name": "qwen2.5-coder:7b"
  },
  "$version": 3
}
```
