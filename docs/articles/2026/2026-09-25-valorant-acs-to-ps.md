---
title: "VALORANTのPerformance ScoreはACSから何が変わったのか"
description: "VALORANTのACSからPerformance Scoreへの変更点を、Riot公式情報と実試合データから考察する。内部MMRとの関連についても整理。"
date: 2026-09-25
tags:
  - VALORANT
  - Performance Score
  - ACS
  - MMR
---

# VALORANTのPerformance ScoreはACSから何が変わったのか

2026年9月のPatch 13.06で、VALORANTでは従来の **ACS（Average Combat Score / 平均バトルスコア）** に代わって、0～500の **Performance Score（パフォーマンススコア）** が導入された。

Riotの公式説明では、Performance Scoreは「ダメージやキルだけでなく、試合への貢献をより総合的に評価する」ための指標とされている。

実際にゲーム内で確認すると、従来のACSとはかなり異なる項目が評価対象として表示される。

この記事では、

- ACSからPerformance Scoreになって何が変わったのか
- 逆に、何が変わっていないのか
- Performance Scoreと内部MMRにはどのような関係がありそうか

について、Riotの公式情報と実際の試合データをもとに考察する。

!!! note "この記事について"

    Performance Scoreおよび内部MMRの正確な計算式はRiotから公開されていない。

    そのためこの記事では、

    - Riotが公式に発表している内容
    - ゲーム内で直接確認できる内容
    - 実試合データから推測できる内容

    をなるべく区別して扱う。

---

## 結論

先に結論を書く。

ACSからPerformance Scoreへの変更は、

> **キルやダメージを中心に「個人の戦闘結果」を測る指標から、戦闘結果を土台にしつつ「チームの戦闘にどう関与したか」まで評価する指標への拡張**

と見るのが、現時点では最も自然だと思う。

大まかに整理すると次のようになる。

| 要素 | ACS → Performance Scoreでの変化についての考察 |
| --- | --- |
| キル | Performance Scoreでも強く関係している |
| ダメージ | Performance Scoreでも強く関係している |
| アシスト | 明示的に評価され、追加の説明力を持つ可能性が高い |
| トレード | 新たに明示された評価項目 |
| デス | 「調整デス」として明示的な評価対象になった |
| ユーティリティー | 評価項目として明示されている |
| 設置・解除 | 独立した表示項目として評価される |
| ファーストキル | ACSに見られた「ラウンド早期のKillを強く評価する構造」に相当する大きな追加効果は確認しづらい。ただしFirst Kill自体の価値が下がったとは言えない |

特に重要なのは、

> 「キルやダメージを軽視して、その代わりにサポート行動を評価する指標になった」わけではなさそう

という点だ。

実際の試合データを見る限り、KillやDamageはPerformance Scoreでも依然として強く関係している。

そのうえで、

- Assist
- Trade
- Death
- Utility
- Objective

といった、ACSでは十分に表現できなかった要素まで評価範囲を広げたと考えるのがよさそうだ。

概念的には、

```text
Performance Score
  ≒ Combat
  + Team Interaction
  + Utility / Objective
```

のようなイメージに近い。ただし、これは計算式を示したものではない。

また、KillやDamageについても「ACS時代と同じ重みで評価されている」とまでは言えない。分かるのは、Performance ScoreでもKillとDamageが主要な要素に見えるというところまでである。

内部MMRとの関係については、Performance Scoreの導入に合わせてMMRの算出方法も変更された、という公式発表は確認できない。

Riotは2026年1月のPatch 12.00で内部MMRの算出方法を変更した際には、その変更をパッチノートで明確にアナウンスしている。一方、Patch 13.06ではPerformance Scoreの導入について説明されているものの、内部MMRの変更には触れられていない。

この違いは、今回MMR側を同時に変更したのではなく、**表示上の評価指標をACSからPerformance Scoreへ更新し、既存の内部MMRのPerformance評価に近づけた**という仮説を考えるうえで、意味のある状況証拠になる。

ただし、告知がないことはMMRが変更されなかった証明ではない。また、この状況証拠だけでPerformance ScoreがACSより内部MMRの評価に近いとまでは確認できない。両者がどの程度似ているかは、現在公開されている情報だけでは判断できず、ここは検証可能な追加情報を待つ仮説として扱う。

