# 株クラTPG リリース前 総合テスト結果

| 項目 | 内容 |
|---|---|
| 対象 | `index.html`（6,364行）／ `assets/`（82件） |
| 検証環境 | `http://localhost:8000`（`python3 -m http.server`）／ Chromium |
| 検証日 | 2026-09-01 |
| 対象ファイルのSHA-256 | `2fb64ee9db96e3c6660a0f9ab739195a4378cd44790dd598ae2fd5d41dc2b3fa` |
| コードの変更 | **なし**（検証前後でハッシュ一致を確認済み） |

## 総括

| 判定 | 件数 |
|---|---|
| OK | 28 |
| NG | 4 |
| 一部NG | 2 |
| 検証不可 | 0（機械検証範囲では全項目に結論あり） |

**NG 4件**：`se_clear.mp3` 欠落／OGP画像の不整合／中断時に残るタイマー2種／`fifth` テーマとサウンドトラック解禁条件の食い違い。
いずれもゲーム進行を止める不具合ではありませんが、`se_clear.mp3` と OGP はリリース前に対応を推奨します。

---

## ■ 称号（37種）

### 1. 各称号の判定条件がコード上で正しく実装されているか — **OK**

構成：**本体25種 ＋ 冠12種 = 37種**（`allTitles().length` で確認）。
うちタイム系10種（`b01`＋`b02`〜`b10`）、条件系15種（`n01`〜`n11`＋キャラ系4）、冠12種。

| ID | 称号 | 実装箇所 | 条件式 | 判定 |
|---|---|---|---|---|
| b01–b10 | タイム系 | 5539 `baseTitleFor()` | `totalMs < maxMs` の最小 `maxMs` を選択 | OK |
| n01 | 株クラRPGメンバー | 5859 | クリアで無条件 | OK |
| n02 | 握力ゴリラ | 5861 | 累計打鍵 `>= 3000` | OK |
| n03 | ガチホ民 | 5862 | `state.timeoutCount === 0` | OK |
| n04 | お祈り投資家 | 5863 | `0 < lastRemainingMs <= 1000` | OK |
| n05 | 市場に居続けた者 | 5864 | 所持イラスト `>= 11` | OK |
| n06 | 塩漬け職人 | 6114 | 同一キャラ連続時間切れ `>= 5` | OK |
| n07 | 狼狽売りの達人 | 6195 | `state.miss === 20` の瞬間 | OK |
| n08 | 高値掴みの天才 | 6115 | `state.stageIndex === 3` で時間切れ | OK |
| n09 | ナンピンマスター | 4393 | 連続時間切れ `>= 3` の相手を撃破 | OK |
| n10 | 損切使い | 4129 | 戦闘中に「最初から始める」 | OK |
| n11 | 株クラ民 | 5980 | X投稿ボタンを押す | OK |
| w_* / l_* | キャラ4人×勝敗 | 5757 `awardCharaTitle()` | 撃破時 `win` ／ 時間切れ時 `lose` | OK |
| c01–c08 | 冠 | 5820 `earnedCrowns()` | 後述 | OK |

### 2. タイム系10種の境界値 — **OK**

各閾値の **−1ms / ちょうど / +1ms** を全点検証。判定は `totalMs < maxMs`（**未満**）で一貫。

| 閾値 | −1ms | ちょうど | +1ms |
|---|---|---|---|
| 50,000 | b10 株クラの頂点 | b09 フルFIRE | b09 |
| 60,000 | b09 | b08 市場の支配者 | b08 |
| 70,000 | b08 | b07 インデックスの魔術師 | b07 |
| 80,000 | b07 | b06 億り人 | b06 |
| 90,000 | b06 | b05 相場を読む者 | b05 |
| 110,000 | b05 | b04 複利の探究者 | b04 |
| 130,000 | b04 | b03 分散の使い手 | b03 |
| 150,000 | b03 | b02 積立の徒 | b02 |
| 180,000 | b02 | b01 見習い投資家 | b01 |

`0ms` → b10、`999999ms` → b01。範囲外でも必ず何かに落ちる（`|| TITLE_BASE[0]`）。

