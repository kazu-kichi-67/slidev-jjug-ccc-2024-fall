---
theme: seriph
colorSchema: light
background: https://cover.sli.dev
title: Java x Spring Boot製アプリケーションのコールドスタートに立ち向かう！
class: text-center
drawings:
  persist: false
transition: fade
mdc: true
hideInToc: true
lineNumbers: true
---

## Java x Spring Boot製アプリケーションのコールドスタートに立ち向かう！

## 〜暖機運転のアプローチいろいろやってみた〜

<br>

2024/10/27 Presentation for JJUG CCC 2024 Fall

菊地 和真@kazu_kichi_67

<div class="abs-br m-6 flex gap-2">
  <a href="https://x.com/kazu_kichi_67" target="_blank" alt="X" title="Open in X"
    class="text-xl slidev-icon-btn opacity-50 !border-none !hover:text-white">
    <carbon-logo-x />
  </a>
  <a href="https://github.com/kazu-kichi-67" target="_blank" alt="GitHub" title="Open in GitHub"
    class="text-xl slidev-icon-btn opacity-50 !border-none !hover:text-white">
    <carbon-logo-github />
  </a>
</div>

---
layout: section
hideInToc: true
---

TODO: ハッシュタグのQR

---
src: ./pages/who-am-i.md
hide: false
---

---
hideInToc: true
---

# Agenda

***

<Toc maxDepth="2"/>

---
layout: section
---

# 本題に入る前に準備

---
level: 2
hideInToc: true
---

# 背景

***

<br>

- Java 21
- Spring Boot 3.2
- Amazon EKS（Elastic Kubernetes Service）上で動く
- Onion Architecture
- ECサイトにおける注文システムのバックエンドAPI

---
layout: section
hideInToc: true
---

<div id="highlight">
なんか、最初のリクエストだけ<br>
異常に遅くない・・？？
</div>

---
layout: section
hideInToc: true
---

<div id="highlight">
地味に困ってる人多い気がするけど、
情報少なくない・・？？
</div>

---
layout: section
hideInToc: true
---

<div id="highlight">
よし、登壇しよう！！
</div>

---
level: 2
hideInToc: true
---

# モチベーション

***

<br>

### 話すこと

- なぜ遅いのか？（簡単に）
- 暖機運転のアプローチ
- Javaが提供するアプローチ

<br>

### 話さないこと

- 各アプローチの詳細な説明
- パフォーマンス問題に対する銀の弾丸
- 背景によって最適解は異なるため、各々で計測・検証をお願いします🙏

<br>

<v-click>

### → 最初のアイデア出しの一助になれば幸いです

</v-click>

---
level: 2
hideInToc: true
---

# コールドスタートとは何か？

***

<br>

- スタートアップ：アプリケーションが起動するまでの時間
- <span v-mark.red>ウォームアップ：ピークパフォーマンスに達するまでの時間</span>
- スタートアップ + ウォームアップ => コールドスタート

---
level: 2
hideInToc: true
---

# ウォームアップの段階

***

<br>

- <span v-mark.red>初回リクエスト特有の遅延</span>：1回のリクエストでそこそこ効く
- JITコンパイラ（Just In Time Compile）による最適化：一般的にC1で数千、C2で3万回程度のリクエストが必要

<div class="grid grid-cols-2 gap-4">

<div>

<br>

```mermaid {scale: 0.5}
---
config:
  xyChart:
    titleFontSize: 30
    xAxis:
      showLabel: false
      titleFontSize: 30
    yAxis:
      showLabel: false
      titleFontSize: 30
  themeVariables:
    xyChart:
      plotColorPalette: "#146b8c"
---
xychart-beta
  title "レイテンシーの遷移イメージ"
  x-axis "Request" 0 --> 10
  y-axis "Time" 0 --> 1
  line [1, 0.2, 0.19, 0.18, 0.17, 0.16, 0.15, 0.15, 0.15, 0.15]
```

</div>

<div>

出典元: https://shipilev.net/talks/j1-Oct2011-21682-benchmarking.pdf

<img src="/Java_Runtime_Lifecycle.png"/>

</div>

</div>

<style>
p {
    font-size: 10pt
}
</style>

---
level: 2
hideInToc: true
---

# ウォームアップの重要性

***

<br>

- アプリケーションの生存時間は短くなっている
  - コンテナやサーバレスの普及
    - Podの再起動、オートスケールアップ
  - アジャイル開発による、高速なリリースサイクル
- <span v-mark.red>恩恵よりもデメリットが目立つようになってきた</span>

---
level: 2
hideInToc: true
---

# どうやって暖機するのか？

***

<br>

- <span v-mark.red>実際にAPIを呼び出してから、ユーザーのリクエストを受け付ける</span>
  - 例えば、、
  - Spring Boot Actuatorでアプリケーションの状態を可視化
  - kubernetesのStartup Probeでリクエストを投げる
- 参照系は比較的容易、では<span v-mark.circle.orange>更新系</span>は？

<br>

### 参考