---

## 今回確認したデータ

今回の考察では、主に以下のデータを確認した。

- 同一アカウントの連続したCompetitive 13試合
- 各試合のPerformance Score詳細
- Tracker上のK/D/A、ACS、First Killなど
- 複数試合について、全10プレイヤー分の
    - Performance Score
    - K/D/A
    - Trade
    - First Kill
    - Plant
    - Defuse

First Killについては、追加データを含めておよそ **80 player-match** 程度を比較した。

さらにFirst Kill単独の見かけ上の相関だけで判断せず、

- Kill
- Death
- Assist
- Trade
- First Kill

を同時に説明変数として扱い、Performance Scoreとの関係を見る簡易的な線形回帰も行った。試合ごとのラウンド数やスコア分布の違いをそのまま混ぜないよう、試合ごとの差も考慮して比較している。

この分析では、Killは強い正の関係、AssistとTradeも正の関係、Deathは負の関係を示し、First Killは他の変数を考慮するとかなり小さい関係となった。

ただし、今回取得できていないDamage、Adjusted Kill、Adjusted Death、Utility、Agent、Killが発生した状況などもPerformance Scoreへ影響している可能性が高い。そのため、この回帰はRiotの計算式や正式な係数を推定するものではなく、観測できた複数要素のうち何が追加の説明力を持ちそうかを見る簡易分析として扱う。

この規模のサンプルから正確な計算式は逆算できないが、ACSとPerformance Scoreの評価傾向の違いを考える材料として、いくつかの傾向が確認できた。

---

## ACSは何を評価していたのか

ACSも単純なキル数ランキングではなかった。

Riotは過去にCombat Scoreについて、少なくとも

- キル
- 与ダメージ
- First Blood
- 連続キル
- ダメージによるAssist

などがスコアに影響すると説明している。

また当時から、

- 敵を発見する
- 敵を遅延する
- 味方を回復する

といった **非ダメージ系の貢献をより評価する方法を検討している** とも述べていた。

かなり単純化すれば、ACSは

`Damage + Kill + Early Kill + Multi Kill + Assist`

のような、**Combatを中心とした評価指標**だったと考えられる。

Riot自身も旧Combat Scoreについて、試合における「core combat」でのPerformanceを示す指標として説明していた。

---

## Performance Scoreで評価されているもの

現在のPerformance Score詳細画面では、少なくとも以下の項目が表示される。

- ダメージ
- 調整キル数
- 調整デス数
- トレード
- アシスト数
- ユーティリティー用途
- 設置数
- 解除数

この時点ですでに、ACSとはかなり方向性が異なる。

Patch 13.06でRiotはPerformance Scoreについて、

> ダメージやキルだけでなく、試合への貢献をより総合的に評価するための0～500の数値

と説明している。

またPerformance Scoreは、

- Competitive
- Unrated
- Swiftplay
- Premier

で使用され、これらのモードではMVPおよびスコアボード順位もPerformance Scoreによって決定されるようになった。

---

## Performance Scoreでも引き続き重要なもの：KillとDamage

Performance Scoreになったからといって、

**KillやDamageが重要でなくなったわけではない。**

自分の連続した13試合を見ると、

- Damage
- Adjusted Kill
- ACS

はいずれもPerformance Scoreとかなり強く連動していた。

AssistやUtilityが多くても、

- Damageが低い
- Adjusted Killが低い

試合ではPerformance Scoreも低いケースが確認できる。

逆に、戦闘スタッツが非常に高い試合では基本的にPerformance Scoreも高い。

したがって、

> Performance Scoreはサポートプレイヤーを評価するためにCombatを軽視した指標

という理解は適切ではなさそうだ。

むしろ、

`Performance Score = Combat Base + Team Interaction + Utility / Objective`

という構造をイメージした方が分かりやすい。

**ACSで評価していたCombatを重要な要素として残し、その外側に評価範囲を広げた**

と考えるのが自然だと思う。

---

## 大きく変わったもの：Trade

ACSとの違いとして特に興味深いのが **Trade** である。

Performance Scoreでは「トレード」が独立した評価項目として存在する。

つまり単純に、

> 敵を倒した

という結果だけではなく、

