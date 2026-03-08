# 游玩记录

游玩记录API用于管理用户的游戏分数和评级数据。

## 获取用户游玩记录

### 获取记录列表
获取指定用户的游玩记录，支持分页、排序和筛选。

**端点**: `GET /records/{username}`

**路径参数**:
| 参数名 | 类型 | 必填 | 描述 |
|--------|------|------|------|
| username | string | 是 | 用户名（不区分大小写） |

**查询参数**:
| 参数名 | 类型 | 必填 | 描述 |
|--------|------|------|------|
| scope | string | 否 | 记录范围："b50" (默认) 或 "all" |
| page_size | integer | 否 | 每页记录数，默认50 |
| sort_by | string | 否 | 排序字段："rating" (默认) 或 "score" |
| order | string | 否 | 排序顺序："desc" (默认) 或 "asc" |

**认证**: 需要Bearer Token（只能访问自己的记录或公开记录）

**成功响应** (200 OK):
```json
{
  "records": [
    {
      "song_id": "song_001",
      "song_title": "Paradigm",
      "difficulty": "Massive",
      "level": 1.0,
      "score": 950000,
      "rating": 12.5,
      "played_at": "2024-01-01T12:00:00Z",
      "song_level_id": "song_001_massive"
    },
    {
      "song_id": "song_002",
      "song_title": "Another Song",
      "difficulty": "Invaded",
      "level": 2.0,
      "score": 980000,
      "rating": 15.2,
      "played_at": "2024-01-01T13:00:00Z",
      "song_level_id": "song_002_invaded"
    }
  ],
  "pagination": {
    "total": 120,
    "page_size": 50,
    "current_page": 1,
    "total_pages": 3
  },
  "summary": {
    "total_records": 120,
    "average_rating": 14.8,
    "best_rating": 16.5,
    "last_updated": "2024-01-02T00:00:00Z"
  }
}
```

## 提交游玩记录

### 上传分数
提交新的游玩记录或更新现有记录。

**端点**: `POST /records/{username}`

**路径参数**:
| 参数名 | 类型 | 必填 | 描述 |
|--------|------|------|------|
| username | string | 是 | 用户名（不区分大小写） |

**认证**: 需要Bearer Token（只能提交到自己的账户）

**请求格式**: `application/json`

**请求体**:
```json
{
  "play_records": [
    {
      "song_level_id": "song_001_massive",
      "score": 950000
    }
  ],
  "is_replace": false,
  "upload_token": "optional_upload_token"
}
```

**字段说明**:
| 字段 | 类型 | 必填 | 描述 |
|------|------|------|------|
| play_records | array | 是 | 游玩记录数组 |
| play_records[].song_level_id | string | 是 | 歌曲难度唯一标识符 |
| play_records[].score | integer | 是 | 分数 (0-1000000) |
| is_replace | boolean | 否 | 是否覆盖最佳记录，默认false |
| upload_token | string | 否 | 上传令牌（某些操作需要） |

**成功响应** (200 OK 或 201 Created):
```json
{
  "success": true,
  "message": "记录已提交",
  "records": [
    {
      "song_level_id": "song_001_massive",
      "score": 950000,
      "rating": 12.5,
      "is_new_best": true,
      "previous_best": null
    }
  ]
}
```

## 导出B50图片

### 生成B50图片
生成Best 50（最佳50首歌）的评级图片。

**端点**: `GET /records/{username}/export/b50`

**路径参数**:
| 参数名 | 类型 | 必填 | 描述 |
|--------|------|------|------|
| username | string | 是 | 用户名（不区分大小写） |

**认证**: 需要Bearer Token

**成功响应** (200 OK):
- Content-Type: `image/png`
- 响应体为PNG图片的二进制数据

**示例**:
```bash
curl -H "Authorization: Bearer <token>" \
  "https://api.prp.icel.site/records/username/export/b50" \
  --output b50_image.png
```



## Python示例

```python
import aiohttp
import asyncio
from typing import Dict, Any, Optional, List

class PRPRecords:
    def __init__(self, base_url="https://api.prp.icel.site"):
        self.base_url = base_url

    async def get_user_records(self, username: str, access_token: str,
                              scope: str = "b50", page_size: int = 50) -> Dict[str, Any]:
        """获取用户游玩记录"""
        url = f"{self.base_url}/records/{username.lower()}"
        params = {
            'scope': scope,
            'page_size': page_size,
            'sort_by': 'rating',
            'order': 'desc'
        }
        headers = {'Authorization': f'Bearer {access_token}'}

        async with aiohttp.ClientSession() as session:
            async with session.get(url, params=params, headers=headers) as response:
                if response.status == 200:
                    return await response.json()
                else:
                    raise Exception(f"获取用户记录失败: {response.status}")

    async def upload_score(self, username: str, access_token: str,
                          song_level_id: str, score: int,
                          overwrite_best: bool = False) -> Dict[str, Any]:
        """上传分数（使用song_level_id）"""
        url = f"{self.base_url}/records/{username.lower()}"
        headers = {
            'Authorization': f'Bearer {access_token}',
            'Content-Type': 'application/json'
        }

        payload = {
            "play_records": [{
                "song_level_id": song_level_id,
                "score": score
            }],
            "is_replace": overwrite_best
        }

        async with aiohttp.ClientSession() as session:
            async with session.post(url, json=payload, headers=headers) as response:
                if response.status in (200, 201):
                    return await response.json()
                else:
                    error_text = await response.text()
                    raise Exception(f"上传分数失败: {response.status}, {error_text}")

    async def get_b50_image(self, username: str, access_token: str) -> bytes:
        """获取B50图片"""
        url = f"{self.base_url}/records/{username.lower()}/export/b50"
        headers = {'Authorization': f'Bearer {access_token}'}

        async with aiohttp.ClientSession() as session:
            async with session.get(url, headers=headers) as response:
                if response.status == 200:
                    return await response.read()
                else:
                    raise Exception(f"获取B50图片失败: {response.status}")


# 使用示例
async def main():
    records_client = PRPRecords()
    access_token = "your_access_token"

    # 获取用户记录
    user_records = await records_client.get_user_records("username", access_token)
    print(f"总记录数: {user_records['pagination']['total']}")

    # 上传分数
    result = await records_client.upload_score("username", access_token,
                                              "song_001_massive", 950000)
    print(f"上传结果: {result['message']}")

    # 获取B50图片
    image_data = await records_client.get_b50_image("username", access_token)
    with open("b50.png", "wb") as f:
        f.write(image_data)
    print("B50图片已保存")

# asyncio.run(main())
```

## 数据结构

### 游玩记录
| 字段 | 类型 | 描述 |
|------|------|------|
| song_id | string | 歌曲ID |
| song_title | string | 歌曲标题 |
| difficulty | string | 难度名称 |
| level | number | 难度定数 |
| score | integer | 分数 (0-1000000) |
| rating | number | 计算出的评级 |
| played_at | string | 游玩时间（ISO 8601） |
| song_level_id | string | 歌曲难度唯一标识符 |


## 注意事项

1. **权限控制**: 用户只能访问自己的记录或公开记录
2. **分数范围**: 分数应在0-1000000之间
3. **重复上传**: 默认不覆盖最佳记录，除非设置`is_replace: true`
4. **song_level_id**: 上传分数时需要正确的`song_level_id`，可通过歌曲API获取
5. **图片格式**: B50图片为PNG格式，可能需要处理二进制数据
6. **分页**: 记录API支持分页，默认返回50条记录