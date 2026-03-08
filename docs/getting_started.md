# 快速开始

本指南将帮助您快速开始使用PRP API。

## 前提条件

1. **Python 3.8+**: 确保已安装Python 3.8或更高版本
2. **网络访问**: 能够访问 `https://api.prp.icel.site`
3. **PRP账户**: 在[PRP网站](https://prp.icel.site)注册账户

## 安装

### 方法1: 直接使用requests库
```bash
pip install requests
```

### 方法2: 使用aiohttp进行异步请求
```bash
pip install aiohttp
```

### 方法3: 安装完整依赖（包含示例）
```bash
pip install -r requirements.txt
```

## 第一步: 获取访问令牌

所有API请求都需要认证。首先获取访问令牌：

```python
import requests

# 登录获取令牌
response = requests.post(
    "https://api.prp.icel.site/user/login",
    data={"username": "your_username", "password": "your_password"}
)

if response.status_code == 200:
    token_data = response.json()
    access_token = token_data["access_token"]
    print(f"访问令牌: {access_token}")
else:
    print(f"登录失败: {response.status_code}")
```

## 第二步: 调用API

获取令牌后，就可以调用受保护的API了：

```python
# 使用令牌访问API
headers = {"Authorization": f"Bearer {access_token}"}

# 获取用户信息
response = requests.get(
    "https://api.prp.icel.site/user/me",
    headers=headers
)

if response.status_code == 200:
    user_info = response.json()
    print(f"用户信息: {user_info}")
```

## 常用操作示例

### 1. 搜索歌曲
```python
def search_songs(query: str, access_token: str):
    # 获取所有歌曲
    headers = {"Authorization": f"Bearer {access_token}"}
    response = requests.get("https://api.prp.icel.site/songs", headers=headers)

    if response.status_code == 200:
        songs = response.json()

        # 客户端搜索（API目前不支持服务器端搜索）
        results = []
        for song in songs:
            if query.lower() in song.get('title', '').lower():
                results.append(song)

        return results
```

### 2. 上传分数
```python
def upload_score(username: str, access_token: str,
                 song_level_id: str, score: int):
    url = f"https://api.prp.icel.site/records/{username}"
    headers = {
        "Authorization": f"Bearer {access_token}",
        "Content-Type": "application/json"
    }

    payload = {
        "play_records": [{
            "song_level_id": song_level_id,
            "score": score
        }],
        "is_replace": False
    }

    response = requests.post(url, json=payload, headers=headers)
    return response.json()
```

### 3. 获取B50图片
```python
def get_b50_image(username: str, access_token: str, output_file: str = "b50.png"):
    url = f"https://api.prp.icel.site/records/{username}/export/b50"
    headers = {"Authorization": f"Bearer {access_token}"}

    response = requests.get(url, headers=headers, stream=True)

    if response.status_code == 200:
        with open(output_file, 'wb') as f:
            for chunk in response.iter_content(chunk_size=8192):
                f.write(chunk)
        print(f"图片已保存到: {output_file}")
```

## 异步示例

如果您需要高性能或并发请求，建议使用异步客户端：

```python
import aiohttp
import asyncio

async def async_example():
    async with aiohttp.ClientSession() as session:
        # 登录
        form_data = aiohttp.FormData()
        form_data.add_field('username', 'your_username')
        form_data.add_field('password', 'your_password')

        async with session.post(
            "https://api.prp.icel.site/user/login",
            data=form_data
        ) as response:
            token_data = await response.json()
            access_token = token_data["access_token"]

        # 使用令牌
        headers = {"Authorization": f"Bearer {access_token}"}
        async with session.get(
            "https://api.prp.icel.site/user/me",
            headers=headers
        ) as response:
            user_info = await response.json()
            print(user_info)

# 运行异步函数
# asyncio.run(async_example())
```

## 错误处理

始终处理可能的错误：

```python
import requests
from requests.exceptions import RequestException

def safe_api_call(func, *args, **kwargs):
    try:
        response = func(*args, **kwargs)
        response.raise_for_status()  # 检查HTTP错误
        return response.json()
    except RequestException as e:
        print(f"请求错误: {e}")
        return None
    except ValueError as e:
        print(f"JSON解析错误: {e}")
        return None

# 使用安全调用
result = safe_api_call(
    requests.get,
    "https://api.prp.icel.site/user/me",
    headers=headers
)
```

## 运行示例代码

本项目包含完整的示例代码：

1. **基础示例**: `examples/basic_usage.py`
   ```bash
   python examples/basic_usage.py
   ```

2. **高级客户端**: `examples/advanced_client.py`
   ```bash
   python examples/advanced_client.py
   ```

3. **命令行工具**: `examples/cli_tool.py`
   ```bash
   # 查看帮助
   python examples/cli_tool.py --help

   # 登录
   python examples/cli_tool.py login username password

   # 搜索歌曲
   python examples/cli_tool.py search "歌曲名"

   # 上传分数
   python examples/cli_tool.py upload "歌曲名" "难度" 分数
   ```

## 下一步

- 查看[API端点文档](songs.md)了解所有可用端点
- 学习[错误处理](errors.md)最佳实践
- 探索[高级功能](upload.md)如文件上传
- 参考[示例代码](../examples/)获取完整实现

## 获取帮助

- **API文档**: 访问 [https://api.prp.icel.site/docs](https://api.prp.icel.site/docs) 查看交互式文档
- **问题反馈**: 在GitHub仓库提交Issue
- **社区支持**: 加入PRP Discord社区获取帮助