> **味方の戦闘とどのような関係で敵を倒したのか**

まで区別して評価していることになる。

自分の13試合でも、

- Trade / roundが高い試合では良い評価
- Trade / roundが低い試合では悪い評価

という比較的はっきりした傾向がPerformance画面に表示された。

これはACSにはなかった視点である。

概念的には、

`Kill`

だけではなく、

`Kill + Relation to Teammates`

を見るようになった、と考えることができる。

!!! caution "「カバー」とTradeは同じではない"

    ここでゲーム上確認できるのは、あくまで「Trade」という結果。

    例えば、

    - 味方とクロスを組んでいる
    - 味方がピークした際に撃てる位置を維持している
    - 敵にピークさせない角度を保持している

    といった「潜在的なカバー」までPerformance Scoreが評価している証拠はない。

---

## Assistの評価も大きくなった可能性が高い

Assistについては、最近のRiotの方向性ともよく一致している。

Patch 12.05では、

- Assist Banner
- Killfeed上でのAssist表示
- Assistに使われたAbilityの表示

などが追加された。

この際Riotは、Assistについて

> ラウンド勝利に大きく貢献する一方、これまで十分な評価を得られていなかった

という趣旨の説明をしている。

さらに、この変更自体を

> チームプレイを認識し、推奨する

ためのものとして説明している。

その約半年後にPerformance Scoreが導入された。

実データでは、Kill / Death / Assist / Tradeなどを同時に考慮した簡易分析でも、AssistとTradeにはPerformance Scoreに対する追加の正方向の関係が残った。

この結果は、Combatの結果だけでは説明しきれない部分をAssistやTradeが補っている、という見方と比較的よく整合する。

例えばあるOmenの試合では、

| 項目 | 値 |
| --- | ---: |
| K/D/A | 15 / 15 / 9 |
| ACS | 192 |
| Performance Score | 273 |
| Assist / round | 0.41 |
| Trade / round | 0.18 |

となっていた。

別のSkyeでは、

| 項目 | 値 |
| --- | ---: |
| K/D/A | 16 / 16 / 8 |
| ACS | 210 |
| Performance Score | 279 |
| Assist / round | 0.33 |
| Trade / round | 0.17 |

だった。

これらは、AssistやTradeが多い試合でPerformance Scoreも高かった例として参考になる。ただし、ACSとPerformance Scoreは別の指標であり、

- 尺度
- 平均値
- 分布
- 上限
- 各要素の重み

が同じである根拠はない。たとえばPerformance ScoreからACSを引いた数値を、そのままAssistやTradeによる加点とみなすことはできない。

見るべきなのは絶対値の差ではなく、Combat成績が近いプレイヤー同士でPerformance Scoreに差があるか、KillやDeathなどを考慮した後でもAssistやTradeが説明に寄与するかである。今回の簡易分析では、この意味でAssistとTradeが追加の説明力を持つ可能性が確認できた。

!!! note "AssistがKillより重要という意味ではない"

    AssistやTradeの評価が以前より大きくなったように見えることと、KillよりAssistの方が重要になったことは別の話。

    実データでは依然としてKillやDamageの影響が非常に大きい。

    Performance Scoreでは、従来のCombat評価に対してAssistやTradeが追加の説明力を持つようになった、と考える方が適切だと思う。

---

## Deathも明示的な評価軸になった

ACSでは、

`Death = -X点`

という形で、Deathそのものを直接減点する指標ではなかった。

もちろん早く死ねば、その後DamageやKillを取る機会を失うため間接的には不利になる。

しかしCombat Scoreの中心は、自分が敵に対して行った戦闘結果だった。

Performance Scoreには明確に、

**「調整デス数（Adjusted Death）」**

という項目が存在する。

自分のデータでも、

- Adjusted Deathが少ない → 良い評価
- Adjusted Deathが多い → 悪い評価

というかなり素直な傾向が確認できた。

つまり新しい指標では、

> 何人倒したか

だけではなく、

> **何回、どのような形で自分が失われたか**

もCombat Performanceへより明示的に含めている可能性がある。

---

## 「調整キル」「調整デス」とは何なのか

Performance Scoreの中でも特に興味深いのが、

- Adjusted Kill
- Adjusted Death

である。

Adjusted Killは単純な

