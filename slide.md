#### Omotesando.rb #56
- - -

### パターンマッチと右代入が
### 便利になる gem をつくった

---

#### 自己紹介
- - -

* 名前：osyo
* Twitter : [@pink_bangbi](https://twitter.com/pink_bangbi)
* github  : [osyo-manga](https://github.com/osyo-manga)
* ブログ  : [Secret Garden(Instrumental)](http://secret-garden.hatenablog.com)
* 趣味で Ruby にパッチを投げたり bugs.ruby で気になったチケットをブログにまとめたりしてる
    * Ruby 2.5 〜 2.7 までオレオレ機能を追加した
    * Ruby 3.0 では機能追加できなかった…
* Ruby で一番好きな機能は Refinements
* オンラインなのであちこちの地域.rb に参加してる                      <!-- .element: class="fragment" -->
<br>
    * [Fukuoka.rb](https://fukuokarb.connpass.com/)
[Hamada.rb](https://hamadarb.connpass.com/)
[kawasaki.rb](https://kawasakirb.connpass.com/)
[nikotama.rb](https://nikotamarb.connpass.com/)
[Sendai.rb](https://sendairb.connpass.com/)
[Shibuya.rb](https://shibuyarb.connpass.com/)
[Shinjuku.rb](https://shinjukurb.connpass.com/)
[Tama.rb](https://tamarb.connpass.com/)
[Toyama.rb](https://toyamarb.connpass.com/)
[Gotanda.rb](https://gotanda-rb.connpass.com/)
[Entaku.rb](https://entakurb.doorkeeper.jp/)
[Grow.rb](https://growrb.doorkeeper.jp/)
[Hamamatsu.rb](https://hmrb.doorkeeper.jp/)
[kanazawa.rb](https://kzrb.doorkeeper.jp/)
[Kobe.rb](https://koberb.doorkeeper.jp/)
[Machida.rb](https://machidarb.doorkeeper.jp/)
[mitaka.rb](https://mitakarb.doorkeeper.jp/)
[Ruby Tuesday](https://ruby-tuesday.doorkeeper.jp/)
[toruby](https://toruby.doorkeeper.jp/)
[西日暮里.rb](https://nishinipporirb.doorkeeper.jp/)
<br>

---

### パターンマッチと右代入
### 便利になる gem をつくった

---

## 注意！
- - -

* この記事は開発版の Ruby 3.0 で検証しています
* Ruby 3.0 がリリースがされているときには挙動が変わっている可能性があるので注意してください

---

#### Ruby 2.7 / 3.0 でパターンマッチが入った
- - -

* Ruby 2.7 / 3.0 でパターンマッチが入りました
* これは以下のような感じで使うことができる

```ruby
def check(user)
  case user
  in { name:, age: (..20) }
    "#{name}は魔法少女です"
  in { name:, age: (20..) }
    "#{name}は魔法少女ではありません"
  else
    "わからない"
  end
end

p check({ name: "ほむ", age: 14 })
# => "ほむは魔法少女です"
p check({ name: "マミ", age: 30 })
# => "マミは魔法少女ではありません"
p check(["まどか", 14])
# => "わからない"
```

---

#### 任意のクラスでパターンマッチできる
- - -

* #deconstruct_keys を定義することで拡張できる

```ruby
class Time
  # in に書いた Hash のキーを受け取る
  # 返した Hash でパターンマッチを行う
  def deconstruct_keys(keys)
    { year: year, month: month, day: day }
  end

  def to_four_seasons
    case self
    in { month: (3..5) }  then "spring"
    in { month: (6..8) }  then "summer"
    in { month: (9..11) } then "autumn"
    in { month: (1..2) } | { month: 12 } then "winter"
    end
  end
end

p Time.now.to_four_seasons
# => "autumn"
```

---

#### 応用するとこんな事ができる
- - -

```ruby
module PatternMatchWithMethod
  # メソッドを呼び出すような拡張を行う
  def deconstruct_keys(keys)
    keys.to_h { |name| [name, public_send(name)] }
  end
end

class Time
  include PatternMatchWithMethod

  def to_four_seasons
    case self
    in { month: (3..5) }  then "spring"
    in { month: (6..8) }  then "summer"
    in { month: (9..11) } then "autumn"
    in { month: (1..2) } | { month: 12 } then "winter"
    end
  end
end

p Time.now.to_four_seasons
# => "autumn"
```

>>>

* いろいろと便利に使える

```ruby
class Array
  include PatternMatchWithMethod
end

# 配列のメソッドの結果を取得する
case ["homu", "mami", "mado"]
in { first:, last:, size: }
end
p first  # => "homu"
p last   # => "mado"
p size   # => 3

# インスタンスに extend して生やすこともできる
case { name: "homu", age: 14 }.extend PatternMatchWithMethod
in { keys:, values: }
end
p keys     # => [:name, :age]
p values   # => ["homu", 14]
```

---

## これを gem 化した

---

#### [pattern_matchable](https://github.com/osyo-manga/gem-pattern_matchable)
- - -

```ruby
require "pattern_matchable"

class Time
  # PatternMatchable を include するとすぐに使える
  include PatternMatchable

  def to_four_seasons
    case self
    in { month: (3..5) }  then "spring"
    in { month: (6..8) }  then "summer"
    in { month: (9..11) } then "autumn"
    in { month: (1..2) } | { month: 12 } then "winter"
    end
  end
end

p Time.now.to_four_seasons
# => "autumn"
```

### オープンクラスに直越 include したくないですよね？     <!-- .element: class="fragment" -->

---

### Refinements の時間だオラァ！！

---


#### using PatternMatchable するだけで拡張できる
- - -

```ruby
require "pattern_matchable"

# このファイル内の記事全てに反映される
using PatternMatchable

class Time
  def to_four_seasons
    case self
    in { month: (3..5) }  then "spring"
    in { month: (6..8) }  then "summer"
    in { month: (9..11) } then "autumn"
    in { month: (1..2) } | { month: 12 } then "winter"
    end
  end
end
p Time.now.to_four_seasons  # => "autumn"

case ["homu", "mami", "mado"]
in { first:, last:, size: }
end
p [first, last, size]
# => ["homu", "mado", 3]
```

---

#### 特定のクラスのみ拡張もできる
- - -

```ruby
require "pattern_matchable"

class Time
  # Time クラスのみ拡張する
  using PatternMatchable Time

  def to_four_seasons
    case self
    in { month: (3..5) }  then "spring"
    in { month: (6..8) }  then "summer"
    in { month: (9..11) } then "autumn"
    in { month: (1..2) } | { month: 12 } then "winter"
    end
  end
end

p Time.now.to_four_seasons  # => "autumn"

# Array は拡張されない
# error: ["homu", "mami", "mado"] (NoMatchingPatternError)
case ["homu", "mami", "mado"]
in { first:, last:, size: }
end
```

---

## Refinements は最高だな！！

---

## おまけ：右代入の話

---

#### 右代入
- - -

* Ruby 3.0 で右代入が入る予定
* これは左辺の値を右辺の変数に代入する構文

```ruby
result = 42
# を
42 => result
```

* また以下のような機能も入る予定   <!-- .element: class="fragment" -->

```ruby
user = { name: "homu", age: 14 }
case user
in { name:, age: }
end
# を
user => { name:, age: }
p [name, age]   # => ["homu", 14]
```
<!-- .element: class="fragment" -->

### これって分割代入じゃね？         <!-- .element: class="fragment" -->

>>>

* ちなみにパターンマッチなので以下のように書くこともできる

```ruby
homu = { name: "homu", age: 14 }

# 特定のクラスのみ代入できる
homu => { name: String => name, age: Integer => age }
p [name, age]   # => ["homu", 14]


mami = { name: :mami, age: 14 }
# これはエラー
# error: {:name=>:mami, :age=>14} (NoMatchingPatternError)
mami => { name: String => name, age: Integer => age }
p [name, age]   # => ["homu", 14]
```


---

## まとめ

---

#### まとめ
- - -

* Ruby 2.7 / 3.0 でパターンマッチという機能が実装された         <!-- .element: class="fragment" -->
    * パターンマッチ便利
* パターンマッチは #deconstruct_keys を定義することで任意のクラスをパターンマッチすることができる         <!-- .element: class="fragment" -->
* <p class="fragment">[pattern_matchable](https://github.com/osyo-manga/gem-pattern_matchable) を使うと簡単に既存のクラスを拡張できるよ         </p>
* Refinements は最高だな！！         <!-- .element: class="fragment" -->
* 右代入も最高だな！！         <!-- .element: class="fragment" -->

---

### 宣伝
- - -

* [Machida.rb #07 - Machida.rb | Doorkeeper](https://machidarb.doorkeeper.jp/events/113497)
    * 今週の金曜日だよ！
    * yukicoderの問題をみんなで解いてみるよ！
* [Ruby Advent Calendar 2020](https://qiita.com/advent-calendar/2020/ruby)
    * 今年もやるよ！！！
    * みんな参加してね！
* [Ruby 3.0 Advent Calendar 2020](https://qiita.com/advent-calendar/2020/ruby3)
    * Ruby 3.0 についての Advent Calendar だよ！！！
    * 寂しいのでみんな参加してね！！
* [令和時代の基礎文法最速マスター](https://qiita.com/advent-calendar/2020/reiwa_saisoku)
    * 参加して！！！




---

### ご清聴
### ありがとうございました

