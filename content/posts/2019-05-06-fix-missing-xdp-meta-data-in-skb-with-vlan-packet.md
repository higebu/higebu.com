---
title: "XDP metadata が VLAN パケットのときに失われるバグを修正した"
slug: fix-missing-xdp-meta-data-in-skb-with-vlan-packet
date: 2019-05-06T14:30:00Z
categories: 
- "Tech"
tags: 
- "ebpf"
- "xdp"
- "network"
- "linux"
- "debian"
---


## パッチ

パッチは [こちら][1] で、stable では [4.19.37][2] か [5.0.10][3] からこの修正が入っています。

内容としては `bpf_xdp_adjust_meta()` でパケットに付けたメタデータが skb のビルドプロセス内の `skb_vlan_untag()` (さらに詳しく言うと `skb_reorder_vlan_header()` ) で失われて tc の cls_bpf などで参照できない問題を修正したものです。

## Linux へのパッチの送り方について

公式ドキュメントにパッチの送り方 [Submitting patches: the essential guide to getting your code into the kernel][4] が書いてあるので、これを読めばだいたいわかるようになっていました。

また、今回はネットワーク関連だったので、 [netdev FAQ][5] も読んだところ、バグフィックスは [net][6] ブランチ、新機能は [net-next][7] ブランチということがわかり、 [net][6] ブランチにパッチを送りました。

その他に注意する点として、メールの送り方があり、これは [Email clients info for Linux][8] というページにまとまっています。

自分はこれを読まずに `git imap-send` から Gmail Web Clientでメールしてしまい、 [タブが全滅したパッチ][9] を送ってしまいました。

最終的に Thunderbird でメールしましたが、 `git send-email` が一番シンプルで楽そうです。

## Debian でのカーネルのビルド方法について

[Debian Linux Kernel Handbook][10] の [Chapter 4. Common kernel-related tasks][11] が詳しいです。

`LOCALVERSION` を付けておくとわかりやすいので下記のようなコマンドでビルドしています。

```shell
make -j`nproc` LOCALVERSION=-hoge bindeb-pkg
```

## 感想

- VLAN の処理をハードウェアオフロードしない場合、カーネルが VLAN の処理をするけど、 skb をビルドするときに VLAN ヘッダはパケットデータからは消えてしまうということ自体初めて知ったし、ドライバのソースを追ったりして、全体的に良い経験になった
- たぶん今まで VLAN と `bpf_xdp_adjust_meta()` を組み合わせて使ったことのある人がいなかった
- メール難しい
- eBPF/XDP と Linux Network Stack を組み合わせてパケットをいじる方法についてかなり詳しくなった

 [1]: https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=d85e8be2a5a02869f815dd0ac2d743deb4cd7957
 [2]: https://cdn.kernel.org/pub/linux/kernel/v4.x/ChangeLog-4.19.37
 [3]: https://cdn.kernel.org/pub/linux/kernel/v5.x/ChangeLog-5.0.10
 [4]: https://www.kernel.org/doc/html/latest/process/submitting-patches.html
 [5]: https://www.kernel.org/doc/html/latest/networking/netdev-FAQ.html
 [6]: https://git.kernel.org/pub/scm/linux/kernel/git/davem/net.git
 [7]: https://git.kernel.org/pub/scm/linux/kernel/git/davem/net-next.git
 [8]: https://www.kernel.org/doc/html/latest/process/email-clients.html
 [9]: https://patchwork.ozlabs.org/patch/1085113/
 [10]: https://kernel-team.pages.debian.net/kernel-handbook/index.html
 [11]: https://kernel-team.pages.debian.net/kernel-handbook/ch-common-tasks.html