### 3. 条件系のシミュレーション — **OK**（7項目すべて）

| 称号 | 検証方法 | 結果 |
|---|---|---|
| 塩漬け職人 | 同一キャラで強制時間切れ→リトライを5回 | 1〜4回目 未取得 → **5回目で取得** |
| ナンピンマスター | 3連続時間切れ後に撃破 | 連続数3で撃破 → **取得**。撃破後 `timeoutStreak` は 0 にリセット |
| 四天王箱推し（c08） | 4人全員で時間切れを経験 | 3人目まで `false` → **4人目で `allCharasTimedOut()` が `true`**、冠一覧にも `c08` が入る |
| 握力ゴリラ | 累計打鍵 2999 / 3000 | 2999 で未取得 → **3000 で取得** |
| 狼狽売りの達人 | ミス 19 / 20 / 21 | 19 未取得 → **20 で取得**、21 でも維持 |
| お祈り投資家 | 残り 0 / 1 / 1000 / 1001ms | 0=否、**1=可、1000=可**、1001=否 |
| 高値掴みの天才 | 4番手（`stageIndex===3`）で時間切れ | **取得** |

### 4. 冠12種の判定 — **OK**

| 条件 | 得られた冠 |
|---|---|
| 正確率100・最大コンボ150・無傷・1回目 | c02 c03 c04 **c06** |
| 同上・10回目 | c02 c03 c04 **c05** |
| 同上・100回目 | c02 c03 c04 c05 **c07** |
| コンボ149・ミスあり・時間切れあり・5回目 | （なし） |
| 上に「4人全員で時間切れ」を追加 | **c08** |
| c01 邂逅の（1%抽選） | 20,000回で **0.95%**（期待値1%と整合） |

キャラ冠4種（`w_micchan` / `l_micchan` / `w_nimushi` / `l_nempan`）は項目3で取得を確認。

### 5. 獲得済み称号がすべて選択画面に出るか — **OK**

全37種を解放した状態で称号一覧を描画。
- 表示行 **42行** = 称号37 ＋「冠なし」1 ＋ 見出し4（`titles-sub`）
- 全37種の表示名が一覧に存在（未表示 0件）
- 「？？？」表示 **0件**

### 6. 装備の選択・保存・復元 — **OK**

- `saveEquippedTitle("b05","c04")` → `selectedBaseTitle:"b05"` / `selectedCrownTitle:"c04"` が localStorage に保存
- **未獲得IDを指定した場合は保存されない**（`loadProgress()` が `unlockedTitles` に含まれるIDのみ通す）
- 復元は `loadProgress()` 経由で型・存在チェックを通過したものだけ

---

## ■ ご褒美イラスト

### 7. 抽選の重み付け — **OK**

`REWARD_WEIGHT_NEW = 5` / `REWARD_WEIGHT_OWN = 1`

**所持0枚から1000回**（全10種が未所持＝重み均等）

| ID | 回数 | | ID | 回数 |
|---|---|---|---|---|
| pair_mi_ni | 115 | | pair_ne_po | 98 |
| solo_micchan | 108 | | solo_ponkotsu | 96 |
| pair_mi_ne | 108 | | pair_ni_po | 95 |
| solo_nimushi | 107 | | pair_ni_ne | 85 |
| solo_nempan | 104 | | pair_mi_po | 84 |

10種すべてが 84〜115（期待100）に収まり、偏りなし。

**5枚所持で1000回**

| | 回数 |
|---|---|
| 未所持5種の合計 | 838 |
| 所持済み5種の合計 | 162 |
| 比 | **5.17 : 1**（期待 5.00 : 1） |

重み付けは仕様どおり。

### 8. 11枚揃うまでの平均回数 — **OK**（1000回シミュレーション）

| 統計 | 値 |
|---|---|
| 平均 | **14.93 回** |
| 中央値 | 14 回 |
| 最小 | 11 回（＝最速。毎回新規を引いた場合） |
| 90パーセンタイル | 19 回 |
| 最大 | 29 回 |

### 9. 10枚揃った次のクリアで集合絵が100%か — **OK**

