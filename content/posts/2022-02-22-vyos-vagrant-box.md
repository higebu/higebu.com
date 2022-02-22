---
title: "久しぶりにVyOSのVagrant Boxを更新した"
date: 2022-02-22T14:18:09+09:00
draft: false
categories:
  - Tech
tags:
  - vyos
  - vagrant
slug: vyos-vagrant-box
keywords: vyos,vagrant
---

![VyOS Vagrant Boxes](/images/vyos-vagrant-box-search-result.png)

# VyOSのVagrant Box更新のお知らせ

5年ほど前に作成し、 [Vagrant Cloud](https://app.vagrantup.com/boxes/search?utf8=%E2%9C%93&sort=downloads&provider=&q=vyos) で公開したVyOSのVagrant Boxですが、その後メンテせずに放置してしまっていました。

使っていた方々には大変ご迷惑をおかけしました。

しかし、 [Ansible公式ブログ](https://www.ansible.com/blog/fumbling-through-networking) など、いろいろなところで `vagrant init higebu/vyos` と記載されているのを思い出し、今更ですが、更新することにしました。

現在は [higebu/vyos](https://app.vagrantup.com/higebu/boxes/vyos) [vyos/current](https://app.vagrantup.com/vyos/boxes/current) 共に最新のrolling releaseのVyOSを利用できるようになっています。

また、GitHub Actionsで自動更新されるようにしていますので、本体に大きな変更がない限り、常に最新版が利用できるはずです。

# 使い方

使い方は下記の通りです。

```shell
vagrant plugin install vagrant-vyos
vagrant init vyos/current
vagrant up
```

# 裏話

VMイメージをビルドするためのAnsible Playbookが https://github.com/vyos/vyos-vm-images にあるのですが、そのままでは元々あったlibvirt向けのplaybookも動かなかったため、いろいろと修正し、VirtualBox向けのplaybookも追加しました。

修正したplaybookはforkした https://github.com/higebu/vyos-vm-images にあり、[Pull Request](https://github.com/vyos/vyos-vm-images/pull/22) もしています。

Vagrant Box以外のplaybookも壊れている可能性があるので、時間があれば修正していきたいと思っています。

以上です。