`Kills / Rounds`

とは一致しない。

実際の試合では、

`7 kills / 16 rounds = 0.438`

なのに、

`Adjusted Kill = 0.53`

と表示されるケースがあった。

逆に、

`11 kills / 13 rounds = 0.846`

なのに、

`Adjusted Kill = 0.79`

となるケースもある。

したがって、

**Adjusted Kill ≠ Kills / Rounds**

であることは確認できる。

つまりPerformance Scoreでは、Killを単純に「1 Kill = 1」として扱ってはいない。

何を使って補正しているかはRiotから公開されていない。

考えられる候補としては、

- 人数状況
- Killの発生タイミング
- Tradeかどうか
- ラウンドへの影響
- 戦闘に関与した他プレイヤー
- その他のCombat Context

などがある。

ただし、この部分は現時点では推測である。

---

## First Killの扱いはどう変わったのか

当初、自分は

> ACSからPerformance ScoreになってFirst Killの価値が下がったのではないか

と考えていた。

ただし追加データを確認した結果、この表現は少し強すぎると思う。

### ACSではEarly Killが構造的に高く評価されていた

旧Combat Scoreでは、Riot自身が

- First Blood
- Streak

がCombat Scoreに影響すると説明していた。

つまりACSは、ラウンド序盤のKillを高く評価する性格を持っていた。

---

### Performance Scoreでは大きな「独立加点」は確認しづらい

追加で複数試合の全プレイヤーについて、

- Kill
- Death
- Assist
- Trade
- First Kill
- Performance Score

を比較した。

約80 player-match程度のデータを見る限り、First Kill単独とPerformance Scoreの関係はそれほど強くなかった。

さらにKill / Death / Assist / Trade / First Killを同時に説明変数として扱う簡易的な線形回帰でも、

**First Kill数そのものによる大きな独立効果は確認しづらかった。**

この分析の目的はFirst Kill単独の相関を見ることではなく、他の観測できる戦闘スタッツを考慮した後に追加の説明力が残るかを見ることだった。

分かりやすい例として、同じ試合に以下の2人がいた。

| | K/D/A | Trade | First Kill | Performance Score |
| --- | ---: | ---: | ---: | ---: |
| Player A | 21 / 17 / 3 | 5 | 5 | 287 |
| Player B | 21 / 17 / 3 | 6 | 3 | 280 |

KDAは完全に同一。

Player AはFirst Killが2多いが、Performance Scoreの差は7だった。

もちろん、

- Damage
- Utility
- Adjusted Kill

などが同じわけではないため、この比較だけでFirst Killの係数を求めることはできない。

別の試合では次の例もある。

| | K/D/A | Trade | First Kill | Performance Score |
| --- | ---: | ---: | ---: | ---: |
| Cypher | 11 / 16 / 6 | 0 | 4 | 170 |
| Omen | 11 / 15 / 9 | 3 | 0 | 190 |

CypherはFirst Killを4回取っている。

一方OmenはFirst Killが0だが、

- Assistが3多い
- Tradeが3多い
- Deathが1少ない

という違いがあり、Performance ScoreではOmenの方が20高い。

この結果も、

> First Killに非常に大きな独立ボーナスが存在する

というモデルとはあまり整合しない。

---

### ただし「First Killの価値が下がった」とはまだ言えない

ここは明確に区別したい。

今回のデータから言えそうなのは、

> **Performance Scoreでは、First Kill数そのものにACSのような大きな独立加点が付いている形跡は確認しづらい**

というところまで。

これは、

> First Killが重要ではなくなった

ことを意味しない。

First Killは通常、

- Killとして評価される
- Damageを伴う
- 5v4を作る
- ラウンド勝率に大きく影響する

という非常に価値の高い行動である。

また、

**First Killであること自体がAdjusted Killの計算に含まれている可能性**

も否定できない。

そのため現時点では、

#### 言えそうなこと

> ACSのようにFirst Killを独立して非常に強く評価する構造は、Performance Scoreでは見えにくくなった。

#### まだ言えないこと

> First Killそのものの価値が低下した。

という切り分けが適切だと思う。

---

## UtilityとObjectiveも評価対象になった

Performance Scoreには、

- Utility
- Plant
- Defuse

も明示的に存在する。