通常10種を所持した状態で1000回抽選 → **1000/1000 (100%)** が `REWARD_FINAL`（`id:"all"`）かつ `isNew: true`。

### 10. 11枚揃ったあとの挙動 — **OK**

全11種所持で1000回抽選 → 例外なし。11種から均等に選ばれ（74〜103回）、`isNew` は **常に false**。

---

## ■ 画面遷移

### 11. 全画面の遷移経路 — **OK**

localStorage を空にして通しプレイ。`showScreen()` を計測して得た実際の経路：

**初回プレイ（エピローグ）**
```
story → start → title → intro → game → defeat
              → intro → game → defeat
              → intro → game → defeat
              → final → game → defeat        ← 4人目は final 演出
              → story(EPILOGUE) → clear
```
**トゥルーエンド（通常10枚所持で集合絵を初獲得）**
```
title → intro→game→defeat ×3 → final→game→defeat → clear
      → （1秒後に導入オーバーレイ）→ story(TRUE_END) → clear
```
- 初回起動はデータなしで `story`（プロローグ）から開始
- プロローグ完了で `storySeen: true`、2回目以降は `start` から
- トゥルーエンド後の戻り先は `clear`（スコア `00:00.78` と「Xで結果を投稿」が維持されている）
- 完了後のタイトル背景は `title_bg_true_cleared_ending.jpeg`

### 12. 各画面での打鍵が判定に回らないか — **OK**

`title / profile / gallery / soundtrack / titles / clear / defeat / timeup / intro / final / story / start` の
12画面で `a`〜`g` を送出。**全画面で `state.correct` `state.miss` ともに増加 0**。

### 13. 「最初から始める」「タイトルへ戻る」の遷移先 — **OK**

| 起点 | ボタン | 遷移先 |
|---|---|---|
| 人物紹介 | タイトルへ戻る | title |
| ギャラリー | タイトルへ戻る | title |
| サウンドトラック | タイトルへ戻る | title |
| 称号一覧 | タイトルへ戻る | title |
| 戦闘中 | 最初から始める | title（`n10 損切使い` を取得） |
| クリア画面 | 最初から始める | title（`n10` は**取得しない**＝仕様どおり） |

### 14. 演出中の中断で予約タイマーが残らないか — **一部NG**

| 中断タイミング | 結果 |
|---|---|
| 登場演出（intro）中 | **OK** — 3秒後も `title` のまま（勝手に戦闘へ行かない） |
| 最後の相手の演出（final）中 | **OK** — 3.5秒後も `title` のまま |
| ハート消滅演出中 | **NG（軽微）** — 下記 NG-3a |
| IME監視中 | **NG（軽微）** — 下記 NG-3b |

---

## ■ タイマーとスコア

### 15. 各ステージの制限時間 — **OK**

| ステージ | 制限時間 | 問題数（＝HP） |
|---|---|---|
| 1人目 | 60秒 | 5 |
| 2人目 | 60秒 | 6 |
| 3人目 | 60秒 | 6 |
| 4人目 | **90秒** | 7 |

`STAGE_TIME_MS = [60000, 60000, 60000, 90000]` で仕様どおり。

### 16. ミス減点3秒、下限0秒 — **OK**

`MISS_PENALTY_MS = 3000`。制限60秒での残り時間：

| ミス回数 | 0 | 1 | 5 | 19 | 20 | 21 | 100 |
|---|---|---|---|---|---|---|---|
| 残り(ms) | 60000 | 57000 | 45000 | 3000 | **0** | **0** | **0** |

`Math.max(0, ...)` により負値にならない。

### 17. 撃破画面・時間切れ画面の滞在時間が加算されないか — **OK**

| 画面 | 遷移直後 | 1.5秒滞在後 | 増分 |
|---|---|---|---|
| 時間切れ画面 | 1,998ms | 1,998ms | **0** |
| 撃破画面 | 2,999ms | 2,999ms | **0** |

`state.tickTimer` も `null`（停止済み）。

### 18. 残り10秒の警告が1回だけか — **OK**

