<!-- Start -->

# Date and DateTime

> # 時間と日付

<!-- End -->

```@meta
CurrentModule = Base.Dates
```

<!-- Start -->
The `Dates` module provides two types for working with dates: [`Date`](@ref) and [`DateTime`](@ref), representing day and millisecond precision, respectively; both are subtypes of the abstract [`TimeType`](@ref).
> `Dates`モジュールは日付を扱うための2つの型を提供します：それぞれ、日とミリ秒の精度を表す[` Date`](@ref) と[`DateTime`](@ref) どちらも抽象的な[`TimeType`](@ref) のサブタイプです。
<!-- End -->

<!-- Start -->
The motivation for distinct types is simple: some operations are much simpler, both in terms of code and mental reasoning, when the complexities of greater precision don't have to be dealt with.
> 異なるタイプの動機付けは簡単です。コードや精神的な推論の両方で、より正確な複雑さを処理する必要がない場合、操作がはるかに簡単です。
<!-- End -->

<!-- Start -->
For example, since the [`Date`](@ref) type only resolves to the precision of a single date (i.e. no hours, minutes, or seconds), normal considerations for time zones, daylight savings/summer time, and leap seconds are unnecessary and avoided.
> 例えば、[Date`](@ref)型は単一の日付の精度(つまり、時間、分、または秒)で解決されるため、タイムゾーン、夏時間、閏秒の通常の考慮事項不要で避けられます。
<!-- End -->

<!-- Start -->
Both [`Date`](@ref) and [`DateTime`](@ref) are basically immutable [`Int64`](@ref) wrappers.
> [`Date`](@ref) と[` DateTime`](@ref) は基本的には不変の[`Int64`](@ref) ラッパーです。
<!-- End -->

<!-- Start -->
The single `instant` field of either type is actually a `UTInstant{P}` type, which represents a continuously increasing machine timeline based on the UT second [^1]. 
> どちらのタイプの単一の「インスタント」フィールドは、実際にはUT second [^1]に基づいてマシンのタイムラインが連続的に増加することを表す `UTInstant {P}`タイプです。
<!-- End -->

<!-- Start -->
The [`DateTime`](@ref) type is not aware of time zones (*naive*, in Python parlance), analogous to a *LocalDateTime* in Java 8. 
> [`DateTime`](@ref)型は、Java 8の* LocalDateTime *に似ているタイムゾーン(Pythonの意味では* naive *)を認識しません。
<!-- End -->

<!-- Start -->
Additional time zone functionality can be added through the [TimeZones.jl package](https://github.com/JuliaTime/TimeZones.jl/), which compiles the [IANA time zone database](http://www.iana.org/time-zones). 
> 追加のタイムゾーン機能は[TimeZones.jlパッケージ](https://github.com/JuliaTime/TimeZones.jl/)から追加することができ、[IANAタイムゾーンデータベース](http：//www.iana。組織/時間帯)。
<!-- End -->

<!-- Start -->
Both [`Date`](@ref) and [`DateTime`](@ref) are based on the [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) standard, which follows the proleptic Gregorian calendar.
> [`Date`](@ref)と[` DateTime`](@ref)の両方は[ISO 8601](https://en.wikipedia.org/wiki/ISO_8601)標準に基づいており、それは麻痺性グレゴリオ暦カレンダー。
<!-- End -->

<!-- Start -->
One note is that the ISO 8601 standard is particular about BC/BCE dates. 
> 1つの注意点は、ISO 8601規格はBC / BCE日付に特有であることです。
<!-- End -->

<!-- Start -->
In general, the last day of the BC/BCE era, 1-12-31 BC/BCE, was followed by 1-1-1 AD/CE, thus no year zero exists.
> 一般的にBC / BCE時代の最終日であるBC / BCE 1-12-31は、1-1-1 AD / CEの後に続き、したがって0は存在しません。
<!-- End -->

<!-- Start -->
The ISO standard, however, states that 1 BC/BCE is year zero, so `0000-12-31` is the day before `0001-01-01`, and year `-0001` (yes, negative one for the year) is 2 BC/BCE, year `-0002` is 3 BC/BCE, etc.
> しかし、ISO規格ではBC / BCEが1年であると記載されているので、0000-12-31は0001-01-01の前の日であり、-0001は年である(yes、 )は2 BC / BCE、年-0002は3 BC / BCEなどです。
<!-- End -->


[^1]:
   <!-- The notion of the UT second is actually quite fundamental. 
<!-- End -->

    UT秒の概念は、実際には非常に基本的なものです。
   <!-- There are basically two different notions of time generally accepted, one based on the physical rotation of the earth (one full rotation = 1 day), the other based on the SI second (a fixed, constant value). 
<!-- End -->

    地球の物理的な回転(1回転= 1日)に基づくものと、SI秒に基づくもの(固定された一定の値)とに基づいて、時間の一般的に受け入れられる2つの異なる概念が基本的に存在する。
   <!-- > These are radically different! 
<!-- End -->

    これらは根本的に異なります！
   <!-- Think about it, a "UT second", as defined relative to the rotation of the earth, may have a different absolute length depending on the day! 
<!-- End -->

    それについて考えると、地球の回転に関連して定義された「UT秒」は、日によって異なる絶対的な長さを持つことがあります！
   <!-- Anyway, the fact that [`Date`](@ref) and [`DateTime`](@ref) are based on UT seconds is a simplifying, yet honest assumption so that things like leap seconds and all their complexity can be avoided. 
<!-- End -->

    とにかく、[`Date`](@ref)と[` DateTime`](@ref)がUT秒に基づいているという事実は、閏秒やその複雑さを避けることができるように単純化しても正直な仮定です。
   <!-- This basis of time is formally called [UT](https://en.wikipedia.org/wiki/Universal_Time) or UT1.
<!-- End -->

    この時間基準は正式に[UT](https://en.wikipedia.org/wiki/Universal_Time)またはUT1と呼ばれています。
   <!-- Basing types on the UT second basically means that every minute has 60 seconds and every day has 24 hours and leads to more natural calculations when working with calendar dates.
<!-- End -->

    UT秒のベースタイプは基本的に、1分ごとに60秒があり、毎日24時間あり、カレンダーの日付を扱うときに自然な計算が行われることを意味します。


<!-- Start -->

## Constructors

> ## コンストラクタ

<!-- End -->

[`Date`](@ref) and [`DateTime`](@ref) types can be constructed by integer or [`Period`](@ref) types, by parsing, or through adjusters (more on those later):

```jldoctest
julia> DateTime(2013)
2013-01-01T00:00:00

julia> DateTime(2013,7)
2013-07-01T00:00:00

julia> DateTime(2013,7,1)
2013-07-01T00:00:00

julia> DateTime(2013,7,1,12)
2013-07-01T12:00:00

julia> DateTime(2013,7,1,12,30)
2013-07-01T12:30:00

julia> DateTime(2013,7,1,12,30,59)
2013-07-01T12:30:59

julia> DateTime(2013,7,1,12,30,59,1)
2013-07-01T12:30:59.001

julia> Date(2013)
2013-01-01

julia> Date(2013,7)
2013-07-01

julia> Date(2013,7,1)
2013-07-01

julia> Date(Dates.Year(2013),Dates.Month(7),Dates.Day(1))
2013-07-01

julia> Date(Dates.Month(7),Dates.Year(2013))
2013-07-01
```

<!-- Start -->
[`Date`](@ref) or [`DateTime`](@ref) parsing is accomplished by the use of format strings. 
> [`Date`](@ref)または[` DateTime`](@ref)の解析は、フォーマット文字列の使用によって行われます。
<!-- End -->

<!-- Start -->
Format strings work by the notion of defining *delimited* or *fixed-width* "slots" that contain a period to parse and passing the text to parse and format string to a [`Date`](@ref) or [`DateTime`](@ref) constructor, of the form `Date("2015-01-01","y-m-d")` or `DateTime("20150101","yyyymmdd")`.
> 書式文字列は、解析して文字列を解析して文字列を[`Date`](@ref)または[` DateTime]に構文解析して渡すためのピリオドを含む* delimited *または* fixed-width * "slots" `Date(" 2015-01-01 "、" ymd ")`または `DateTime(" 20150101 "、" yyyymmdd ")`の形式の ``(@ref)コンストラクタを使用します。
<!-- End -->


