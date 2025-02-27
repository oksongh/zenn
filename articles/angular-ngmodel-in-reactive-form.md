---
title: "[Angular]リアクティブフォームとテンプレートフォームを混ぜて使う"
emoji: "🐷"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [Angular]
published: false
published_at: 2025-03-13 21:00
---
# 結論

普通はどちらかに寄せるべきだが、リアクティブフォームとテンプレートフォームを同時に使いたい場合がある。
テンプレートフォームをリアクティブフォームの外に置く場合は問題ない。中に置くときはテンプレートに`(ngModelOptions)={standalone:true}`を追加する。

# Angularのフォーム

Angularを使う以上避けて通れないフォームの使い方は前提知識としておくが、要約を書いておく。
基本的には[公式のチュートリアル](https://angular.jp/tutorials/learn-angular)をみておけばOKだと思う。

## フォームの基本
Angularには[リアクティブフォーム](https://angular.jp/tutorials/learn-angular/17-reactive-forms)と[テンプレートフォーム](https://angular.jp/tutorials/learn-angular/15-forms)の2種類の記法がある。

チュートリアルには以下の通り書いてある。

## リアクティブフォーム
チュートリアル[リアクティブフォーム](https://angular.jp/tutorials/learn-angular/17-reactive-forms)については以下のサンプルコードが書いてある。
ユーザーの入力はすべてFormGroup,FormArray,FormControlといったオブジェクトに詰め込まれ、それらを参照して表示を変える。
入れ子になったフォームに適している。

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

本題かつ結論。
リアクティブフォームとテンプレートフォームを同時に使いたい場合がある。
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
        <label>Favorite Framework:
            <input id="framework" type="text" [(ngModel)]="favoriteFramework" />
        </label>
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
```

![NG01350](/images/angular-ngmodel-in-reactive-form/NG01350.png)


`<input id="framework" type="text" [(ngModel)]="favoriteFramework"/>`がリアクティブフォームの要素の一つとして扱われてしまっているからダメ、ということらしい。
このエラーを読んでみると、2つ解決方法が書いてある。

1. リアクティブフォームに統一しろ
可能なら統一するべき。

1. standalone:trueを入れろ
何らかの理由で統一できないときの対処法。
`[ngModelOptions]="{standalone:true}"`を使うと外側のリアクティブフォーム部分から独立したフォームとして扱える。以下使用例。


```ts:OKByStandalone
@Component({
  selector: 'app-root',
  standalone: true,
  template: `
    <!-- reactive form -->
    <form [formGroup]="profileForm">
        <label>
            name:
            <input type="text" formControlName="name" />
        </label>

        <!-- template form -->
        <label>
            Favorite Framework:
            <input id="framework" type="text" [(ngModel)]="favoriteFramework"
                [ngModelOptions]="{standalone:true}"/>
        </label>
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
```