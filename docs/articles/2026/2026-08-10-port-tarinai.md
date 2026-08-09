---
title: VALORANT / Discord ネットワーク障害 調査まとめ
date: 2026-08-10
description: VALORANTとDiscordのネットワーク障害を、Intel I226-VのEEEとRTX830のMAP-E環境から調査した記録です。
tags:
  - VALORANT
  - Discord
  - ネットワーク
  - RTX830
  - MAP-E
  - トラブルシューティング
---

# VALORANT / Discord ネットワーク障害 調査まとめ

## 0.ここだけ手書き

こちらはネットワークの問題をchatGPT 5.6 Sol君にまとめさせながら調査したものを最後に出力させたもの。
私から伝えられることは「v6プラス(ポート割り当て240個←めっちゃ少ない)を使用する場合はポートセービングIPマスカレード機能がついているルーターを使おう」くらい。
自分はYAMAHA のルーターを使用しているせいか上位機種（[RTX1300 (希望小売価格： 295,900円(税込))](https://network.yamaha.com/products/routers/rtx1300/index)）をおすすめされましたが(買えません)、通常のルーターは多分似たような機能が入っているので気にしなくていい説はあります。

## 1. 概要

VALORANTで以下の問題が発生していた。

- マップ読み込みが極端に遅く、他プレイヤーが1ラウンド終える程度までロードが続くことがある
- Discord VCで `RTC Connecting` が長時間続き、`No Route` になることがある
- 調査途中から、VALORANTやDiscordで数秒程度の「瞬断」も発生するようになった

調査の結果、少なくとも **2種類の独立した問題が重なっていた** と判断している。

| 問題 | 原因・有力仮説 | 現在の状態 |
|---|---|---|
| 数秒間のネットワーク瞬断 | Intel I226-V の EEE（省電力イーサネット） | **ほぼ確定・解決** |
| Discord `RTC Connecting → No Route` 等のUDP接続問題 | RTX830 + MAP-E環境でのUDP外部ポート逼迫 | **非常に有力・300秒化後は再発なし** |

---

## 2. 環境

### PC

- Windows 11
- RAM 64GB
- ASUS TUF GAMING H770-PRO WIFI
- Ethernet Controller: Intel I226-V

### ネットワーク

- ルーター: Yamaha RTX830
- 回線: ドコモ光
- ISP: GMOとくとくBB
- IPv4 over IPv6: v6プラス / JPIX / MAP-E
- PC LAN IP: `192.168.100.8`
- RTX830 LAN IP: `192.168.100.1`

RTX830のMAP-Eで利用可能なIPv4外部ポートは以下の15ブロック。

```text
7296-7311
11392-11407
15488-15503
19584-19599
23680-23695
27776-27791
31872-31887
35968-35983
40064-40079
44160-44175
48256-48271
52352-52367
56448-56463
60544-60559
64640-64655
```

各16ポート × 15ブロックなので、

```text
16 × 15 = 240 ports
```

利用可能なMAP-E外部IPv4ポートは **240個**。

---

# 3. 元々発生していた問題

## 3.1 VALORANT

VALORANTのマップ読み込みが不定期に極端に遅くなる。

症状が重い場合、

```text
マップロード開始
↓
他プレイヤーは既に試合開始
↓
1ラウンド程度終了
↓
ようやくロード完了
```

という状態になる。

SSD、CPU、RAM、GPUなどについて調査したが、

- Samsung Magician正常
- SMART正常
- SSDスキャン正常
- Windows Event Viewerにストレージ系異常なし
- VALORANT再インストール済み
- アンチウイルス除外済み

などから、ストレージやPC性能が主原因である可能性は低下した。

---

## 3.2 Discord VC

Discord VCで、

```text
RTC Connecting
↓
No Route
```

となり、数十秒～数分接続できないことがあった。

Discordログを確認すると、

```text
RTC Control WebSocket
↓
正常に接続
↓
AUTHENTICATING
↓
READY
↓
UDP media connection開始
↓
Connection timed out 3 times
↓
RTC_CONNECTING => NO_ROUTE
```

となっていた。

つまり、

**DiscordのWebSocket/TCP制御接続は正常だが、音声用UDP接続だけが成立しない**

という状態だった。

---

# 4. Discord障害時のパケットキャプチャ

Pktmonで障害中の通信を取得した。

PCからDiscord media serverへのUDP送信は実際に発生していた。

しかし障害中は、

```text
PC → Discord UDP
PC → Discord UDP
PC → Discord UDP
...
```

に対して、

```text
Discord → PC
```

の応答が確認できなかった。

その後、ある時点から突然応答が返るようになり、

```text
ConnectionAttemptFinished: succeeded
Connected ... protocol: udp
RTC connected to media server
```

となった。

接続成立後のUDP ping RTTは約50ms台で正常だった。

したがって、

**DiscordクライアントがUDPを送っていなかったことが原因ではない**

ことが確認できた。

---

# 5. スマートフォンでも同じDiscord障害を確認

PCでDiscord VCへの接続に失敗している最中、

- 同じRTX830配下
- Wi-Fi接続
- モバイルデータ通信OFF

のスマートフォンでも同じVCへの接続に失敗した。

これにより、

```text
Windows
Intel I226-V
Discord Desktop
PC固有設定
```

だけが原因である可能性は大幅に低下した。

問題はRTX830またはそれより上流に存在する可能性が高いと判断した。

---

# 6. RTX830 / MAP-E NAT調査

RTX830ではNAT descriptor 20000をMAP-Eに使用。

```text
NAT/IPマスカレード 動作タイプ : 2
```

RTX830 Type2では、TCPについては外部ポート節約が行われる一方、

**UDPについては基本的に1セッションにつき1外部ポートを使用する。**

MAP-Eで利用可能なのは240ポートなので、

```text
最大240個程度のUDP NAT mapping
```

という制約が存在する。

---

## 6.1 NATテーブルでVALORANT系UDPを多数確認

RTX830のNAT詳細を確認すると、PC `192.168.100.8` から以下のような宛先へのUDP mappingが大量に存在していた。

```text
43.229.65.1:8181
151.106.248.1:8181
151.106.246.1:8181
162.249.75.1:8181
```

VALORANTはUDP `7000-8000` および `8180-8181` などを利用する。

これらのセッションが、MAP-Eで割り当てられた240個の外部ポートのかなりの部分を使用していた。

PCAPでも短時間に利用されるUDP source port数が増加し、

```text
約199
↓
218
↓
227
↓
235
↓
239
```

近くまで達する場面があった。

PC側source port数とRTX830外部ポート数は完全に同一ではないものの、UDPセッション数が非常に多いことを示す材料となった。

---

# 7. NATセッション総数との違い

RTX830では、

```text
Current/Max : 約319 / 65534
Peak        : 3001
```

などとなっていた。

これはNATセッションテーブル全体の上限には全く達していない。

ただし、

```text
65534
```

はNATセッションテーブル全体の話であり、

```text
MAP-Eで使用可能なUDP外部ポート = 240
```

とは別の制約。

したがって、

```text
NATセッション数が319しかない
↓
UDPポート枯渇ではない
```

とは判断できない。

TCPではType2のポート節約が機能するため、多数のTCPセッションとUDPポート逼迫が同時に存在できる。

---

# 8. UDP NAT timer

RTX830のNAT timerは未設定だったため、UDPについてもデフォルトの

```text
900秒
```

が使用されていた。

つまり、一度作られたUDP mappingが比較的長時間保持される。

VALORANTが多数のUDP mappingを短時間に作った場合、

```text
VALORANT
↓
多数のUDP mapping作成
↓
使用終了
↓
mappingが最大900秒程度保持
↓
MAP-Eの240ポートを長時間占有
↓
Discord等が新しいUDP mappingを作ろうとする
↓
利用可能ポート不足
↓
RTC Connecting / No Route
```

という状態になる可能性が考えられた。

---

# 9. Discord接続成功時刻との相関

障害中に使用されていた古いUDP mappingについて、

```text
最後に使用された時刻 + 900秒
```

を計算すると、Discord UDP接続が突然成功した時刻とかなり近いケースが確認された。

これは、

```text
古いUDP mappingがタイムアウト
↓
MAP-E外部ポート解放
↓
Discordが新しいmapping取得
↓
接続成功
```

という仮説と整合する。

ただし、これだけで完全な因果関係が証明されたわけではない。

---

# 10. UDP NAT timerを300秒へ変更

対策としてRTX830に、

```text
nat descriptor timer 20000 protocol=udp 300
save
```

を設定。

UDP dynamic NAT mappingの保持時間を、

```text
900秒
↓
300秒
```

へ短縮した。

TCPには影響しない。

---

## 10.1 300秒化後

変更後、以前発生していた、

```text
Discord RTC Connecting
↓
No Route
```

などのUDP接続問題は、現在まで再発していない。

そのため、

**「MAP-Eの限定された外部UDPポート + RTX830 Type2 + 900秒の長いNAT保持時間」**

が元々のUDP接続障害に関与していた可能性はさらに高まった。

ただし、発生頻度の関係から完全な証明とはせず、

**非常に有力な原因仮説であり、300秒設定が有効な対策として機能している**

と評価する。

---

# 11. 300秒化後に新たな「瞬断」を認識

300秒化後、

- Discord VCの音声が数秒途切れる
- VALORANTメニュー画面で通信が数秒止まる

という別の現象を認識した。

当初、

```text
NAT timerを300秒にした副作用
```

の可能性も検討した。

しかしパケットキャプチャを行った結果、別原因であることが判明した。

---

# 12. VALORANT瞬断時のPCAP

15:24前後にVALORANTで瞬断が発生。

PCAPを確認すると、

```text
15:24:30.328571
↓
約3.24秒
↓
15:24:33.570206
```

の間、Intel I226-V側の通信が完全に停止していた。

復旧直後には、

```text
DHCP
ARP
IPv6 Neighbor Solicitation
IPv6 Duplicate Address Detection
Router Solicitation
MLD
```

など、NIC接続後のネットワーク初期化処理が発生していた。

これはNAT mappingの問題ではなく、

**WindowsからEthernetインターフェース自体が一度切断された**

ことを示していた。

---

# 13. 15:32 / 15:35にも同じ現象を捕捉

別のPCAPでは、

```text
15:32:03.268
↓
15:32:06.475

約3.207秒
```

および、

```text
15:35:10.391
↓
15:35:13.602

約3.2秒
```

のEthernet通信停止を確認。

15:35のVALORANT/Riot UDP `:8181` も、

```text
15:35:10.345
↓
15:35:14.313
```

まで約3.97秒停止していた。

Discord UDPも同時に停止していた。

つまり、

**VALORANTやDiscordだけではなく、Ethernet通信そのものが停止していた。**

---

# 14. Windows Event Logでリンク断を確認

PCAPの時刻とWindows System Event Logを比較。

例:

```text
2026/08/09 15:32:03
e2fnexpress ID 27
Intel(R) Ethernet Controller I226-V
ネットワーク・リンクが切断されました。

2026/08/09 15:32:06
e2fnexpress ID 32
Intel(R) Ethernet Controller I226-V
ネットワークのリンクが 1Gbps 全二重通信で確立されました。
```

15:35についても、

```text
15:35:10 ID 27 Link disconnected
15:35:13 ID 32 Link established
```

とPCAPに完全一致した。

その後さらに、

```text
15:37:42
15:38:12
15:38:31
15:39:52
15:42:49
15:43:12
15:48:45
15:49:06
15:49:18
15:53:31
15:55:17
15:56:17
16:00:04
16:00:59
16:01:27
16:03:23
16:04:02
16:04:26
16:04:40
```

など非常に高頻度でID 27 → ID 32が発生していた。

リンク断は通常約3秒で復旧していた。

---

# 15. RTX830側の挙動

RTX830でもリンク復帰後、

```text
[DHCPD] LAN1(port2) Allocates 192.168.100.8
```

が、

```text
15:32:08
15:35:15
```

などに記録されていた。

Windows側の、

```text
Link Down
↓
約3秒
↓
Link Up
↓
DHCP再取得
```

と一致する。

---

# 16. 原因: Intel I226-V の EEE

調査中、Intel I226-Vのドライバを再インストールしていた。

以前の通常運用では、

```text
省電力イーサネット / Energy Efficient Ethernet
EEE = OFF
```

としていた。

しかしドライバ再インストールによって、

```text
EEE = ON
```

へ戻っていたことが判明。

タイミング的にも、今回の高頻度な3秒リンク断が発生し始めた時期と一致した。

---

# 17. EEEをOFFへ戻した結果

EEEを再びOFFへ変更。

変更直前には、

```text
16:03:23
16:04:02
16:04:26
16:04:40
```

と非常に短い間隔でリンク断が発生していた。

EEE OFF変更後はリンク断が停止。

数時間後、

```powershell
Get-WinEvent -FilterHashtable @{
    LogName      = 'System'
    ProviderName = 'e2fnexpress'
    Id           = 27,32
    StartTime    = (Get-Date).AddHours(-2)
}
```

で確認したところ、

```text
ID 27 / ID 32
該当イベントなし
```

となった。

---

# 18. EEE問題の結論

以下が成立した。

```text
以前の運用
EEE OFF
↓
高頻度リンク断なし

ドライバ再インストール
↓
EEE ON
↓
ID 27 / ID 32 が高頻度発生
↓
PCAP上でも約3秒Ethernet完全停止
↓
VALORANT / Discordも同時に瞬断

EEE OFFへ戻す
↓
それまで数十秒～数分間隔だったリンク断が停止
↓
数時間後もID 27 / 32なし
```

このため、

**今回途中から発生した数秒間のネットワーク瞬断は、Intel I226-VのEEEが原因だったとほぼ確定。**

現在はEEE OFFで運用する。

---

# 19. 最終的な問題の分離

## 問題A: I226-Vリンク瞬断

### 症状

- VALORANTが突然数秒止まる
- Discord VCが数秒途切れる
- その他Ethernet通信も同時停止

### 原因

**Intel I226-VのEnergy Efficient Ethernet (EEE)**

ドライバ再インストールによってEEEがONへ戻っていた。

### 証拠

- Windows `e2fnexpress ID 27`
- 約3秒後の `ID 32`
- PCAP上の約3.2秒のEthernet完全停止
- DHCP / IPv6再初期化
- VALORANT / Discord停止時刻との完全一致
- EEE OFF後、数時間リンク断なし

### 現在

```text
EEE = OFF
```

**解決済みと判断。**

---

## 問題B: Discord / VALORANT UDP接続問題

### 症状

Discord:

```text
RTC Connecting
↓
No Route
```

VALORANT:

- 極端なマップロード遅延
- UDP通信確立の問題が疑われる

### 有力原因

```text
JPIX v6プラス / MAP-E
        +
利用可能IPv4外部ポート約240個
        +
RTX830 NAT Type2
        +
UDPは1 mappingにつき1外部ポート
        +
VALORANTが多数のUDP mappingを生成
        +
デフォルトUDP NAT timer 900秒
```

による、

**MAP-E UDP外部ポートの一時的な逼迫・枯渇**

が最有力。

### 証拠

- Discord WebSocket接続は正常
- UDP mediaだけ `NO_ROUTE`
- PCは実際にUDPを送信
- 障害中Discord media serverから応答なし
- 後から突然UDP接続成功
- 同じLANのスマートフォンでもDiscord VC接続失敗
- RTX830 Type2
- MAP-E外部ポート240個
- VALORANT系UDP mappingが多数存在
- UDP利用数が240近くまで増える場面あり
- 900秒経過によるmapping解放とDiscord成功時刻が近いケースあり
- UDP NAT timerを300秒へ変更後、問題が現在まで再発していない

### 対策

```text
nat descriptor timer 20000 protocol=udp 300
save
```

### 現在

```text
UDP NAT timer = 300秒
```

変更後、元々のUDP接続問題は再発していない。

**非常に有力な原因仮説であり、300秒化が有効な対策として機能している状態。**

---

# 20. 現在の最終設定

## Intel I226-V

```text
Energy Efficient Ethernet / 省電力イーサネット
= Disabled
```

## Yamaha RTX830

```text
nat descriptor timer 20000 protocol=udp 300
```

---

# 21. 現在の状態

現時点では、

```text
I226-V link down (ID 27)
→ 数時間再発なし

Discord RTC Connecting / No Route
→ UDP timer 300秒化後、再発なし

VALORANT / Discord 3秒瞬断
→ EEE OFF後、再発なし
```

となっている。

したがって現在は設定を追加変更せず、

```text
EEE OFF
+
UDP NAT timer 300秒
```

をベースラインとして継続運用する。

---

# 22. 今後再発した場合

## 数秒間のネットワーク瞬断

最初に確認する。

```powershell
Get-WinEvent -FilterHashtable @{
    LogName      = 'System'
    ProviderName = 'e2fnexpress'
    Id           = 27,32
    StartTime    = (Get-Date).AddHours(-2)
} |
Select-Object TimeCreated, Id, Message |
Sort-Object TimeCreated
```

`ID 27 → ID 32` が再発していればI226-Vリンク問題を再調査する。

再発しない場合、別原因として扱う。

---

## Discord RTC Connecting / No Route

EEE問題とは分離して調査する。

必要に応じて、

- Discord WebRTCログ
- Pktmon PCAP
- RTX830 NAT状態
- 同じLAN上の別端末での再現確認

を行う。

特に、

```text
UDP NAT timer 300秒でも再発するか
```

を重要な判断材料とする。

---

# 23. 最終評価

### I226-V / EEE問題

**確度: 非常に高い / ほぼ確定**

EEE OFF後、直前まで高頻度だったリンクフラップが数時間完全に停止したため。

### MAP-E / UDPポート問題

**確度: 高いが完全証明ではない**

複数の独立した観測結果がUDP外部ポート逼迫仮説と整合し、UDP NAT timerを900秒から300秒へ変更後は実害が再発していない。

今後300秒設定のまま長期間再発しなければ、この仮説への確信度はさらに上がる。