| ケース | 結果 |
|---|---|
| 通常のカウントダウンで10秒を跨ぐ | 11秒台 0回 → **10秒台で1回** → 8秒/4秒でも1回のまま |
| ミス減点11回で40秒→7秒へ一気に跨ぐ | **1回のみ**。その後の再描画でも増えない |

`state.warned` はステージ開始（`setupBattle`）ごとにリセットされ、リトライでも鳴り直す。

### 19. クリアタイムの表示形式と境界値 — **OK**

| 入力(ms) | 表示 | | 入力 | 表示 |
|---|---|---|---|---|
| 0 | 00:00.00 | | **59996** | **00:59.99** |
| 1 | 00:00.00 | | 59999 | 00:59.99 |
| 10 | 00:00.01 | | 60000 | 01:00.00 |
| 999 | 00:00.99 | | 3599999 | 59:59.99 |
| 1000 | 00:01.00 | | 3600000 | 60:00.00 |

**59.996秒 → `00:59.99`**（切り捨てのため `00:60.00` にならない）。
`-5` / `NaN` / `Infinity` / `"x"` / `null` はすべて `00:00.00`。

---

## ■ localStorage

### 20. 初回訪問（データなし） — **OK**

`loadBest()` が `null` を返し、`loadProgress()` は全項目が既定値（称号0・イラスト0・clearCount 0・各種 false）。累計打鍵 0、`storySeen`/`epilogueSeen`/`trueEndSeen`/`soundEnabled` すべて `false`。

### 21. 旧データ（項目が欠けている状態） — **OK**

`{"bestTimeMs":12345}` のみを入れても、欠けた項目はすべて既定値に補完され例外なし。

### 22. 壊れたJSON — **OK**

| 内容 | 結果 |
|---|---|
| `{壊れた` | `loadBest()` → null、既定値で継続 |
| `[1,2,3]`（配列） | 同上（配列は明示的に拒否） |
| `null` | 同上 |
| `"文字列"` | 同上 |

`try/catch` で保護され、いずれも例外なし。

### 23. 未知のIDが混ざっている場合 — **OK**

投入：`unlockedTitles:["b01","__unknown__","w_micchan",123,null]` / `unlockedRewards:["solo_micchan","__nope__","solo_micchan"]` / `clearCount:"5"` / `noTimeoutClear:"yes"` / `totalKeystrokes:-99` / `selectedBaseTitle:"b10"`（未獲得）/ `selectedCrownTitle:"__x__"` / `trueEndSeen:1`

| 項目 | 結果 |
|---|---|
| unlockedTitles | `["b01","w_micchan"]`（未知ID・数値・null を除去） |
| unlockedRewards | `["solo_micchan"]`（未知IDと**重複**を除去） |
| clearCount | `0`（文字列 `"5"` は不採用） |
| noTimeoutClear / trueEndSeen | `false`（`=== true` 判定のため） |
| totalKeystrokes | `0`（負値を不採用） |
| selectedBaseTitle / CrownTitle | `null`（未獲得・未知IDを不採用） |

### 24. 「最初から始める」でデータが消えないか — **OK**

戦闘中に実行しても、`clearCount:7` / `bestTimeMs:55555` / `totalKeystrokes:1234` / イラスト2枚 / `storySeen:true` はすべて保持。称号は `n10 損切使い` が1つ増えるだけ。

### 25. 保存される全キーの一覧と更新タイミング — **OK**

**localStorage キーは `kabukura-rpg-best` の1つのみ**（1つのJSONオブジェクトに全項目を格納）。