これはCombat中心だったACSとの大きな違いである。

ただしUtilityについては、単純な使用回数だけで評価を決めているとは考えにくい。

自分のデータでは、

- Utility 1.7 / round → 良い評価
- Utility 1.8 / round → 悪い評価

というケースが存在した。

そのため、

`Score = Utility Uses × 固定係数`

のような単純な式ではなさそうだ。

考えられるものとして、

- エージェントごとの期待値
- Utilityの種類
- Utilityが実際に敵へ作用したか
- Utilityによる味方のKillへの貢献

などがある。

ただし具体的な計算方法は公開されていない。

---

## ACS → Performance Scoreで変わったこと

ここまでを整理する。

### Combat以外まで評価範囲が広がった

ACSは主に「core combat」を評価する指標だった。

Performance ScoreではRiot自身が、

**DamageやKillだけでなく、より広いMatch Contributionを見る**

と説明している。

---

### Tradeが独立した評価軸になった

敵を倒したという結果だけでなく、

**味方との戦闘上の関係**

が明示的に評価されるようになった。

---

### Assistの相対的重要度が上がった可能性が高い

実データでも、ACSとPerformance Scoreの差を説明する重要な要素に見える。

またRiot自身も2026年に、Assistについて

**これまで十分な評価を得られていなかった**

という趣旨の説明をしている。

---

### Deathが明示的な評価項目になった

Adjusted Deathという項目が存在する。

単純なKill数だけでなく、

**自分がどの程度倒されたか**

もPerformance評価に含まれる。

---

### Utility / Plant / Defuseが可視化された

Combat結果以外のラウンド貢献も評価範囲に含まれている。

---

## Performance Scoreでも引き続き重要なもの

以下はPerformance Scoreでも引き続き重要であることが確認できる。

### Killは依然として重要

Performance ScoreでもKillとScoreの間には強い関係がある。

---

### Damageも依然として重要

DamageもPerformance Scoreの中心的な要素に見える。

---

### Combat Performanceが土台

AssistやUtilityだけが多ければ高Scoreになるわけではない。

DamageやAdjusted Killが低い試合では、Performance Scoreも高くなりにくい。

つまり、

> ACSをやめてSupport Scoreにした

わけではない。

より正確には、

> **ACSが見ていたCombatを残したまま、TeamplayやUtilityまで評価範囲を広げた**

という変更だと思う。

---

## Performance Scoreと内部MMRは同じものなのか

ここからは内部MMRについて考える。

結論としては、

**Performance Score = 内部MMRの可視化**

と考える根拠はない。

Riotは旧Combat Scoreについても、

> Combat Scoreとランク更新に使用するPerformance Calculationは1対1ではない

と説明していた。

Rank UpdateではCombat Scoreに使われる一部のデータを利用するものの、それだけで決まるわけではなく、特に勝敗や勝利の決定度が重要だと説明されている。

またRiotは以前から、

**MMRと表示上のRank Rating（RR）は別のシステム**

として説明している。

さらに2026年1月のPatch 12.00では、

**内部MMRの算出方法そのものを変更した**

ことが公式に発表された。

一方でPerformance Scoreが導入されたのは2026年9月。

つまり、

- MMR算出方法の変更：2026年1月
- Performance Score導入：2026年9月

であり、そもそも変更時期が一致していない。

したがって、

`Performance Score → そのままMMRへ入力`

という単純な関係を仮定する理由はない。

---

## Performance ScoreとMMRは無関係なのか

ただし、

> Performance ScoreとMMRは全く別のデータを使っている

と考える必要もない。

VALORANTのサーバーは試合中に、

- Damage
- Kill
- Death
- Assist
- Trade
- Utility
- Objective
- 対戦相手
- ラウンド状況

などのGame Telemetryを保持できる。

そのため、概念的には次のような構造が考えられる。

```text
Game Telemetry
      |
      +---- Performance Score
      |
      +---- MMR Performance Evaluation
```

つまり、

**同じ生データの一部を材料として、目的の異なる2つの評価を行っている**

という考え方である。

Performance Scoreの目的は、

> その試合でどの程度良いPerformanceだったかを示す

こと。

MMRの目的は、

> このプレイヤーの実力をどの位置に推定すべきか

を決めること。

