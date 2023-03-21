# Shell GPT 中文使用说明文档
Shell GPT 由 OpenAI 的 ChatGPT (GPT-3.5) 提供支持的命令行生产力工具。作为开发人员，我们可以利用 ChatGPT 功能生成 shell 命令、代码片段、评论和文档等。忘记备忘单和注释，使用此工具，您可以直接在终端中获得准确的答案，并且您可能会发现自己减少了日常的 Google 搜索，从而节省了宝贵的时间和精力。
<div align="center">
    <img src="https://i.ibb.co/cNLN99f/output-rescaled.gif" width="800"/>
</div>

项目地址：https://github.com/TheR1D/shell_gpt

## 安装
```shell
pip install shell-gpt
```
您需要一个 OpenAI API 密钥，您可以在[此处](https://beta.openai.com/account/api-keys) 生成一个。
如果设置了 `$OPENAI_API_KEY` 环境变量，它将被使用，否则，系统将提示您输入密钥，然后将其存储在 `~/.config/shell_gpt/.sgptrc` 中。

## 用法
`sgpt` 有多种用例，包括简单查询、shell 查询和代码查询。
### 简单查询
我们可以将它用作普通的搜索引擎，询问任何事情：
```shell
sgpt "nginx default config file location"
# -> The default configuration file for Nginx is located at /etc/nginx/nginx.conf.
```
```shell
sgpt "docker show all local images"
# -> You can view all locally available Docker images by running: `docker images`
```
```shell
sgpt "mass of sun"
# -> = 1.99 × 10^30 kg
```
### 转换
无需搜索转换公式或使用单独的转换网站即可转换各种单位和测量值。您可以转换时间、距离、重量、温度等单位。
```shell
sgpt "1 hour and 30 minutes to seconds"
# -> 5,400 seconds
```
```shell
sgpt "1 kilometer to mile"
# -> 1 kilometer is equal to 0.62137 miles.
```
```shell
sgpt "$(date) to Unix timestamp"
# -> The Unix timestamp for Thu Mar 2 00:13:11 CET 2023 is 1677327191.
```
### Shell 命令
你有没有发现自己忘记了常用的 shell 命令，比如 `chmod`，需要在线查找语法？使用 `--shell` 选项，您可以在终端中快速找到并执行您需要的命令。
```shell
sgpt --shell "make all files in current directory read only"
# -> chmod 444 *
```
由于我们正在接收有效的 shell 命令，我们可以使用 `eval $(sgpt --shell "make all files in current directory read only")` 来执行它，但这不是很方便，我们可以使用 `--execute` (或 `--shell` `--execute` 的快捷方式 `-se`) 参数：
```shell
sgpt --shell --execute "make all files in current directory read only"
# -> chmod 444 *
# -> Execute shell command? [y/N]: y
# ...
```
Shell GPT 知道您正在使用的操作系统和“$SHELL”，它将为您拥有的特定系统提供 shell 命令。例如，如果您要求 `sgpt` 更新您的系统，它会根据您的操作系统返回一个命令。这是一个使用 macOS 的示例：
```shell
sgpt -se "update my system"
# -> sudo softwareupdate -i -a
```
同样的提示，在 Ubuntu 上使用时，会产生不同的建议：
```shell
sgpt -se "update my system"
# -> sudo apt update && sudo apt upgrade -y
```


让我们尝试一些 docker 容器：
```shell
sgpt -se "start nginx using docker, forward 443 and 80 port, mount current folder with index.html"
# -> docker run -d -p 443:443 -p 80:80 -v $(pwd):/usr/share/nginx/html nginx
# -> Execute shell command? [y/N]: y
# ...
```
此外，我们可以在提示中提供一些参数名称，例如，将输出文件名传递给 ffmpeg：
```shell
sgpt -se "slow down video twice using ffmpeg, input video name \"input.mp4\" output video name \"output.mp4\""
# -> ffmpeg -i input.mp4 -filter:v "setpts=2.0*PTS" output.mp4
# -> Execute shell command? [y/N]: y
# ...
```
我们可以在我们的提示中应用额外的 shell 魔法，在这个例子中将文件名传递给 ffmpeg：
```shell
ls
# -> 1.mp4 2.mp4 3.mp4
sgpt -se "using ffmpeg combine multiple videos into one without audio. Video file names: $(ls -m)"
# -> ffmpeg -i 1.mp4 -i 2.mp4 -i 3.mp4 -filter_complex "[0:v] [1:v] [2:v] concat=n=3:v=1 [v]" -map "[v]" out.mp4
# -> Execute shell command? [y/N]: y
# ...
```
由于 ChatGPT 还可以对输入文本进行汇总和分析，我们可以要求它生成提交消息：
```shell
sgpt "Generate git commit message, my changes: $(git diff)"
# -> Commit message: Implement Model enum and get_edited_prompt() func, add temperature, top_p and editor args for OpenAI request.
```
或者要求它在日志中查找错误并提供更多详细信息：
```shell
sgpt "check these logs, find errors, and explain what the error is about: ${docker logs -n 20 container_name}"
# ...
```
### 生成代码
使用 `--code` 参数，我们可以仅查询代码作为输出，例如：
```shell
sgpt --code "Solve classic fizz buzz problem using Python"
```
```python
for i in range(1, 101):
    if i % 3 == 0 and i % 5 == 0:
        print("FizzBuzz")
    elif i % 3 == 0:
        print("Fizz")
    elif i % 5 == 0:
        print("Buzz")
    else:
        print(i)
```
由于它是有效的 python 代码，我们可以将输出重定向到文件：
```shell
sgpt --code "solve classic fizz buzz problem using Python" > fizz_buzz.py
python fizz_buzz.py
# 1
# 2
# Fizz
# 4
# Buzz
# Fizz
# ...
```

### 聊天
要开始聊天会话，请使用 `--chat` 选项，后跟唯一的会话名称和提示：
```shell
sgpt --chat number "please remember my favorite number: 4"
# -> I will remember that your favorite number is 4.
sgpt --chat number "what would be my favorite number + 4?"
# -> Your favorite number is 4, so if we add 4 to it, the result would be 8.
```
您还可以使用聊天会话通过提供额外线索来迭代改进 ChatGPT 的建议：
```shell
sgpt --chat python_requst --code "make an example request to localhost using Python"
```
```python
import requests

response = requests.get('http://localhost')
print(response.text)
```
要求 ChatGPT 为我们的请求添加缓存。
```shell
sgpt --chat python_request --code "add caching"
```
```python
import requests
from cachecontrol import CacheControl

sess = requests.session()
cached_sess = CacheControl(sess)

response = cached_sess.get('http://localhost')
print(response.text)
```
我们可以使用 `--code` 或 `--shell` 选项来启动 `--chat`，这样您就可以不断优化结果：
```shell
sgpt --chat sh --shell "What are the files in this directory?"
# -> ls
sgpt --chat sh "Sort them by name"
# -> ls | sort
sgpt --chat sh "Concatenate them using FFMPEG"
# -> ffmpeg -i "concat:$(ls | sort | tr '\n' '|')" -codec copy output.mp4
sgpt --chat sh "Convert the resulting file into an MP3"
# -> ffmpeg -i output.mp4 -vn -acodec libmp3lame -ac 2 -ab 160k -ar 48000 final_output.mp3
```

### 聊天会话
要列出所有当前聊天会话，请使用 `--list-chat` 选项：
```shell
sgpt --list-chat
# .../shell_gpt/chat_cache/number
# .../shell_gpt/chat_cache/python_request
```
要显示与特定聊天会话相关的所有消息，请使用“--show-chat”选项后跟会话名称：
```shell
sgpt --show-chat number
# user: please remember my favorite number: 4
# assistant: I will remember that your favorite number is 4.
# user: what would be my favorite number + 4?
# assistant: Your favorite number is 4, so if we add 4 to it, the result would be 8.
```

### 请求缓存
使用“--cache”（默认）和“--no-cache”选项控制缓存。此缓存适用于对 OpenAI API 的所有“sgpt”请求：
```shell
sgpt "what are the colors of a rainbow"
# -> The colors of a rainbow are red, orange, yellow, green, blue, indigo, and violet.
```
下一次，相同的查询将立即从本地缓存中获取结果。请注意，`sgpt "what are the colors of a rainbow" --temperature 0.5` 将发出新请求，因为我们没有在之前的请求中提供 `--temperature`（同样适用于 `--top-probability`） .

这只是我们可以使用 ChatGPT 模型执行的操作的一些示例，我相信您会发现它对您的特定用例很有用。

### 运行时配置文件
您可以在运行时配置文件 `~/.config/shell_gpt/.sgptrc` 中设置一些参数：
```text
# API 密钥，也可以定义 OPENAI_API_KEY 环境。
OPENAI_API_KEY=your_api_key
# OpenAI 主机，如果您想使用代理，则很有用。
OPENAI_API_HOST=https://api.openai.com
# 每个聊天会话的最大缓存消息量。
CHAT_CACHE_LENGTH=100
# 聊天缓存文件夹。
CHAT_CACHE_PATH=/tmp/shell_gpt/chat_cache
# 请求缓存长度（数量）。
CACHE_LENGTH=100
# 请求缓存文件夹。
CACHE_PATH=/tmp/shell_gpt/cache
# 以秒为单位的请求超时。
REQUEST_TIMEOUT=60
```

### 完整的参数列表
```shell
╭─ Arguments ───────────────────────────────────────────────────────────────────────────────────────────────╮
│   prompt      [PROMPT] 生成补全的提示。                                                                      │
╰───────────────────────────────────────────────────────────────────────────────────────────────────────────╯
╭─ Options ─────────────────────────────────────────────────────────────────────────────────────────────────╮
│ --temperature      FLOAT RANGE [0.0<=x<=1.0]  生成输出的随机性. [默认: 1.0]                                   │
│ --top-probability  FLOAT RANGE [0.1<=x<=1.0]  限制最高可能的标记（单词）。 [默认：1.0]                           │
│ --chat             TEXT                       通过 id 关注对话（聊天模式）。[默认：None]                        │
│ --show-chat        TEXT                       显示来自提供的聊天 ID 的所有消息。 [默认：None]                    │
│ --list-chat                                   列出所有现有的聊天 ID。 [默认：no-list-chat]                     │
│ --shell                                       提供 shell 命令作为输出。                                      │
│ --execute                                     将执行 --shell 命令。                                         │
│ --code                                        提供代码作为输出。 [默认： no-code]                              │
│ --editor                                      打开 $EDITOR 以提供提示。 [默认：no-editor]                     │
│ --cache                                       缓存完成结果。 [默认：cache]                                    │
│ --animation                                   打字机动画。 [默认：animation]                                 │
│ --spinner                                     在 API 请求期间显示加载微调器。 [默认：spinner]                   │
│ --help                                        显示此消息并退出。                                             │
╰───────────────────────────────────────────────────────────────────────────────────────────────────────────╯
```


## Docker
使用 `OPENAI_API_KEY` 环境变量和用于存储缓存的 docker 卷运行容器：
```shell
docker run --rm \
           --env OPENAI_API_KEY="your OPENAI API key" \
           --volume gpt-cache:/tmp/shell_gpt \
       ghcr.io/ther1d/shell_gpt --chat rainbow "what are the colors of a rainbow"
```

使用别名和“OPENAI_API_KEY”环境变量的对话示例：
```shell
alias sgpt="docker run --rm --env OPENAI_API_KEY --volume gpt-cache:/tmp/shell_gpt ghcr.io/ther1d/shell_gpt"
export OPENAI_API_KEY="your OPENAI API key"
sgpt --chat rainbow "what are the colors of a rainbow"
sgpt --chat rainbow "inverse the list of your last answer"
sgpt --chat rainbow "translate your last answer in french"
```

您还可以使用提供的`Dockerfile`构建您自己的镜像：
```shell
docker build -t sgpt .
```
