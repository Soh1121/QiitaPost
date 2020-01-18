MacにVMwareで仮想環境を構築して、Ubuntuをインストールしました。
キーボードでの入力が思うようにいかず、設定変更に苦戦したので後学のために書き残します。
自身で行った設定は３つです。

# 環境
- VMware：VMware Fusion 11 Pro
- Ubuntu：Ubuntu 18.04 LTS（日本語Remix）
- PC：MacbookPro（JISキーボード）

# インストール直後の状況
- 『英数』キーが『Capslock』として動作
- 入力メソッドをmozc（Google日本語入力）に変更すると、英字入力の際も変換が有効になる

# 改善目標
- 通常のMacbookと同様の動作
    - 『かな』キーでローマ字入力
        - 『英数』キーで英数字直接入力

# Step１：キーボードの選択
Ubuntuをインストールすると、デスクトップ画面でキーボードの選択が立ち上がります。
そちらの選択は『MacBook/MacBook Pro (Intl.)』を選択します。

# Step2：『英数』キーから『Capslock』をはずす
キーボードを選択した直後では、『英数』キーに『Capslock』が割り当てられており、macと同様のキー操作ができません。
そこで、『英数』キーから『Capslock』をはずすように修正します。

## 現在の設定の確認
まずは現在の設定を確認してみましょう。
`$ setxkbmap -print`
上記コマンドを端末で入力すると下記の結果が出力されると思います。

```bash:結果
xkb_keymap {
    xkb_keycodes  { include "evdev+aliases(qwerty)" };
    xkb_types     { include "complete"  };
    xkb_compat    { include "complete"  };
    xkb_symbols   { include "pc+jp+us:2+inet(evdev)+capslock(escape)"   };
    xkb_geometry  { include "pc(pc105)" };
};
```
こちらの結果から、`xkb_symbols`を見ると`pc`、`jp`、`us`、`inet(evdev)`、`capslock(escape)`を読み込んでいることがわかります。

## 設定ファイルの編集
`pc`、`jp`といったそれぞれのオプションの細かい設定は、`/usr/share/X11/xkb/symbols/`にすべて書かれています。
ここの`jp`を編集すれば問題は解決します。
編集箇所は２か所です。
47行目付近と186行目付近の２か所を下記のように変更します。

```bash:/usr/share/X11/xkb/symbols/jp
    key <CAPS> { [ Eisu_toggle, Caps_Lock ] };
    ↓
    key <CAPS> { [ Eisu_toggle ] };
```

書き換えたら再起動して設定を反映させます。

# Step3：『英数』キーでIMEオフ
ここまでで、無事『英数』キーがCapslockから開放されました。
上部のバーの入力ソースのアイコンから、『ツール』＞『プロパティ』で『Mozcプロパティ』画面が開きます。
この画面で『キー設定の選択』の『編集』を選びます。

『Mozcキー設定』画面を下の方にスクロールしていくと、下の画像のように『英数』キーに関わる部分が出てきます。
MS-IMEの標準の設定では『英数入力切替』となっています。
![Eisu-英数入力切替.png](https://qiita-image-store.s3.amazonaws.com/0/281085/ea8c0e5c-e29a-157a-6f12-b933195d225d.png)

これを『IMEを無効化』に変更します。
![Eisu-IMEを無効化.png](https://qiita-image-store.s3.amazonaws.com/0/281085/3c75a11f-3823-2e23-f7d6-ca2590863d4b.png)

以上で目的としていた動作になります。

# 参考
[mac/linux/windowsに全対応した日本語入力切り替えキー](https://yhara.jp/2018/05/22/kana-eisu-mac-linux-win)