| フィールド | 更新関数 | 更新タイミング |
|---|---|---|
| `bestTimeMs` / `bestAccuracy` / `bestCombo` | `saveBest()` 5702 | クリア画面表示時（自己ベスト更新時のみ良い方を採用） |
| `clearCount` | `saveBest()` | クリアのたびに +1 |
| `noTimeoutClear` | `saveBest()` | クリア時（一度でも無傷なら以後 true） |
| `unlockedTitles` | `saveBest()` / `unlockTitles()` 5749 | クリア時＋称号取得の瞬間（撃破・時間切れ・ミス20・X投稿・途中離脱） |
| `unlockedRewards` | `saveBest()` | クリア時（新規入手した1枚を追加） |
| `selectedBaseTitle` / `selectedCrownTitle` | `saveEquippedTitle()` 5394 | クリア画面の称号選択、および初回クリア時の自動装備 |
| `totalKeystrokes` | `addTotalKeystrokes()` 5795 | クリア時（`awardClearConditionTitles` から） |
| `storySeen` | `saveStorySeen()` 5933 | プロローグを見終わった時 |
| `epilogueSeen` | `saveEpilogueSeen()` 5923 | エピローグを見終わった時 |
| `trueEndSeen` | `saveTrueEndSeen()` 5913 | トゥルーエンドを見終わった時 |
| `soundEnabled` | `saveSoundSetting()` 5943 | 音のON/OFFを切り替えた時 |

通しプレイ後、実際に保存されていた14キー：
`bestAccuracy, bestCombo, bestTimeMs, clearCount, epilogueSeen, noTimeoutClear, selectedBaseTitle, selectedCrownTitle, soundEnabled, storySeen, totalKeystrokes, trueEndSeen, unlockedRewards, unlockedTitles`

書き込みは全箇所 `try/catch` で保護（プライベートブラウズ等で失敗しても進行は止まらない）。

---

## ■ テーマ切替

### 26. 4段階の優先順位 — **OK**（ただし閾値について注記あり）

| 状態 | テーマ | 背景 | BGM |
|---|---|---|---|
| クリア 0〜2回 | `normal` | title_bg.jpeg | bgm_title.mp3 |
| クリア 3〜5回 | `flawless` | title_bg_flawless.jpeg | bgm_title_flawless.mp3 |
| クリア 6回以上 | `fifth` | bg_5th.jpeg | bgm_title_5th.mp3 |
| トゥルーエンド後 | `true_end` | title_bg_true_cleared_ending.jpeg | bgm_title_true_cleared_ending.mp3 |

**優先順位は正しく `true_end > fifth > flawless > normal`。** トゥルーエンド後はクリア回数0回でも `true_end` が選ばれます。

> **注記** 現在の条件は `fifth` が `clearCount >= 6`、`flawless` が `clearCount >= 3` です。
> ご指定は「5回」「2回」でしたが、コード上のコメント（「6回クリアしたら」「2周目をクリアしたら」）とは一致しています。意図どおりかご確認ください。

### 27. 素材がない場合のフォールバック — **OK**

`true_end` を選択中の状態で素材を順に落とした結果：

| 欠落させたもの | 背景 | BGM |
|---|---|---|
| （なし） | true_cleared_ending | true_cleared_ending |
| true_end の背景 | **bg_5th へ降格** | true_cleared_ending（維持） |
| ＋ true_end のBGM | bg_5th | **bgm_title_5th へ降格** |
| ＋ fifth の背景 | **title_bg_flawless へ降格** | bgm_title_5th |
| ＋ fifth のBGM | title_bg_flawless | **bgm_title_flawless へ降格** |
| ＋ flawless 両方 | title_bg.jpeg | bgm_title.mp3 |
| 全部 | （無し＝下地色） | （無し＝無音） |

**背景とBGMは独立して降格**し、全滅時も例外を出さずに空を返します。

---

## ■ 音（コード上の検証）

### 28. 音OFF時に Audio/AudioContext が生成されないか — **NG（軽微）**

下記 NG-4 を参照。**音は一切鳴りませんが、AudioContext と効果音9本の先読みは実行されます。**

| タイミング | AudioContext | `<audio>`要素 | 先読み | 実際の発音 |
|---|---|---|---|---|
| 起動直後 | 未生成 | 0 | 0 | — |
| スタート画面を抜けた後（音OFFのまま） | **running** | 0 | **9本** | 0 |
| 音OFFで `playSe`/`playBgm` を呼ぶ | running | 0 | 9本 | **0（`canPlay()` が false）** |

### 29. BGMのループ設定が正しく反映されるか — **OK**

