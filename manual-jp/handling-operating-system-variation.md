<!-- Start -->

## Handling Operating System Variation

> ## オペレーティングシステムのバリエーションの処理

<!-- End -->
<!-- Start -->
When dealing with platform libraries, it is often necessary to provide special cases for various platforms. 
> プラットフォームライブラリを扱う際には、さまざまなプラットフォームに特別なケースを提供する必要があることがよくあります。
<!-- End -->
<!-- Start -->
The variable `Sys.KERNEL` can be used to write these special cases. 
> 変数 `Sys.KERNEL`を使用して、これらの特殊なケースを書くことができます。
<!-- End -->
<!-- Start -->
There are several functions in the `Sys` module intended to make this easier: `isunix`, `islinux`, `isapple`, `isbsd`, and `iswindows`. 
> `isunix`、` islinux`、 `isapple`、` isbsd`、 `iswindows`のような` Sys`モジュールにはいくつかの機能があります。
<!-- End -->
<!-- Start -->
These may be used as follows:
> これらは次のように使用できます。
<!-- End -->

```julia
if Sys.iswindows()
    some_complicated_thing(a)
end
```

<!-- Start -->
Note that `islinux` and `isapple` are mutually exclusive subsets of `isunix`. 
`islinux`と` isapple`は `isunix`の相互排他的なサブセットです。
<!-- End -->
<!-- Start -->
Additionally, there is a macro `@static` which makes it possible to use these functions to conditionally hide invalid code, as demonstrated in the following examples.
>さらに、以下の例に示すように、これらの関数を使用して条件付きで無効なコードを隠すことを可能にするマクロ@staticがあります。
<!-- End -->

<!-- Start -->
Simple blocks:
> 単純なブロック：
<!-- End -->

```
ccall((@static Sys.iswindows() ? :_fopen : :fopen), ...)
```

Complex blocks:

```julia
@static if Sys.islinux()
    some_complicated_thing(a)
else
    some_different_thing(a)
end
```

<!-- Start -->
When chaining conditionals (including `if`/`elseif`/`end`), the `@static` must be repeated for each level (parentheses optional, but recommended for readability):
> 条件を連結する場合（if if / `else if` /` end`を含む）、レベルごとに `@static`を繰り返す必要があります（括弧は省略可能ですが、読みやすくするために推奨されます）。
<!-- End -->

```julia
@static Sys.iswindows() ? :a : (@static Sys.isapple() ? :b : :c)
```
