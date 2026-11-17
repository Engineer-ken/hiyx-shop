# hiyo-cardshop-fabric (Minecraft 26.3 / Fabric)

hiyo-plug 1.9 (CardShopPlugin) を Fabric サーバー用 mod に移植したものです。
CARD通貨・店NPC・中央銀行(買取窓口)・入場料NPC・改札/鉄道運賃・家賃・エリトラ課税・非ログイン税・国からの給付金・カジノを含みます。
**中央銀行の買取窓口は Shift+クリックで1スタックずつ売却できます。**

## 1. jar の作り方(自分のPCにJava不要)
1. GitHub で空のリポジトリを作り、このフォルダの中身をそのままアップロード(`.github` フォルダも含める)。
2. リポジトリの **Actions** タブ → `build` が自動で走る(動かないときは `build` → **Run workflow**)。
3. 完了したら実行結果の **Artifacts → hiyo-cardshop-jar** をダウンロード。
   中の `hiyo-cardshop-fabric-1.0.0.jar` が mod 本体です(`-sources.jar` は不要)。

自分のPCで作る場合: JDK 25 を入れて `./gradlew build` → `build/libs/`。

## 2. サーバーへの導入
- `mods/` に次の2つを入れる: `hiyo-cardshop-fabric-1.0.0.jar` と **Fabric API 0.166.2+26.3**
- 初回起動で `config/cardshop/config.yml` が作られます(中身は元プラグインと同じキー)。

## 3. 元プラグインのデータ引き継ぎ
`plugins/CardShopPlugin/` の中身を `config/cardshop/` にコピーしてください(同じワールドを使うこと)。

| ファイル | 引き継ぎ |
|---|---|
| config.yml / cards.yml / bank.yml / benefits.yml / rent.yml / companyincome.yml / lastseen.txt / casino.yml / train/company.txt / web/ / entrynpcs.yml / cashbacknpcs.yml | そのまま使えます |
| shopnpcs.yml(店NPCの在庫) | 旧形式(Base64 NBT)も読み込みを試みます |
| shop.yml(`/shop` の出品) | 旧形式のアイテムは読めないためスキップされます(ログに出ます) |
| 所持中のCARD・登録済みエリトラ | 旧プラグインの印も認識します |

NPC(Mannequin)はワールドに残っているものをそのまま使います(UUIDで紐付け)。

## 4. 権限
`/cardadmin` などの管理操作は、OP(権限レベル2以上)または Fabric Permissions API のノード `cardshop:admin` を持つ人。
`cardshop:casino`(既定: 全員)、`cardshop:logintax.exempt`(非ログイン税の免除・既定: なし)。

## 5. 元プラグインからの違い(動作を変えた点)
- 拾得/インベントリ操作でのCARD名義切替 → 1秒ごとの確認で切り替え(イベントが無いため)
- 看板で駅を登録/編集した場合 → プレイヤー周辺を1秒ごとに走査して登録(チャンク読込時にも登録)
- 乗り物の駅通過判定 → 乗車中のプレイヤーだけを毎tick確認
- エリトラ → 飛行開始の次tickに「未登録/残高不足」なら着地させる
- GUI → 売買後に開き直さず、その場で表示を更新(カーソルがずれません)
- `/shop buy` で購入時にアイテムが二重に入るバグ(元の `hasInventorySpace`)を修正
- 使われていなかった `ShopGuiManager` は移植していません
- 店NPCの見た目は /summon(NBT)で生成します。スキンURLがクライアントに拒否される場合は既定スキンで表示されます

## 6. ビルドが通らなかったら
このmodは Minecraft 26.3 の本体なしで書かれているため、API名の差でコンパイルエラーが出る可能性があります。
Minecraft固有の呼び出しは `src/main/java/yourname/cardshop/util/Mc.java` と `Gui.java` / `Npc.java` に集約してあるので、
Actions のエラーログを貼れば、その数か所を直すだけで済みます。
