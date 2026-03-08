# Paradigm Reboot Prober API 文档

[中文](README.md) | [English](README_EN.md)

Paradigm Reboot Prober (PRP) 是一个为音游《Paradigm: Reboot》开发的查分器网站。本文档提供了部分API参考。

## 快速开始

### 基础URL
```
https://api.prp.icel.site
```

### 认证方式
PRP API使用OAuth2密码流程进行认证。首先通过`/user/login`端点获取访问令牌，然后在后续请求的`Authorization`头中使用该令牌。

```python
import requests

# 获取访问令牌
response = requests.post("https://api.prp.icel.site/user/login",
                         data={"username": "your_username", "password": "your_password"})
token = response.json()["access_token"]

# 使用令牌访问受保护的端点
headers = {"Authorization": f"Bearer {token}"}
response = requests.get("https://api.prp.icel.site/user/me", headers=headers)
```

### 基本示例
查看[示例代码](examples/)了解各种编程语言的完整示例。

## API端点概览

### 用户认证
- `POST /user/login` - 用户登录获取访问令牌
- `GET /user/me` - 获取当前用户信息
- `POST /user/me/upload-token` - 获取上传令牌

### 歌曲管理
- `GET /songs` - 获取歌曲列表
- `GET /songs/{song_id}` - 获取指定歌曲详情

### 游玩记录
- `GET /records/{username}` - 获取用户的游玩记录
- `POST /records/{username}` - 提交游玩记录
- `GET /records/{username}/export/b50` - 导出B50图片


## 详细文档
- [用户认证](docs/authentication.md)
- [歌曲管理](docs/songs.md)
- [游玩记录](docs/records.md)
- [错误处理](docs/errors.md)


## 贡献
欢迎提交Issue和Pull Request来改进本文档。

## 许可证
本项目采用MIT许可证 - 查看[LICENSE](LICENSE)文件了解详情。