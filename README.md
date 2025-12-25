# mackerel-plugin-snmp-table

mackerel-agent のメトリックプラグインとして動作します。

テーブル状になっている SNMP の値を取得し、それぞれをメトリックに変換します。

## 使い方

```
Usage of ./mackerel-plugin-snmp-table:
  -conf filename
        config filename (default "config.yml")
  -preview
        show table
```

## 詳細設定
mackerel-plugin-snmp-table の設定について説明します。

このGitレポジトリには、サンプルファイル `config.yml.sample` があります。以下に示す設定例が含まれています。コピーしてお使いください。

### MIBの用意

MIB（Management Information Base）ファイルを mackerel-plugin-snmp-table に登録すると、設定項目などをMIB形式で扱うことができます。

設定例は以下のとおりです。

```
mib:
  directory:
    - "/usr/share/snmp/mibs/"
  modules:
    - SNMPv2-MIB
    - IF-MIB
```

`directory`にMIBファイルを格納するフォルダーを指定し、`modules`に読み込むMIBファイル名を列挙します。子フォルダーは探索しないので、MIBファイルは`directory`のフォルダーの直下に置いてください。

MIBファイルはベンダー各社から提供されています。

- Red Hat Enterprise Linuxやその派生ディストリビューションの場合は、net-snmp-libsパッケージをインストールすると、 `/usr/share/snmp/mibs/`フォルダーに`SNMPv2-MIB`や`IF-MIB`などのMIBファイルが置かれます。
- Debian GNU/Linux・Ubuntuの場合は、snmp-mibs-downloaderパッケージ（non-freeセクション）をインストールすると、`/var/lib/snmp/mibs/ietf/`フォルダーに`SNMPv2-MIB`や`IF-MIB`などのMIBファイルが置かれます。

## 接続先の設定

接続先と、テーブルでの表現にしたいOIDを選択します。

```
target:
  ipaddress: "192.168.220.1"
  community: public
  oid: ".1.3.6.1.2.1.2.2"
```

SNMP バージョンは、v2c、ポートは 161/udp で固定されております。

## メトリックの設定

得られたテーブルからメトリックに変換する設定を行います。

以下の設定例では、前述のOID (IF-MIB::ifTable) の続きとなっており、 ifDescr ごとに、ifInOctets と ifOutOctets のメトリックを作り出します。

```
metric:
  - prefix: "octets.{{.MetricValue}}"
    key: "{{.ifDescr}}"
    value:
      - name: ifInOctets
        diff: true
        unit: integer
        label: ifInOctets
      - name: ifOutOctets
        diff: true
        unit: integer
        label: ifOutOctets
```

設定の値の確認には、`-preview` オプションをつけて起動することで、OIDごとの値がタブ区切り形式で表示されます。
