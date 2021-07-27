# 概要
好きなチームの(次の)試合等のイベント情報を､会場周辺の衛星写真と天気予報をあわせて表示

# Link
- [デモ(Replit)](https://webappfinal.cgjf0086.repl.co/)
- [ソースコード(Replit)](https://replit.com/@cgjf0086/WebAppFinal)
- [ソースコード(Github)](https://github.com/cgjf0086/WebAppFinal)

# 使用方法
APIキーを設定してください  

- Replit
  - [Secret](https://docs.replit.com/tutorials/08-storing-secrets-and-history)機能を利用して環境変数にAPIキーを設定してください
- その他の環境
  - `.env.local`等の外部に公開されないファイルで環境変数にAPIキーを設定してください
- **ベタ書きはおすすめしません**
  - 公開しないように気を付けてください

**<注意>**  
これらのAPIキーはVueで使用するため､環境変数の名前の先頭には`VUE_APP_`を付けてください ([参考](https://cli.vuejs.org/guide/mode-and-env.html#environment-variables))

`src/App.vue`内ではSecret機能を利用して以下のように設定しています

- [TheSportDB](https://www.thesportsdb.com/api.php) : `VUE_APP_THESPORTSDB_KEY`
  - 無料プランのAPIキーは "1"
- [Mapbox](https://docs.mapbox.com/help/glossary/access-token/) : `VUE_APP_MAPBOX_KEY`
- [OpenWeather](https://openweathermap.org/appid) : `VUE_APP_OPENWEATHERMAP_KEY`

```javascript
// Set your TheSportsDB API key
const db_APIkey = process.env.VUE_APP_THESPORTSDB_KEY // '1'

// Set your Mapbox API key
const mapbox_APIKey = process.env.VUE_APP_MAPBOX_KEY

// Set your OpenWeather API key
const weather_APIKey = process.env.VUE_APP_OPENWEATHER_KEY
```

# 流れ
1. _TheSportsDB_からデータが提供されているすべてのスポーツを取得し､スポーツのセレクトボックスを描画 (アプリ側)
2. 地図(_Mapbox_)を初期状態で描画 (アプリ側)
3. スポーツをセレクトボックスから選択 (ユーザー側)
4. ユーザーが選択したスポーツから､そのスポーツのすべてのリーグ情報を取得し､リーグのセレクトボックスを描画 (アプリ側)
5. リーグをセレクトボックスから選択 (ユーザー側)
6. ユーザーが選択したリーグから､そのリーグのすべてのチーム情報を取得し､チームのセレクトボックスを描画 (アプリ側)  
    1. チーム情報が取得できない場合は､エラーを表示
7. チームをセレクトボックスから選択 (ユーザー側)
8. ユーザーが選択したチームから､_TheSportsDB_からそのチームのイベント情報を取得し､描画 (アプリ側)
    1. もし1対1形式の試合･イベントでないなら取得した情報から会場情報の抽出  (アプリ側)
    2. そうでないならば､`TeamID`をもとに_TheSportsDB_から両チームの情報を取得し､ホーム･アウェイのチームを表示  (アプリ側)
        - ホームチーム情報から､会場情報の抽出
9. 抽出した会場情報から_Mapbox Geocoding API_を使用し位置情報(`Lng`/`Lat`)を取得  (アプリ側)
    - 位置情報(`Lng`/`Lat`)をもとに会場周辺の天気情報を_OpenWeather_から取得  (アプリ側)
        - イベント日時が無料プランで取得できる7日後までであれば、イベント当日の天気予報を表示
        - それ以降の日時では､7日後の天気予報を表示
    - 位置情報(`Lng`/`Lat`)をもとに_Mapbox_で衛星地図の表示  (アプリ側)

# Documents
- 使用したAPI
  - [TheSportsDB](https://www.thesportsdb.com/api.php)
  - [Mapbox](https://docs.mapbox.com/)
  - [Mapbox Geocoding API](https://docs.mapbox.com/api/search/geocoding/)
  - [OpenWeather](https://openweathermap.org/api)

- 使用したPackages
  - [Vue@3.1.5](https://v3.vuejs.org/guide/introduction.html)
  - [Vue-multiselect@2.1.6](https://vue-multiselect.js.org/)
  - [Mapbox GL JS@2.3.1](https://docs.mapbox.com/mapbox-gl-js/api/)
  - [i18n-iso-countries@6.8.0](https://github.com/michaelwittig/node-i18n-iso-countries)

# Todo
- カスタムマーカーを描画し、クリックで情報を表示([GeoJSON](https://docs.mapbox.com/help/glossary/geojson/))
- ステータスによって文字色の変更
- デザインの変更
- スマホ対応
- チーム詳細情報表示の改善(v-htmlにXSS攻撃に関する脆弱性があるため)
- ~~Forward Geocodeの精度向上(国によるフィルタリング)~~
- ~~Forward Geocodeで得られる位置情報が実際の場所と合致しない場合のエラー処理~~  
(poi型データを正しく返さないAPI側のエラーのため現状修正不可)
- ~~お気に入り機能の追加~~
- キャッシュの使用
- ユーザーデータの保存
- お気に入り機能の改善
- Vue3対応の[Vue-Multiselect@3.0.0alpha](https://github.com/shentao/vue-multiselect/tree/next)での実装(Alpha版のため見送り)

# 問題点
- TheSportsDBが無料プランであるので`nextevent`を取得できない  
  -> 代わりに`lastevent`を取得  
  -> APIKEYによってAPIパラメータの変更  
  -> 現在の天気(Daily)を表示
- 有料プランであれば､ライブスコアも表示可能
- Mapbox Geocoderが正しいpoi型データを返さない
  - Real Madrid  
  - etc
