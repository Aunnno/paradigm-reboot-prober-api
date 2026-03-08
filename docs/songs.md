# 歌曲管理

歌曲管理API提供了对《Paradigm: Reboot》游戏歌曲信息的访问和管理功能。

## 获取歌曲列表

### 列出所有歌曲
获取游戏中的所有歌曲，包含各个难度的信息。

**端点**: `GET /songs`

**参数**:
| 参数名 | 类型 | 必填 | 描述 |
|--------|------|------|------|
| - | - | - | 目前无查询参数 |

**认证**: 可选。如果提供认证令牌，响应会包含用户的游玩记录。

**成功响应** (200 OK):
```json
[
  {
    "song_id": "song_001",
    "title": "Paradigm",
    "artist": "Artist Name",
    "genre": "Original",
    "bpm": 180,
    "cover": "https://example.com/cover.jpg",
    "difficulty": "Massive",
    "level": 1.0,
    "difficulty_id": "massive",
    "song_level_id": "song_001_massive"
  },
  {
    "song_id": "song_001",
    "title": "Paradigm",
    "artist": "Artist Name",
    "genre": "Original",
    "bpm": 180,
    "cover": "https://example.com/cover.jpg",
    "difficulty": "Invaded",
    "level": 2.0,
    "difficulty_id": "invaded",
    "song_level_id": "song_001_invaded"
  }
  // ...更多难度条目
]
```

**注意**: API返回的是每个难度的独立条目。通常需要按`song_id`分组来获取完整的歌曲信息。

## 获取歌曲详情

### 获取单个歌曲信息
获取指定歌曲的详细信息，包含所有难度和可能的用户游玩记录。

**端点**: `GET /songs/{song_id}`

**路径参数**:
| 参数名 | 类型 | 必填 | 描述 |
|--------|------|------|------|
| song_id | string | 是 | 歌曲ID |

**查询参数**:
| 参数名 | 类型 | 必填 | 描述 |
|--------|------|------|------|
| src | string | 否 | 数据来源，默认为"prp" |

**认证**: 可选。如果提供认证令牌，响应会包含用户的游玩记录。

**成功响应** (200 OK):
```json
{
  "song": {
    "id": "song_001",
    "title": "Paradigm",
    "artist": "Artist Name",
    "genre": "Original",
    "bpm": 180,
    "cover": "https://example.com/cover.jpg",
    "length": 120,
    "notes": 500,
    "released_at": "2023-01-01"
  },
  "levels": [
    {
      "difficulty": "Massive",
      "level": 1.0,
      "difficulty_id": "massive",
      "song_level_id": "song_001_massive",
      "notes": 300,
      "designer": "Designer A"
    },
    {
      "difficulty": "Invaded",
      "level": 2.0,
      "difficulty_id": "invaded",
      "song_level_id": "song_001_invaded",
      "notes": 400,
      "designer": "Designer B"
    }
  ],
  "user_records": [
    {
      "score": 950000,
      "rating": 12.5,
      "played_at": "2024-01-01T12:00:00Z"
    }
  ]
}
```



## Python示例

```python
import aiohttp
import asyncio
from typing import List, Dict, Any

class PRPSongs:
    def __init__(self, base_url="https://api.prp.icel.site"):
        self.base_url = base_url

    async def get_songs(self, access_token: str = None) -> List[Dict[str, Any]]:
        """获取歌曲列表"""
        url = f"{self.base_url}/songs"
        headers = {}
        if access_token:
            headers['Authorization'] = f'Bearer {access_token}'

        async with aiohttp.ClientSession() as session:
            async with session.get(url, headers=headers) as response:
                if response.status == 200:
                    return await response.json()
                else:
                    raise Exception(f"获取歌曲列表失败: {response.status}")

    async def search_songs(self, query: str, access_token: str = None) -> List[Dict[str, Any]]:
        """搜索歌曲（客户端实现）"""
        all_songs = await self.get_songs(access_token)

        # 按song_id去重
        unique_songs = {}
        for item in all_songs:
            song_id = item.get('song_id')
            if song_id not in unique_songs:
                unique_songs[song_id] = {
                    'id': song_id,
                    'title': item.get('title'),
                    'artist': item.get('artist'),
                    'genre': item.get('genre'),
                    'bpm': item.get('bpm'),
                    'cover': item.get('cover'),
                    'difficulties': []
                }

            # 添加难度信息
            difficulty_info = {
                'difficulty': item.get('difficulty'),
                'level': item.get('level'),
                'difficulty_id': item.get('difficulty_id'),
                'song_level_id': item.get('song_level_id')
            }
            unique_songs[song_id]['difficulties'].append(difficulty_info)

        # 搜索匹配的歌曲
        matched = []
        for song in unique_songs.values():
            if query.lower() in song.get('title', '').lower():
                matched.append(song)

        return matched

    async def get_song_details(self, song_id: str, access_token: str = None) -> Dict[str, Any]:
        """获取歌曲详情"""
        url = f"{self.base_url}/songs/{song_id}"
        params = {'src': 'prp'}
        headers = {}
        if access_token:
            headers['Authorization'] = f'Bearer {access_token}'

        async with aiohttp.ClientSession() as session:
            async with session.get(url, params=params, headers=headers) as response:
                if response.status == 200:
                    return await response.json()
                else:
                    raise Exception(f"获取歌曲详情失败: {response.status}")

# 使用示例
async def main():
    songs_client = PRPSongs()

    # 搜索歌曲
    matched_songs = await songs_client.search_songs("paradigm")
    for song in matched_songs:
        print(f"歌曲: {song['title']}, 艺术家: {song['artist']}")
        for diff in song['difficulties']:
            print(f"  难度: {diff['difficulty']}, 定数: {diff['level']}")

# asyncio.run(main())
```

## 数据结构

### 歌曲信息
| 字段 | 类型 | 描述 |
|------|------|------|
| id / song_id | string | 歌曲唯一标识符 |
| title | string | 歌曲标题 |
| artist | string | 艺术家 |
| genre | string | 音乐类型 |
| bpm | number | 每分钟节拍数 |
| cover | string | 封面图片URL |
| length | number | 歌曲长度（秒） |
| notes | number | 总音符数 |
| released_at | string | 发布日期（ISO 8601） |

### 难度信息
| 字段 | 类型 | 描述 |
|------|------|------|
| difficulty | string | 难度名称（Massive/Invaded/Detected/Reboot） |
| level | number | 难度定数 |
| difficulty_id | string | 难度标识符 |
| song_level_id | string | 歌曲难度唯一标识符（用于上传分数） |
| notes | number | 该难度的音符数 |
| designer | string | 谱面设计师 |

## 注意事项

1. **数据格式**: API返回的是每个难度的独立条目，需要客户端进行聚合
2. **搜索功能**: 搜索是在客户端实现的，服务器端目前不提供搜索参数
3. **缓存**: 考虑缓存歌曲数据以减少API调用