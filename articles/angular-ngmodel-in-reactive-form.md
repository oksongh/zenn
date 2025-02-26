---
title: ""
emoji: "🐷"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: []
published: false
---

# Angularのフォーム

Angularを使う以上、どうせ避けて通れないのでフォームの使い方は前提知識としておくが、要約を書いておく。
基本的には[公式のチュートリアル](https://angular.jp/tutorials/learn-angular)をみておけばOKだと思う。

## フォームの基本
Angularには[リアクティブフォーム](https://angular.jp/tutorials/learn-angular/17-reactive-forms)と[テンプレートフォーム](https://angular.jp/tutorials/learn-angular/15-forms)の2種類の記法がある。

チュートリアルには以下の通り書いてある。

## リアクティブフォーム
チュートリアル[リアクティブフォーム](https://angular.jp/tutorials/learn-angular/17-reactive-forms)については以下のサンプルコードが書いてある。
** ユーザーの入力はすべてFormGroup,FormArray,FormControlというオブジェクトに詰め込まれ、それらを参照することで表示を変える。 **
入力と出力がはっきり分かれているのが特徴だと思う。

例ではprofileFormを参照するときは`{{profileForm.value.name}}`のようにプロパティとして定義された変数`profileForm`からデータを取得している。

```ts:リアクティブフォーム
import {Component} from '@angular/core';
import {FormGroup, FormControl} from '@angular/forms';
import {ReactiveFormsModule} from '@angular/forms';

@Component({
  selector: 'app-root',
  template: `
    <form [formGroup]="profileForm" (ngSubmit)="handleSubmit()">
      <input type="text" formControlName="name" />
      <input type="email" formControlName="email" />
      <button type="submit">Submit</button>
    </form>

    <h2>Profile Form</h2>
    <p>Name: {{ profileForm.value.name }}</p>
    <p>Email: {{ profileForm.value.email }}</p>
  `,
  imports: [ReactiveFormsModule],
})
export class AppComponent {
  profileForm = new FormGroup({
    name: new FormControl(''),
    email: new FormControl(''),
  });

  handleSubmit() {
    alert(this.profileForm.value.name + ' | ' + this.profileForm.value.email);
  }
}
```

## テンプレートフォーム

チュートリアル[テンプレートフォーム](https://angular.jp/tutorials/learn-angular/15-forms)については以下のサンプルコードが書いてある。
`[(ngModel)]="foo"`と書くと定義したプロパティ(foo)を参照して表示し、ユーザーの入力をプロパティへ挿入する、という双方向にデータのやり取りを行う（公式は双方向バインディングと呼んでいる）。
つまり、`[(ngModel)]="foo"`は入力も出力も担っている。

```ts:テンプレートフォーム
import {Component} from '@angular/core';
import {FormsModule} from '@angular/forms';

@Component({
  selector: 'app-user',
  template: `
    <p>Username: {{ username }}</p>
    <p>{{ username }}'s favorite framework: {{ favoriteFramework }}</p>
    <label for="framework">
      Favorite Framework:
      <input id="framework" type="text" [(ngModel)]="favoriteFramework" />
    </label>
  `,
  imports: [FormsModule],
})
export class UserComponent {
  favoriteFramework = '';
  username = 'youngTech';
}

```

# リアクティブフォームとテンプレートフォームを同時に使いたい

本題。
結論。
リアクティブフォームとテンプレートフォームを同時に使いたいときがある。
テンプレートフォーム形式のフォームをリアクティブフォームの外に置く場合は問題ないが、中に置くときはテンプレートに`(ngModelOptions)={standalone:true}`を追加する。

## テンプレートフォームをリアクティブフォームの外に置く

特に問題ない。
```ts:テンプレートフォームが外にある
import 'zone.js';
import { Component } from '@angular/core';
import { bootstrapApplication } from '@angular/platform-browser';
import {FormGroup, FormControl} from '@angular/forms';
import {ReactiveFormsModule} from '@angular/forms';
import {FormsModule} from '@angular/forms';

@Component({
  selector: 'app-root',
  standalone: true,
  template: `
    <!-- reactive form -->
    <form [formGroup]="profileForm">
      <label>name:<input type="text" formControlName="name" /></label>
    </form>

    <!-- template form -->
    <label>
      Favorite Framework:<input id="framework" type="text" [(ngModel)]="favoriteFramework" />
    </label>
    <h2>Profile Form</h2>
    <p>{{ profileForm.value.name }}'s favorite framework: {{ favoriteFramework }}</p>
  `,
  imports: [ReactiveFormsModule,FormsModule],
})
export class App {
  profileForm = new FormGroup({
    name: new FormControl(''),
  });
  favoriteFramework = '';
}
```
## リアクティブフォームの中にテンプレートフォームを置く

以下のコードではリアクティブフォームの`<form [formGroup]="profileForm"></form>`の中にテンプレートフォームの`[(ngModel)]`が入っている。
このとき、NG01350というエラーが出る。
<!-- リアクティブフォームの中ならリアクティブフォームに統一すべきだと思うが、 -->
<!-- そんなしなくてはならないときもある。 -->
<!-- 表示の切り替えだけ`[(ngModel)]`で、本体の入力フォームはリアクティブフォームで書きたいときとか。 -->

<!-- ただ混載するだけだとブラウザのコンソールにエラーが出る。 -->


基本的に standalone:trueとすると

```ts:Error(NG01350)
import 'zone.js';
import { Component } from '@angular/core';
import { bootstrapApplication } from '@angular/platform-browser';
import { FormGroup, FormControl } from '@angular/forms';
import { ReactiveFormsModule } from '@angular/forms';
import { FormsModule } from '@angular/forms';

@Component({
  selector: 'app-root',
  standalone: true,
  template: `
    <!-- reactive form -->
    <form [formGroup]="profileForm">
      <label>name:<input type="text" formControlName="name" /></label>
       <!-- template form -->
      <label>Favorite Framework:<input id="framework" type="text" [(ngModel)]="favoriteFramework" /> </label>
    </form>

    <h2>Profile Form</h2>
    <p>{{ profileForm.value.name }}'s favorite framework: {{ favoriteFramework }}</p>
  `,
  imports: [ReactiveFormsModule, FormsModule],
})
export class App {
  profileForm = new FormGroup({
    name: new FormControl(''),
  });
  favoriteFramework = '';
}
bootstrapApplication(App);
```

![NG01350](/images/angular-ngmodel-in-reactive-form/NG01350.png)


このエラーを読んでみると、2つ提案が書いてある。
画像に枠つけて
1. リアクティブフォームにしろ
<!-- `[(ngModel)]`をformControlに登録できないからformControlNameを使って、リアクティブフォームにしたらどうか？とのこと。 -->

1. standalone:trueを入れろ

`[ngModelOptions]="{standalone:true}"`

これを使うと外側のリアクティブフォーム部分とよい。


余談

```ts:微修正
@Component({
  template: `
        ...
        <input id="framework" type="text" formControlName="framework" [(ngModel)]="favoriteFramework" /> 
        ...
  `,
})
export class App {
  profileForm = new FormGroup({
    name: new FormControl(''),
    // add
    framework:new FormControl('')
  });
  ...
}

```