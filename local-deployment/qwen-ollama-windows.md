# Qwen + Ollama on Windows：本地部署实战笔记

日期：2026-08-31  
目标目录：`D:\bigmodel`  
当前已安装模型：`qwen3.6:27b`


> Windows 环境下使用 Ollama 部署并运行 Qwen 的实战记录，包含目录规划、模型下载、运行脚本、LoRA / Adapter、硬件观察与常见问题。

## 目录

- [1. 目标与结论](#1-目标与结论)
- [2. 参考网站](#2-参考网站)
- [3. 目录结构](#3-目录结构)
- [4. 安装 Ollama](#4-安装-ollama)
- [5. 设置模型目录环境变量](#5-设置模型目录环境变量)
- [6. 启动 Ollama 服务](#6-启动-ollama-服务)
- [7. 下载 Qwen 模型](#7-下载-qwen-模型)
- [8. 运行模型](#8-运行模型)
- [9. 当前脚本](#9-当前脚本)
- [10. LoRA / Adapter 研究流程](#10-lora--adapter-研究流程)
- [11. 关于完整 GGUF 模型和 LoRA 的区别](#11-关于完整-gguf-模型和-lora-的区别)
- [12. GPU / NPU 观察](#12-gpu--npu-观察)
- [13. 常见问题](#13-常见问题)
- [14. 安全与来源检查建议](#14-安全与来源检查建议)

## 1. 目标与结论

本次目标是在 Windows 电脑上把通义千问/Qwen 系列模型安装到 `D:\bigmodel`，并通过 Ollama 本地运行，后续可以研究 LoRA/adapter 挂载方式。

最终采用方案：

- 运行框架：Ollama
- 安装目录：`D:\bigmodel\ollama-app`
- 模型目录：`D:\bigmodel\ollama-models`
- LoRA/adapter 预留目录：`D:\bigmodel\adapters`
- 脚本目录：`D:\bigmodel\scripts`
- 当前模型：`qwen3.6:27b`

本机硬件观察：

- 内存：32 GB
- GPU：Intel Arc 集显
- NPU：Intel AI Boost
- 实际运行状态：Ollama 当前主要使用 CPU + 内存，`ollama ps` 显示 `100% CPU`
- 27B 模型能跑，但内存占用高，首次加载慢

## 2. 参考网站

Ollama 官方：

- Windows 下载页：https://ollama.com/download/windows
- Ollama 模型库：https://ollama.com/library
- Qwen3.6 模型页：https://ollama.com/library/qwen3.6
- Modelfile 文档：https://docs.ollama.com/modelfile
- GPU 文档：https://docs.ollama.com/gpu

Qwen / LoRA 相关：

- Qwen 官方站：https://qwen.ai
- Qwen Hugging Face 主页：https://huggingface.co/Qwen
- Hugging Face Qwen LoRA 搜索：https://huggingface.co/models?search=qwen%20lora%20gguf
- ModelScope 模型搜索：https://modelscope.cn/models

## 3. 目录结构

当前采用的目录结构：

```text
D:\bigmodel
├─ adapters
├─ downloads
├─ ollama-app
├─ ollama-models
└─ scripts
```

说明：

- `downloads`：安装包、手动下载的模型文件等
- `ollama-app`：Ollama 程序目录
- `ollama-models`：Ollama 拉取的模型权重和 manifest
- `adapters`：以后放 LoRA adapter
- `scripts`：启动、测试、创建 LoRA 派生模型的脚本

## 4. 安装 Ollama

下载安装器：

```powershell
New-Item -ItemType Directory -Force -Path 'D:\bigmodel\downloads'
Invoke-WebRequest -Uri 'https://ollama.com/download/OllamaSetup.exe' -OutFile 'D:\bigmodel\downloads\OllamaSetup.exe'
```

安装到 D 盘：

```powershell
New-Item -ItemType Directory -Force -Path 'D:\bigmodel\ollama-app','D:\bigmodel\ollama-models','D:\bigmodel\adapters','D:\bigmodel\scripts'
Start-Process -FilePath 'D:\bigmodel\downloads\OllamaSetup.exe' -ArgumentList '/DIR="D:\bigmodel\ollama-app"' -Wait
```

验证 Ollama：

```powershell
& 'D:\bigmodel\ollama-app\ollama.exe' --version
```

本次安装版本为：

```text
0.33.2
```

## 5. 设置模型目录环境变量

变量名：

```text
OLLAMA_MODELS
```

变量值：

```text
D:\bigmodel\ollama-models
```

图形界面设置方式：

```text
Win + R
sysdm.cpl
高级
环境变量
用户变量
新建
```

PowerShell 设置方式：

```powershell
[Environment]::SetEnvironmentVariable("OLLAMA_MODELS", "D:\bigmodel\ollama-models", "User")
```

设置后重新打开 PowerShell，验证：

```powershell
$env:OLLAMA_MODELS
```

应该输出：

```text
D:\bigmodel\ollama-models
```

如果还想直接输入 `ollama run ...`，还需要把下面目录加入 `Path`：

```text
D:\bigmodel\ollama-app
```

否则就继续使用完整路径：

```powershell
& 'D:\bigmodel\ollama-app\ollama.exe' run qwen3.6:27b
```

## 6. 启动 Ollama 服务

后台启动：

```powershell
Start-Process -FilePath 'D:\bigmodel\ollama-app\ollama.exe' -ArgumentList 'serve' -WindowStyle Hidden
```

确认服务可用：

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:11434/api/tags' -Method Get
```

停止服务：

```powershell
Get-Process ollama -ErrorAction SilentlyContinue | Stop-Process -Force
```

## 7. 下载 Qwen 模型

下载本次使用的模型：

```powershell
& 'D:\bigmodel\ollama-app\ollama.exe' pull qwen3.6:27b
```

本次结果：

```text
NAME           ID              SIZE
qwen3.6:27b    9d5803d493a9    17 GB
```

模型实际保存在：

```text
D:\bigmodel\ollama-models
```

检查模型列表：

```powershell
& 'D:\bigmodel\ollama-app\ollama.exe' list
```

检查当前运行状态：

```powershell
& 'D:\bigmodel\ollama-app\ollama.exe' ps
```

本机观察到：

```text
PROCESSOR    100% CPU
CONTEXT      4096
```

## 8. 运行模型

普通模式，关闭 thinking：

```powershell
& 'D:\bigmodel\ollama-app\ollama.exe' run qwen3.6:27b --think=false
```

Thinking 模式：

```powershell
& 'D:\bigmodel\ollama-app\ollama.exe' run qwen3.6:27b --think=true
```

进入交互界面后会看到：

```text
>>>
```

直接输入问题即可。

退出方式：

```text
/bye
```

或按：

```text
Ctrl + C
```

## 9. 当前脚本

普通聊天脚本：

```text
D:\bigmodel\scripts\Chat-Qwen3.6-27B.cmd
```

内容：

```cmd
@echo off
"D:\bigmodel\ollama-app\ollama.exe" run qwen3.6:27b --think=false
pause
```

Thinking 聊天脚本：

```text
D:\bigmodel\scripts\Chat-Qwen3.6-27B-Thinking.cmd
```

内容：

```cmd
@echo off
"D:\bigmodel\ollama-app\ollama.exe" run qwen3.6:27b --think=true
pause
```

启动服务脚本：

```text
D:\bigmodel\scripts\Start-Ollama-D-BigModel.cmd
```

用途：后台启动 Ollama 服务。

测试脚本：

```text
D:\bigmodel\scripts\Test-Qwen3.6-27B.ps1
```

运行：

```powershell
powershell -ExecutionPolicy Bypass -File 'D:\bigmodel\scripts\Test-Qwen3.6-27B.ps1'
```

成功时输出类似：

```text
local-ok
```

## 10. LoRA / Adapter 研究流程

Ollama 通过 `Modelfile` 的 `ADAPTER` 指令挂载 LoRA。

重要限制：

- LoRA 必须和基础模型匹配
- 当前基础模型是 `qwen3.6:27b`
- 优先使用 `.gguf` LoRA adapter
- 不要把 Qwen2.5、Qwen3 14B/32B、Llama、Mistral 等模型的 LoRA 硬套到 `qwen3.6:27b`
- 普通 `.safetensors` LoRA 不一定能被 Ollama 直接使用

示例 Modelfile：

```text
FROM qwen3.6:27b
ADAPTER D:\bigmodel\adapters\your-qwen-lora-adapter.gguf
PARAMETER num_ctx 8192
```

本次创建过的 LoRA 工具脚本：

```text
D:\bigmodel\scripts\Create-Qwen3.6-27B-LoRA-GGUF.ps1
```

用法：

```powershell
powershell -ExecutionPolicy Bypass -File 'D:\bigmodel\scripts\Create-Qwen3.6-27B-LoRA-GGUF.ps1' -AdapterPath 'D:\bigmodel\adapters\your-adapter.gguf'
```

默认会创建模型名：

```text
qwen3.6-27b-lora
```

之后运行：

```powershell
& 'D:\bigmodel\ollama-app\ollama.exe' run qwen3.6-27b-lora --think=false
```

## 11. 关于完整 GGUF 模型和 LoRA 的区别

LoRA adapter：

- 体积通常较小
- 依赖基础模型
- 用 `ADAPTER` 挂载
- 适合研究风格、领域能力、任务微调效果

完整 GGUF 模型：

- 是完整模型权重
- 不能当作 LoRA 挂载
- 需要单独创建 Ollama 模型
- 通常通过 `FROM D:\path\model.gguf` 创建

本次未把第三方去限制/解限类完整 GGUF 模型接入 Ollama。

## 12. GPU / NPU 观察

本机任务管理器观察：

- CPU 上升明显
- 内存接近 32 GB 上限
- GPU/NPU 没有明显占用

原因：

- Ollama 当前在这台机器上主要用 CPU 跑
- Intel AI Boost NPU 通常不会被 Ollama 用于这种 GGUF LLM 推理
- Intel Arc 集显在 Windows + Ollama 下不一定能自动被使用

可以尝试 Vulkan：

```powershell
Get-Process ollama -ErrorAction SilentlyContinue | Stop-Process -Force
$env:OLLAMA_VULKAN = "1"
Start-Process "D:\bigmodel\ollama-app\ollama.exe" -ArgumentList "serve" -WindowStyle Hidden
```

然后检查：

```powershell
& 'D:\bigmodel\ollama-app\ollama.exe' ps
```

如果仍显示：

```text
100% CPU
```

说明当前没有成功用上 Intel GPU。

## 13. 常见问题

### 13.1 为什么看到 Thinking 或英文思考过程？

Qwen 新模型支持 thinking。普通聊天建议用：

```powershell
& 'D:\bigmodel\ollama-app\ollama.exe' run qwen3.6:27b --think=false
```

注意参数必须写成：

```text
--think=false
```

不要写成：

```text
--think false
```

否则 `false` 可能被当成用户输入。

### 13.2 为什么内存占用很高？

`qwen3.6:27b` 是 27B 级模型，Ollama 包约 17 GB。32 GB 内存可以运行，但空间很紧，首次加载会慢，运行时内存占用高是正常现象。

### 13.3 想更快怎么办？

可以安装更小的 Qwen 模型，例如 14B、8B 或 4B 级别。越小速度越快，质量一般也会下降。

示例：

```powershell
& 'D:\bigmodel\ollama-app\ollama.exe' pull qwen3:8b
```

### 13.4 如何删除模型？

```powershell
& 'D:\bigmodel\ollama-app\ollama.exe' rm qwen3.6:27b
```

删除前建议先确认：

```powershell
& 'D:\bigmodel\ollama-app\ollama.exe' list
```

## 14. 安全与来源检查建议

下载社区模型或 LoRA 前建议检查：

- 页面是否明确说明 base model
- 是否匹配 `Qwen3.6-27B`
- 是否是 `.gguf` adapter
- license 是否允许本地研究使用
- README 是否说明训练数据和用途
- 是否有明显的恶意、钓鱼、可疑脚本

不要随意运行模型仓库里的未知脚本。对于 Ollama，本地研究优先使用 `.gguf` 文件和 `Modelfile`，不要执行不明 `.bat`、`.ps1`、`.exe`。
