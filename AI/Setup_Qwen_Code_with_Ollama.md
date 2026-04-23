# Setup Qwen Code with Ollama
## Ollama
```
ollama pull qwen2.5-coder:7b
ollama list
ollama run qwen2.5-coder:7b

Check availability
curl http://localhost:11434/v1/models
```
## Ollama increase context with custom model file
```
cat .\Modelfile.txt
FROM qwen2.5-coder:7b
PARAMETER num_ctx 32768
PARAMETER num_predict 8192
PARAMETER temperature 0.2
PARAMETER top_p 0.9

ollama create qwen-coder-32k -f .\Modelfile.txt

ollama list
..
NAME                     ID              SIZE      MODIFIED
qwen-coder-32k:latest    3a62ee5cf6ce    4.7 GB    18 seconds ago
qwen2.5-coder:7b         dae161e27b0e    4.7 GB    35 minutes ago
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
        "id": "qwen-coder-32k:latest",
        "name": "Qwen2.5 32K (Ollama)",
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
    "name": "qwen-coder-32k:latest"
  },
  "$version": 3
}
```
