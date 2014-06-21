---
layout: post
title: "TypeScriptでダブル配列のTrieを作ってみた"
description: ""
category: ""
tags: [typescript,trie]
---
{% include JB/setup %}

## きっかけ
作り始めた動機は3つ。
1. もともと検索などで使われる技術に関わっていたいと思っていた
2. ちょうど仕事でTypeScriptを使っていた
3. オープンソースとして何か作ってみたかった

そんなこんなで(datrie4ts)[http://github.com/hase1031/datrie4ts]を作った。
まだ全ての機能を兼ね揃えているわけではありませんが、前方一致検索はできていると思う。


## Trieについて
Trieについては(日本語入力を支える技術)[http://amzn.to/1p2tnRJ]を読みながら勉強した。

実はTrieって検索は速いけど、構築コストが高いのであまりJSとかに向いていない。
(ローカルストレージとかに辞書を保存しない限りリロードすると消えるため)

そう、ただ作ってみたかっただけ。笑


Trieを一言で説明すると、エッジに文字情報の付いた木構造。
本には実装方法としてLOUDSとダブル配列があったが、今回はダブル配列を選択。
ダブル配列は使用するメモリ量が少ないのが特徴らしい。


## 開発について
キーのaddやremoveができるTrieを作ろうと思うと実装が複雑になるため、まずは静的なダブル配列を実装した。
ここで言う静的というのは、最初にTrie木を作ったら、それ以降変更しないということ。

インターフェイスについては、[dart](http://chasen.org/~taku/software/darts/)を見たりして考えて作った。
その結果、構築、完全一致、共通接頭辞検索を用意した。

~~~
        /**
         * Builds the double array.
         *
         * @param keys list of key, keys must be sorted alphabetically.
         * @param results value corresponding to key
         */
        build(keys:Array<string>, results:Array<ResultType>): number

        /**
         * Searches result exactly.
         *
         * @param key
         * @param n number of node
         */
        exactMatchSearch(key:string, n:number): ResultType

        /**
         * Searches results partially.
         *
         * @param key
         * @param n number of node
         */
        commonPrefixSearch(key:string, n:number): Array<ResultType>
~~~

実装方法についてはもっとスマートな方法があるかもしれないので、改善していきたい。
特にダブル配列の空き要素を探すところがただの線形探索になっているのでもっと速くできるはず。


## 今後
addとremoveを実装したい。
そして自分でこのライブラリを使っていきたい。

動的なダブル配列はどうしても処理が重くなってしまうが、まずは動くようにするだけでも。


