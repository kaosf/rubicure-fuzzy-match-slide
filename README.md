% Rubicureをあいまい検索対応強化してみた
% ka ([kaosfield](http://www.kaosfield.net))
% 2015-08-22

# 皆大好き便利なRubicure

```sh
gem install rubicure
```

# 私が思う一つの問題点

```ruby
Precure.splashstar
#=>
# NoMethodError: undefined method `splashstar' for #<Rubicure::Core:0x007f60ca7da078>
#         from /home/username/.rbenv/versions/2.2.3/lib/ruby/gems/2.2.0/gems/rubicure-0.2.8.1/lib/rubicure/core.rb:18:in `method_missing'
#         from /home/username/.rbenv/versions/2.2.3/lib/ruby/gems/2.2.0/gems/rubicure-0.2.8.1/lib/rubicure.rb:20:in `method_missing'
#         from (irb):2
#         from /home/username/.rbenv/versions/2.2.3/bin/irb:11:in `<main>'
```

正しくは

```ruby
Precure.splash_star
#=> {:series_name=>"splash_star", :title=>"ふたりはプリキュア Splash☆Star", :started_date=>Sun, 05 Feb 2006, :ended_date=>Sun, 28 Jan 2007, :girls=>["cure_bloom", "cure_egret"]}
```

# 規則が覚えられない

<ul>
<li>`yes_go_go`</li>
<li>`yes_gogo`</li>
</ul>

<div class="incremental">
どちらが正しいか分かりますか？

正解は `yes_gogo`
</div>

# なので適当に覚えていればいいようにした

```sh
gem install rubicure_fuzzy_match
```

これでRubicureが強化される

# `Rubicure::Seriese` に `fuzzy_find` が追加される

<div class="incremental">
```ruby
Rubicure::Seriese.fuzzy_find 'yesgogo'
#=> {:series_name=>"yes_gogo", :title=>"Yes！ プリキュア5 Go Go！", :started_date=>Sun, 03 Feb 2008, :ended_date=>Sun, 25 Jan 2009, :girls=>["cure_dream", "cure_rouge", "cure_lemonade", "cure_mint", "cure_aqua", "milky_rose"]}
```

```ruby
Rubicure::Seriese.fuzzy_find 'yes gogo'
#=> {:series_name=>"yes_gogo", :title=>"Yes！ プリキュア5 Go Go！", :started_date=>Sun, 03 Feb 2008, :ended_date=>Sun, 25 Jan 2009, :girls=>["cure_dream", "cure_rouge", "cure_lemonade", "cure_mint", "cure_aqua", "milky_rose"]}
```

```ruby
Rubicure::Seriese.fuzzy_find '5 gogo'
#=> {:series_name=>"yes_gogo", :title=>"Yes！ プリキュア5 Go Go！", :started_date=>Sun, 03 Feb 2008, :ended_date=>Sun, 25 Jan 2009, :girls=>["cure_dream", "cure_rouge", "cure_lemonade", "cure_mint", "cure_aqua", "milky_rose"]}
```

```ruby
Rubicure::Seriese.fuzzy_find '555'
#=> {:series_name=>"yes_gogo", :title=>"Yes！ プリキュア5 Go Go！", :started_date=>Sun, 03 Feb 2008, :ended_date=>Sun, 25 Jan 2009, :girls=>["cure_dream", "cure_rouge", "cure_lemonade", "cure_mint", "cure_aqua", "milky_rose"]}
```
</div>

# かなり適当な検索で作品を取ってこれる

```ruby
Rubicure::Seriese.fuzzy_find 'ss'
#=> {:series_name=>"splash_star", :title=>"ふたりはプリキュア Splash☆Star", :started_date=>Sun, 05 Feb 2006, :ended_date=>Sun, 28 Jan 2007, :girls=>["cure_bloom", "cure_egret"]}

Rubicure::Seriese.fuzzy_find 'ゴプリ'
#=> {:series_name=>"go_princess", :title=>"Go!プリンセスプリキュア", :started_date=>Sun, 01 Feb 2015, :girls=>["cure_flora", "cure_mermaid", "cure_twinkle", "cure_scarlett"]}

Rubicure::Seriese.fuzzy_find 'ハト'
#=> {:series_name=>"heart_catch", :title=>"ハートキャッチプリキュア！", :started_date=>Sun, 07 Feb 2010, :ended_date=>Sun, 30 Jan 2011, :girls=>["cure_blossom", "cure_marine", "cure_sunshine", "cure_moonlight"]}

Rubicure::Seriese.fuzzy_find '姫プリ'
#=> {:series_name=>"go_princess", :title=>"Go!プリンセスプリキュア", :started_date=>Sun, 01 Feb 2015, :girls=>["cure_flora", "cure_mermaid", "cure_twinkle", "cure_scarlett"]}
```

# 作品名の正規化

`Rubicure::Seriese.regularize` メソッドで正規化が出来る

```ruby
Rubicure::Seriese.regularize 'splashstar' #=> "ふたりはプリキュア Splash☆Star"
Rubicure::Seriese.regularize 'スマプリ' #=> "スマイルプリキュア！"
Rubicure::Seriese.regularize 'スマイプリ' #=> "スマイルプリキュア！"
Rubicure::Seriese.regularize 'ハピチャ' #=> "ハピネスチャージプリキュア！"
```

`fuzzy_find` の中で使ってる

# 実装

[seamusabshere/fuzzy_match](https://github.com/seamusabshere/fuzzy_match)

なんかこんなgemがあって適当にいい感じに合うものを探してくれる仕組みが簡単に作れた

```ruby
require 'fuzzy_match'

fm = FuzzyMatch.new ['プリキュア', 'ナージャ']
fm.find 'ナンジャ' #=> "ナージャ"
fm.find 'プリプリ' #=> "プリキュア"
fm.find 'どれみ'   #=> nil
```

# fuzzy_matchについて

今朝コンビニで時間潰してる時に偶然知った

アルゴリズムとか全然分かってない

他にもっとなんか簡単に実現出来るのがあるかも(これが一番怖い…)

# 問題点

日本語で「スプラッシュスター」とか「マックスハート」とかで検索しても大丈夫にしたい

```ruby
Rubicure::Seriese.regularize 'スプラッシュスター' #=> "フレッシュプリキュア！"
```

「プリンセスプリキュア」がかなり誤検出のもとになっていて何もしないと「フレプリ」「スマプリ」がこれになってしまう

自前でパターンを追加して今は回避してる

例外を追加しまくっていくのは綺麗とは思えないので何とかしたい

# おまけ

RSpecじゃなくてtest-unit (gemの方)を使っている

データ駆動テストをやってみてる

```ruby
class TestRubicureFuzzyMatch < Test::Unit::TestCase
  data(
    "ふたり"      => ["ふたり"     , "ふたりはプリキュア"            ],
    "初代"        => ["初代"       , "ふたりはプリキュア"            ],
    ...
    "姫プリ"      => ["姫プリ"     , "Go!プリンセスプリキュア"       ])
  test '.regularize' do |data|
    assert_equal data[1], Rubicure::Seriese.regularize(data[0])
  end
  ...
end
```

参考: [Ruby - Test::Unitでテストを書く - Qiita](http://qiita.com/repeatedly/items/727b08599d87af7fa671)

#

おわり