| ケース | loop | loopStart | loopEnd |
|---|---|---|---|
| タイトルBGM（`loopStart:0`） | true | 0 | 30.772（＝曲長） |
| `loopStart:5, loopEnd:12` を指定 | true | **5** | **12** |
| 不正値（`loopStart:9999`, `loopEnd:5`） | true | **0** | **17.866（曲長）** ＝安全側に倒れる |
| サウンドトラック試聴 | **false** | — | — |

### 30. 効果音のプリロードが動くか — **OK**

音ON時に9本すべてデコード完了：
`se_type / se_compleate / se_miss / se_click / se_final_slash / se_defeat / se_final_intro / se_timeup / se_warning`
`se_clear.mp3` のみ `broken` に登録（→ NG-1）。

### 31. 画面遷移時に効果音が停止するか — **OK**

| ケース | 結果 |
|---|---|
| `showScreen()` 単体 | 止まらない（＝仕様。遷移そのものは音を切らない） |
| 撃破画面へ | 遷移前の2本が**すべて停止**し、新たに撃破音1本のみ（ノードIDで照合済み） |
| 時間切れ画面へ | 同上 |
| 最初から始める | **0本**（新規再生なし） |

### 32. 音源が存在しない場合に例外が出ないか — **OK**

存在しないSE／BGM、`null` / `undefined` / `{}` / `""` を渡しても**例外なし**。失敗した src は `broken` に登録され、以後リクエストされません。

---

## ■ 素材の参照

### 33. 参照ファイルパスの実在確認 — **NG**

コード内の `assets/...` 参照 **96件**を抽出。**実在91件 / 欠落5件**。
うち3件はコメント内の記述例のため実害なし。**実害のある欠落は2件**。

| パス | 参照元（行） | 状態 | 影響 |
|---|---|---|---|
| `assets/se_clear.mp3` | 2521 `AUDIO.se.clear` | **404** | **NG-1**：クリア画面の効果音が鳴らない |
| `assets/ogp2.jpg` | 6335 `syncOgpTags()` | **404** | **NG-2**：OGP画像 |
| `assets/se_x.mp3` | 2513 | 欠落 | コメント内の記述例。実害なし |
| `assets/xxx.mp3` | 2497–2498 | 欠落 | 同上 |
| `assets/micchan.png` | 2570 | 欠落 | 同上（「画像が届いたら」のメモ） |

**実プレイ1周の通信ログでも404は `se_clear.mp3` の1件のみ**でした。

#### 参考：コードから参照されていない `assets/` 配下のファイル（計 35.8MB）

デプロイ容量の削減候補です（動作には影響しません）。

| ファイル | サイズ | 備考 |
|---|---|---|
| `artbook_0905.pdf` | 4.6MB | 旧版PDF |
| `artbook_0907.pdf` | 7.1MB | 旧版PDF |
| `artbook_0909.pdf` | 8.2MB | 旧版PDF |
| `artbook_0911.pdf` | 9.2MB | 旧版PDF |
| `bgm_title_true_cleared_ending.mp4` | 3.0MB | mp3 の元動画と思われる |
| `title_bg_true_cleared_ending.jpeg.jpeg` | 504KB | 拡張子が二重。同名 `.jpeg` と同一サイズ |
| `artbook/60.jpeg` | 692KB | `ARTBOOK.pages` は 00〜13 の14枚のみ参照 |
| `finale.jpg` | 372KB | — |
| `sound/`（ディレクトリ） | 2.8MB | — |

---

## ■ エラー

### 34. 未捕捉のJSエラー・未処理のPromise拒否 — **OK**

localStorage を空にした状態から、`error` / `unhandledrejection` / `console.error` を捕捉して2通り通しプレイ。

| 経路 | 未捕捉エラー | Promise拒否 |
|---|---|---|
| 初回プレイ（プロローグ→4戦→エピローグ→クリア） | **0件** | **0件** |
| トゥルーエンド（4戦→クリア→導入→TRUE_END→クリア） | **0件** | **0件** |

`console.warn` は **1件**（OGP同期。→ NG-2）。

---

