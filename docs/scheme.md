# Calendar
カレンダーに関連するデータを格納するスキーマ

## day
カレンダーのそれぞれの日付にコンディションを記録する

| name | type | description |
| -- | -- | -- |
| id | uuid | 各日付のid |
| condition | int | 0を基準に-2 ~~ 2。高いほど良い |

## day_comment
dayには複数のコメントがつき得る

| name | type | description |
| -- | -- | -- |
| id | uuid | -- |
| day_id | uuid | dayのid |
| created | time | タイムスタンプ |
| updated | time | 更新日
| comment | text | コメント本文 | 
| priority | bool | trueの場合優先度が高い |

# Trafck
服薬やイベントなどのその時々の記録を行う

## event
このeventは3時間ごとに通知を飛ばしてconditionの記録を促す。
それによって太陽の推移を記録くする。
通知を無視した場合

| name | type | description |
| -- | -- | -- |
| id | uuid | -- |
| comment | text[option] | イベントの説明 | 
| condition | int | 基本を0として-2 ~ 2の間の数値 |
| is_archived | bool | 廃止したどうか |
| priority | bool | trueの場合優先度が高い |

## tag
| name | type | description |
| -- | -- | -- |
| id | uuid | -- |
| event_id | uuid | 紐つけさきのイベントid |
| tag_group_id | uuid | タブグループのid |
| event_id | uuid | イベントのid |
| tag_name | text | タグ名 |
| condition | int | 基本を0として-2 ~ 2の間の数値 |

## tag_group
タグたちのグループの集まり。
一つのtagに対して複数のidが集まる。

| name | type | description |
| -- | -- | -- |
| id | uuid | -- |
| name | uuid | どのtagに属する農家のid |
| created | time | 記録日 |
| updated | time | 更新日時 |

# Sleep
睡眠はぶつ切りの可能性を考慮し下職となっている、
複数存在しうるがday_idに紐ついている、

## log
| name | type | description |
| -- | -- | -- |
| id | uuid | -- |
| day_id | uuid | dayのid |
| created | time | 記録日 |
| updated | time | 更新日時 |
| start | 開始日時 | 睡眠開始　|
| end | 終了時間 | 起床時刻 |