<!-- Start -->
Delimited slots are marked by specifying the delimiter the parser should expect between two subsequent periods; so `"y-m-d"` lets the parser know that between the first and second slots in a date string like `"2014-07-16"`, it should find the `-` character. 
> 区切られたスロットは、パーサーが後続の2つのピリオドの間に期待する区切り文字を指定することによってマークされます。 `` y-m-d '`はパーサに、` `2014-07-16" `のような日付文字列の最初のスロットと2番目のスロットの間に` -`文字があることを認識させます。
<!-- End -->

<!-- Start -->
The `y`, `m`, and `d` characters let the parser know which periods to parse in each slot.
> `y`、` m`、 `d`文字はパーサーが各スロットで解析する期間を知ることを可能にします。
<!-- End -->

<!-- Start -->
Fixed-width slots are specified by repeating the period character the number of times corresponding to the width with no delimiter between characters. 
> 固定幅スロットは、区切り文字なしの幅に対応する回数だけピリオド文字を繰り返して指定します。
<!-- End -->

<!-- Start -->
So `"yyyymmdd"` would correspond to a date string like `"20140716"`. 
> `` yyyymmdd "`は `` 20140716 "`のような日付文字列に対応します。
<!-- End -->

<!-- Start -->
The parser distinguishes a fixed-width slot by the absence of a delimiter, noting the transition `"yyyymm"` from one period character to the next.
> パーサーは区切り文字がないことによって固定幅のスロットを識別し、ある期間文字から次の文字への移行「yyyymm」を注目します。
<!-- End -->

<!-- Start -->
Support for text-form month parsing is also supported through the `u` and `U` characters, for abbreviated and full-length month names, respectively. 
> テキスト形式の月の構文解析のサポートは、それぞれ短縮形および完全長の月名の場合は、「u」および「U」文字を使用してサポートされています。
<!-- End -->

<!-- Start -->
By default, only English month names are supported, so `u` corresponds to "Jan", "Feb", "Mar", etc.
> デフォルトでは、英語の月名のみがサポートされているので、「u」は「Jan」、「Feb」、「Mar」などに対応します。
<!-- End -->

<!-- Start -->
And `U` corresponds to "January", "February", "March", etc.
> そして`U`は`Junuary`、`February`、`March`などに対応します。
<!-- End -->

#Similar to other name=>value mapping functions [`dayname()`](@ref) and [`monthname()`](@ref), custom locales can be loaded by passing in the `locale=>Dict{String,Int}` mapping to the `MONTHTOVALUEABBR` and `MONTHTOVALUE` dicts for abbreviated and full-name month names, respectively.
カスタムロケールは、他のname=>値マッピング関数[`dayname()`](@ref) と[`monthname()`](@ref)と同様に、 `locale => Dict {String,Int}` `MONTHTOVALUEABBR`と `MONTHTOVALUE`へのマッピングでは、それぞれ短縮名とフルネーム月名が指定されています。

<!-- Start -->
One note on parsing performance: using the `Date(date_string,format_string)` function is fine if only called a few times. 
> 解析のパフォーマンスに関する1つの注意： `Date(date_string、format_string)`関数を使うのは、何度か呼び出されただけで問題ありません。
<!-- End -->

<!-- Start -->
If there are many similarly formatted date strings to parse however, it is much more efficient to first create a [`Dates.DateFormat`](@ref), and pass it instead of a raw format string.
> しかし、同様にフォーマットされた日付文字列が多く解析された場合は、最初に[`Dates.DateFormat`](@ref)を作成し、生の書式文字列の代わりに渡す方が効率的です。
<!-- End -->


```jldoctest
julia> df = DateFormat("y-m-d");