# NG 一覧

## NG-1 `se_clear.mp3` が存在せず、クリア効果音が鳴らない

- **現象**：クリア画面に入るたびに `assets/se_clear.mp3` が 404。効果音が一切鳴らない。
- **原因**：`AUDIO.se.clear` が参照するファイルが `assets/` に未配置。
- **該当箇所**：`index.html` 2521行 `clear: "assets/se_clear.mp3",` ／ 再生は 4544行 `playSe(AUDIO.se.clear)`
- **影響**：`loadSeBuffer` の `catch` で `broken` に登録されるため例外・進行停止はなし。純粋に「音が鳴らない」のみ。
- **対処案**：音源を配置するか、`AUDIO.se.clear` の行を削除する（削除しても `playSe` は無指定を無視するため安全）。

## NG-2 OGP画像の指定が実体と食い違い、毎回コンソール警告が出る

- **現象**：起動のたびに以下の警告が出る。
  ```
  [OGP] 静的タグが定数とズレています。<head> を修正してください:
  meta[property="og:image"] = .../assets/ogp.jpg?v=2 / 定数 = .../assets/ogp2.jpg
  ```
  さらに `syncOgpTags()` は警告後に**タグを書き換える**ため、読み込み後の `og:image` は存在しない `ogp2.jpg` を指す。
- **原因**：`<head>` の静的タグは `assets/ogp.jpg?v=2`（実在）だが、JS 側の定数は `assets/ogp2.jpg`（**不在**）。
- **該当箇所**：`index.html` 21行・26行（静的タグ）／ 6335行（`syncOgpTags` 内の定数）
- **影響**：
  - X等のクローラーはJSを実行しないため、**実際のOGP表示は `ogp.jpg?v=2` が使われ問題なし**
  - ブラウザ上では毎回警告が出る＋DOM上のタグが404を指す
- **対処案**：`ogp2.jpg` を配置するか、6335行を `assets/ogp.jpg?v=2` に合わせる。
- 補足：`twitter:image`（27行）は `syncOgpTags` の対象外のため書き換わらず、`ogp.jpg?v=2` のまま。

## NG-3 「最初から始める」で一部のタイマーが解除されない

`restartFromTitle()`（4122行〜）は `introTimer` / `timeupTimer` / `glowTimer` / `penaltyTimer` / `finalTimers` / `trueEndTimer` / `tickTimer` を解除しますが、次の2つが漏れています。

### NG-3a `state.heartTimers`（ハート消滅演出）

- **現象**：ハートが消える演出（0.45秒）の途中で「最初から始める」を押すと、予約が1件残ったままタイトルへ戻る。0.8秒後にコールバックが発火し、非表示の戦闘画面のハートDOMが 5→4 に減る。
- **該当箇所**：4122行 `restartFromTitle()` に `clearHeartEffects()` の呼び出しがない
- **影響**：**表示上の実害なし**。`target.parentNode` の存在チェックがあり例外は出ず、次の戦闘開始時に `renderHp()` → `clearHeartEffects()` で確実に片付き、ハートも正しい数（5個）で再描画されることを確認済み。

### NG-3b `state.imeTimer`（IME警告）

- **現象**：ゲーム開始から5秒以内に「最初から始める」を押すと、`imeTimer` が解除されずタイトル画面滞在中に発火し、`#ime-warning` が `hidden = false` にされる。
- **該当箇所**：4122行 `restartFromTitle()` に `clearTimeout(state.imeTimer)` がない ／ 発火箇所は 6022行
- **影響**：`#ime-warning` は `#screen-game` の内側にあるためタイトル画面では見えない。ただし**次にゲームを始めた瞬間から警告が出た状態**になる（本来は開始5秒後に出る）。有効打鍵を1つ入れれば `noteValidKey()` が消すため、実害は限定的。

## NG-4 音OFFでもAudioContextの生成と効果音の先読みが走る