[Liveness and Readiness Probes with Spring Boot](https://spring.io/blog/2020/03/25/liveness-and-readiness-probes-with-spring-boot)

[Liveness, Readiness, and Startup Probes](https://kubernetes.io/docs/concepts/configuration/liveness-readiness-startup-probes/)

---
layout: section
---

# 暖機運転のアプローチ

---
level: 2
---

# 繰り返し呼べるように環境を整える

***

<br>

### 注文APIについて考えてみる

<v-clicks>

- 暖機用のユーザ、商品を準備する
- 自動で注文をキャンセルする仕組みが必要
- キャンセルを繰り返すユーザに対して罰則があればその対象外とする
- 検索に載せないなど、一般ユーザには買えない仕組みが欲しい
- 計測や分析側への影響も考慮する必要あり

</v-clicks>

<br>

<v-click>

- ❌ めっちゃつらい！！

</v-click>

---
level: 2
---

# 特殊ルートの実装

***

```java {*|1-2|3|4|5-6|7-9|*}
@Repository
public class OrderRepositoryImpl implements OrderRepository {
	public OrderId create(Order order) {
    if (order.userId().isWarmup()) {
      // 暖機運転用の特殊ルート
      return OrderId.ofWarmup();
    } else {
      // 注文作成
    }
  } 
}
```

<v-clicks>

- ⭕️ 小規模かつ、暫定であれば・・??
- 🔺 暖機効果がやや低い
- ❌ 実装・メンテナンスコスト大
- ❌ ドメインモデルに余計な関心ごとが入り込む

</v-clicks>

---
level: 2
---

# Dynamic Dependency Injection

***

```java {*|1,3|2,4-5|7-12|9|11|*}
@Configuration
@RequiredArgsConstructor
public class WarmupConfiguration {
  private final BeanFactory beanFactory;
  private final WarmupService warmupService;

  @Bean
  @Primary
  @Scope(value = "prototype", proxyMode = ScopedProxyMode.INTERFACES)
	public OrderRepository orderRepository() {
		return beanFactory.getBean(warmupService.getMode() + "OrderRepository", OrderRepository.class);
	}
}
```

<v-clicks>

- ⭕️ 暖機の関心ごとが散らばらない
- 🔺 暖機効果がやや低い
- 🔺 実装・メンテナンスコスト中
- ❌ 都度DIされることによる性能懸念

</v-clicks>

---
level: 2
---

# Dynamic Data Source

***

<br>

```java {*|3|6-7|8|*}
@Component
@RequiredArgsConstructor
public class DynamicRoutingDataSource extends AbstractRoutingDataSource {
  private final WarmupService warmupService;

  @Override
  protected Object determineCurrentLookupKey() {
		return warmupService.getMode();
	}
}
```

---
level: 2
hideInToc: true
---

# Dynamic Data Source

***

<br>

```java {*|3|11-13|15-16|*}
@Configuration
@RequiredArgsConstructor
public class DynamicDataSourceConfiguration {
  private final DynamicRoutingDataSource dynamicRoutingDataSource;

  ・・・

  @Bean
  @Primary
	public DataSource dataSource() {
    Map<Object, DataSource> dataSources = new LinkedHashMap<>();
    dataSources.put(WarmupMode.PROD, prodDataSource());
    dataSources.put(WarmupMode.WARMUP, warmupDataSource());

    dynamicRoutingDataSource.setTargetDataSources(dataSources);
    dynamicRoutingDataSource.setDefaultTargetDataSource(prodDataSource());

		return dynamicRoutingDataSource;
	}
}
```

---
level: 2
hideInToc: true
---

# Dynamic Data Source

***

<br>

<v-clicks>

- ⭕️ データを自由に準備できるため、暖機の自由度は高い
- ⭕️ 暖機効果が高い
- ⭕️ 実装・メンテナンスコスト小
- ❌ 都度DIされることによる性能懸念
- ❌ インフラの整備が必要
- ❌ 接続先が切り替わらなかった場合の事故リスク

</v-clicks>

<br>

<v-click>

### 結論

- どのアプローチもデメリット（痛み）を伴う

</v-click>

---
layout: section
---

# 暖機運転以外のアプローチ


---
level: 2
---

# Class Data Sharing（CDS）

***

<br>

- [JEP 310: Application Class-Data Sharing](https://openjdk.org/jeps/310)
- [JEP 341: Default CDS Archives](https://openjdk.org/jeps/341)
- [JEP 350: Dynamic CDS Archives](https://openjdk.org/jeps/350)

---
level: 2
---

# Native化（GraalVM）

***

<br>

- AOTコンパイル（Ahead of Time Compile）
- ⭕️ 起動時間
- ⭕️ パフォーマンス
- ❌ 対応ライブラリが限られる
- ❌ 導入のハードルが高い
- ❌ コンパイルに時間がかかる

---
level: 2
---

# CRaC(Coordinated Restore at Checkpoint)

***

<br>

- AWS LambdaのSnapStartで活用されている
- Linux KernelのCheckpoint/Restore in Userspace（CRIU）を利用するため、実行環境に依存する
- adoptium/temurin-buildでは未対応
  - [Including CRac for container image](https://github.com/adoptium/temurin-build/issues/3604)

---
level: 2
---

# Project Leyden

***

<br>

- [Project Leyden](https://openjdk.org/projects/leyden/)

---

# まとめ

***

<br>

- 暖機運転を行うのが望ましい
- <span v-mark.red>コストやリスクのトレードオフを考慮し、本当に必要なところだけ導入する</span>
- Javaの今後のバージョンアップにも注目!!

---
layout: center
class: text-center
hideInToc: true
---

# End

Thank you for listening!

良いJava Lifeを！

<img src="/QR_X.png"/>