似ているようで目的が異なる。

---

## MMRでは「絶対値」より「期待値との差」が重要なのではないか

過去のRiotの説明には、Performanceについて興味深い考え方が登場する。

例えば旧Performanceタブでは、

- 自分より高ランクの相手を繰り返し倒す
- 自分より高ランクの相手と接戦を繰り広げる

といった場合に、

**「想定以上のPerformance」**

として認識すると説明されていた。

この考え方をMMRに当てはめると、

単純なPerformanceの絶対値より、

`Observed Performance - Expected Performance`

の方が実力推定には有用である。

例えば、同じPerformance Score 300だったとしても、

- 自分よりかなり強い相手に対して出した300
- 自分よりかなり弱い相手に対して出した300

では、実力推定上の意味は異なるはずである。

そのため内部MMRを概念的に表すなら、

`MMR変化 ≒ 勝敗による評価 + 実際のPerformanceと期待Performanceとの差`

のような構造を考えると、過去のRiotの説明とは比較的よく整合する。

!!! warning "これはRiotが公開したMMRの計算式ではない"

    上記はあくまで説明用の概念モデル。

    Riotが公開しているのは、

    - MMRが実力推定に利用されていること
    - RRとは別のシステムであること
    - PerformanceがRank Updateの一部に利用されること
    - 相手との相対的な強さによって「想定以上のPerformance」が認識される考え方
    - 2026年1月に内部MMRの算出方法を変更したこと

    までであり、現在の具体的なMMR計算式は公開されていない。

---

## RR増減からMMR変化を逆算できるか

RRとMMRは無関係ではない。

Riotの説明では、現在の表示Rankと内部MMRの位置関係によってRRの増減量が調整される。

一般的には、

- MMRが表示Rankより高い → 勝利時に多くRRを得やすい
- MMRと表示Rankが近い → RR増減が比較的均衡する
- MMRが表示Rankより低い → 勝利時のRRが少なく、敗北時の減少が大きくなりやすい

という関係がある。

ただし、

> この試合で+23RRだったから、MMRも23相当上がった

というような逆算はできない。

RRは、

- 勝敗
- ラウンド差
- Performance
- 現在のRankとMMRの位置関係

などを含む表示側の仕組みであり、MMRそのものではないためである。

今回RRの増減も確認したが、Performance Scoreとの単純な対応関係は見られなかった。

そのため今回の考察では、RRをPerformance Scoreと内部MMRの関係を推定する主要な材料にはしていない。

---

## Performance ScoreからMMRについて何が分かるのか

Performance Scoreから内部MMRを直接逆算することはできない。

ただしPerformance Scoreは、

> **Riotが現在「良いPerformance」をどのような要素に分解して捉えようとしているのか**

を見る材料としては非常に興味深い。

現在のPerformance Scoreでは、

`Kill / Damage`

だけでなく、

`Trade / Assist / Death / Utility / Objective`

まで明示的に扱っている。

もしMMR側のPerformance評価でも同じGame Telemetryの一部を利用しているなら、

**内部MMRの個人Performance評価も、単純なK/DやACSよりかなり多面的である可能性**

は十分考えられる。

ただし、

> Performance ScoreでTradeが高評価
>
> ↓
>
> MMRでもTradeが同じ係数で高評価

とは言えない。

Performance ScoreとMMRは、目的の異なるシステムだからである。

---

## まとめ

ACSからPerformance Scoreへの変更を一言で表すなら、

> **「どれだけ敵を倒したか」を中心とするCombat評価から、「チームの戦闘にどれだけ有効に関与したか」まで含む評価への拡張**

と考えている。

KillとDamageは、今回のデータでもPerformance Scoreの土台に見える。

ただし、ACS時代と同じ係数・同じ重みで評価されているかは分からない。

一方で、

- Trade
- Assist
- Adjusted Death
- Utility
- Plant / Defuse

といった要素が明示的に評価されるようになった。

First Killについては、

> **ACSのような大きな独立加点が存在する形跡は、今回のデータでは確認しづらかった**

というところまでは言えそうだ。

ただし、

> First Killそのものの価値が低下した

とまでは言えない。

Killとしての価値や5v4を作る価値は当然残っており、Adjusted Killの中へ何らかの形で組み込まれている可能性もある。