- **現象**：`soundEnabled: false` のままでも、スタート画面を抜けた時点で AudioContext が `running` になり、効果音9本（計約400KB）がダウンロード・デコードされる。
- **原因**：`unlockAudio()`（3402行）が `audio.enabled` を見ずに `ensureAudioContext()` と `preloadSe()` を実行する。
- **該当箇所**：`index.html` 3402〜3420行
- **影響**：**音は一切鳴らない**（`canPlay()` が `audio.enabled` を見て false を返すため、発音数は常に0）。実害は「音を使わない人にも約400KBの通信とデコード処理が発生する」点のみ。音ON時に即座に鳴らすための設計上の意図的な先読みとも読めます。
- 補足：`<audio>` 要素は一切生成されません（完全に Web Audio API のみ）。

---

# 参考：仕様の食い違い（不具合ではないが要確認）

| # | 内容 | 該当箇所 |
|---|---|---|
| A | `fifth` テーマは **6回**クリアで切替（`clearCount >= 6`）だが、サウンドトラック `title_5th` の解禁は **5回**（`clearCount >= 5`）。曲だけ1周早く聞ける。 | 2295行 / 2547行 |
| B | `flawless` テーマは **3回**クリアで切替（`clearCount >= 3`）。コメントは「2周目をクリアしたら」。サウンドトラック `title_flawless` の解禁は **2回**。 | 2301行 / 2545行 |
| C | サウンドトラック `title_cleared`（`bgm_title_cleared.mp3`）は **1回**クリアで解禁されるが、対応するタイトルテーマは存在しない（`cleared` テーマは削除済み）。ゲーム中には流れない曲。 | 2543行 |
| D | `SOUNDTRACK` 全9曲の `author` が空文字のまま。空なら非表示になるため崩れはしない。 | 2541〜2558行 |
| E | 効果音のファイル名が `se_compleate.mp3`（`complete` の綴り違い）。動作に影響なし。 | 2520行 |

---

# 人の目・耳での確認が必要な項目

機械的に検証できないため、実機での確認をお願いします。

## 音
1. **各BGMの聞こえ方** — 音量バランス（BGM 0.2〜0.4、SE 1.0〜1.2）が実際に耳で心地よいか
2. **BGMのループの継ぎ目** — 全曲 `loopStart: 0`（曲頭に戻る）設定のため、繋がりが不自然でないか。特に新規2曲（`bgm_title_5th` / `bgm_title_true_cleared_ending`）
3. **効果音のタイミングと重なり** — 打鍵音・呪文完了音・撃破音が同時に鳴ったときにうるさくないか
4. **サウンドトラックの試聴** — 曲が最後まで鳴り、終了後に「再生中」表示が消えるか（コード上は確認済み、実音での確認が必要）
5. **残り10秒の警告音**が緊迫感として適切か

## 画像・見た目
6. **全画像の内容が正しいか** — ファイル名と中身の対応（例：`miccahnSolo.jpeg` `miccahnNimushi.jpeg` は綴りが `miccahn`）
7. **各画面のレイアウト** — 960×640 での余白・文字の詰まり具合
8. **称号一覧画面の枠合わせ** — 背景イラストの枠とUIの重なり（±1px以内で実装済みだが目視確認推奨）
9. **アニメーションの見え方** — ハート消滅、白フラッシュ、暗転、ホワイトアウトの速度感
10. **ご褒美イラスト・アートブック（14ページ）・ジャケット画像**の表示

## 動作環境
11. **iPad / iPhone Safari での動作** — タッチ操作、ソフトウェアキーボード、画面サイズ
12. **macOS Safari での音** — AudioContext の `interrupted` 対策を入れた後の実挙動（Chromium では検証済み）
13. **他ブラウザ**（Firefox / Edge）での表示と音
14. **実際のキーボード入力による打鍵判定** — IME OFF/ON の切り替え、日本語入力時の警告表示
15. **デプロイ後のOGP表示** — X等でリンクを貼ったときのカード画像

## 体験
16. **難易度バランス** — 制限時間 60/60/60/90 秒と問題数 5/6/6/7 が適切か
17. **シナリオの読みやすさ** — プロローグ・エピローグ・トゥルーエンドのテンポと改行位置
18. **称号のヒント文言**が推測可能な難易度か