julia> dt = Date("2015-01-01",df)
2015-01-01

julia> dt2 = Date("2015-01-02",df)
2015-01-02
```

<!-- Start -->
You can also use the `dateformat""` string macro. 
> `dateformat""`文字列マクロを使うこともできます。
<!-- End -->

<!-- Start -->
This macro creates the `DateFormat` object once when the macro is expanded and uses the same `DateFormat` object even if a code snippet is run multiple times.
> このマクロは、マクロが展開されたときに `DateFormat`オブジェクトを作成し、コードスニペットが複数回実行されたとしても同じ` DateFormat`オブジェクトを使用します。
<!-- End -->


```jldoctest
julia> for i = 1:10^5
           Date("2015-01-01", dateformat"y-m-d")
       end
```

<!-- Start -->
A full suite of parsing and formatting tests and examples is available in [`tests/dates/io.jl`](https://github.com/JuliaLang/julia/blob/master/test/dates/io.jl).
> 構文解析と書式設定のテストとサンプルの完全なスイートは、[`tests/dates/io.jl`](https://github.com/JuliaLang/julia/blob/master/test/dates/io.jl)で利用できます。

<!-- Start -->

### Durations/Comparisons

> ### 継続時間/比較

<!-- End -->

<!-- Start -->
Finding the length of time between two [`Date`](@ref) or [`DateTime`](@ref) is straightforward given their underlying representation as `UTInstant{Day}` and `UTInstant{Millisecond}`, respectively.
> 2つの[Date`](@ref)または[`DateTime`](@ref)の間の時間の長さは、それらの基礎となる表現がそれぞれ` UTInstant {Day} `と` UTInstant {Millisecond} `であることを前提としています。
<!-- End -->

<!-- Start -->
The difference between [`Date`](@ref) is returned in the number of [`Day`](@ref), and [`DateTime`](@ref) in the number of [`Millisecond`](@ref). 
> [`Date`](@ref)の差は[` Day`](@ref)と[`DateTime`](@ref)の数で返され、[` Millisecond`](@ref )。
<!-- End -->

