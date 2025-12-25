
---
title: "基于 LibreOffice 的 Word 转 PDF HTTP 服务"
date: 2025-12-25 18:14:32
tags: [LibreOffice, Word, PDF]
---



# 基于 LibreOffice 的 Word 转 PDF HTTP 服务 —— libreoffice-pdf-service

在很多项目中，我们会遇到将 Word 文档转换成 PDF 的需求。尤其是在自动化办公、文件存档、内容分发等场景下，PDF 格式以其稳定的排版和广泛的兼容性成为首选。但很多库或在线服务在处理复杂文档时存在**格式偏差、样式错乱**等问题。

为了解决这些问题，我基于 LibreOffice 构建了一个轻量级的 Word 转 PDF HTTP 服务，并将其开源到 GitHub：[wjl6/libreoffice-pdf-service](https://github.com/wjl6/libreoffice-pdf-service)

## 🚀 项目简介

libreoffice-pdf-service 是一个独立的 Word 转 PDF 服务，核心依赖 LibreOffice 的强大转换能力，通过 HTTP 接口 提供文件上传并返回 PDF，也支持字节流转换。

相比传统集成转换库，它的优势是：

- **转换精度高** — LibreOffice 原生支持 doc/docx/odt 等格式，输出 PDF 保持文档样式与布局完整。

- **可独立部署** — 服务与主业务系统解耦，不影响主应用性能。

- **轻松扩展** — 支持 REST API，可集成至各种语言或系统。

## ✨ 核心特性

- ✅ 基于 LibreOffice 提供高保真转换

- ✅ RESTful API：通过 HTTP 提供转换服务

- ✅ 支持文件上传 & 字节数组调用

- ✅ Docker 一键部署，易于上线

- ✅ 健康检查、自动清理临时文件

- ✅ 内置安全验证（文件类型白名单、大小限制等）

- ✅ 可配置转换超时和资源限制

## 📦 架构设计

服务整体架构如下：

```text
客户端 APP / 服务
         │
    HTTP 请求 (Word 文件)
         │
   libreoffice-pdf-service
         │
    Docker 中的 LibreOffice
         │
     返回 PDF 文件
```

服务接受客户端的上传，调用 LibreOffice 进行 headless 转换，然后返回 PDF。

## 🛠 快速开始

### 1️⃣ 准备

确保已安装 Docker 或 Docker Compose。

### 2️⃣ 构建镜像

```bash

cd docker
docker build -t libreoffice-service:latest .
```

### 3️⃣ 运行服务

#### 使用 Docker Compose（推荐）

```bash
cd docker
docker-compose up -d
```

#### 或单个 Docker

```bash
docker run -d \
  --name libreoffice-service \
  -p 8100:8080 \
  -e TEMP_DIR=/tmp/libreoffice-convert \
  -e PORT=8080 \
  -e MAX_FILE_SIZE=104857600 \
  -e CONVERT_TIMEOUT=300 \
  libreoffice-service:latest
```

### 4️⃣ 验证服务

#### 健康检查：

```bash
curl http://localhost:8100/health
```

#### 上传 Word 转 PDF：

```bash
curl -X POST \
  -F "file=@test.docx" \
  http://localhost:8100/convert \
  -o output.pdf
```

## 📘 API 文档

### 🧪 健康检查

#### 请求

```text
GET /health
```

#### 返回状态

```json
{
  "status": "healthy",
  "libreoffice": "available"
}
```

### 📤 文件上传转换

#### 请求参数

- `file` — Word 文件（支持 .doc / .docx / .odt / .rtf）

#### 返回

- 成功：PDF 文件流

- 失败：错误信息 JSON

### 🧠 字节数组转换

无需上传 multipart，只需发送文件字节流：

```text
POST /convert/bytes
Content-Type: application/octet-stream
```

## ⚙️ 配置说明

服务支持通过环境变量配置：

|变量名|说明|默认值|
|---|---|---|
|PORT|服务端口|8080|
|TEMP_DIR|临时文件目录|/tmp/libreoffice-convert|
|MAX_FILE_SIZE|最大文件大小（字节）|104857600|
|CONVERT_TIMEOUT|转换超时时间（秒）|300|
## 🧩 常见问题

### ❌ 服务无法启动

- 检查 Docker 日志：`docker logs libreoffice-service`

- 确认端口未被占用

- 确保 LibreOffice 正常安装

### 📝 转换失败

- 确认文件格式支持

- 检查是否超过大小限制

- 查看日志获取详细错误

### ⏱ 超时或大文件

- 调整 CONVERT_TIMEOUT

- 考虑将大文件拆分再转换

## 📈 实战集成

项目内提供了 Java 示例客户端，可直接在 Spring Boot 项目中集成调用服务，实现自动化文件转换。

## 🧠 总结

这个基于 LibreOffice 的 PDF 转换服务解决了很多项目中常见的 Word 到 PDF 转换难题，尤其是对排版、样式要求严格的业务场景。它无需集成复杂的 SDK，只需通过 HTTP 接口调用，大大简化了使用和部署流程。

欢迎大家：

- ✨ ⭐ Star

- ✨ 提 Issue / PR

- ✨ Fork 并改进！

**源码地址**：

👉 [https://github.com/wjl6/libreoffice-pdf-service](https://github.com/wjl6/libreoffice-pdf-service)