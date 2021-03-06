## GraphQL

<img class="no-frame" src="resources/images/graphql_logo.png" alt="" style="width:25%">

---

### What's GraphQL ?

<ul class="good">
  <li>Demand Driven Architecture</li>
  <li>Hierarchical Composition</li>
  <li>Solid schema system</li>
</ul>

---

リモートデータから欲しい部分だけ取得できる仕組み

---

### Query example

<iframe src="https://www.graphqlhub.com/playground?query=%7B%0A%20%20graphQLHub%0A%20%20github%20%7B%0A%20%20%20%20repo(name%3A%20%22typed-css-modules%22%2C%20ownerUsername%3A%20%22quramy%22)%20%7B%0A%20%20%20%20%20%20name%0A%20%20%20%20%20%20owner%20%7B%0A%20%20%20%20%20%20%20%20login%0A%20%20%20%20%20%20%20%20company%0A%20%20%20%20%20%20%7D%0A%20%20%20%20%20%20issues(limit%3A%201)%20%7B%0A%20%20%20%20%20%20%20%20comments%20%7B%0A%20%20%20%20%20%20%20%20%20%20body%0A%20%20%20%20%20%20%20%20%7D%0A%20%20%20%20%20%20%7D%0A%20%20%20%20%7D%0A%20%20%7D%0A%7D%0A" frameborder="0" style="width:80%;height:350px"></iframe>

---

<p class="smaller">
  React Europe 2016の参加者の16% が本番利用している
</p>

<img src="resources/images/react_europe.png" alt="">

<p class="link smaller"><a href="https://youtu.be/etax3aEe2dA?t=37://youtu.be/etax3aEe2dA?t=37s" target="_blank">
<span class="fa fa-youtube-play"></span> Dan Schafer - GraphQL at Facebook at react-europe 2016</a></p>

---

...GraphQLとWebAppをどうやって繋げたら良いのか？

---

## Relay

<img class="no-frame" src="resources/images/relay_logo.svg" alt="" style="width:30%">

---

GraphQLとReactをいい感じに統合するフレームワーク

---

<div style="display:flex">
  <div>
    <p>Flux</p>
    <img src="resources/images/flux-diagram-white-background.png" alt="" height="220">
    <ul class="bad" style="font-size: smaller">
    <li>
      非同期の部分が複雑...
    </li>
    </ul>
  </div>
  <p style="width:40px"></p>
  <div>
    <p>Relay</p>
    <img src="resources/images/relay-architecture.png" alt="" height="220">
    <ul class="good" style="font-size: smaller">
    <li>
      データ管理はRelayに丸投げ！
    </li>
    </ul>
  </div>
</div>

---

<p class="smaller">
  GraphQLのfragment
<span class="fa fa-arrow-right"></span>
  React.Componentのprops
</p>


```js
function UserComponent({user}) {
  return (
    <div>
      <span>{user.name}</span>
      <img src={user.profileUrl} />
    </div>
  );
}

export const User = Relay.createContainer(UserComponent, {
  fragments: {
    user: () => Relay.QL`
      fragment on User {
        name, profileUrl
      }
    `,
  }
});
```

---

いや、でも俺、Angularの方が慣れてるんだけど...

---

<span class="fa fa-lightbulb-o" style="font-size: 120px"></span>

~~React~~ Angular Component にすればいいだけじゃね？

---

### Relay Core API

RelayからReact依存をひっぺがす取り組み

<a href="https://github.com/facebook/relay/issues/559" target="_blank">issues/559</a>で議論されている

<a href="https://github.com/andimarek/generic-relay" target="_blank">andimarek/generic-relay</a> に何かそれっぽい奴がいる


---

### やってみた

---

#### Reindex + GraphQL + Relay + Angular2

<img class="no-frame" src="resources/images/reindex_logo.svg" alt=""  style="width:180px; margin: 0 40px 0 0;">
<img class="no-frame" src="resources/images/graphql_logo.png" alt=""  style="width:160px; margin: 0 40px 0 0;">
<img class="no-frame" src="resources/images/relay_logo.svg" alt=""    style="width:160px; margin: 0 40px 0 0;">
<img class="no-frame" src="resources/images/shield-large.png" alt=""  style="width:160px;">

---

### Demonstration

<iframe class="demo" src="resources/demo/ng2-relay/" frameborder="0" style="height: 480px"></iframe>

---

### Code

RelayContainerになるやつにGraphQL書く

