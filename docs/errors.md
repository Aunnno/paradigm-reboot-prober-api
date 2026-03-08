# 错误处理

本文档描述了PRP API可能返回的错误代码和响应格式。

## 错误响应格式

所有错误响应都遵循相同的JSON格式：

```json
{
  "detail": "错误描述信息",
  "error_code": "ERROR_CODE",
  "status_code": 400
}
```

**字段说明**:
| 字段 | 类型 | 描述 |
|------|------|------|
| detail | string | 人类可读的错误描述 |
| error_code | string | 机器可读的错误代码 |
| status_code | integer | HTTP状态代码 |

## HTTP状态代码

PRP API使用标准的HTTP状态代码：

| 状态码 | 描述 |
|--------|------|
| 200 OK | 请求成功 |
| 201 Created | 资源创建成功 |
| 400 Bad Request | 请求格式错误 |
| 401 Unauthorized | 认证失败或令牌无效 |
| 403 Forbidden | 权限不足 |
| 404 Not Found | 资源不存在 |
| 409 Conflict | 资源冲突（如用户名已存在） |
| 422 Unprocessable Entity | 数据验证失败 |
| 429 Too Many Requests | 请求过于频繁 |
| 500 Internal Server Error | 服务器内部错误 |

## 常见错误代码

### 认证错误 (401)
| 错误代码 | 描述 | 解决方案 |
|----------|------|----------|
| INVALID_CREDENTIALS | 用户名或密码错误 | 检查凭据是否正确 |
| TOKEN_EXPIRED | 访问令牌已过期 | 重新登录获取新令牌 |
| INVALID_TOKEN | 令牌格式无效 | 检查令牌格式 |
| MISSING_TOKEN | 缺少认证令牌 | 在请求头中添加Authorization |

### 权限错误 (403)
| 错误代码 | 描述 | 解决方案 |
|----------|------|----------|
| PERMISSION_DENIED | 没有访问权限 | 检查用户权限 |
| NOT_OWNER | 不能访问其他用户的资源 | 只能访问自己的资源 |
| ADMIN_REQUIRED | 需要管理员权限 | 使用管理员账户 |

### 资源错误 (404)
| 错误代码 | 描述 | 解决方案 |
|----------|------|----------|
| USER_NOT_FOUND | 用户不存在 | 检查用户名 |
| SONG_NOT_FOUND | 歌曲不存在 | 检查歌曲ID或标题 |
| RECORD_NOT_FOUND | 游玩记录不存在 | 检查记录ID |

### 验证错误 (400, 422)
| 错误代码 | 描述 | 解决方案 |
|----------|------|----------|
| VALIDATION_ERROR | 数据验证失败 | 检查请求数据格式 |
| INVALID_SCORE | 分数超出范围 (0-1000000) | 调整分数值 |
| INVALID_DIFFICULTY | 难度无效 | 使用有效的难度值 |
| MISSING_REQUIRED_FIELD | 缺少必填字段 | 提供所有必填字段 |

### 冲突错误 (409)
| 错误代码 | 描述 | 解决方案 |
|----------|------|----------|
| USERNAME_EXISTS | 用户名已存在 | 选择其他用户名 |
| DUPLICATE_RECORD | 重复的记录 | 使用overwrite参数或跳过 |

### 请求限制 (429)
| 错误代码 | 描述 | 解决方案 |
|----------|------|----------|
| RATE_LIMIT_EXCEEDED | 请求过于频繁 | 降低请求频率 |

## 错误处理示例

