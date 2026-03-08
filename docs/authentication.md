# 认证

PRP API 使用基于OAuth2的Bearer Token认证方式。所有受保护的端点都需要在请求头中包含有效的访问令牌。

## 获取访问令牌

### 用户登录
通过`/user/login`端点获取访问令牌。

**端点**: `POST /user/login`

**请求格式**: `application/x-www-form-urlencoded`

**参数**:
| 参数名 | 类型 | 必填 | 描述 |
|--------|------|------|------|
| username | string | 是 | 用户名 |
| password | string | 是 | 密码 |

**示例请求**:
```bash
curl -X POST "https://api.prp.icel.site/user/login" \
  -d "username=your_username" \
  -d "password=your_password"
```

**成功响应** (200 OK):
```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "token_type": "bearer"
}
```

**错误响应**:
- 401 Unauthorized: 用户名或密码错误
- 400 Bad Request: 请求格式错误

## 使用访问令牌

获取访问令牌后，在后续请求的`Authorization`头中使用它：

```
Authorization: Bearer <access_token>
```

**示例**:
```bash
curl -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..." \
  "https://api.prp.icel.site/user/me"
```

## 令牌有效期

访问令牌目前没有严格的过期时间限制，但建议在长时间不使用时重新登录获取新令牌。


## 获取当前用户信息

### 获取用户信息
获取当前认证用户的详细信息。

**端点**: `GET /user/me`

**认证**: 需要Bearer Token

**成功响应** (200 OK):
```json
{
  "id": 123,
  "username": "your_username",
  "email": "user@example.com",
  "created_at": "2024-01-01T00:00:00Z",
  "updated_at": "2024-01-01T00:00:00Z"
}
```


## 获取上传令牌

某些上传操作需要额外的上传令牌。

**端点**: `POST /user/me/upload-token`

**认证**: 需要Bearer Token

**成功响应** (200 OK):
```json
{
  "upload_token": "upload_token_123456"
}
```

## Python示例

```python
import aiohttp
import asyncio

class PRPAuth:
    def __init__(self, base_url="https://api.prp.icel.site"):
        self.base_url = base_url
        self.access_token = None

    async def login(self, username: str, password: str) -> bool:
        """用户登录"""
        url = f"{self.base_url}/user/login"
        form_data = aiohttp.FormData()
        form_data.add_field('username', username)
        form_data.add_field('password', password)

        async with aiohttp.ClientSession() as session:
            async with session.post(url, data=form_data) as response:
                if response.status == 200:
                    data = await response.json()
                    self.access_token = data.get('access_token')
                    return True
                else:
                    print(f"登录失败: {response.status}")
                    return False

    async def get_user_info(self) -> dict:
        """获取当前用户信息"""
        if not self.access_token:
            raise ValueError("请先登录")

        url = f"{self.base_url}/user/me"
        headers = {'Authorization': f'Bearer {self.access_token}'}

        async with aiohttp.ClientSession() as session:
            async with session.get(url, headers=headers) as response:
                if response.status == 200:
                    return await response.json()
                else:
                    raise Exception(f"获取用户信息失败: {response.status}")

# 使用示例
async def main():
    auth = PRPAuth()
    if await auth.login("your_username", "your_password"):
        user_info = await auth.get_user_info()
        print(f"用户信息: {user_info}")

# asyncio.run(main())
```

## 注意事项

1. **安全存储**: 请妥善保管访问令牌，避免泄露
2. **密码安全**: 不要在客户端硬编码密码
3. **错误处理**: 始终处理认证失败的情况
4. **重试机制**: 对于过期令牌，应重新登录获取新令牌