```js
import { Component , Input , NgZone } from "@angular/core";
import * as Relay from 'generic-relay';
import { ConnectRelay } from "../connectRelay";

export const UserContainer = Relay.createGenericContainer('UserContainer', {
  fragments: {
    user: () => Relay.QL`
      fragment user on User {
        id, name, profileUrl
      }
    `
  }
});
:
```

---

自作DecoratorでComponentにRelayContainer結合する

```js
:
@ConnectRelay({ container: UserContainer }) // RelayとつなぐオレオレDecorator
@Component({
  selector: 'user',
  template: `
  <div>
    <div class="user" ng-if="user">
      <img class="avatar" [src]="user.profileUrl" *ngIf="user.profileUrl" />
      <p class="avatar" *ngIf="!user.profileUrl"><span class="fa fa-user"></span></p>
      <h3>{{user?.name}}</h3>
    </div>
  </div>
  `,
})
export class UserComponent {
  // ここにGraphQLのQuery Dataが注入される
  private user: { name: string; profileUrl: string };
  constructor(private ngZone: NgZone) { }
}
```
<p class="smaller">
参考:
  <a class="link" href="https://tech.small-improvements.com/2016/02/25/angular-2-and-relay/" target="_blank">tech.small-improvements.com/2016/02/25/angular-2-and-relay/</a>
</p>

---

### 実現できることはreact-relayと一緒

<ul class="good">
  <li>Fragmentを使ったcolocation
  <p class="smaller">Viewが必要とするデータをComponentで宣言できる</p>
  </li>
  <li>Client Storeデータの管理
  <p class="smaller">Mutation発行 -> Cache(Store)のデータ管理をRelayに任せることができる</p>
  </li>
</ul>

---

### イケてないとこ

<ul>
<li>
<p class="smaller">
  generic-relayは結構古い(最終commitが今年1月)ので, 本家とRelayのAPIが少し違う
</p>
</li>
<li>
<p class="smaller">
  `Relay.QL`はbabel-relay-pluginで事前にcompileする必要がある. <br/> TypeScriptを使う場合は多段transpile必須
</p>
</li>
<li>
<p class="smaller">
  そもそもTypeScriptでのComponent型宣言とFragmentの記述が冗長
</p>
</li>
<li>
<p class="smaller">
Decoratorからng2のDI空間にアクセスする方法がない<br />別の方法で考えた方が良さげ(妙案募集中)
</p>
</li>
</ul>

---

# Summary

---

### 結論

 * Flux/Redux で疲弊している人は挑戦する価値があるかも.
 * Relayを使ったSPAのパターンはReact/Angularとあまり関係ない
 * Relay Coreがlaunchされるまでは、React + Relayで慣れておくのが無難

---

# Thank you!

by @Quramy

---

## Appendix

GraphQL, Relayを勉強するのに役立ったリソース, ツール達

---

### 1/2

<ul class="good">
  <li>
    <a href="https://learngraphql.com/" target="_blank">Learn GraphQL</a>
    <p class="smaller">対話的にGraphQLを触りながら学べるworkshop</p>
  </li>
  <li>
    <a href="https://github.com/skevy/graphiql-app" target="_blank">graphiql-app</a>
    <p class="smaller">GraphiQL(GraphQLの対話ツール)のElectronラッパー. <br>複数クエリの保存編集が出来るため, サーバー作成中に便利</p>
  </li>
  <li>
    <a href="https://www.graphqlhub.com/" target="_blank">GraphQL Hub</a>
    <p class="smaller">GithubやGiphyなど、有名なWebサービスをGraphQLでラップしたエンドポイントが公開されている. サーバを立てずにGraphQLが触れるが、Relayには対応していない</p>
  </li>
</ul>

---

### 2/2

<ul class="good">
  <li>
    <a href="https://chrome.google.com/webstore/detail/graphql-network/igbmhmnkobkjalekgiehijefpkdemocm?hl=en-GB" target="_blank">GraphQL Network</a>
    <p class="smaller">GraphQLの通信内容を表示してくるChrome Extension. <br> GraphQLを使ったアプリ開発時に便利</p>
  </li>
  <li>
    <a href="https://facebook.github.io/relay/docs/graphql-relay-specification.html#content" target="_blank">GraphQL Relay Specification</a>
    <p class="smaller">本家のGuide. RelayでGraphQLを使う際の約束事が書かれている. </p>
  </li>
  <li>
    <a href="https://www.reindex.io/" target="_blank">Reindex</a>
    <p class="smaller">Relayにも対応しているGraphQLのBaaS. <br />Relayを一通り勉強した後に触った方が良いかも</p>
  </li>
</ul>
