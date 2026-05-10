# API Interface

DBのスキーマ（`docs/scheme.md`）は正規化されていますが、実際のAPIはフロントエンドでの利用を想定し、これらをJoinしたネスト構造のデータを提供します。

## 1. Calendar / Day

### `GET /api/calendar`
カレンダー画面（月間表示など）のためのAPI。指定した期間（月など）の `day` データと、紐づく重要な情報をJoinして取得します。

**Query Parameters:**
- `year`: int
- `month`: int

**Response:**
```json
[
  {
    "id": "uuid",             // day.id
    "date": "2026-05-11",     // カレンダーの日付
    "condition": 1,           // day.condition
    "priority_comments": [
      // day_commentから priority=true のものをJoin
      {
        "id": "uuid",
        "comment": "よく眠れた"
      }
    ],
    "sleep_summary": {
      // 該当day_idに紐づくsleep logから算出
      "total_hours": 7.5
    }
  }
]
```

### `GET /api/days/:day_id`
1日の詳細画面のためのAPI。`day`テーブルを起点に、`day_comment`、睡眠記録（`log`）、およびその日発生した `event`（とそれに紐づく `tag`）を全てJoinして返します。

**Response:**
```json
{
  "id": "uuid",             // day.id
  "condition": 1,
  "comments": [
    // day_commentをJoin
    {
      "id": "uuid",
      "comment": "今日は調子が良い",
      "priority": true,
      "created": "2026-05-11T10:00:00Z",
      "updated": "2026-05-11T10:00:00Z"
    }
  ],
  "sleep_logs": [
    // sleep logをJoin
    {
      "id": "uuid",
      "start": "2026-05-10T23:30:00Z",
      "end": "2026-05-11T07:00:00Z",
      "created": "2026-05-11T07:00:00Z",
      "updated": "2026-05-11T07:00:00Z"
    }
  ],
  "events": [
    // その日に該当するeventをJoin (※eventテーブルに発生日時が含まれる想定)
    {
      "id": "uuid",
      "comment": "朝の記録",
      "condition": 2,
      "is_archived": false,
      "priority": false,
      "created": "2026-05-11T09:00:00Z",
      "tags": [
        // tagとtag_groupをJoin
        {
          "id": "uuid",
          "tag_name": "コーヒー",
          "condition": 1,
          "tag_group": {
            "id": "uuid",
            "name": "飲食"
          }
        }
      ]
    }
  ]
}
```

## 2. Track (Event & Tag)

### `POST /api/events`
3時間ごとの通知時や、任意のタイミングでのイベント記録を作成します。フロントエンドからはイベント情報とタグ情報を同時に送信し、バックエンドでトランザクションを貼って `event` と `tag` テーブルへインサートします。

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

### `GET /api/tag-groups`
入力画面でタグをサジェストするために、タググループとそれに紐づくタグの履歴（ユニークな `tag_name` など）をまとめて取得します。

**Response:**
```json
[
  {
    "id": "uuid", // tag_group.id
    "name": "体調",
    "recent_tags": [
      "頭痛",
      "腹痛",
      "肩こり"
    ]
  }
]
```

## 3. CRUD Operations

以下のエンドポイントは各エンティティの個別操作（作成・更新・削除）用です。
子要素は関連する親のIDなどをURIに含めます。

- **Day Comments**
  - `POST /api/days/:day_id/comments`
  - `PUT /api/comments/:comment_id`
  - `DELETE /api/comments/:comment_id`
- **Sleep Logs**
  - `POST /api/days/:day_id/sleep-logs`
  - `PUT /api/sleep-logs/:log_id`
  - `DELETE /api/sleep-logs/:log_id`
- **Events**
  - `PUT /api/events/:event_id`
  - `DELETE /api/events/:event_id`
- **Tags**
  - `POST /api/events/:event_id/tags`
  - `DELETE /api/tags/:tag_id`
