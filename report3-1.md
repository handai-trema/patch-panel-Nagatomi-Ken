##パッチパネルの機能拡張
授業で説明したパッチの追加と削除以外に， 
以下の機能をパッチパネルに追加してください． 

1. ポートのミラーリング 
2. パッチとポートのミラーリングの一覧 

それぞれpatch_panelのサブコマンドとして実装してください．

###はじめに
今回のプログラムは[こちら](https://github.com/handai-trema/patch-panel-yamatchan/blob/master/report.md)のレポートを参考に作成した。

##解答


コートは以下のようになっている

 * [lib/patch_panel.rb](https://github.com/handai-trema/patch-panel-Nagatomi-Ken/blob/develop/lib/patch_panel.rb)
 * [bin/patch_panel](https://github.com/handai-trema/patch-panel-Nagatomi-Ken/blob/develop/bin/patch_panel)


以下のような環境で動作確認を行った

```
vswitch('patch_panel') { datapath_id 0xabc }

vhost ('host1') { ip '192.168.0.1' }
vhost ('host2') { ip '192.168.0.2' }
vhost ('host3') { ip '192.168.0.3' }

link 'patch_panel', 'host1'
link 'patch_panel', 'host2'
link 'patch_panel', 'host3'
```

まず、ポート１、ポート２の間にパッチを生成し、ポート１からポート２にパケットを送信した。

```
./bin/trema run lib/patch_panel.rb -c patch_panel.conf 

./bin/patch_panel create 0xabc 1 2

./bin/patch_panel dump_flow patch_panel
NXST_FLOW reply (xid=0x4):
 cookie=0x0, duration=26.302s, table=0, n_packets=0, n_bytes=0, idle_age=26, priority=0,in_port=1 actions=output:2
 cookie=0x0, duration=26.297s, table=0, n_packets=0, n_bytes=0, idle_age=26, priority=0,in_port=2 actions=output:1

./bin/trema send_packets --source host1 --dest host2
./bin/trema show_stats host1
Packets sent:
  192.168.0.1 -> 192.168.0.2 = 1 packet
./bin/trema show_stats host2
Packets received:
  192.168.0.1 -> 192.168.0.2 = 1 packet
./bin/trema show_stats host3
```

上記の通り、パッチが生成され、パケットがホスト１からホスト２に伝送されている。ホスト３は何も受信していない。
次に、ポート１をポート３へミラーリングし、パッチとミラーリングの一覧を表示した。

```
./bin/patch_panel create_mirror 0xabc 1 3
./bin/trema dump_flow patch_panel
NXST_FLOW reply (xid=0x4):
 cookie=0x0, duration=10.376s, table=0, n_packets=0, n_bytes=0, idle_age=10, priority=0,in_port=1 actions=output:2,output:3
 cookie=0x0, duration=10.37s, table=0, n_packets=0, n_bytes=0, idle_age=10, priority=0,in_port=2 actions=output:1,output:3

./bun/patch_panel list
patch 1 & 2
mirror 1 > 3
```

以上の結果から、テーブルが更新され、一覧の表示から１と２の間にパッチが生成され、ポート１をポート３へミラーリングしていることがわかる。


最後に、もう１度ホスト１からホスト２にパケットを送信した。

```
./bin/trema send_packets --source host1 --dest host2
./bin/trema show_stats host1
Packets sent:
  192.168.0.1 -> 192.168.0.2 = 2 packets
./bin/trema show_stats host2
Packets recieved:
  192.168.0.1 -> 192.168.0.2 = 2 packets
./bin/trema show_stats host3
Packets recieved:
  192.168.0.1 -> 192.168.0.2 = 1 packet
```
以上の結果より、ポートのミラーリングをしたことで、ホスト３がパケットを受信していることがわかる。






