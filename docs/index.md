# PRP API 文档

欢迎使用Paradigm Reboot Prober (PRP) API文档。PRP是一个为音游《Paradigm: Reboot》开发的查分器网站。

## 文档导航

### 入门指南
- [快速开始](getting_started.md) - 立即开始使用API
- [认证](authentication.md) - 用户登录和令牌管理

### API参考
- [歌曲管理](songs.md) - 歌曲搜索和详细信息
- [游玩记录](records.md) - 分数上传和记录查询
- [文件上传](upload.md) - CSV和图片上传
- [错误处理](errors.md) - 错误代码和故障排除

### 资源
- [示例代码](../examples/) - 完整的代码示例
- [交互式文档](https://api.prp.icel.site/docs) - Swagger UI
- [GitHub仓库](https://github.com/PRProber/paradigm-reboot-prober-backend) - 后端源代码

## API概述

PRP API提供以下主要功能：

### 用户系统
- 用户注册和登录
- 个人信息管理
- 访问令牌认证

### 歌曲数据
- 完整的歌曲数据库
- 多难度谱面信息
- 歌曲搜索功能

### 游玩记录
- 分数上传和更新
- 个人游玩记录查询
- B50（最佳50首歌）计算

### 数据导出
- B50图片生成
- CSV数据导出
- 统计趋势分析

## 基本概念

### 认证方式
PRP API使用OAuth2 Bearer Token进行认证。所有受保护的端点都需要在请求头中包含有效的访问令牌。

### 数据结构
- **歌曲 (Song)**: 游戏中的歌曲，包含多个难度
- **难度 (Difficulty)**: 歌曲的难度等级（Massive/Invaded/Detected/Reboot）
- **游玩记录 (PlayRecord)**: 用户的游戏分数记录
- **B50**: 用户最佳的50首歌的评级总和

### 常用术语
- **song_level_id**: 歌曲和难度的唯一组合标识符，用于上传分数
- **rating**: 根据分数和歌曲定数计算出的评级
- **scope**: 记录查询范围，"b50"表示最佳50首，"all"表示所有记录

## 快速示例

```python
import requests

# 1. 登录获取令牌
response = requests.post("https://api.prp.icel.site/user/login",
                         data={"username": "test", "password": "test123"})
token = response.json()["access_token"]

# 2. 访问受保护端点
headers = {"Authorization": f"Bearer {token}"}
response = requests.get("https://api.prp.icel.site/user/me", headers=headers)
print(response.json())
```

## 支持与反馈

- **问题报告**: 在GitHub仓库提交Issue

## 更新日志

### v1.0 (当前)
- 完整的用户认证系统
- 歌曲数据库查询
- 游玩记录管理
- 文件上传功能
- 数据导出（B50图片、CSV）

---

**注意**: 本API仅用于个人非商业用途。请勿滥用API，尊重服务器资源。