### API仕様: ユーザー登録API
**API ID:** user_registration_api
**エンドポイント:** /api/register
**HTTPメソッド:** POST
**リクエスト:**
| 名前 | 型 | 必須 |
|------|----|----|
| name | string | yes |
| email | string | yes |
| password | string | yes |

**レスポンス:**
| 名前 | 型 |
|------|----|
| userId | string |
| message | string |

**制約:** メール形式チェック, パスワード8文字以上