---
title: Zabbix の VMware 監視機能のバグフィックスを取り込んでもらえた話
slug: zabbix-vmware-monitoring-bug-fix
date: 2014-06-23T00:00:00+09:00
categories: 
- "Tech"
tags: 
- "zabbix"
- " vmware"
---


今回取り込んでもらえたのは、100台以上の ESXi が登録されないというバグに対するパッチです。

このバグはチケットを作る前から中の方に報告はしていたのですが、なかなか直らなかったので、自分でパッチを作って報告しました。

2.2.1 の頃からパッチを当てたものを使っていて、 2.2.2 が出たときに今後もパッチを作り続けなければならないのかなと思ったのですが、他の影響力のあるユーザー企業さんが同じ問題にぶつかり、急に優先度が上がって取り込まれたということでよかったです。

とにかく、 OSS でバグを見つけたら、報告したり、パッチを作って送ったりすることは大事です。

なかなか取り込んでくれなくても、開発者は多忙なので、取り込んでもらえるまで諦めないことも大事です。

以下パッチの詳細です。

JIRA のチケット
---------------

https://support.zabbix.com/browse/ZBX-7721

チケットにしないと気づいてもらえないので、チケットにするのは大事です。

英語が間違っているかもしれませんが、間違ってたらすみませんと思いながら気にせず投稿しています。

間違ってたり、わからなかったりしたら突っ込んでくれるはずなので、大丈夫だと思っています。

このバグの原因について
----------------------

オートディスカバリで vCenter 配下の Hypervisor を取ってくる機能は、 VMware の API の RetrievePropertiesEx を使って実現しています。

この API はデフォルトでは100個しか取ってこれず、次のリストがある場合、 token を返すようになっています。

この token を使って、 ContinueRetrievePropertiesEx という API を叩くと、101台目以降が取得できるのですが、実装されていませんでした。

ソースを見るとわかるのですが、 libcurl で直に SOAP の XML を投げていて、胸が熱くなります。

2.2.2 向けのパッチ
------------------

2.2.4 RC1 か 2.3.0 から直っているのですが、 2.2.2 を使っていて、困っている奇特な方はお使いください。

<script src="https://gist.github.com/higebu/9826313.js"></script>

```bash
wget http://downloads.sourceforge.net/project/zabbix/ZABBIX%20Latest%20Stable/2.2.2/zabbix-2.2.2.tar.gz
tar xf zabbix-2.2.2.tar.gz
patch -p1 -d zabbix-2.2.2 < ZBX-7721-zabbix-2.2.2.patch
```

2.2.3 はスルーするつもりなので、作っていません。

今後
----

100台以上は取れるようになったのですが、実は200台以上が取れていないように見えるので、調査中です。