そしてPerformance Scoreと内部MMRは同一ではない。

現時点では、

> **一部のGame Telemetryを共有している可能性はあるが、異なる目的を持つ別の評価システム**

と考えるのが最も安全だと思う。

Patch 12.00では内部MMRの算出方法変更が告知され、Patch 13.06ではPerformance Scoreが導入された一方でMMR変更の告知はなかった。この違いは、表示指標を既存の内部Performance評価に近づけたという仮説を考える状況証拠になる。ただし、これだけでPerformance ScoreがACSよりMMRに近いと確認できるわけではない。

整理すると、

**Performance Score**

> その試合でどの程度良いPerformanceだったかを評価する。

**内部MMR**

> プレイヤーの実力を推定し、今後どの強さの相手とマッチさせるべきかを判断する。

という違いがある。

この区別を押さえておくと、ACS・Performance Score・RR・MMRの関係はかなり整理しやすい。

---

## 現時点では分からないこと

今回のデータからも、以下についてはまだ判断できない。

- Performance Scoreの正確な計算式
- Adjusted Killの定義
- Adjusted Deathの定義
- Agentごとの補正の有無
- Mapごとの補正の有無
- Utilityの具体的な評価方法
- First KillがAdjusted Killへ与える影響
- Tradeの正確な判定条件
- Performance ScoreとMMRで共有される入力データ
- 現在のMMRにおける個人Performanceと勝敗の正確な比率

またRiot自身もPatch 13.06で、Performance Scoreについて今後もフィードバックを見ながら更新していく予定だとしている。

そのためこの記事は、

**2026年9月時点の公式情報と、少数の実試合データから現在の仕様を考察したもの**

として読んでほしい。

---

## 参考資料

### Riot Games - [VALORANT パッチノート 13.06](https://playvalorant.com/ja-jp/news/game-updates/valorant-patch-notes-13-06/)

2026年9月22日公開。

Performance Scoreの導入について、

- ACSに代わって導入
- 0～500の数値
- DamageやKillだけではなく、より総合的なMatch Contributionを評価
- MVPおよびScoreboard順位に使用
- 今後もFeedbackを見ながら更新予定

と説明されている。

### Riot Games - [VALORANT パッチノート 12.05](https://playvalorant.com/ja-jp/news/game-updates/valorant-patch-notes-12-05/)

2026年3月17日公開。

Assist BannerやKillfeedのAssist表示を追加。

RiotはAssistについて、ラウンド勝利に大きく貢献するにもかかわらず、従来十分な評価を得られていなかったという趣旨の説明をしている。

また、これらの変更をTeamplayを認識・推奨するためのものとして説明している。

### Riot Games - [VALORANT パッチノート 12.00](https://playvalorant.com/ja-jp/news/game-updates/valorant-patch-notes-12-00/)

2026年1月6日公開。

Competitive Updateとして、

**内部MMRの算出方法を変更した**

ことが明記されている。

目的はMatch Qualityの向上と、試合ごとの体験をより一貫させることとされている。

### Riot Games - [VALORANT パッチノート 1.11](https://playvalorant.com/ja-jp/news/game-updates/valorant-patch-notes-1-11/)

2020年10月27日公開。

バトルスコアの算出方法について、ダメージ以外のアシストも考慮する変更が明記されている。

### Riot Games - [Ask VALORANT #7](https://playvalorant.com/ja-jp/news/dev/ask-valorant-7/)

2020年9月10日公開。

旧Combat Scoreについて、

- Kill
- Damage
- First Blood
- Streak

などが評価要素であることを説明。

また、

**Combat ScoreとRank Updateに使用するPerformance Calculationは1対1ではない**

ことも明言されている。

さらに旧Performanceタブの評価について、自分より高ランクの相手に対して繰り返し良い戦闘を行った場合などに「想定以上のPerformance」として認識する考え方が説明されている。

### Riot Games - [Ask VALORANT「ランクレーティング エディション」](https://playvalorant.com/ja-jp/news/dev/ask-valorant-rank-rating-edition/)

2021年3月18日公開。

MMRとRRの違いや、

- MMRがPlayer Skillの推定に利用されること
- 表示RankとMMRの位置関係によってRR増減量が変化すること

などが説明されている。
