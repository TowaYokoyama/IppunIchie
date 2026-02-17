# 一分一会（Ippun-Ichie）

1分話すと、聞き手役のアバターが反応を返してくれる iPhone 向けスピーキング練習アプリです。  
「点数をつける」のではなく、「相手にどう伝わったか」を短時間で振り返る体験を目指しています。

## どんな作品か

- プレゼン・説明・Web面接・質問応答の練習を、1人で気軽に回せるトレーニングアプリ
- 60秒で話して、文字起こしと分析結果、アバターのリアクションを確認
- 音声データは保存せず、テキスト分析中心で軽量に運用
- iOS（SwiftUI）+ AWSサーバーレス（Lambda/API Gateway/DynamoDB）で構成

## 体験フロー

1. Apple Sign In でログイン
2. 練習モードを選ぶ（プレゼン・説明・Web面接・質問応答）
3. 60秒話す（端末で録音と文字起こし）
4. 分析結果とアバター反応を確認
5. 履歴で振り返る

## プレビュー
<img width="585" height="1266" alt="IMG_5241" src="https://github.com/user-attachments/assets/268ee0f4-ce5a-4d6d-8cf8-dc04392ec678" />
<img width="585" height="1266" alt="IMG_5244" src="https://github.com/user-attachments/assets/1a2967fd-edfc-47e4-8c9e-b916aa41e3df" />
<img width="585" height="1266" alt="IMG_5244" src="https://github.com/user-attachments/assets/3631ecfa-56ae-4f10-bc78-5102f6805a2b" />
<img width="585" height="1266" alt="IMG_5242" src="https://github.com/user-attachments/assets/953ce390-2e70-4cbc-8103-e3c371466eb0" />
<img width="1170" height="2532" alt="IMG_5240" src="https://github.com/user-attachments/assets/719bbb1f-0c74-4f70-bdd8-7fcb1591519e" />


## 主な機能

- Apple Sign In 認証
- 4モードの1分間スピーキング（プレゼン・説明・Web面接・質問応答）
- iOS Speech Framework による文字起こし
- バックエンドでのテキスト分析（明瞭さ・構成・自信）
- アバター反応表示（SceneKit）
- 練習履歴の保存と表示
- インサイト機能（成長推移、週間サマリー、パーソナライズお題）

## 開発状況（MVP）

- 基本フローは実装済み（ログイン、練習、分析、履歴）
- 和風の本番3Dアバターモデルは今後追加予定（現状は仮モデルベース）

## 技術スタック

- iOS: Swift, SwiftUI, Speech Framework, SceneKit, AuthenticationServices
- Backend: TypeScript, Node.js 20.x, AWS SDK v3, Zod
- Infra: AWS CDK v2, API Gateway, Lambda, DynamoDB, CloudWatch, AWS Budgets
- Monorepo: npm workspaces

## 技術的考察（設計判断）

- 1分固定セッション: 練習開始の心理的負荷を下げ、継続しやすさを優先
- 点数化より反応: 評価ストレスを下げるため、アバターのリアクションと短いコメント中心
- 音声データ非保存: プライバシー配慮として、サーバー送信は文字起こしテキスト中心
- ルールベース + 拡張可能ML: MVPでは軽量で安定稼働、将来的に高度分析へ段階拡張
- サーバーレス構成: 少人数運用でもコストを抑え、運用負荷を最小化
- 共通型の分離: `packages/shared` で iOS/Backend 間のAPI契約ズレを抑制
- 運用ガード: レート制限、CloudWatchアラーム、Budget通知で異常検知を早期化

## ディレクトリ構成（主要）

```text
ippun-ichie/
├── apps/
│   ├── ios/
│   │   ├── IppunIchie.xcodeproj/                      # Xcode プロジェクト
│   │   └── IppunIchie/IppunIchie/
│   │       ├── Views/                                  # 画面（Session/Result/History 等）
│   │       ├── Managers/                               # 認証・録音・履歴管理
│   │       ├── Services/                               # API通信・顔/音声解析
│   │       ├── Models/                                 # 分析結果やモード定義
│   │       └── Theme/                                  # UIテーマ定義
│   └── backend/
│       └── src/
│           ├── handlers/                               # analyze/history/topic/insights API
│           ├── services/                               # 分析ロジック・DynamoDB・レート制限
│           ├── schemas/                                # Zodスキーマ
│           ├── utils/                                  # logger/response/validation 等
│           ├── router.ts                               # ルーティング定義
│           └── index.ts                                # Lambda エントリーポイント
├── infra/
│   └── aws/
│       ├── lib/ippun-ichie-stack.ts                   # CDK スタック本体
│       └── bin/app.ts                                  # CDK エントリーポイント
├── packages/
│   └── shared/src/index.ts                             # 共通型定義
├── docs/                                                # 設計・要件・セットアップ資料
└── README.md
```

## API（代表）

- `POST /analyze` : 発話テキストを分析して反応を返す
- `GET /topic` : モードに応じたお題を返す
- `GET /history` : 練習履歴を取得する
- `GET /insights/growth` : 成長推移を取得する
- `GET /insights/weekly-summary` : 週間サマリーを取得する
