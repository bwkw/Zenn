---
title: "「良いコード/悪いコードで学ぶ設計入門」の理解をPHP(Laravel)で深めよか"
emoji: "👻"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Laravel", "設計", "品質", "ミノ駆動本"]
published: false
---

# TL;DR

- 「良いコード/悪いコードで学ぶ設計入門」をテーマに、PHP(Laravel)を使用してプログラミング設計原則を詳解
- 各トピックごとに「問題のコード」例と「改良されたコード」例を PHP(Laravel)で提供し、良い設計原則に従う方法を具体的に提示
- これを読めば、設計原則の理解が深まり自身の開発に活かせるはず
- もちろん全トピックを扱うわけではないので、気になった人はぜひ購入を

https://gihyo.jp/book/2022/978-4-297-12783-1

# 第 1 章 悪しき構造の弊害を知覚する

## 1. 意味不明な命名

### 問題のコード

メソッドやクラスの命名においては、技術的な詳細や使用している技術そのものを表す名前（「技術駆動命名(Technology-driven naming)」という）よりも、ビジネスロジックを反映した名前を使用すべきという考え方があります。

以下に、技術駆動命名のコード例を示します。

```php
class UserController extends Controller {
    public function getUserFromDB($id) {
        $user = User::find($id);
        return view('user.show', ['user' => $user]);
    }
}
```

### 改良されたコード

この `getUserFromDB` メソッドは、その名前がデータベースからユーザーを取得するという実装詳細を含んでいます。しかし、メソッドの主な目的は、特定のユーザーの詳細を表示することです。したがって、このような場合には以下のように名前を変更すべきです。

```php
class UserController extends Controller {
    public function show($id) {
        $user = User::find($id);
        return view('user.show', ['user' => $user]);
    }
}
```

この改良されたコードでは、 `show` という名前がビジネスロジックをより直接的に反映しています。

## 2. 理解を困難にする条件分岐のネスト

### 問題のコード

条件分岐が深くネストされてしまうと、コードの可読性は大きく低下し、結果的に理解が困難になる傾向があります。さらに、条件の複雑さが増すと、それに伴ってバグの発生率も高まる可能性があります。

以下に、このネストした条件分岐の問題を具体的に示すコード例を示します。

```php
class UserController extends Controller {
    public function show($id) {
        $user = User::find($id);

        if ($user) {
            if ($user->isActive()) {
                return view('user.show', ['user' => $user]);
            } else {
                return redirect('home')->with('error', 'User is not active');
            }
        } else {
            return redirect('home')->with('error', 'User not found');
        }
    }
}
```

### 改良されたコード

問題のコードを見ると、二つの条件、ユーザーが存在するかどうか、そしてユーザーがアクティブかどうかという条件がネストしています。

このようなネストした条件分岐は、「ガード節」を使用して平坦化することができます。ガード節とは、関数の先頭でエラーケースをチェックして早期にリターンするというテクニックです。

以下に、ガード節を使用して改善したコードを示します。

```php
class UserController extends Controller {
    public function show($id) {
        $user = User::find($id);

        if (!$user) {
            return redirect('home')->with('error', 'User not found');
        }

        if (!$user->isActive()) {
            return redirect('home')->with('error', 'User is not active');
        }

        return view('user.show', ['user' => $user]);
    }
}
```

## 3. さまざまな悪魔を招きやすいデータクラス

### 問題のコード

データクラスとは、データの格納と取得のみを行うクラスを指します。これらのクラスがビジネスロジックを持たないために起こる問題は、そのデータを操作するためのロジックがクラス外部に散在してしまう可能性があるという点です。

例えば、以下のようなコードを考えてみましょう。

```php
class User {
    public $name;
    public $email;
}

class UserService {
    public function changeEmail(User $user, $newEmail) {
        // メールアドレスの形式検証
        if (!filter_var($newEmail, FILTER_VALIDATE_EMAIL)) {
            throw new Exception('Invalid email format');
        }

        $user->email = $newEmail;
    }
}
```

### 改良されたコード

このコードでは、`User` クラスは単にデータを保持するだけのデータクラスで、それを操作するためのロジックは`UserService` クラスに存在します。これにより、`User`クラスがそのデータの整合性を自身で保証することができなくなり、全ての操作が `UserService` を通じて行われることが必ずしも保証されていないため、データの不整合が発生するリスクがあります。

これらの問題を解決するためには、開発者がデータとそのデータを操作するロジックを一緒に持つようにクラスを設計する必要があります。

```php
class User {
    public $name;
    public $email;

    public function __construct($name, $email) {
        if (!filter_var($email, FILTER_VALIDATE_EMAIL)) {
            throw new Exception('Invalid email format');
        }
        $this->name = $name;
        $this->email = $email;
    }

    public function changeEmail($newEmail) {
        if (!filter_var($newEmail, FILTER_VALIDATE_EMAIL)) {
            throw new Exception('Invalid email format');
        }
        $this->email = $newEmail;
    }
}
```
