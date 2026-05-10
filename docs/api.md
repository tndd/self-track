# API Interface

DBのスキーマ（`docs/scheme.md`）は正規化されていますが、実際のAPIはフロントエンドの「3つのタブ（Track, Daily, Sleep）」それぞれの画面構成で扱いやすい形に最適化して提供します。

## 1. Track (Track Tab)

時々の出来事（イベント）の記録や、そのタイムライン表示のためのAPIです。

### `GET /api/tracks`
Trackタブでのタイムライン一覧表示用API。指定期間のイベントとそれに紐づくタグを取得します。

**Query Parameters:**
- `start_date`: YYYY-MM-DD
- `end_date`: YYYY-MM-DD

**Response:**
```json
[
  {
    "id": "uuid",
    "comment": "少し疲れてきた",
    "condition": -1,
    "priority": false,
    "is_archived": false,
    "created": "2026-05-11T14:00:00Z",
    "tags": [
      {
        "id": "uuid",
        "tag_name": "頭痛",
        "condition": -2,
        "tag_group": {
          "id": "uuid",
          "name": "体調"
        }
      }
    ]
  }
]
```

### `POST /api/tracks`
イベント（時々の記録）を作成します。

**Request Body:**
```json
{
  "comment": "少し疲れてきた",
  "condition": -1,
  "priority": false,
  "tags": [
    {
      "tag_group_id": "uuid",
      "tag_name": "頭痛",
      "condition": -2
    }
  ]
}
```

### `GET /api/tags/groups`
タグ入力時のサジェスト用一覧を取得します。

---

## 2. Daily (Daily Tab)

日単位の記録（1日の全体コンディション、日記的コメント）のためのAPIです。
他のリソース（TrackやSleep）とは分離し、Dailyタブ専用の情報のみを扱います。

### `GET /api/daily`
カレンダーや月間の一覧表示用API。

**Query Parameters:**
- `year`: int
- `month`: int

**Response:**
```json
[
  {
    "id": "uuid",             // day.id
    "date": "2026-05-11",
    "condition": 1,
    "priority_comments": [
      {
        "id": "uuid",
        "comment": "全体的に良い一日だった"
      }
    ]
  }
]
```

### `GET /api/daily/:day_id`
特定の一日の詳細（日記画面）を取得します。

**Response:**
```json
{
  "id": "uuid",
  "date": "2026-05-11",
  "condition": 1,
  "comments": [
    {
      "id": "uuid",
      "comment": "今日は調子が良い",
      "priority": true,
      "created": "2026-05-11T10:00:00Z",
      "updated": "2026-05-11T10:00:00Z"
    }
  ]
}
```

### `POST /api/daily/:day_id/comments`
該当日の日記（コメント）を追加します。

---

## 3. Sleep (Sleep Tab)

睡眠記録（本睡眠・昼寝）のためのAPIです。Dailyから切り離し、独立したリストとして管理します。

### `GET /api/sleeps`
Sleepタブでの睡眠記録の一覧表示用API。昼寝などもそれぞれ独立したレコードとして取得します。

**Query Parameters:**
- `start_date`: YYYY-MM-DD
- `end_date`: YYYY-MM-DD

**Response:**
```json
[
  {
    "id": "uuid",
    "day_id": "uuid", // DB上は対象のdayに紐づく
    "start": "2026-05-10T23:30:00Z",
    "end": "2026-05-11T07:00:00Z",
    "duration_hours": 7.5,
    "created": "2026-05-11T07:00:00Z"
  },
  {
    "id": "uuid",
    "day_id": "uuid",
    "start": "2026-05-11T13:00:00Z",
    "end": "2026-05-11T14:30:00Z",
    "duration_hours": 1.5,
    "created": "2026-05-11T14:30:00Z"
  }
]
```

### `POST /api/sleeps`
睡眠記録を追加します。

**Request Body:**
```json
{
  "day_id": "uuid",
  "start": "2026-05-10T23:30:00Z",
  "end": "2026-05-11T07:00:00Z"
}
```

---

## 4. CRUD Operations (個別リソース)

詳細な更新・削除などは以下のエンドポイントで行います。

- **Track (Events/Tags)**
  - `PUT /api/tracks/:event_id`
  - `DELETE /api/tracks/:event_id`
  - `DELETE /api/tags/:tag_id`
- **Daily (Comments)**
  - `PUT /api/daily/comments/:comment_id`
  - `DELETE /api/daily/comments/:comment_id`
- **Sleep**
  - `PUT /api/sleeps/:sleep_id`
  - `DELETE /api/sleeps/:sleep_id`
