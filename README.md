# 顔の向き検出システム

リアルタイムで顔の下向き角度を検出し、姿勢の改善をサポートするWebアプリケーションです。

## 機能

- **リアルタイム検出**: MediaPipe Face Meshを使用して顔のランドマークを検出
- **デバイス傾き補正**: スマホの傾きセンサーと組み合わせて正確な顔の向きを計算
- **30度警告**: 顔が30度以上下を向くと視覚的に警告
- **スムーズな表示**: 角度の急激な変化を平滑化してリアルタイムに表示

## 技術スタック

- **MediaPipe Face Mesh**: 顔のランドマーク検出（468ポイント）
- **DeviceOrientation API**: スマホの傾きセンサーデータ取得
- **Canvas API**: リアルタイムでの顔メッシュ可視化

## 顔の向きの計算方法

実際の顔の下向き角度は以下のように計算されます：

```
顔の向き = 顔の下向き角度 + デバイスの前傾角度
```

### 詳細

1. **顔の下向き角度**:
   - 顔のランドマーク（額、鼻、顎）から顔の傾きを計算
   - 正面を向いている状態を0度とする

2. **デバイスの前傾角度**:
   - DeviceOrientationEventのbeta値を使用
   - デバイスが立っている状態（90度）を基準に、前傾した分を加算
   - 例: デバイスが平面（0度）の場合、90度の傾きとして計算

3. **合成**:
   - 両方の角度を合算して実際の顔の下向き角度を算出
   - スマホを平面に置いて見下ろした場合も正確に計算

## 使い方

1. **開始ボタンを押す**
   - カメラとモーションセンサーへのアクセス許可を求められます

2. **許可を与える**
   - カメラ: 前面カメラで顔を撮影するため
   - モーションセンサー: デバイスの傾きを取得するため（iOS 13+では明示的な許可が必要）

3. **顔をカメラに向ける**
   - リアルタイムで顔のメッシュが表示されます
   - 顔の向きが計算され、画面に表示されます

4. **姿勢をチェック**
   - 顔が30度以上下を向くと警告が表示されます
   - 顔の傾きとデバイスの傾きも個別に確認できます

## 表示される情報

- **顔の下向き角度**: 実際の顔の下向き角度（メイン表示）
- **顔の傾き**: カメラに対する顔の傾き角度
- **デバイスの傾き**: スマホ本体の傾き角度

## ブラウザ要件

- **カメラアクセス**: getUserMedia API対応
- **モーションセンサー**: DeviceOrientation API対応
- **推奨ブラウザ**:
  - iOS Safari 13+
  - Android Chrome 最新版
  - Android Firefox 最新版

## 注意事項

### iOS (Safari)
- iOS 13以降では、DeviceOrientationEventへのアクセスに明示的な許可が必要です
- 設定 > Safari > モーションと画面の向きのアクセス を有効にしてください

### Android
- Chrome、Firefoxでは通常、追加の設定なしで動作します
- セキュアコンテキスト（HTTPS）が必要な場合があります

### デスクトップ
- カメラは動作しますが、モーションセンサーがないため、デバイスの傾きは0として扱われます
- 顔の傾きのみが計算されます

## 技術的な詳細

### 使用しているランドマーク

- **額の中心** (index 10): 顔の上端
- **鼻の付け根** (index 168): 顔の中心参照点
- **鼻の先端** (index 1): 顔の向きの検出
- **顎の先** (index 152): 顔の下端

### 角度計算アルゴリズム

```javascript
// 顔の中心線のベクトル（額から顎へ）
faceVectorY = chin.y - forehead.y
faceVectorZ = chin.z - forehead.z

// ピッチ角度を計算
pitchAngle = atan2(faceVectorZ, faceVectorY) * (180 / π)

// デバイスの傾きを加算
deviceAngleContribution = 90 - deviceOrientation.beta
faceDirection = faceAngle + deviceAngleContribution
```

### 平滑化

急激な角度変化を抑えるため、指数移動平均を使用：

```javascript
smoothedAngle = previousAngle * 0.7 + newAngle * 0.3
```

## ローカルで実行

```bash
# HTTPSサーバーを起動（カメラとセンサーアクセスにHTTPSが必要な場合）
python3 -m http.server 8000

# またはNode.jsの場合
npx http-server -p 8000
```

ブラウザで `http://localhost:8000` にアクセスしてください。

## ライセンス

MIT License

## 参考

- [MediaPipe Face Mesh](https://google.github.io/mediapipe/solutions/face_mesh.html)
- [DeviceOrientation Event Specification](https://w3c.github.io/deviceorientation/)