### Python错误处理
```python
import aiohttp
import asyncio
from typing import Dict, Any

class PRPClient:
    def __init__(self, base_url="https://api.prp.icel.site"):
        self.base_url = base_url

    async def handle_request(self, method: str, endpoint: str, **kwargs) -> Dict[str, Any]:
        """处理请求并统一错误处理"""
        url = f"{self.base_url}{endpoint}"

        try:
            async with aiohttp.ClientSession() as session:
                async with session.request(method, url, **kwargs) as response:
                    data = await response.json() if response.content_length else {}

                    if 200 <= response.status < 300:
                        return data
                    else:
                        # 提取错误信息
                        error_detail = data.get('detail', 'Unknown error')
                        error_code = data.get('error_code', 'UNKNOWN_ERROR')

                        # 根据错误代码进行特定处理
                        if response.status == 401:
                            if error_code == 'TOKEN_EXPIRED':
                                raise TokenExpiredError(error_detail)
                            else:
                                raise AuthenticationError(error_detail)
                        elif response.status == 403:
                            raise PermissionError(error_detail)
                        elif response.status == 404:
                            raise NotFoundError(error_detail)
                        elif response.status == 429:
                            raise RateLimitError(error_detail)
                        else:
                            raise APIError(f"{error_code}: {error_detail}", response.status)

        except aiohttp.ClientError as e:
            raise ConnectionError(f"网络连接错误: {str(e)}")

# 自定义异常类
class APIError(Exception):
    def __init__(self, message: str, status_code: int = 500):
        super().__init__(message)
        self.status_code = status_code

class AuthenticationError(APIError):
    def __init__(self, message: str):
        super().__init__(message, 401)

class PermissionError(APIError):
    def __init__(self, message: str):
        super().__init__(message, 403)

class NotFoundError(APIError):
    def __init__(self, message: str):
        super().__init__(message, 404)

class RateLimitError(APIError):
    def __init__(self, message: str):
        super().__init__(message, 429)

class TokenExpiredError(AuthenticationError):
    def __init__(self, message: str = "访问令牌已过期"):
        super().__init__(message)

# 使用示例
async def example():
    client = PRPClient()

    try:
        # 尝试访问需要认证的端点
        result = await client.handle_request('GET', '/user/me',
                                           headers={'Authorization': 'Bearer invalid_token'})
        print(f"结果: {result}")

    except TokenExpiredError:
        print("令牌过期，需要重新登录")
        # 重新登录逻辑
    except AuthenticationError as e:
        print(f"认证失败: {e}")
    except PermissionError as e:
        print(f"权限不足: {e}")
    except NotFoundError as e:
        print(f"资源未找到: {e}")
    except RateLimitError as e:
        print(f"请求过于频繁: {e}")
        # 等待一段时间后重试
    except APIError as e:
        print(f"API错误 ({e.status_code}): {e}")
    except ConnectionError as e:
        print(f"网络错误: {e}")
```

### 重试机制
对于某些可恢复的错误（如令牌过期、速率限制），可以实现重试机制：

```python
import asyncio
import time
from typing import Callable, Any

async def retry_with_backoff(
    func: Callable,
    max_retries: int = 3,
    base_delay: float = 1.0,
    max_delay: float = 30.0,
    retry_on: tuple = (TokenExpiredError, RateLimitError, ConnectionError)
):
    """指数退避重试机制"""
    retries = 0
    last_exception = None

    while retries <= max_retries:
        try:
            return await func()
        except retry_on as e:
            last_exception = e
            retries += 1

            if retries > max_retries:
                break

            # 计算等待时间（指数退避）
            delay = min(base_delay * (2 ** (retries - 1)), max_delay)
            print(f"重试 {retries}/{max_retries}, 等待 {delay}秒...")
            await asyncio.sleep(delay)

    raise last_exception

# 使用重试机制
async def make_request_with_retry():
    client = PRPClient()

    async def request_func():
        return await client.handle_request('GET', '/user/me',
                                         headers={'Authorization': 'Bearer token'})

    try:
        return await retry_with_backoff(request_func)
    except Exception as e:
        print(f"所有重试都失败: {e}")
        raise
```

## 调试建议

### 1. 检查请求格式
```python
# 打印请求详情
import pprint

def debug_request(method, url, **kwargs):
    print(f"请求: {method} {url}")
    if 'headers' in kwargs:
        print("请求头:")
        pprint.pprint(kwargs['headers'])
    if 'json' in kwargs:
        print("请求体:")
        pprint.pprint(kwargs['json'])
```

### 2. 使用日志记录
```python
import logging

logging.basicConfig(level=logging.DEBUG)
logger = logging.getLogger(__name__)

# 在请求前后记录
logger.debug(f"发送请求到 {url}")
response = await session.request(...)
logger.debug(f"收到响应: {response.status}")
```

### 3. 验证数据
```python
def validate_upload_data(song_title: str, difficulty: str, score: int):
    errors = []

    if not song_title or len(song_title.strip()) == 0:
        errors.append("歌曲标题不能为空")

    valid_difficulties = ['Massive', 'Invaded', 'Detected', 'Reboot',
                         'M', 'I', 'D', 'R']
    if difficulty not in valid_difficulties:
        errors.append(f"难度无效。有效值: {', '.join(valid_difficulties)}")

    if not (0 <= score <= 1000000):
        errors.append(f"分数必须在0-1000000之间，当前: {score}")

    return errors
```

## 常见问题解决

### Q: 收到401错误但凭据正确
A: 检查用户名和密码是否包含特殊字符，确保正确编码。

### Q: 上传分数失败但歌曲存在
A: 确保使用正确的`song_level_id`，而不是歌曲ID。通过歌曲API获取正确的`song_level_id`。

### Q: CSV上传部分失败
A: 检查失败记录的错误信息。常见问题包括歌曲标题不匹配、难度格式错误、分数超出范围。

### Q: 获取B50图片返回错误
A: 确保用户名正确且用户有足够的游玩记录生成B50图片。

### Q: 请求被限制 (429)
A: 降低请求频率，实现指数退避重试机制。