<!-- Start -->
Similarly, comparing [`TimeType`](@ref) is a simple matter of comparing the underlying machine instants (which in turn compares the internal [`Int64`](@ref) values).
> 同様に、[`TimeType`](@ref)を比較することは、基礎をなすマシンインスタントを比較する単純な問題です。(内部の[Int64`](@ref)値を比較します)。
<!-- End -->

```jldoctest
julia> dt = Date(2012,2,29)
2012-02-29

julia> dt2 = Date(2000,2,1)
2000-02-01

julia> dump(dt)
Date
  instant: Base.Dates.UTInstant{Base.Dates.Day}
    periods: Base.Dates.Day
      value: Int64 734562

julia> dump(dt2)
Date
  instant: Base.Dates.UTInstant{Base.Dates.Day}
    periods: Base.Dates.Day
      value: Int64 730151

julia> dt > dt2
true

julia> dt != dt2
true

julia> dt + dt2
ERROR: MethodError: no method matching +(::Date, ::Date)
[...]

julia> dt * dt2
ERROR: MethodError: no method matching *(::Date, ::Date)
[...]

julia> dt / dt2
ERROR: MethodError: no method matching /(::Date, ::Date)
[...]

julia> dt - dt2
4411 days

julia> dt2 - dt
-4411 days

julia> dt = DateTime(2012,2,29)
2012-02-29T00:00:00

julia> dt2 = DateTime(2000,2,1)
2000-02-01T00:00:00

julia> dt - dt2
381110400000 milliseconds
```

<!-- Start -->

## Accessor Functions

> ## アクセサ関数

<!-- End -->


<!-- Start -->
Because the [`Date`](@ref) and [`DateTime`](@ref) types are stored as single [`Int64`](@ref) values, date parts or fields can be retrieved through accessor functions. 
> [`Date`](@ref)型と[` DateTime`](@ref)型は単一の[`Int64`](@ref)値として格納されるので、アクセサの関数を使って日付部分やフィールドを取り出すことができます。
<!-- End -->

<!-- Start -->
The lowercase accessors return the field as an integer:
> 小文字のアクセサはフィールドを整数として返します。
<!-- End -->

```jldoctest tdate
julia> t = Date(2014, 1, 31)
2014-01-31

julia> Dates.year(t)
2014

julia> Dates.month(t)
1

julia> Dates.week(t)
5

julia> Dates.day(t)
31
```

<!-- Start -->
While propercase return the same value in the corresponding [`Period`](@ref) type:
> 先頭大文字は対応する[`Period`](@ref)型と同じ値を返します：
<!-- End -->

```jldoctest tdate
julia> Dates.Year(t)
2014 years

julia> Dates.Day(t)
31 days
```

<!-- Start -->
Compound methods are provided, as they provide a measure of efficiency if multiple fields are needed at the same time:
> 複合メソッドが提供されています、効率的に複数のフィールドが同時に必要な場合の効率的な手段です。
<!-- End -->

```jldoctest tdate
julia> Dates.yearmonth(t)
(2014, 1)

julia> Dates.monthday(t)
(1, 31)

julia> Dates.yearmonthday(t)
(2014, 1, 31)
```

<!-- Start -->
One may also access the underlying `UTInstant` or integer value:
> 基礎となる `UTInstant`または整数値にアクセスすることもできます：
<!-- End -->


```jldoctest tdate
julia> dump(t)
Date
  instant: Base.Dates.UTInstant{Base.Dates.Day}
    periods: Base.Dates.Day
      value: Int64 735264

julia> t.instant
Base.Dates.UTInstant{Base.Dates.Day}(735264 days)

julia> Dates.value(t)
735264
```

<!-- Start -->

## Query Functions

> ## クエリ関数

<!-- End -->

<!-- Start -->
Query functions provide calendrical information about a [`TimeType`](@ref). 
> クエリ関数は[`TimeType`](@ref)に関するカレンダー情報を提供します。
<!-- End -->

<!-- Start -->
They include information about the day of the week:
> それらには曜日に関する情報が含まれています：
<!-- End -->

```jldoctest tdate2
julia> t = Date(2014, 1, 31)
2014-01-31

julia> Dates.dayofweek(t)
5

julia> Dates.dayname(t)
"Friday"

julia> Dates.dayofweekofmonth(t) # 5th Friday of January
5
```

Month of the year:

```jldoctest tdate2
julia> Dates.monthname(t)
"January"

julia> Dates.daysinmonth(t)
31
```

<!-- Start -->
As well as information about the [`TimeType`](@ref)'s year and quarter:
> [`TimeType`](@ref)の年と四半期に関する情報だけでなく、
<!-- End -->


```jldoctest tdate2
julia> Dates.isleapyear(t)
false

julia> Dates.dayofyear(t)
31

julia> Dates.quarterofyear(t)
1

julia> Dates.dayofquarter(t)
31
```

<!-- Start -->
The [`dayname()`](@ref) and [`monthname()`](@ref) methods can also take an optional `locale` keyword that can be used to return the name of the day or month of the year for other languages/locales.
> [`dayname()`](@ref) と[`monthname()`](@ref) メソッドは、オプションの `locale`キーワードを取ることができます。これは、 他の言語/ロケールの場合
<!-- End -->

<!-- Start -->
There are also versions of these functions returning the abbreviated names, namely [`dayabbr()`](@ref) and [`monthabbr()`](@ref). 
> これらの関数のバージョンも、短縮名[@ dayabbr() `](@ref)と[` monthabbr() `](@ref)を返します。
<!-- End -->

<!-- Start -->
First the mapping is loaded into the `LOCALES` variable:
> 最初にマッピングは `LOCALES`変数にロードされます：
<!-- End -->


```jldoctest tdate2
julia> french_months = ["janvier", "février", "mars", "avril", "mai", "juin",
                        "juillet", "août", "septembre", "octobre", "novembre", "décembre"];

julia> french_monts_abbrev = ["janv","févr","mars","avril","mai","juin",
                              "juil","août","sept","oct","nov","déc"];

julia> french_days = ["lundi","mardi","mercredi","jeudi","vendredi","samedi","dimanche"];

julia> Dates.LOCALES["french"] = Dates.DateLocale(french_months, french_monts_abbrev, french_days, [""]);
```

<!-- Start -->
The above mentioned functions can then be used to perform the queries:
> 上記の関数を使用してクエリを実行することができます。
<!-- End -->


```jldoctest tdate2
julia> Dates.dayname(t;locale="french")
"vendredi"

julia> Dates.monthname(t;locale="french")
"janvier"

julia> Dates.monthabbr(t;locale="french")
"janv"
```

<!-- Start -->
Since the abbreviated versions of the days are not loaded, trying to use the function `dayabbr()` will error.
> 日付の短縮バージョンはロードされないので、関数 `dayabbr()`を使用しようとするとエラーになります。
<!-- End -->

```jldoctest tdate2
julia> Dates.dayabbr(t;locale="french")
ERROR: BoundsError: attempt to access 1-element Array{String,1} at index [5]
Stacktrace:
 [1] #dayabbr#6(::String, ::Function, ::Int64) at ./dates/query.jl:114
 [2] (::Base.Dates.#kw##dayabbr)(::Array{Any,1}, ::Base.Dates.#dayabbr, ::Int64) at ./<missing>:0 (repeats 2 times)
```

<!-- Start -->

## TimeType-Period Arithmetic

> ## TimeType-期間算術

<!-- End -->

<!-- Start -->
It's good practice when using any language/date framework to be familiar with how date-period arithmetic is handled as there are some [tricky issues](https://codeblog.jonskeet.uk/2010/12/01/the-joys-of-date-time-arithmetic/) to deal with (though much less so for day-precision types).
> いくつかの[厄介な問題]があるため、日時計算がどのように処理されるかを熟知するために、任意の言語/日付フレームワークを使用するとよい習慣です(https://codeblog.jonskeet.uk/2010/12/01/the-joys-対処する(日付精度の型の場合はそれほど多くありません)
<!-- End -->

<!-- Start -->
The `Dates` module approach tries to follow the simple principle of trying to change as little as possible when doing [`Period`](@ref) arithmetic. 
> `Dates`モジュールのアプローチは、` `Period``(@ref)算術演算を行うときにできるだけ変更を行わないという簡単な原則に従います。
<!-- End -->

<!-- Start -->
This approach is also often known as *calendrical* arithmetic or what you would probably guess if someone were to ask you the same calculation in a conversation. 
> このアプローチは、しばしば*カレンダー*算術とか、誰かがあなたに会話の中で同じ計算を依頼するかどうかを推測するものとしてよく知られています。
<!-- End -->

<!-- Start -->
Why all the fuss about this? Let's take a classic example: add 1 month to January 31st, 2014. 
> なぜこれについてのすべての大騒ぎするのでしょうか？従来の例を考えてみましょう：2014年1月31日に1か月を追加します。
<!-- End -->

<!-- Start -->
What's the answer? Javascript will say [March 3](http://www.markhneedham.com/blog/2009/01/07/javascript-add-a-month-to-a-date/) (assumes 31 days). 
> なにが答えでしょうか？ Javascriptは[3月3日](http://www.markhneedham.com/blog/2009/01/07/javascript-add-a-month-to-a-date/) と表示されます(31日間を想定)。
<!-- End -->

<!-- Start -->
PHP says [March 2](http://stackoverflow.com/questions/5760262/php-adding-months-to-a-date-while-not-exceeding-the-last-day-of-the-month) (assumes 30 days). 
> PHPは[3月2日](http://stackoverflow.com/questions/5760262/php-adding-months-to-a-date-while-not-exceeding-the-last-day-of-the-month) 30日と仮定)。
<!-- End -->

<!-- Start -->
The fact is, there is no right answer. 
> 実際、正解はありません。
<!-- End -->

<!-- Start -->
In the `Dates` module, it gives the result of February 28th. 
> `Dates`モジュールでは、2月28日の結果が得られます。
<!-- End -->

<!-- Start -->
How does it figure that out? I like to think of the classic 7-7-7 gambling game in casinos.
> どうやってを把握るんでしょうか？私はカジノで古典的な7-7-7の賭けるゲームを考えるのが好きです。
<!-- End -->

<!-- Start -->
Now just imagine that instead of 7-7-7, the slots are Year-Month-Day, or in our example, 2014-01-31.
> 今度は、7-7-7の代わりに、年 - 月 - 日、またはこの例の2014-01-31をスロットとします。
<!-- End -->

<!-- Start -->
When you ask to add 1 month to this date, the month slot is incremented, so now we have 2014-02-31.
> この日付に1か月を追加するようにリクエストすると、月のスロットが増えているので、今は2014-02-31です。
<!-- End -->

<!-- Start -->
Then the day number is checked if it is greater than the last valid day of the new month; if it is (as in the case above), the day number is adjusted down to the last valid day (28). 
> 次に、新しい月の最後の有効日よりも大きいかどうかがチェックされます。そうであれば(上記の場合と同様に)、曜日番号は最後の有効日(28)まで調整されます。
<!-- End -->

<!-- Start -->
What are the ramifications with this approach? Go ahead and add another month to our date, `2014-02-28 + Month(1) == 2014-03-28`.
> このアプローチにはどのような影響がありますか？ 2014-02-28 + Month(1)== 2014-03-28`にもう1ヶ月追加してください。
<!-- End -->

<!-- Start -->
What? Were you expecting the last day of March? Nope, sorry, remember the 7-7-7 slots. 
> 何？あなたは3月の最終日を期待していましたか？いいえ、申し訳ありませんが、7-7-7スロットを忘れないでください。
<!-- End -->

<!-- Start -->
As few slots as possible are going to change, so we first increment the month slot by 1, 2014-03-28, and boom, we're done because that's a valid date. 
> できるだけスロットを変更する予定がないので、最初に月のスロットを1、2014-03-28で増やして、ブームにします。これは有効な日付なので完了しました。
<!-- End -->

<!-- Start -->
On the other hand, if we were to add 2 months to our original date, 2014-01-31, then we end up with 2014-03-31, as expected. 
> 一方、元の日付2014-01-31に2か月を追加すると、予想どおり2014-03-31になります。
<!-- End -->

<!-- Start -->
The other ramification of this approach is a loss in associativity when a specific ordering is forced (i.e. adding things in different orders results in different outcomes). 
> このアプローチの別の分岐は、特定の順序付けが強制される(すなわち、異なる順序のものを追加すると結果が異なる)場合の結合性の喪失である。
<!-- End -->

<!-- Start -->
For example:
> 例えば：
<!-- End -->


```jldoctest
julia> (Date(2014,1,29)+Dates.Day(1)) + Dates.Month(1)
2014-02-28

julia> (Date(2014,1,29)+Dates.Month(1)) + Dates.Day(1)
2014-03-01
```

<!-- Start -->
What's going on there? In the first line, we're adding 1 day to January 29th, which results in 2014-01-30; then we add 1 month, so we get 2014-02-30, which then adjusts down to 2014-02-28.
> ここで何が起こっているのでしょうか？ 最初の行では、1月29日に1日追加します。その結果、2014-01-30になります。 1ヶ月追加すると2014-02-30となり、2014-02-28に調整されます。
<!-- End -->

<!-- Start -->
In the second example, we add 1 month *first*, where we get 2014-02-29, which adjusts down to 2014-02-28, and *then* add 1 day, which results in 2014-03-01. 
> 2番目の例では、*最初に*1ヶ月を加算するので2014-02-29は2014-02-28に、*それから*1日を加算して2014-03-01になります。
<!-- End -->

<!-- Start -->
One design principle that helps in this case is that, in the presence of multiple Periods, the operations will be ordered by the Periods' *types*, not their value or positional order; this means `Year` will always be added first, then `Month`, then `Week`, etc. 
> この場合に役立つ設計原則の1つは、複数の期間が存在する場合、その操作はその値または位置の順序ではなく、期間のタイプ*によって順序付けされることです。 `Year`が常に追加され、` Month`、 `Week`などが追加されます。
<!-- End -->

<!-- Start -->
Hence the following *does* result in associativity and Just Works:
> したがって、以下のことは連想性とJust Worksをもたらします：
<!-- End -->

```jldoctest
julia> Date(2014,1,29) + Dates.Day(1) + Dates.Month(1)
2014-03-01

julia> Date(2014,1,29) + Dates.Month(1) + Dates.Day(1)
2014-03-01
```

<!-- Start -->
Tricky? Perhaps. 
トリッキー？ おそらく。
<!-- End -->


<!-- Start -->
What is an innocent `Dates` user to do?
> 無垢な`Dates`ユーザーはどうすれば良いのでしょうか？ 
<!-- End -->

<!-- Start -->
The bottom line is to be aware that explicitly forcing a certain associativity, when dealing with months, may lead to some unexpected results, but otherwise, everything should work as expected. 
> 結論として、月を扱うときに明示的に特定の結合性を強制することは、予期しない結果を招くことがあることに注意してください。
<!-- End -->

<!-- Start -->
Thankfully, that's pretty much the extent of the odd cases in date-period arithmetic when dealing with time in UT (avoiding the "joys" of dealing with daylight savings, leap seconds, etc.).
>ありがたいことに、これは、UTでの時間処理(夏時間、閏秒などを扱うことの「喜び」を避けて)を扱うときの日間演算における奇妙なケースの程度です。
<!-- End -->

<!-- Start -->
As a bonus, all period arithmetic objects work directly with ranges:
> ボーナスとして、すべてのピリオド算術オブジェクトは範囲で直接動作します。
<!-- End -->

```jldoctest
julia> dr = Date(2014,1,29):Date(2014,2,3)
2014-01-29:1 day:2014-02-03

julia> collect(dr)
6-element Array{Date,1}:
 2014-01-29
 2014-01-30
 2014-01-31
 2014-02-01
 2014-02-02
 2014-02-03

julia> dr = Date(2014,1,29):Dates.Month(1):Date(2014,07,29)
2014-01-29:1 month:2014-07-29

julia> collect(dr)
7-element Array{Date,1}:
 2014-01-29
 2014-02-28
 2014-03-29
 2014-04-29
 2014-05-29
 2014-06-29
 2014-07-29
```

<!-- Start -->

## Adjuster Functions

## アジャスター機能

<!-- End -->

<!-- Start -->
As convenient as date-period arithmetics are, often the kinds of calculations needed on dates take on a *calendrical* or *temporal* nature rather than a fixed number of periods. 
日付期間の算術と同じくらい便利であるように、日付に必要な計算の種類は、固定された数ではなく、カレンダー*または*時間的な性質を取ることがよくあります。
<!-- End -->

<!-- Start -->
Holidays are a perfect example; most follow rules such as "Memorial Day = Last Monday of May", or "Thanksgiving = 4th Thursday of November". 
> 祝日は完璧な例です。 ほとんどが「メモリアルデー= 5月の最後の月曜日」、または「感謝祭= 11月の第4木曜日」などのルールに従います。
<!-- End -->

<!-- Start -->
These kinds of temporal expressions deal with rules relative to the calendar, like first or last of the month, next Tuesday, or the first and third Wednesdays, etc.
> これらの種類の時間表現は、カレンダーに関連するルール(月の最初または最後、次の火曜日、または1番目および3番目の水曜日など)を処理します。
<!-- End -->

<!-- Start -->
The `Dates` module provides the *adjuster* API through several convenient methods that aid in simply and succinctly expressing temporal rules. 
> `Dates`モジュールは、時間的ルールを簡潔かつ簡潔に表現するのに役立ついくつかの便利なメソッドによって* adjuster * APIを提供します。
<!-- End -->

<!-- Start -->
The first group of adjuster methods deal with the first and last of weeks, months, quarters, and years.
> アジャスターメソッドの最初のグループは、最初と最後の週、月、四半期、および年を処理します。
<!-- End -->

<!-- Start -->
They each take a single [`TimeType`](@ref) as input and return or *adjust to* the first or last of the desired period relative to the input.
> それらはそれぞれ入力として単一の[`TimeType`](@ref)を取り、*入力に対する最初の、または最後の*に調整します。
<!-- End -->

```jldoctest
julia> Dates.firstdayofweek(Date(2014,7,16)) #Adjusts the input to the Monday of the input's week

2014-07-14

julia> Dates.lastdayofmonth(Date(2014,7,16)) # Adjusts to the last day of the input's month

2014-07-31

julia> Dates.lastdayofquarter(Date(2014,7,16)) # Adjusts to the last day of the input's quarter

2014-09-30
```

<!-- Start -->
The next two higher-order methods, [`tonext()`](@ref), and [`toprev()`](@ref), generalize working with temporal expressions by taking a `DateFunction` as first argument, along with a starting [`TimeType`](@ref).
> 次の2つの高次メソッド[`tonext()`](@ref)と [`toprev()`](@ref) は、第1引数として`DateFunction`を取り、 開始[`TimeType`](@ref)です。
<!-- End -->
<!-- Start -->
A `DateFunction` is just a function, usually anonymous, that takes a single [`TimeType`](@ref) as input and returns a [`Bool`](@ref), `true` indicating a satisfied adjustment criterion.
> `DateFunction`は単一の[`TimeType`](@ref) を入力として受け取り、満たされた調整基準を示す[`Bool`](@ref)、 `true`を返します。
<!-- End -->
<!-- Start -->
For example:
例えば：
<!-- End -->

```jldoctest
julia> istuesday = x->Dates.dayofweek(x) == Dates.Tuesday <!-- > Returns true if the day of the week of x is Tuesday
<!-- End -->

(::<!-- 1) (generic function with 1 method)
<!-- End -->


julia> Dates.tonext(istuesday, Date(2014,7,13)) # 2014-07-13 is a Sunday
2014-07-15

julia> Dates.tonext(Date(2014,7,13), Dates.Tuesday) # Convenience method provided for day of the week adjustments
2014-07-15
```

<!-- Start -->
This is useful with the do-block syntax for more complex temporal expressions:
これは、より複雑な時間的表現のためのdo-block構文で便利です：
<!-- End -->

```jldoctest
julia> Dates.tonext(Date(2014,7,13)) do x
           #Return true on the 4th Thursday of November (Thanksgiving)
           Dates.dayofweek(x) == Dates.Thursday &&
           Dates.dayofweekofmonth(x) == 4 &&
           Dates.month(x) == Dates.November
       end
2014-11-27
```

<!-- Start -->
The [`Base.filter()`](@ref) method can be used to obtain all valid dates/moments in a specified range:
> [`Base.filter()`](@ref)メソッドは、指定された範囲内のすべての有効な日付/時刻を取得するために使用できます：
<!-- End -->


```jldoctest
# Pittsburgh street cleaning; Every 2nd Tuesday from April to November
# Date range from January 1st, 2014 to January 1st, 2015
julia> dr = Dates.Date(2014):Dates.Date(2015);

julia> filter(dr) do x
           Dates.dayofweek(x) == Dates.Tue &&
           Dates.April <= Dates.month(x) <= Dates.Nov &&
           Dates.dayofweekofmonth(x) == 2
       end
8-element Array{Date,1}:
 2014-04-08
 2014-05-13
 2014-06-10
 2014-07-08
 2014-08-12
 2014-09-09
 2014-10-14
 2014-11-11
```

<!-- Start -->
Additional examples and tests are available in [`test/dates/adjusters.jl`](https://github.com/JuliaLang/julia/blob/master/test/dates/adjusters.jl).
> 追加のサンプルとテストは、[`test/dates/adjusters.jl`](https://github.com/JuliaLang/julia/blob/master/test/dates/adjusters.jl)で利用できます。
<!-- End -->

<!-- Start -->

## Period Types

> ## 期間タイプ

<!-- End -->

<!-- Start -->
Periods are a human view of discrete, sometimes irregular durations of time. 
> 期間とは、不規則な、時には不規則な時間の人間の視点です。
<!-- End -->

<!-- Start -->
Consider 1 month; it could represent, in days, a value of 28, 29, 30, or 31 depending on the year and month context.
> 1ヶ月を考慮してください。 年間および月の状況に応じて、28日、29日、30日または31日の値を日数で表すことができます。
<!-- End -->

<!-- Start -->
Or a year could represent 365 or 366 days in the case of a leap year. 
> または、うるう年の場合、365日または366日を表すことがあります。
<!-- End -->

<!-- Start -->
[`Period`](@ref) types are simple [`Int64`](@ref) wrappers and are constructed by wrapping any `Int64` convertible type, i.e. `Year(1)` or `Month(3.0)`. 
> [`Period`](@ref) 型は単純な[Int64`](@ref)ラッパーで、`Int64`変換可能な型、つまり `Year(1)`や `Month(3.0)`をラップして構築されます。
<!-- End -->

<!-- Start -->
Arithmetic between [`Period`](@ref) of the same type behave like integers, and limited `Period-Real` arithmetic is available.
> 同じ型の[`Period`](@ref)間の算術演算は整数のように振る舞い、限られた` Period-Real`算術演算が利用できます。
<!-- End -->

```jldoctest
julia> y1 = Dates.Year(1)
1 year

julia> y2 = Dates.Year(2)
2 years

julia> y3 = Dates.Year(10)
10 years

julia> y1 + y2
3 years

julia> div(y3,y2)
5

julia> y3 - y2
8 years

julia> y3 % y2
0 years

julia> div(y3,3) # mirrors integer division
3 years
```

<!-- Start -->

## Rounding

> ## 丸め

<!-- End -->

<!-- Start -->
[`Date`](@ref) and [`DateTime`](@ref) values can be rounded to a specified resolution (e.g., 1 month or 15 minutes) with [`floor()`](@ref), [`ceil()`](@ref), or [`round()`](@ref):
> [`floor()`](@ref)、[@]、[@]で、[日付](@ref)と[`DateTime`](@ref)の値を指定した解像度(例えば1月または15分) `ceil()`](@ref)、または `` round() `](@ref)：
<!-- End -->

```jldoctest
julia> floor(Date(1985, 8, 16), Dates.Month)
1985-08-01

julia> ceil(DateTime(2013, 2, 13, 0, 31, 20), Dates.Minute(15))
2013-02-13T00:45:00

julia> round(DateTime(2016, 8, 6, 20, 15), Dates.Day)
2016-08-07T00:00:00
```

<!-- Start -->
Unlike the numeric [`round()`](@ref) method, which breaks ties toward the even number by default, the [`TimeType`](@ref)[`round()`](@ref) method uses the `RoundNearestTiesUp` rounding mode. 
> 数値[`round()`](@ref) メソッドとは異なり、[`TimeType`](@ref) [`round()`](@ref) メソッドはデフォルトで偶数につながる `RoundNearestTiesUp`丸めモードを使います。
<!-- End -->

<!-- Start -->
(It's difficult to guess what breaking ties to nearest "even" [`TimeType`](@ref) would entail.) 
> (最も近い "even" [`TimeType`](@ref) に付随する結びつきがどうなるかを推測するのは難しいです。)
<!-- End -->

<!-- Start -->
Further details on the available `RoundingMode` s can be found in the [API reference](@ref stdlib-dates).
> 利用可能な `RoundingMode`の詳細は、[APIリファレンス](@ref stdlib-dates)にあります。
<!-- End -->


<!-- Start -->
Rounding should generally behave as expected, but there are a few cases in which the expected behaviour is not obvious.
> 丸めは一般的には期待どおりに動作するはずですが、予想される動作が明白でないケースがいくつかあります。
<!-- End -->

<!-- Start -->

### Rounding Epoch

> ### 時(とき)の丸め処理

<!-- End -->

<!-- Start -->
In many cases, the resolution specified for rounding (e.g., `Dates.Second(30)`) divides evenly into the next largest period (in this case, `Dates.Minute(1)`).
> 多くの場合、丸めのために指定された解像度(例えば、Dates.Second(30))は、次に大きい期間(この場合、Dates.Minute(1))に均等に分割されます。
<!-- End -->

<!-- Start -->
But rounding behaviour in cases in which this is not true may lead to confusion. 
> しかし、これが真でない場合の丸め動作は、混乱を招く可能性があります。
<!-- End -->

<!-- Start -->
What is the expected result of rounding a [`DateTime`](@ref) to the nearest 10 hours?
> [`DateTime`](@ref) を10時間単位で四捨五入するとどのような結果になるのでしょうか？
<!-- End -->

```jldoctest
julia> round(DateTime(2016, 7, 17, 11, 55), Dates.Hour(10))
2016-07-17T12:00:00
```

<!-- Start -->
That may seem confusing, given that the hour (12) is not divisible by 10. 
> 時間(12)が10で割り切れないことを考えると、混乱しているように思えるかもしれません。
<!-- End -->

<!-- Start -->
The reason that `2016-07-17T12:00:00` was chosen is that it is 17,676,660 hours after `0000-01-01T00:00:00`, and 17,676,660 is divisible by 10.
> `2016-07-17T12：00：00` を選択した理由は、 `0000-01-01T00：00：00` から17676,660時間後であり、17676,660は10で割り切れるからです。
<!-- End -->

<!-- Start -->
As Julia [`Date`](@ref) and [`DateTime`](@ref) values are represented according to the ISO 8601 standard, `0000-01-01T00:00:00` was chosen as base (or "rounding epoch") from which to begin the count of days (and milliseconds) used in rounding calculations. 
> Julia [`Date`](@ref) と[`DateTime`](@ref) の値は ISO 8601 規格に従って表現されているので、 `0000-01-01T00：00：00` がベースとして選択された( "エポック" )を使用して、丸め計算で使用された日数(およびミリ秒)のカウントを開始します。
<!-- End -->

<!-- Start -->
(Note that this differs slightly from Julia's internal representation of [`Date`](@ref) s using Rata Die notation; but since the ISO 8601 standard is most visible to the end user, `0000-01-01T00:00:00` was chosen as the rounding epoch instead of the `0000-12-31T00:00:00` used internally to minimize confusion.)
> (Rata Die表記を使用したJuliaの [`Date`](@ref) の内部表現とは若干異なりますが、ISO 8601規格はエンドユーザにとって最も目に見えるので、`0000-01-01T00：00：00` は混乱を最小限に抑えるために内部的に使用される `0000-12-31T00：00：00` の代わりに丸めエポックとして選ばれました。)
<!-- End -->

<!-- Start -->
The only exception to the use of `0000-01-01T00:00:00` as the rounding epoch is when rounding to weeks. 
> 丸めのエポックとして '0000-01-01：00：00`を使用する唯一の例外は、数週間に丸めたときです。
<!-- End -->

<!-- Start -->
Rounding to the nearest week will always return a Monday (the first day of the week as specified by ISO 8601). 
> 最も近い週に四捨五入すると、常に月曜日(ISO 8601で指定された週の最初の曜日)が返されます。
<!-- End -->

<!-- Start -->
For this reason, we use `0000-01-03T00:00:00` (the first day of the first week of year 0000, as defined by ISO 8601) as the base when rounding to a number of weeks.
> この理由から、数週間に丸めたとき、ベースとして `0000-01-03T00：00：00`(ISO 8601で定義されているように、0000年の最初の週の最初の日)を使用します。
<!-- End -->

<!-- Start -->
Here is a related case in which the expected behaviour is not necessarily obvious: What happens when we round to the nearest `P(2)`, where `P` is a [`Period`](@ref) type? 
> これは、期待される振る舞いが必ずしも明白でないという関連するケースです：Pが[`Period`](@ref)型である最も近い` P(2) `に丸めるとどうなりますか？
<!-- End -->

<!-- Start -->
In some cases (specifically, when `P <: Dates.TimePeriod`) the answer is clear:
> いくつかの場合(具体的には、`P <：Dates.TimePeriod`)、その答えは明確です：
<!-- End -->

```jldoctest
julia> round(DateTime(2016, 7, 17, 8, 55, 30), Dates.Hour(2))
2016-07-17T08:00:00

julia> round(DateTime(2016, 7, 17, 8, 55, 30), Dates.Minute(2))
2016-07-17T08:56:00
```

<!-- Start -->
This seems obvious, because two of each of these periods still divides evenly into the next larger order period. 
> これらの期間のうちの2つの期間のうちの2つが依然として次のより大きな期間に均等に分割されるため、これは明らかなようです。
<!-- End -->

<!-- Start -->
But in the case of two months (which still divides evenly into one year), the answer may be surprising:
> しかし2カ月(1年に均等に分割されている)の場合、その答えは驚くかもしれません。
<!-- End -->

```jldoctest
julia> round(DateTime(2016, 7, 17, 8, 55, 30), Dates.Month(2))
2016-07-01T00:00:00
```

<!-- Start -->
Why round to the first day in July, even though it is month 7 (an odd number)? The key is that months are 1-indexed (the first month is assigned 1), unlike hours, minutes, seconds, and milliseconds (the first of which are assigned 0).
> 月が7(奇数)であっても、なぜ7月の最初の日に丸めますか？ キーは、時、分、秒、およびミリ秒(最初に0が割り当てられている)とは異なり、月は1つのインデックス付き(最初の月は1が割り当てられます)です。
<!-- End -->

<!-- Start -->
This means that rounding a [`DateTime`](@ref) to an even multiple of seconds, minutes, hours, or years (because the ISO 8601 specification includes a year zero) will result in a [`DateTime`](@ref) with an even value in that field, while rounding a [`DateTime`](@ref) to an even multiple of months will result in the months field having an odd value.
> つまり、ISO 8601仕様では年がゼロであるため、[`DateTime`](@ref) を秒、分、時、または偶数倍に丸めると、[`DateTime`](@ref) をそのフィールドに偶数の値で置き換え、[`DateTime`](@ref)を偶数倍の月数に丸めれば月フィールドは奇数値になります。
<!-- End -->
<!-- Start -->
Because both months and years may contain an irregular number of days, whether rounding to an even number of days will result in an even value in the days field is uncertain.
> 月と年の両方に不規則な日数が含まれている可能性があるため、偶数日に四捨五入すると曜日フィールドに偶数の値が返されるかどうかは不明です。
<!-- End -->

<!-- Start -->
See the [API reference](@ref stdlib-dates) for additional information on methods exported from the `Dates` module.
> `Dates`モジュールからエクスポートされたメソッドの詳細については、[APIリファレンス](@ref stdlib-dates)を参照してください。
<!-- End -->
