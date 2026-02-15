# 掲示板（認証ユーザー限定）- 要件・仕様（MVP）

## 目的
登録（認証）ユーザーのみがスレッド閲覧・作成、投稿閲覧・作成できる掲示板を提供する。

## 前提（技術）
- 認証: Firebase Authentication
- データ: Cloud Firestore
- アクセス制御: Firestore Security Rules
- UIは後で指定（まず機能とデータ・権限を確定）

## スコープ（MVP）
- ログイン/ログアウト
- threads（スレッド）: 一覧 / 作成 / 詳細
- posts（投稿）: 一覧 / 作成（スレッド配下）
- 未認証ユーザーは一切アクセス不可（read/write禁止）
- ページング（limit + cursor）

## 受け入れ条件（Done）
- 未ログインで threads/posts の read/write が permission-denied
- ログイン後にスレッド作成→一覧に反映
- スレッド内で投稿作成→詳細に反映
- authorUid の偽装（request.auth.uid と不一致）は拒否
- README に起動手順・最小テスト手順（ルール検証）が記載されている

## 権限・ロール
- user: 認証済みの一般ユーザー（閲覧・作成）
- admin: 将来拡張（削除/モデレーション）。MVPでは必須にしないが設計余地は残す。

## データモデル（Firestore）
### threads/{threadId}
- title: string (1..100) 必須
- authorUid: string 必須（= request.auth.uid）
- createdAt: timestamp 必須（serverTimestamp 推奨）
- updatedAt: timestamp 必須（serverTimestamp 推奨）

（任意: postCount, lastPostAt は将来拡張）

### threads/{threadId}/posts/{postId}
- body: string (1..2000) 必須
- authorUid: string 必須（= request.auth.uid）
- createdAt: timestamp 必須（serverTimestamp 推奨）
- updatedAt: timestamp 必須（serverTimestamp 推奨）

## クエリ仕様
- threads 一覧: createdAt desc, pageSize + cursor
- posts 一覧: createdAt asc（表示用） or desc（取得最適化）※実装で統一

## Firestore Security Rules（要点）
- 共通: request.auth != null のみ許可
- create: request.resource.data.authorUid == request.auth.uid を必須
- バリデーション:
  - title: 1..100
  - body: 1..2000
- update/delete:
  - MVP: 原則なし（未対応） or 作成者のみ（必要になったら追加）
  - adminロール対応は将来拡張

## 画面（UIは後で指定）
- ログイン画面
- スレッド一覧（作成導線）
- スレッド詳細（投稿一覧 + 投稿フォーム）

## 将来拡張
- adminモデレーション（削除、BAN）
- 通報/ブロック
- 画像添付（Storage）
- 通知（FCM）
- App Check の強制