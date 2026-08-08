---
title: ✨Shell 脚本请求接口
published: 2025-07-09
pinned: false
description: "记录使用 Shell 脚本批量读取项目 ID 并请求接口的示例，包含日志记录、参数清洗和请求结果输出。"
tags: [Linux, Shell]
category: 技术分享
draft: false
image: ../uploads/20260110/yy.png
---

# Shell 脚本请求接口

```bash
#!/bin/bash

# 定义请求的基本 URL
base_url="<http://127.0.0.1:10001/ReloadTemplate>"

# 定义日志文件
log_file="response_log.txt"

# 清空日志文件（如果存在）
> "$log_file"

# 初始化计数器
count=0

# 读取 project.txt 中的每一行
while IFS= read -r project_id; do
    # 去除前后空格和换行符
    project_id=$(echo "$project_id" | tr -d '\\r' | xargs)

    # 检查项目 ID 是否为空
    if [ -z "$project_id" ]; then
        echo "跳过空行"
        continue
    fi

    # 增加计数器
    count=$((count + 1))

    # 构建完整的请求 URL
    url="${base_url}?project_id=${project_id}"

    # 输出调试信息，包括当前计数
    echo "处理第 $count 条请求，项目 ID: '${project_id}'"
    echo "构建的 URL: '$url'"

    # 使用 --verbose 标志以获取更多调试信息，并将响应写入日志文件
    response=$(curl --verbose -X GET "$url" 2>&1)

    # 检查 curl 命令的返回状态
    if [ $? -ne 0 ]; then
        echo "请求失败: $response" | tee -a "$log_file"
    else
        echo "请求成功: $response" | tee -a "$log_file"
    fi

done < project_ids.txt

# 输出总共处理的条目数
echo "总共处理了 $count 条请求。"

```
