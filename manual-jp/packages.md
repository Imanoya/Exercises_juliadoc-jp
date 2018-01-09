# Packages
#パッケージ

#Julia has a built-in package manager for installing add-on functionality written in Julia. 
Juliaには、Juliaで書かれたアドオン機能をインストールするためのパッケージマネージャが組み込まれています。
#It can also install external libraries using your operating system's standard system for doing so, or by compiling from source. 
また、オペレーティングシステムの標準システムを使用して外部ライブラリをインストールすることも、ソースからコンパイルすることもできます。
#The list of registered Julia packages can be found at [http://pkg.julialang.org](http://pkg.julialang.org).
登録されたJuliaパッケージのリストは、[http://pkg.julialang.org]（http://pkg.julialang.org）にあります。
#All package manager commands are found in the `Pkg` module, included in Julia's `Base` install.
すべてのパッケージマネージャーコマンドはJuliaの `Base`インストールに含まれる` Pkg`モジュールにあります。

#First we'll go over the mechanics of the `Pkg` family of commands and then we'll provide some guidance on how to get your package registered. 
最初に `Pkg`コマンドファミリーの仕組みについて説明します。その後、あなたのパッケージを登録する方法についていくつかのガイダンスを提供します。
#Be sure to read the section below on package naming conventions, tagging versions and the importance of a `REQUIRE` file for when you're ready to add your code to the curated METADATA repository.
curated METADATAリポジトリにコードを追加する準備ができたら、パッケージ命名規則、バージョンのタグ付け、および `REQUIRE`ファイルの重要性に関するセクションを必ず読んでください。

## Package Status
##パッケージステータス

#The [`Pkg.status()`](@ref) function prints out a summary of the state of packages you have installed.
[`Pkg.status（）`]（@ ref）関数は、インストールしたパッケージの状態の要約を表示します。
#Initially, you'll have no packages installed:
最初は、パッケージがインストールされていません：

```julia-repl
julia> Pkg.status()
INFO: Initializing package repository /Users/stefan/.julia/v0.6
INFO: Cloning METADATA from git://github.com/JuliaLang/METADATA.jl
No packages installed.
```

#Your package directory is automatically initialized the first time you run a `Pkg` command that expects it to exist – which includes [`Pkg.status()`](@ref). 
あなたのパッケージディレクトリは、 `` Pkg`status（） ``（@ ref）を含む `` Pkg`コマンドを最初に実行するときに、自動的に初期化されます。
#Here's an example non-trivial set of required and additional packages:
ここでは、必須でない追加パッケージの例を示します。

```julia-repl
julia> Pkg.status()
Required packages:
 - Distributions                 0.2.8
 - SHA                           0.3.2
Additional packages:
 - NumericExtensions             0.2.17
 - Stats                         0.2.6
```

#These packages are all on registered versions, managed by `Pkg`. 
これらのパッケージはすべて、登録されたバージョンであり、 `Pkg`によって管理されています。
#Packages can be in more complicated states, indicated by annotations to the right of the installed package version; we will explain these states and annotations as we encounter them. 
パッケージは、インストールされたパッケージバージョンの右側の注釈で示されるより複雑な状態になることがあります。 これらの状態や注釈が発生したときの説明をします。
#For programmatic usage, [`Pkg.installed()`](@ref) returns a dictionary, mapping installed package names to the version of that package which is #installed:
プログラムでの使用の場合、[`Pkg.installed（）`]（@ ref）はインストールされたパッケージ名をそのパッケージのバージョン（#installed）にマッピングする辞書を返します：

```julia-repl
julia> Pkg.installed()
Dict{String,VersionNumber} with 4 entries:
"Distributions"     => v"0.2.8"
"Stats"             => v"0.2.6"
"SHA"               => v"0.3.2"
"NumericExtensions" => v"0.2.17"
```

## Adding and Removing Packages

#Julia's package manager is a little unusual in that it is declarative rather than imperative.
Juliaのパッケージマネージャーは、命令型ではなく宣言型であるという点で少し珍しいです。
#This means that you tell it what you want and it figures out what versions to install (or remove) to satisfy those requirements optimally – and minimally. 
これは、あなたが望むものを教えて、それらの要件を最適かつ最小限に満たすためにどのバージョンをインストール（または削除）するかを決定することを意味します。
#So rather than installing a package, you just add it to the list of requirements and then "resolve" what needs to be installed. 
したがって、パッケージをインストールするのではなく、要件のリストにパッケージを追加して、インストールする必要があるものを「解決」するだけです。
#In particular, this means that if some package had been installed because it was needed by a previous version of something you wanted, and a newer version doesn't have that requirement anymore, updating will actually remove that package.
特に、以前のバージョンのパッケージで必要とされていたパッケージがインストールされていて、新しいバージョンにそのような要件がない場合、実際にパッケージが削除されます。

#Your package requirements are in the file `~/.julia/v0.6/REQUIRE`. 
パッケージの要件は `〜/ .julia / v0.6 / REQUIRE`ファイルにあります。
#You can edit this file by hand and then call [`Pkg.resolve()`](@ref) to install, upgrade or remove packages to optimally satisfy the requirements, or you can do [`Pkg.edit()`](@ref), which will open `REQUIRE` in your editor (configured via the `EDITOR` or `VISUAL` environment variables), and then automatically call [`Pkg.resolve()`](@ref) afterwards if necessary. 
このファイルを手作業で編集して、[`Pkg.resolve（）`]（@ ref）を呼んでパッケージをインストール、アップグレード、または削除して、要件を最適に満たすようにするか、[`Pkg.edit（）`] @REFIX）はエディタで（ `EDITOR`または` VISUAL`環境変数で設定された） `REQUIRE`を開き、必要に応じて自動的に[` Pkg.resolve（） `]（@ref）を呼び出します。
#If you only want to add or remove the requirement for a single package, you can also use the non-interactive [`Pkg.add()`](@ref) and [`Pkg.rm()`](@ref) commands, which add or remove a single requirement to `REQUIRE` and then call [`Pkg.resolve()`](@ref).
単一のパッケージに対する要件だけを追加または削除したい場合は、非対話的な[`Pkg.add（）`]（@ ref）と[`Pkg.rm（）`]（@ ref）コマンドを使用して、REQUIREに単一の要件を追加または削除し、[`Pkg.resolve（）`]（@ ref）を呼び出します。

#You can add a package to the list of requirements with the [`Pkg.add()`](@ref) function, and the package and all the packages that it depends on will be installed:
[`Pkg.add（）`]（@ ref）関数を使って要件リストにパッケージを追加することができます。パッケージとそれが依存するすべてのパッケージがインストールされます：

```julia-repl
julia> Pkg.status()
No packages installed.

julia> Pkg.add("Distributions")
INFO: Cloning cache of Distributions from git://github.com/JuliaStats/Distributions.jl.git
INFO: Cloning cache of NumericExtensions from git://github.com/lindahua/NumericExtensions.jl.git
INFO: Cloning cache of Stats from git://github.com/JuliaStats/Stats.jl.git
INFO: Installing Distributions v0.2.7
INFO: Installing NumericExtensions v0.2.17
INFO: Installing Stats v0.2.6
INFO: REQUIRE updated.

julia> Pkg.status()
Required packages:
 - Distributions                 0.2.7
Additional packages:
 - NumericExtensions             0.2.17
 - Stats                         0.2.6
```

#What this is doing is first adding `Distributions` to your `~/.julia/v0.6/REQUIRE` file:
これがしているのは、最初に `〜/ .julia / v0.6 / REQUIRE`ファイルに` Distribution`を追加することです：

```
$ cat ~/.julia/v0.6/REQUIRE
Distributions
```

#It then runs [`Pkg.resolve()`](@ref) using these new requirements, which leads to the conclusion that the `Distributions` package should be installed since it is required but not installed. 
次に、これらの新しい要件を使用して[`Pkg.resolve（）`]（@ ref）を実行すると、 `Distribution`パッケージは必須であるがインストールされていないのでインストールする必要があります。
#As stated before, you can accomplish the same thing by editing your `~/.julia/v0.6/REQUIRE` file by hand and then running [`Pkg.resolve()`](@ref) yourself:
前述のように `〜/ .julia / v0.6 / REQUIRE`ファイルを手作業で編集し、[` Pkg.resolve（） `]（@ ref）を実行することで同じことができます：

```julia-repl
$ echo SHA >> ~/.julia/v0.6/REQUIRE

julia> Pkg.resolve()
INFO: Cloning cache of SHA from git://github.com/staticfloat/SHA.jl.git
INFO: Installing SHA v0.3.2

julia> Pkg.status()
Required packages:
 - Distributions                 0.2.7
 - SHA                           0.3.2
Additional packages:
 - NumericExtensions             0.2.17
 - Stats                         0.2.6
```

#This is functionally equivalent to calling [`Pkg.add("SHA")`](@ref), except that [`Pkg.add()`](@ref) doesn't change `REQUIRE` until *after* installation has completed, so if there are problems, `REQUIRE` will be left as it was before calling [`Pkg.add()`](@ref). 
これは、[`Pkg.add（" SHA "）`]（@ ref）を呼び出すのと機能的には同じですが、[`Pkg.add（）`]（@ ref）は* after *のインストールまで `REQUIRE`を変更しません 問題があれば、 `` Pkg.add（） `]（@ ref）を呼び出す前に` `REQUIRE``をそのまま残します。
#The format of the `REQUIRE` file is described in [Requirements Specification](@ref); it allows, among other things, requiring specific ranges of versions of packages.
`REQUIRE`ファイルのフォーマットは、[Requirements Specification]（@ ref）に記述されています。 パッケージのバージョンの特定の範囲を必要とします。

#When you decide that you don't want to have a package around any more, you can use [`Pkg.rm()`](@ref) to remove the requirement for it from the `REQUIRE` file:
パッケージをもう必要としたくない場合は、[`Pkg.rm（）`]（@ ref）を使って `REQUIRE`ファイルからパッケージの要件を削除してください：

```julia-repl
julia> Pkg.rm("Distributions")
INFO: Removing Distributions v0.2.7
INFO: Removing Stats v0.2.6
INFO: Removing NumericExtensions v0.2.17
INFO: REQUIRE updated.

julia> Pkg.status()
Required packages:
 - SHA                           0.3.2

julia> Pkg.rm("SHA")
INFO: Removing SHA v0.3.2
INFO: REQUIRE updated.

julia> Pkg.status()
No packages installed.
```

#Once again, this is equivalent to editing the `REQUIRE` file to remove the line with each package name on it then running [`Pkg.resolve()`](@ref) to update the set of installed packages to match.
再度、これは `REQUIRE`ファイルを編集して各パッケージ名の行を削除し、[` Pkg.resolve（） `]（@ ref）を実行して、インストールされたパッケージのセットを更新するのと同じです。
#While [`Pkg.add()`](@ref) and [`Pkg.rm()`](@ref) are convenient for adding and removing requirements for a single package, when you want to add or remove multiple packages, you can call [`Pkg.edit()`](@ref) to manually change the contents of `REQUIRE` and then update your packages accordingly. 
[`Pkg.add（）`]（@ ref）と[`Pkg.rm（）`]（@ ref）は、単一のパッケージに対する要件の追加と削除に便利ですが、複数のパッケージを追加または削除したい場合、 [`Pkg.edit（）`]（@ ref）を呼んで `REQUIRE`の内容を手動で変更し、それに応じてパッケージを更新することができます。
#[`Pkg.edit()`](@ref) does not roll back the contents of `REQUIRE` if [`Pkg.resolve()`](@ref) fails – rather, you have to run [`Pkg.edit()`](@ref) again to fix the files contents yourself.
[`Pkg.resolve（）`]（@ ref）が失敗した場合、[`Pkg.edit（）`]（@ ref）は `REQUIRE`の内容をロールバックしません。むしろ、` `Pkg.edit （） `]（@ ref）を使ってファイルの内容を修正してください。

#Because the package manager uses libgit2 internally to manage the package git repositories, users may run into protocol issues (if behind a firewall, for example), when running [`Pkg.add()`](@ref).
パッケージマネージャはlibgit2を内部的に使用してパッケージgitリポジトリを管理するため、[`Pkg.add（）`]（@ ref）を実行するときにプロトコルの問題（例えばファイアウォールの背後にある場合）が発生する可能性があります。
#By default, all GitHub-hosted packages wil be accessed via 'https'; this default can be modified by calling [`Pkg.setprotocol!()`](@ref). 
デフォルトでは、GitHubにホストされているすべてのパッケージは 'https'を介してアクセスされます。このデフォルトは[`Pkg.setprotocol！（）`]（@ ref）を呼び出すことによって変更できます。
#The following command can be run from the command line in order to tell git to use 'https' instead of the 'git' protocol when cloning all repositories, wherever they are hosted:
次のコマンドはコマンドラインから実行して、ホストされている場所でも、すべてのリポジトリを複製するときに 'git'プロトコルの代わりに 'https'を使用するように指示できます。

```
git config --global url."https://".insteadOf git://
```

#However, this change will be system-wide and thus the use of [`Pkg.setprotocol!()`](@ref) is preferable.
しかし、この変更はシステム全体で行われるため、[`Pkg.setprotocol！（）`]（@ ref）の使用が望ましいです。

#!!! note
!!! 注意
   # The package manager functions also accept the `.jl` suffix on package names, though the suffix is stripped internally. 
    パッケージマネージャー関数は、パッケージ名に `.jl`接尾辞も受け入れますが、接尾辞は内部的に取り除かれます。
   # For example:
    例えば：

    ```julia
    Pkg.add("Distributions.jl")
    Pkg.rm("Distributions.jl")
    ```

## Offline Installation of Packages
##パッケージのオフラインインストール

#For machines with no Internet connection, packages may be installed by copying the package root directory (given by [`Pkg.dir()`](@ref)) from a machine with the same operating system and environment.
インターネットに接続されていないマシンでは、パッケージのルートディレクトリ（[`Pkg.dir（）`]（@ ref））を同じオペレーティングシステムと環境のマシンからコピーしてインストールすることができます。

#[`Pkg.add()`](@ref) does the following within the package root directory:
[`Pkg.add（）`]（@ ref）は、パッケージのルートディレクトリ内で次のことを行います：

#1. Adds the name of the package to `REQUIRE`.
1. パッケージの名前を `REQUIRE`に追加します。
#2. Downloads the package to `.cache`, then copies the package to the package root directory.
2. パッケージを `.cache`にダウンロードし、パッケージをパッケージのルートディレクトリにコピーします。
#3. Recursively performs step 2 against all the packages listed in the package's `REQUIRE` file.
3. パッケージの `REQUIRE`ファイルにリストされているすべてのパッケージに対して、ステップ2を再帰的に実行します。
#4. Runs [`Pkg.build()`](@ref)
4. [`Pkg.build（）`]（@ ref）を実行します。

#!!! warning
!!!警告
   # Copying installed packages from a different machine is brittle for packages requiring binary external dependencies. 
   別のマシンからインストールされたパッケージをコピーすることは、バイナリの外部依存関係を必要とするパッケージにとって脆弱です。
   # Such packages may break due to differences in operating system versions, build environments, and/or absolute path dependencies.
   このようなパッケージは、オペレーティングシステムのバージョン、ビルド環境、および/または絶対パスの依存関係の違いにより、破損することがあります。

## Installing Unregistered Packages
##登録されていないパッケージのインストール

#Julia packages are simply git repositories, clonable via any of the [protocols](https://www.kernel.org/pub/software/scm/git/docs/git-clone.html#URLS) that git supports, and containing Julia code that follows certain layout conventions. 
Juliaパッケージは単にgitリポジトリで、gitがサポートする[protocols]（https://www.kernel.org/pub/software/scm/git/docs/git-clone.html#URLS）のいずれかを介してクローン可能です。特定のレイアウト規則に従うジュリアコード。
#Official Julia packages are registered in the [METADATA.jl](https://github.com/JuliaLang/METADATA.jl) repository, available at a well-known location [^1]. 
公式のJuliaパッケージは[METADATA.jl]（https://github.com/JuliaLang/METADATA.jl）リポジトリに登録されており、よく知られている場所[^ 1]にあります。
#The [`Pkg.add()`](@ref) and [`Pkg.rm()`](@ref) commands in the previous section interact with registered packages, but the package manager can install and work with unregistered packages too. 
前のセクションの[`Pkg.add（）`]（@ ref）と[`Pkg.rm（）`]（@ ref）コマンドは、登録されたパッケージとやりとりしますが、パッケージマネージャーは登録されていないパッケージ。
#To install an unregistered package, use [`Pkg.clone(url)`](@ref), where `url` is a git URL from which the package can be cloned:
未登録のパッケージをインストールするには、[`Pkg.clone（url）`]（@ ref）を使います。ここで `url`はパッケージをクローンできるgit URLです：

```julia-repl
julia> Pkg.clone("git://example.com/path/to/Package.jl.git")
INFO: Cloning Package from git://example.com/path/to/Package.jl.git
Cloning into 'Package'...
remote: Counting objects: 22, done.
remote: Compressing objects: 100% (10/10), done.
remote: Total 22 (delta 8), reused 22 (delta 8)
Receiving objects: 100% (22/22), 2.64 KiB, done.
Resolving deltas: 100% (8/8), done.
```

#By convention, Julia repository names end with `.jl` (the additional `.git` indicates a "bare" git repository), which keeps them from colliding with repositories for other languages, and also makes Julia packages easy to find in search engines.
習慣的に、Juliaリポジトリ名は `.jl`で終わります（追加の` .git`は "裸の" gitリポジトリを示します）。これは他の言語のリポジトリとの衝突を防ぎます。またJuliaパッケージを検索エンジンで簡単に見つけることができます。
#When packages are installed in your `.julia/v0.6` directory, however, the extension is redundant so we leave it off.
しかし、パッケージがあなたの `.julia / v0.6`ディレクトリにインストールされている場合、その拡張子は冗長ですので、そのまま残しておきます。

#If unregistered packages contain a `REQUIRE` file at the top of their source tree, that file will be used to determine which registered packages the unregistered package depends on, and they will automatically be installed. 
登録されていないパッケージにソースツリーの最上位に `REQUIRE`ファイルがある場合、そのファイルは登録されていないパッケージが依存する登録パッケージを決定するために使用され、自動的にインストールされます。
#Unregistered packages participate in the same version resolution logic as registered packages, so installed package versions will be adjusted as necessary to satisfy the requirements of both registered and unregistered packages.
登録されていないパッケージは、登録されたパッケージと同じバージョン解決ロジックに参加します。したがって、インストールされたパッケージのバージョンは、登録済みパッケージと未登録パッケージの両方の要件を満たすために必要に応じて調整されます。

[^1]:
   # The official set of packages is at [https://github.com/JuliaLang/METADATA.jl](https://github.com/JuliaLang/METADATA.jl), but individuals and organizations can easily use a different metadata repository. This allows control which packages are available for automatic installation. 
   パッケージの公式パッケージは[https://github.com/JuliaLang/METADATA.jl](https://github.com/JuliaLang/METADATA.jl）ですが、個人や組織は簡単に別のメタデータリポジトリを使用できます。これにより、どのパッケージを自動インストールできるかを制御することができます。
   # One can allow only audited and approved package versions, and make private packages or forks available. See [Custom METADATA Repository](@ref) for details.
   監査され承認されたパッケージ・バージョンのみを許可し、プライベート・パッケージまたはフォークを使用可能にすることができます。詳細については、[カスタムMETADATAリポジトリ]（@ ref）を参照してください。

## Updating Packages
##パッケージの更新

#When package developers publish new registered versions of packages that you're using, you will, of course, want the new shiny versions. 
パッケージ開発者があなたが使用しているパッケージの新しい登録バージョンを公開するときには、もちろん、新しい光沢のあるバージョンが必要です。
#To get the latest and greatest versions of all your packages, just do [`Pkg.update()`](@ref):
すべてのパッケージの最新バージョンと最高バージョンを入手するには、単に `` Pkg.update（） `]（@ ref）を実行してください：

```julia-repl
julia> Pkg.update()
INFO: Updating METADATA...
INFO: Computing changes...
INFO: Upgrading Distributions: v0.2.8 => v0.2.10
INFO: Upgrading Stats: v0.2.7 => v0.2.8
```

# The first step of updating packages is to pull new changes to `~/.julia/v0.6/METADATA` and see if any new registered package versions have been published. 
パッケージを更新する最初のステップは、 `〜/ .julia / v0.6 / METADATA`に新しい変更を加え、新しい登録されたパッケージのバージョンが公開されているかどうかを確認することです。
# After this, [`Pkg.update()`](@ref) attempts to update packages that are checked out on a branch and not dirty (i.e. no changes have been made to files tracked by git) by pulling changes from the package's upstream repository.
この後、[`Pkg.update（）`]（@ref）はブランチ上でチェックアウトされたパッケージを更新しようとしますが、ダーティではありません（gitによって追跡されるファイルは変更されません）リポジトリ。
# Upstream changes will only be applied if no merging or rebasing is necessary – i.e. if the branch can be ["fast-forwarded"](https://git-scm.com/book/en/v2/Git-Branching-Basic-Branching-and-Merging).
アップストリームの変更は、マージまたはリベースが必要ない場合、つまりブランチが["早送り"]できる場合にのみ適用されます（https://git-scm.com/book/en/v2/Git-Branching-Basic-ブランチングおよびマージ）。
# If the branch cannot be fast-forwarded, it is assumed that you're working on it and will update the repository yourself.
ブランチが早送りできない場合は、作業中であるとみなされ、自分でリポジトリを更新します。

# Finally, the update process recomputes an optimal set of package versions to have installed to satisfy your top-level requirements and the requirements of "fixed" packages. 
最後に、アップデートプロセスは、最上位レベルの要件と「固定」パッケージの要件を満たすためにインストールされた最適なパッケージバージョンのセットを再計算します。
# A package is considered fixed if it is one of the following:
パッケージは、次のいずれかの場合は修正済みと見なされます。

# 1. **Unregistered:** the package is not in `METADATA` – you installed it with [`Pkg.clone()`](@ref).
1. **登録されていません：**パッケージは `METADATA`にはありません - あなたは[` Pkg.clone（） `]（@ ref）でインストールしました。
# 2. **Checked out:** the package repo is on a development branch.
2. **チェックアウト：**パッケージレポは開発ブランチにあります。
# 3. **Dirty:** changes have been made to files in the repo.
3. ** Dirty：** レポ内のファイルが変更されました。

# If any of these are the case, the package manager cannot freely change the installed version of the package, so its requirements must be satisfied by whatever other package versions it picks.
これらのいずれかが該当する場合、パッケージマネージャはパッケージのインストールされているバージョンを自由に変更することができないため、パッケージの他のパッケージバージョンによってその要件を満たす必要があります。
# The combination of top-level requirements in `~/.julia/v0.6/REQUIRE` and the requirement of fixed packages are used to determine what should be installed.
`〜/ .julia / v0.6 / REQUIRE`のトップレベルの要件と固定パッケージの要件の組み合わせは、何をインストールすべきかを決定するために使用されます。

# You can also update only a subset of the installed packages, by providing arguments to the [`Pkg.update`](@ref) function. 
[`Pkg.update`]（@ ref）関数に引数を与えて、インストールされたパッケージのサブセットのみを更新することもできます。
# In that case, only the packages provided as arguments and their dependencies will be updated:
その場合、引数として提供されるパッケージとその依存関係のみが更新されます：

```julia-repl
julia> Pkg.update("Example")
INFO: Updating METADATA...
INFO: Computing changes...
INFO: Upgrading Example: v0.4.0 => 0.4.1
```

#This partial update process still computes the new set of package versions according to top-level requirements and "fixed" packages, but it additionally considers all other packages except those explicitly provided, and their dependencies, as fixed.
この部分更新プロセスでは、トップレベルの要件と "固定"パッケージに従って新しいパッケージバージョンが計算されますが、明示的に提供されているもの以外のすべてのパッケージとその依存関係は修正されています。

## Checkout, Pin and Free
##チェックアウト、ピン、フリー

#You may want to use the `master` version of a package rather than one of its registered versions. 
登録されたバージョンではなく、 `master`バージョンのパッケージを使用することができます。
#There might be fixes or functionality on master that you need that aren't yet published in any registered versions, or you may be a developer of the package and need to make changes on `master` or some other development branch. 
登録済みのバージョンではまだ公開されていない、またはマスターパッケージや他の開発ブランチを変更する必要があるかもしれません。
#In such cases, you can do [`Pkg.checkout(pkg)`](@ref) to checkout the `master` branch of `pkg` or [`Pkg.checkout(pkg,branch)`](@ref) to checkout some other branch:
そのような場合、 `` pkg.checkout（pkg） `]（@ ref）を実行して、` `pkg`または` `Pkg.checkout（pkg、branch）`（@ ref）の `master`ブランチをチェックアウトすることができます チェックアウトいくつかの他のブランチ：

```julia-repl
julia> Pkg.add("Distributions")
INFO: Installing Distributions v0.2.9
INFO: Installing NumericExtensions v0.2.17
INFO: Installing Stats v0.2.7
INFO: REQUIRE updated.

julia> Pkg.status()
Required packages:
 - Distributions                 0.2.9
Additional packages:
 - NumericExtensions             0.2.17
 - Stats                         0.2.7

julia> Pkg.checkout("Distributions")
INFO: Checking out Distributions master...
INFO: No packages to install, update or remove.

julia> Pkg.status()
Required packages:
 - Distributions                 0.2.9+             master
Additional packages:
 - NumericExtensions             0.2.17
 - Stats                         0.2.7
```

#Immediately after installing `Distributions` with [`Pkg.add()`](@ref) it is on the current most recent registered version – `0.2.9` at the time of writing this. 
[`Pkg.add（）`]（@ ref）で `Distributions 'をインストールした直後には、これを書いている時点では最新の登録済みバージョン - ` 0.2.9`にあります。
#Then after running [`Pkg.checkout("Distributions")`](@ref), you can see from the output of [`Pkg.status()`](@ref) that `Distributions` is on an unregistered version greater than `0.2.9`, indicated by the "pseudo-version" number `0.2.9+`.
[`Pkg.checkout（" Distribution "）`]（@ ref）を実行すると、[`Pkg.status（）`]（@ ref）の出力から、 `Distribution`が登録されていないバージョンより大きい「0.2.9 +」という数字で示される「0.2.9」より大きい。

#When you checkout an unregistered version of a package, the copy of the `REQUIRE` file in the package repo takes precedence over any requirements registered in `METADATA`, so it is important that developers keep this file accurate and up-to-date, reflecting the actual requirements of the current version of the package.
未登録のパッケージをチェックアウトすると、パッケージリポジトリの `REQUIRE`ファイルのコピーが` METADATA`に登録された要件よりも優先されるため、開発者はこのファイルを正確かつ最新の状態に保つことが重要です。現在のバージョンのパッケージの実際の要件を反映しています。
#If the `REQUIRE` file in the package repo is incorrect or missing, dependencies may be removed when the package is checked out. 
パッケージリポジトリの `REQUIRE`ファイルが不正確または欠けている場合、パッケージがチェックアウトされたときに依存関係が削除されることがあります。
#This file is also used to populate newly published versions of the package if you use the API that `Pkg` provides for this (described below).
このファイルは、 `Pkg`がこれを提供するAPI（後述）を使用する場合、新しく公開されたパッケージのバージョンを移植するためにも使用されます。

#When you decide that you no longer want to have a package checked out on a branch, you can "free" it back to the control of the package manager with [`Pkg.free(pkg)`](@ref):
ブランチでパッケージをチェックアウトする必要がなくなったら、[`Pkg.free（pkg）`]（@ ref）を使ってパッケージマネージャのコントロールに "解放"することができます：

```julia-repl
julia> Pkg.free("Distributions")
INFO: Freeing Distributions...
INFO: No packages to install, update or remove.

julia> Pkg.status()
Required packages:
 - Distributions                 0.2.9
Additional packages:
 - NumericExtensions             0.2.17
 - Stats                         0.2.7
```

#After this, since the package is on a registered version and not on a branch, its version will be updated as new registered versions of the package are published.
この後、パッケージはブランチ上ではなく登録バージョン上にあるので、パッケージの新しい登録バージョンが公開されるとそのバージョンが更新されます。

#If you want to pin a package at a specific version so that calling [`Pkg.update()`](@ref) won't change the version the package is on, you can use the [`Pkg.pin()`](@ref) function:
[`Pkg.update（）`]（@ ref）を呼び出すことによってパッケージのバージョンが変更されないように特定のバージョンでパッケージをピン止めしたい場合は、[`Pkg.pin（）` ]（@ ref）関数：

```julia-repl
julia> Pkg.pin("Stats")
INFO: Creating Stats branch pinned.47c198b1.tmp

julia> Pkg.status()
Required packages:
 - Distributions                 0.2.9
Additional packages:
 - NumericExtensions             0.2.17
 - Stats                         0.2.7              pinned.47c198b1.tmp
```

#After this, the `Stats` package will remain pinned at version `0.2.7` – or more specifically, at commit `47c198b1`, but since versions are permanently associated a given git hash, this is the same thing. 
この後、 `Stats`パッケージはバージョン` 0.2.7`で、より具体的にはコミット `47c198b1`で固定されたままですが、バージョンは永続的に所与のgitハッシュに関連付けられているので、これは同じことです。
#[`Pkg.pin()`](@ref) works by creating a throw-away branch for the commit you want to pin the package at and then checking that branch out. 
[`Pkg.pin（）`]（@ ref）は、パッケージをピン止めし、そのブランチをチェックするコミットのスローアウェイブランチを作成することで動作します。
#By default, it pins a package at the current commit, but you can choose a different version by passing a second argument:
デフォルトでは、現在のコミット時にパッケージを固定しますが、2番目の引数を渡して別のバージョンを選択することもできます：

```julia-repl
julia> Pkg.pin("Stats",v"0.2.5")
INFO: Creating Stats branch pinned.1fd0983b.tmp
INFO: No packages to install, update or remove.

julia> Pkg.status()
Required packages:
 - Distributions                 0.2.9
Additional packages:
 - NumericExtensions             0.2.17
 - Stats                         0.2.5              pinned.1fd0983b.tmp
```

#Now the `Stats` package is pinned at commit `1fd0983b`, which corresponds to version `0.2.5`. 
これで、 `Stats`パッケージはコミット` 1fd0983b`に固定され、バージョンは `0.2.5`に対応します。
#When you decide to "unpin" a package and let the package manager update it again, you can use [`Pkg.free()`](@ref) like you would to move off of any branch:
パッケージを "固定解除"してパッケージマネージャに再度更新させる場合、ブランチから移動する場合と同様に[`Pkg.free（）`]（@ ref）を使用することができます：

```julia-repl
julia> Pkg.free("Stats")
INFO: Freeing Stats...
INFO: No packages to install, update or remove.

julia> Pkg.status()
Required packages:
 - Distributions                 0.2.9
Additional packages:
 - NumericExtensions             0.2.17
 - Stats                         0.2.7
```

#After this, the `Stats` package is managed by the package manager again, and future calls to [`Pkg.update()`](@ref) will upgrade it to newer versions when they are published. 
その後、 `Stats`パッケージはパッケージマネージャーによって再度管理され、[` Pkg.update（） `]（@ ref）を呼び出すと、公開時に新しいバージョンにアップグレードされます。
#The throw-away `pinned.1fd0983b.tmp` branch remains in your local `Stats` repo, but since git branches are extremely lightweight, this doesn't really matter; if you feel like cleaning them up, you can go into the repo and delete those branches [^2].
スローアウェイ `pinned.1fd0983b.tmp`ブランチはローカルの` Stats`リポジトリに残っていますが、gitブランチは非常に軽いので、これは本当に問題ではありません。あなたがそれらをきれいにする気がするなら、あなたはレポに行き、それらの枝を削除することができます[^ 2]。

#[^2]:
[^ 2]：
   # Packages that aren't on branches will also be marked as dirty if you make changes in the repo, but that's a less common thing to do.
    リポジトリに変更を加えるとブランチにないパッケージもダーティとマークされますが、あまり一般的ではありません。

## Custom METADATA Repository
##カスタムMETADATAリポジトリ

#By default, Julia assumes you will be using the [official METADATA.jl](https://github.com/JuliaLang/METADATA.jl) repository for downloading and installing packages. 
デフォルトではJuliaはあなたがパッケージをダウンロードしてインストールするために[公式METADATA.jl]（https://github.com/JuliaLang/METADATA.jl）リポジトリを使用することを前提としています。
#You can also provide a different metadata repository location. A common approach is to keep your `metadata-v2` branch up to date with the Julia official branch and add another branch with your custom packages. 
別のメタデータリポジトリの場所を指定することもできます。一般的なアプローチは、Julia公式ブランチであなたの `metadata-v2`ブランチを最新の状態に保ち、あなたのカスタムパッケージを持つ別のブランチを追加することです。
#You can initialize your local metadata repository using that custom location and branch and then periodically rebase your custom branch with the official `metadata-v2` branch. 
そのカスタムロケーションとブランチを使用してローカルメタデータリポジトリを初期化し、公式の `metadata-v2`ブランチでカスタムブランチを定期的にリベースすることができます。
#In order to use a custom repository and branch, issue the following command:
カスタムリポジトリとブランチを使用するには、次のコマンドを発行します。

```julia-repl
julia> Pkg.init("https://me.example.com/METADATA.jl.git", "branch")
```

#The branch argument is optional and defaults to `metadata-v2`. 
ブランチ引数はオプションで、デフォルトは `metadata-v2`です。
#Once initialized, a file named `META_BRANCH` in your `~/.julia/vX.Y/` path will track the branch that your METADATA repository was initialized with. 
初期化されると、 `〜/ .julia / vX.Y /`パスにある `META_BRANCH`という名前のファイルは、METADATAリポジトリが初期化されたブランチを追跡します。
#If you want to change branches, you will need to either modify the `META_BRANCH` file directly (be careful!) or remove the `vX.Y` directory and re-initialize your METADATA repository using the `Pkg.init` command.
ブランチを変更したい場合は、 `META_BRANCH`ファイルを直接修正するか（注意してください）、` vX.Y`ディレクトリを削除し、 `Pkg.init`コマンドを使ってMETADATAリポジトリを再初期化する必要があります。

## Package Development
##パッケージ開発

#Julia's package manager is designed so that when you have a package installed, you are already in a position to look at its source code and full development history. 
Juliaのパッケージマネージャは、パッケージがインストールされているときに、そのソースコードと完全な開発履歴を見ることができるように設計されています。
#You are also able to make changes to packages, commit them using git, and easily contribute fixes and enhancements upstream.
また、パッケージを変更したり、gitを使ってコミットしたり、アップストリームの修正や拡張を簡単に行うこともできます。
#Similarly, the system is designed so that if you want to create a new package, the simplest way to do so is within the infrastructure provided by the package manager.
同様に、システムは、新しいパッケージを作成する場合、パッケージマネージャによって提供されるインフラストラクチャの中で、最も簡単な方法でパッケージを作成できるように設計されています。

### [Initial Setup](@id man-initial-setup)
## [初期設定]（@ id man-initial-setup）

#Since packages are git repositories, before doing any package development you should setup the following standard global git configuration settings:
パッケージはgitリポジトリなので、パッケージ開発を行う前に、以下の標準グローバルgit設定を設定する必要があります。

```
$ git config --global user.name "FULL NAME"
$ git config --global user.email "EMAIL"
```

#where `FULL NAME` is your actual full name (spaces are allowed between the double quotes) and `EMAIL` is your actual email address. 
`FULL NAME`はあなたの実際の完全な名前です（スペースは二重引用符で囲みます）。そして` EMAIL`はあなたの実際の電子メールアドレスです。
#Although it isn't necessary to use [GitHub](https://github.com/) to create or publish Julia packages, most Julia packages as of writing this are hosted on GitHub and the package manager knows how to format origin URLs correctly and otherwise work with the service smoothly. 
Juliaパッケージの作成や公開には[GitHub]（https://github.com/）を使う必要はありませんが、これを書いているJuliaパッケージはGitHubでホストされており、パッケージマネージャは元のURLを正しくフォーマットする方法を知っています さもなければサービスをスムーズに処理してください。
#We recommend that you create a [free account](https://github.com/join) on GitHub and then do:
GitHubで[無料アカウント]（https://github.com/join）を作成し、次の操作を行うことをおすすめします。

```
$ git config --global github.user "USERNAME"
```

#where `USERNAME` is your actual GitHub user name. Once you do this, the package manager knows your GitHub user name and can configure things accordingly. 
ここで `USERNAME`はあなたの実際のGitHubユーザー名です。これを行うと、パッケージマネージャーはあなたのGitHubユーザー名を知っており、それに応じて構成することができます。
#You should also [upload](https://github.com/login?return_to=https%3A%2F%2Fgithub.com%2Fsettings%2Fssh) your public SSH key to GitHub and set up an [SSH agent](https://linux.die.net/man/1/ssh-agent) on your development machine so that you can push changes with minimal hassle. 
公開SSH鍵をGitHubにアップロードして（[https://github.com/login?return_to=https%3A%2F%2Fgithub.com%2Fsettings%2Fssh）、[SSHエージェント]（https： //linux.die.net/man/1/ssh-agent）を使用して、最小限の手間で変更をプッシュできるようにします。
#In the future, we will make this system extensible and support other common git hosting options like [BitBucket](https://bitbucket.org) and allow developers to choose their favorite. 
将来的には、このシステムを拡張可能にし、[BitBucket]（https://bitbucket.org）のような共通のgitホスティングオプションをサポートし、開発者がお気に入りを選択できるようにします。
#Since the package development functions has been moved to the [PkgDev](https://github.com/JuliaLang/PkgDev.jl) package, you need to run `Pkg.add("PkgDev"); import PkgDev` to access the functions starting with `PkgDev.` in the document below.
パッケージ開発機能は[PkgDev]（https://github.com/JuliaLang/PkgDev.jl）パッケージに移動したので、 `Pkg.add（" PkgDev "）;を実行する必要があります。 import PkgDev`を呼び出して、以下の文書の `PkgDev.`で始まる関数にアクセスしてください。

## Making changes to an existing package
##既存のパッケージに変更を加える

### Documentation changes
###ドキュメントの変更

#If you want to improve the online documentation of a package, the easiest approach (at least for small changes) is to use GitHub's online editing functionality. 
パッケージのオンラインドキュメントを改善したいのであれば、（少なくとも小さな変更に対して）最も簡単なアプローチはGitHubのオンライン編集機能を使うことです。
#First, navigate to the repository's GitHub "home page," find the file (e.g., `README.md`) within the repository's folder structure, and click on it. 
まず、リポジトリのGitHubのホームページに移動し、リポジトリのフォルダ構造内のファイル（例： `README.md`）を見つけてクリックします。

#You'll see the contents displayed, along with a small "pencil" icon in the upper right hand corner.
表示される内容は、右上隅に小さな「鉛筆」アイコンとともに表示されます。
#Clicking that icon opens the file in edit mode. 
このアイコンをクリックすると、編集モードでファイルが開きます。
#Make your changes, write a brief summary describing the changes you want to make (this is your *commit message*), and then hit "Propose file change." Your changes will be submitted for consideration by the package owner(s) and collaborators.
変更を行い、変更内容を説明する簡単な要約を書きます（これはあなたの*コミットメッセージ*です）。 そして、 "提案ファイル変更"を押します。変更は、パッケージの所有者と共同作業者が検討するために提出されます。

#For larger documentation changes--and especially ones that you expect to have to update in response to feedback--you might find it easier to use the procedure for code changes described below.
より大きな文書の変更、特にフィードバックに応じて更新する必要がある変更については、後述のコード変更の手順を使用する方が簡単です。

### Code changes
###コードの変更

#### Executive summary
#### エグゼクティブサマリー

#Here we assume you've already set up git on your local machine and have a GitHub account (see above). Let's imagine you're fixing a bug in the Images package:
ここでは、ローカルマシンにgitをすでに設定していて、GitHubアカウントを持っていると仮定します（上記参照）。イメージパッケージにバグを修正しているとしましょう：

```
Pkg.checkout("Images")           # check out the master branch
<here, make sure your bug is still a bug and hasn't been fixed already>
cd(Pkg.dir("Images"))
;git checkout -b myfixes         # create a branch for your changes
<edit code>                      # be sure to add a test for your bug
Pkg.test("Images")               # make sure everything works now
;git commit -a -m "Fix foo by calling bar"   # write a descriptive message
using PkgDev
PkgDev.submit("Images")
```

#The last line will present you with a link to submit a pull request to incorporate your changes.
最後の行には、プルリクエストを送信して変更を組み込むためのリンクが表示されます。

#### Detailed description
#### 詳細な説明

#If you want to fix a bug or add new functionality, you want to be able to test your changes before you submit them for consideration. 
バグを修正したり、新しい機能を追加したりする場合は、変更内容を提出してから検討してください。
#You also need to have an easy way to update your proposal in response to the package owner's feedback. 
また、パッケージ所有者のフィードバックに応じて、簡単に提案を更新する必要があります。
#Consequently, in this case the strategy is to work locally on your own machine; once you are satisfied with your changes, you submit them for consideration.
したがって、この場合、戦略は自分のマシン上でローカルに作業することです。変更に満足したら、検討のために提出します。
#This process is called a *pull request* because you are asking to "pull" your changes into the project's main repository. 
このプロセスは、あなたの変更をプロジェクトのメインリポジトリに「プル」することを要求しているため、プルリクエスト*と呼ばれます。
#Because the online repository can't see the code on your private machine, you first *push* your changes to a publicly-visible location, your own online *fork* of the package (hosted on your own personal GitHub account). 
オンラインリポジトリはあなたのプライベートマシン上のコードを見ることができないので、あなたの変更は公開された場所、パッケージの自分のオンラインフォーク*（自分の個人的なGitHubアカウントでホストされています）にプッシュ*します。
#Let's assume you already have the `Foo` package installed. 
すでに `Foo`パッケージがインストールされているとしましょう。
#In the description below, anything starting with `Pkg.` or `PkgDev.` is meant to be typed at the Julia prompt; anything starting with `git` is meant to be typed in [julia's shell mode](@ref man-shell-mode) (or using the shell that comes with your operating system). 
以下の説明では、「Pkg.」または「PkgDev.」で始まるものは、Juliaプロンプトで入力することを意味しています。 `git`で始まるものは、[juliaのシェルモード]（@ ref man-shell-mode）（またはオペレーティングシステムに付属のシェルを使用して）で入力することを意味します。
#Within Julia, you can combine these two modes:
ジュリア内では、次の2つのモードを組み合わせることができます。

```julia-repl
julia> cd(Pkg.dir("Foo"))          # go to Foo's folder

shell> git command arguments...    # command will apply to Foo
```

#Now suppose you're ready to make some changes to `Foo`. 
今度は `Foo`を少し変更する準備が整ったとしましょう。
#While there are several possible approaches, here is one that is widely used:
いくつかのアプローチがありますが、ここでは広く使用されています：

  * 
  # From the Julia prompt, type [`Pkg.checkout("Foo")`](@ref). 
   Juliaプロンプトで[`Pkg.checkout（" Foo "）`]（@ ref）と入力します。
  # This ensures you're running the latest code (the `master` branch), rather than just whatever "official release" version you have installed.
   これにより、あなたがインストールした "公式リリース"のバージョンだけでなく、最新のコード（ `master`ブランチ）を確実に実行します。
  # (If you're planning to fix a bug, at this point it's a good idea to check again whether the bug has already been fixed by someone else. 
   （あなたがバグを修正しようとしている場合は、この時点でもう一度別の人がバグを修正したかどうか再度確認することをお勧めします。
  # If it has, you can request that a new official release be tagged so that the fix gets distributed to the rest of the community.) 
   それがある場合は、新しい公式リリースにタグを付けて、修正がコミュニティの他の人に配布されるように要求することができます。）
  # If you receive an error `Foo is dirty, bailing`, see [Dirty packages](@ref) below.
   `Foo is dirty、bailing`というエラーが表示された場合は、下記の[Dirty packages]（@ ref）を参照してください。
  * 
  # Create a branch for your changes: navigate to the package folder (the one that Julia reports from [`Pkg.dir("Foo")`](@ref)) and (in shell mode) create a new branch using `git checkout -b <newbranch>`, where `<newbranch>` might be some descriptive name (e.g., `fixbar`). 
   変更のためのブランチを作成します：パッケージフォルダ（Juliaが[`Pkg.dir（" Foo "）`]（@ ref）から報告する）と（シェルモードで） `git checkout -b <newbranch> `と入力します。ここで、` <newbranch> `は説明的な名前です（例：` fixbar`）。
  # By creating a branch, you ensure that you can easily go back and forth between your new work and the current `master` branch (see [https://git-scm.com/book/en/v2/Git-Branching-Branches-in-a-Nutshell](https://git-scm.com/book/en/v2/Git-Branching-Branches-in-a-Nutshell)). 
   ブランチを作成することで、新しい仕事と現在の `master`ブランチ間を簡単に行き来することができます（[https://git-scm.com/book/en/v2/Git-Branching-Branches -in-a-Nutshell]（https://git-scm.com/book/en/v2/Git-Branching-Branches-in-a-Nutshell））。
  # If you forget to do this step until after you've already made some changes, don't worry: see [more detail about branching](@ref man-branch-post-hoc) below.
   すでに変更を加えた後でなければ、このステップを忘れてしまいましたが、下の[分岐の詳細]（@ ref man-branch-post-hoc）を参照してください。
  * 
  # Make your changes. 
   変更を加えます。
  # Whether it's fixing a bug or adding new functionality, in most cases your change should include updates to both the `src/` and `test/` folders. 
   バグを修正しているか新しい機能を追加しているかにかかわらず、ほとんどの場合、変更に `src /`と `test /`フォルダの両方の更新が含まれているはずです。
  # If you're fixing a bug, add your minimal example demonstrating the bug (on the current code) to the test suite; by contributing a test for the bug, you ensure that the bug won't accidentally reappear at some later time due to other changes. 
   バグを修正する場合は、バグを示す最小限のサンプル（現在のコード上）をテストスイートに追加します。バグのテストに貢献することで、他の変更のためにバグが後で偶然に再び現れないようにすることができます。
  # If you're adding new functionality, creating tests demonstrates to the package owner that you've made sure your code works as intended.
   新しい機能を追加している場合、テストを作成することにより、コードオーナーが意図したとおりに動作することをパッケージ所有者に示しています。
  * 
  # Run the package's tests and make sure they pass. There are several ways to run the tests:
   パッケージのテストを実行し、テストが合格することを確認します。テストを実行するにはいくつかの方法があります。

      * 
  #    From Julia, run [`Pkg.test("Foo")`](@ref): this will run your tests in a separate (new) `julia` process.
      Juliaから、 `` Pkg.test（ "Foo"） `]（@ ref）を実行します。これは別の（新しい）` julia`プロセスでテストを実行します。
      * 
  #    From Julia, `include("runtests.jl")` from the package's `test/` folder (it's possible the file has a different name, look for one that runs all the tests): this allows you to run the tests repeatedly in the same session without reloading all the package code; for packages that take a while to load, this can be much faster. 
      Juliaから、パッケージの `test /`フォルダから `include（" runtests.jl "）`（ファイル名が違う可能性があります。すべてのテストを実行するファイルを探してください）、テストを繰り返し実行することができます。すべてのパッケージコードを再ロードせずに同じセッションを実行する。ロードに時間がかかるパッケージでは、はるかに高速になる可能性があります。
  #    With this approach, you do have to do some extra work to make [changes in the package code](@ref man-workflow-tips).
      このアプローチでは、[パッケージコードの変更]（@ ref man-workflow-tips）を行うためにいくつかの余分な作業をする必要があります。
      * 
  #    From the shell, run `julia ../test/runtests.jl` from within the package's `src/` folder.
      シェルから、パッケージの `src /`フォルダ内で `julia ../ test / runtests.jl`を実行してください。
  * 
  # Commit your changes: see [https://git-scm.com/book/en/v2/Git-Basics-Recording-Changes-to-the-Repository](https://git-scm.com/book/en/v2/Git-Basics-Recording-Changes-to-the-Repository).
   変更をコミットしてください：[https://git-scm.com/book/en/v2/Git-Basics-Recording-Changes-to-the-Repository](https://git-scm.com/book/en） / v2 / Git-Basics-レコーディング - リポジトリへの変更）。
  * 
  # Submit your changes: From the Julia prompt, type `PkgDev.submit("Foo")`. 
   変更を送信します：Juliaプロンプトから、 `PkgDev.submit（" Foo "）`と入力します。
  # This will push your changes to your GitHub fork, creating it if it doesn't already exist. 
   これにより、GitHubフォークに変更がプッシュされ、まだ存在しない場合は作成されます。
  # (If you encounter an error, [make sure you've set up your SSH keys](@ref man-initial-setup).) 
   （エラーが発生した場合は、[あなたのSSH鍵を設定したことを確認してください]（@ ref man-initial-setup））
  # Julia will then give you a hyperlink; open that link, edit the message, and then click "submit."
   Juliaはあなたにハイパーリンクを与えます。そのリンクを開き、メッセージを編集して、[送信]をクリックします。
  # At that point, the package owner will be notified of your changes and may initiate discussion.
   その時点で、パッケージ所有者に変更が通知され、ディスカッションが開始されます。
  # (If you are comfortable with git, you can also do these steps manually from the shell.)
  （gitがうまくいけば、シェルから手動でこれらのステップを実行することもできます）。
  * 
  # The package owner may suggest additional improvements. 
   パッケージ所有者は、さらに改善を提案するかもしれません。
  # To respond to those suggestions, you can easily update the pull request (this only works for changes that have not already been merged; for merged pull requests, make new changes by starting a new branch):
   これらの提案に応答するには、プルリクエストを簡単に更新することができます（これは、マージされていない変更に対してのみ機能し、マージされたプルリクエストに対しては、新しいブランチを開始して新しい変更を行います）。

      * 
  #   If you've changed branches in the meantime, make sure you go back to the same branch with `git checkout fixbar` (from shell mode) or [`Pkg.checkout("Foo", "fixbar")`](@ref) (from the Julia prompt). 
      その間にブランチを変更した場合は、 `git checkout fixbar`（シェルモードから）または` `Pkg.checkout（" Foo "、" fixbar "）`]（@ ref ）（Juliaプロンプトから）。
      * 
  #   As above, make your changes, run the tests, and commit your changes.
      上記のように、変更を行い、テストを実行し、変更をコミットします。
      * 
  #   From the shell, type `git push`.  
      シェルから「git push」と入力します。
  #   This will add your new commit(s) to the same pull request; you should see them appear automatically on the page holding the discussion of your pull request.
  これにより、新しいコミットが同じプルリクエストに追加されます。プルリクエストの議論を保持しているページに自動的に表示されるはずです。

  # One potential type of change the owner may request is that you squash your commits. 
  所有者が要求するかもしれない1つの潜在的なタイプの変更は、コミットを縮小することです。
  # See [Squashing](@ref man-squashing-and-rebasing) below.
  下記の[Squashing]（@ ref man-squashing-and-rebasing）を参照してください。

### Dirty packages
###ダーティパッケージ

#If you can't change branches because the package manager complains that your package is dirty, it means you have some changes that have not been committed. 
パッケージマネージャがパッケージが汚れていると告げるためにブランチを変更できない場合は、コミットされていない変更がいくつかあることを意味します。
#From the shell, use `git diff` to see what these changes are; you can either discard them (`git checkout changedfile.jl`) or commit them before switching branches.  
シェルから、 `git diff`を使ってこれらの変更を確認してください。ブランチを切り替える前にそれらを破棄（ `git checkout changedfile.jl`）するかコミットすることができます。
#If you can't easily resolve the problems manually, as a last resort you can delete the entire `"Foo"` folder and reinstall a fresh copy with [`Pkg.add("Foo")`](@ref).
手動で問題を簡単に解決できない場合は、最後の手段として `` Foo '`フォルダ全体を削除し、[` Pkg.add（ "Foo"） `]（@ ref）を使って新しいコピーを再インストールしてください。
#Naturally, this deletes any changes you've made.
当然これにより、変更が削除されます。

### [Making a branch *post hoc*](@id man-branch-post-hoc)
### [ブランチ作成* post hoc *]（@ id man-branch-post-hoc）

#Especially for newcomers to git, one often forgets to create a new branch until after some changes have already been made. 
特にgitの初心者にとっては、いくつかの変更が行われるまで、新しいブランチを作成することを忘れてしまうことがよくあります。
#If you haven't yet staged or committed your changes, you can create a new branch with `git checkout -b <newbranch>` just as usual--git will kindly show you that some files have been modified and create the new branch for you. 
まだ変更を行ったりコミットしたりしていない場合は、 `git checkout -b <newbranch>`で新しいブランチを作成することができます - gitはいくつかのファイルが変更されたことを親切に表示し、君は。
#*Your changes have not yet been committed to this new branch*, so the normal work rules still apply.
*あなたの変更はまだこの新しい支店*にコミットされていない*ので、通常の仕事のルールはまだ適用されます。

#However, if you've already made a commit to `master` but wish to go back to the official `master` (called `origin/master`), use the following procedure:
しかし、あなたがすでに `マスター 'にコミットしていて、公式`マスター'（元/マスターと呼ばれます）に戻る場合は、以下の手順を実行してください：
  * 
  # Create a new branch. This branch will hold your changes.
    新しいブランチを作成します。 このブランチはあなたの変更を保持します。
  * 
  # Make sure everything is committed to this branch.
     すべてがこのブランチにコミットされていることを確認してください。
  * 
  # `git checkout master`. If this fails, *do not* proceed further until you have resolved the problems, or you may lose your changes.
     `git checkout master`です。 これに失敗した場合は、問題を解決するか、変更内容を失うまで、*進むことはできません。
  * 
  # *Reset*`master` (your current branch) back to an earlier state with `git reset --hard origin/master` (see [https://git-scm.com/blog/2011/07/11/reset.html](https://git-scm.com/blog/2011/07/11/reset.html)).
   `master`（あなたの現在のブランチ）を` git reset -hard origin / master`で以前の状態に戻します（[https://git-scm.com/blog/2011/07/11/reset。 html]（https://git-scm.com/blog/2011/07/11/reset.html））。

#This requires a bit more familiarity with git, so it's much better to get in the habit of creating a branch at the outset.
これには、gitの知識がもう少し必要です。最初にブランチを作成するという習慣を習う方がはるかに優れています。

### [Squashing and rebasing](@id man-squashing-and-rebasing)
### [スカッシュとリベース]（@ id man-squashing-and-rebasing）

#Depending on the tastes of the package owner (s)he may ask you to "squash" your commits. 
パッケージ所有者の好みに応じて、彼はコミットを「スカッシュ」するよう求めるかもしれません。
#This is especially likely if your change is quite simple but your commit history looks like this:
これは、変更が非常にシンプルですが、コミット履歴が次のように見える場合は特にそうです：

```
WIP: add new 1-line whizbang function (currently breaks package)
Finish whizbang function
Fix typo in variable name
Oops, don't forget to supply default argument
Split into two 1-line functions
Rats, forgot to export the second function
...
```

#This gets into the territory of more advanced git usage, and you're encouraged to do some reading ([https://git-scm.com/book/en/v2/Git-Branching-Rebasing](https://git-scm.com/book/en/v2/Git-Branching-Rebasing)).
これはより高度なgitの使用法の領域に入り、いくつかの読書をすることをお勧めします（[https://git-scm.com/book/en/v2/Git-Branching-Rebasing](https://git -scm.com/book/en/v2/Git-Branching-Rebasing））。
#However, a brief summary of the procedure is as follows:
ただし、手順の簡単な概要は次のとおりです。

  * 
  # To protect yourself from error, start from your `fixbar` branch and create a new branch with `git checkout -b fixbar_backup`.  
    エラーから身を守るために、 `fixbar`ブランチから始め、` git checkout -b fixbar_backup`を使って新しいブランチを作成してください。
  # Since you started from `fixbar`, this will be a copy. Now go back to the one you intend to modify with `git checkout fixbar`.
     あなたが `fixbar`から始めたので、これはコピーになります。 今度は `git checkout fixbar`で修正したいものに戻ります。
  * 
  # From the shell, type `git rebase -i origin/master`.
     シェルから `git rebase -i origin / master`と入力します。
  * 
  # To combine commits, change `pick` to `squash` (for additional options, consult other sources). 
    コミットを組み合わせるには、 `pick`を` squash`に変更します（追加のオプションについては、他のソースを参照してください）。
  # Save the file and close the editor window.
     ファイルを保存し、エディタウィンドウを閉じます。
  * 
  # Edit the combined commit message.
     結合されたコミットメッセージを編集します。

#If the rebase goes badly, you can go back to the beginning to try again like this:
rebaseがひどくなった場合は、最初に戻って次のように再試行できます。

```
git checkout fixbar
git reset --hard fixbar_backup
```

#Now let's assume you've rebased successfully. 
さて、あなたが正常にリベースしたと仮定しましょう。
#Since your `fixbar` repository has now diverged from the one in your GitHub fork, you're going to have to do a *force push*:
あなたの `fixbar`リポジトリがあなたのGitHubフォークのリポジトリから分岐したので、* force push *をしなければなりません：

  * 
  #  To make it easy to refer to your GitHub fork, create a "handle" for it with `git remote add myfork https://github.com/myaccount/Foo.jl.git`, where the URL comes from the "clone URL" on your GitHub fork's page.
   あなたのGitHubフォークを参照しやすくするために、 `git remote add myfork https：// github.com / myaccount /Foo.jl.git`を使って"ハンドル "を作成してください。URLは" clone URL " "あなたのGitHubフォークのページに。
  * 
  #  Force-push to your fork with `git push myfork +fixbar`. The `+` indicates that this should replace the `fixbar` branch found at `myfork`.
   `git push myfork + fixbar`であなたのフォークに強制的に押し込みます。 `+`は `myfork`にある` fixbar`ブランチを置き換えるべきであることを示します。

## Creating a new Package
##新しいパッケージを作る

### REQUIRE speaks for itself
### REQUIREはそれ自体を話す

#You should have a `REQUIRE` file in your package repository, with a bare minimum directive of what Julia version you expect your users to be running for the package to work. 
あなたのパッケージレポジトリに、パッケージが動作するためにあなたのユーザが動作すると期待しているJuliaバージョンの最小限の指示を持つ `REQUIRE`ファイルが必要です。
#Putting a floor on what Julia version your package supports is done by simply adding `julia 0.x` in this file.
パッケージがサポートしているJuliaのバージョンにフロアを置くには、単にこのファイルに `julia 0.x`を追加するだけです。
#While this line is partly informational, it also has the consequence of whether `Pkg.update()` will update code found in `.julia` version directories. 
この行には部分的に情報がありますが、 `Pkg.update（）`が `.julia`バージョンディレクトリにあるコードを更新するかどうかの結果もあります。
#It will not update code found in version directories beneath the floor of what's specified in your `REQUIRE`.
`REQUIRE`で指定されたフロアの下のバージョンディレクトリにあるコードは更新されません。

#As the development version `0.y` matures, you may find yourself using it more frequently, and wanting your package to support it. 
開発版 `0.y`が成熟するにつれて、より頻繁にそれを使用し、あなたのパッケージがそれをサポートしたいと思うかもしれません。
#Be warned, the development branch of Julia is the land of breakage, and you can expect things to break.
ジュリアの開発枝は破壊の土地であり、物事が壊れることを期待することができます。
#When you go about fixing whatever broke your package in the development `0.y` branch, you will likely find that you just broke your package on the stable version.
開発 `0.y`ブランチであなたのパッケージを壊したものを修正すると、安定版でパッケージを破った可能性が高くなります。

#There is a mechanism found in the [Compat](https://github.com/JuliaLang/Compat.jl) package that will enable you to support both the stable version and breaking changes found in the development version. 
[Compat]（https://github.com/JuliaLang/Compat.jl）パッケージには、安定版と開発版の変更点の両方をサポートするためのメカニズムがあります。
#Should you decide to use this solution, you will need to add `Compat` to your `REQUIRE` file. 
この解決法を使用する場合は、 `REQUIRE`ファイルに` Compat`を追加する必要があります。
#In this case, you will still have `julia 0.x` in your `REQUIRE`. The `x` is the floor version of what your package supports.
この場合、 `REQUIRE`に` julia 0.x`があります。 `x`はあなたのパッケージがサポートしているものの床版です。

#You might also have no interest in supporting the development version of Julia. 
Juliaの開発版のサポートに興味がないかもしれません。
#Just as you can add a floor to the version you expect your users to be on, you can set an upper bound. 
ユーザーが期待しているバージョンにフロアを追加できるように、上限を設定できます。
#In this case, you would put `julia 0.x 0.y-` in your `REQUIRE` file. 
この場合、 `REQUIRE`ファイルに` julia 0.x 0.y-`を入れます。
#The `-` at the end of the version number means pre-release versions of that specific version from the very first commit. 
バージョン番号の末尾にある ` - `は、最初のコミットからその特定のバージョンのリリース前バージョンを意味します。
#By setting it as the ceiling, you mean the code supports everything up to but not including the ceiling version.
これを天井として設定すると、コードは天井バージョンを除くすべてのものをサポートします。

#Another scenario is that you are writing the bulk of the code for your package with Julia `0.y` and do not want to support the current stable version of Julia. If you choose to do this, simply add `julia 0.y-` to your `REQUIRE`. 
もう一つのシナリオは、パッケージのコードの大部分をJulia `0.y`で書いていて、Juliaの現在の安定バージョンをサポートしたくないということです。これを行うことを選択した場合は、単に `julia 0.y-`を `REQUIRE`に追加してください。
#Just remember to change the `julia 0.y-` to `julia 0.y` in your `REQUIRE` file once `0.y` is officially released. 
`0.y`が公式にリリースされた後、あなたの` REQUIRE`ファイルで `julia 0.y-`を `julia 0.y`に変更することを覚えておいてください。
#If you don't edit the dash cruft you are suggesting that you support both the development and stable versions of the same version number! 
ダッシュクルフトを編集しないと、同じバージョン番号の開発版と安定版の両方をサポートしていることを示唆しています！
#That would be madness. 
それは狂気だろう。
#See the [Requirements Specification](@ref) for the full format of `REQUIRE`.
`REQUIRE`のフルフォーマットについては、[Requirements Specification]（@ ref）を参照してください。

#Lastly, in many cases you may need extra packages for testing. 
最後に、多くの場合、テストのために追加のパッケージが必要な場合があります。
#Additional packages which are only required for tests should be specified in the `test/REQUIRE` file. 
テストにのみ必要な追加パッケージは `test / REQUIRE`ファイルで指定する必要があります。
#This `REQUIRE` file has the same specification as the standard `REQUIRE` file.
この `REQUIRE`ファイルは標準の` REQUIRE`ファイルと同じ仕様です。

### Guidelines for naming a package
###パッケージの命名のガイドライン

#Package names should be sensible to most Julia users, *even to those who are not domain experts*.
パッケージ名は、ドメイン専門家でない人でも*ほとんどのジュリアユーザーにとって賢明であるべきです*。
#When you submit your package to METADATA, you can expect a little back and forth about the package name with collaborators, especially if it's ambiguous or can be confused with something other than what it is. 
あなたのパッケージをMETADATAに提出すると、特にあいまいであるか、それ以外のものと混同される可能性がある場合、共同作業者のパッケージ名について少し前もってやりとりができます。
#During this bike-shedding, it's not uncommon to get a range of *different* name suggestions. 
このサイクリング中に、さまざまな名前の提案を得るのは珍しいことではありません。
#These are only suggestions though, with the intent being to keep a tidy namespace in the curated METADATA repository. 
しかし、これらの提案は、企画されたMETADATAリポジトリにきちんとした名前空間を維持することを目的としています。
#Since this repository belongs to the entire community, there will likely be a few collaborators who care about your package name. 
このリポジトリはコミュニティ全体に属しているため、パッケージ名を気にする共同作業者がいる可能性があります。
#Here are some guidelines to follow in naming your package:
あなたのパッケージに名前を付ける際のガイドラインは次のとおりです。

1. 
  # Avoid jargon. 
  専門用語を避けてください。
  #In particular, avoid acronyms unless there is minimal possibility of confusion.
  特に、混乱の可能性が最小限でない限り頭字語は避けてください。
   *   
   # It's ok to say `USA` if you're talking about the USA.
    あなたがアメリカについて話しているなら、「アメリカ」と言うのは大丈夫です。
   *   
   # It's not ok to say `PMA`, even if you're talking about positive mental attitude.
    肯定的な精神的な態度について話しているとしても、「PMA」と言うのは大丈夫じゃない。
2. 
  # Avoid using `Julia` in your package name.
  パッケージ名に `Julia`を使用しないでください。
    * 
   # It is usually clear from context and to your users that the package is a Julia package.
      一般的に、パッケージがJuliaパッケージであることは、コンテキストとユーザーには明らかです。
    * 
   # Having Julia in the name can imply that the package is connected to, or endorsed by, contributors to the Julia language itself.
    ジュリアを名前につけることは、パッケージがジュリア言語そのものへの貢献者と結びついているか、それによって支持されていることを暗示することができます。
3. 
  # Packages that provide most of their functionality in association with a new type should have pluralized names.
  新しいタイプに関連して機能の大部分を提供するパッケージは、複数の名前を持つ必要があります。

    * 
   # `DataFrames` provides the `DataFrame` type.
      `DataFrames`は` DataFrame`型を提供します。
    * 
   #  `BloomFilters` provides the `BloomFilter` type.
      `BloomFilters`は` BloomFilter`型を提供します。
    * 
   #  In contrast, `JuliaParser` provides no new type, but instead new functionality in the `JuliaParser.parse()` function.
      対照的に、 `JuliaParser`は新しい型を提供せず、` JuliaParser.parse（） `関数に新しい機能を提供します。
4. 
  # Err on the side of clarity, even if clarity seems long-winded to you.
  たとえ明快さがあなたに長らく残っていても、明瞭さの面では誤りです。

    *
   # `RandomMatrices` is a less ambiguous name than `RndMat` or `RMT`, even though the latter are shorter.
      `RandomMatrices`は、` RndMat`や `RMT`よりもあまりあいまいではありません。
5. 
  # A less systematic name may suit a package that implements one of several possible approaches to its domain.
  あまり系統的でない名前は、そのドメインへのいくつかの可能なアプローチのうちの1つを実装するパッケージに適しています。

    * 
   #  Julia does not have a single comprehensive plotting package. Instead, `Gadfly`, `PyPlot`, `Winston` and other packages each implement a unique approach based on a particular design philosophy.
    Juliaには、包括的なプロットパッケージが1つありません。代わりに、 `Gadfly`、` PyPlot`、 `Winston`などのパッケージはそれぞれ、特定の設計哲学に基づいた独自のアプローチを実装しています。
    *
   #  In contrast, `SortingAlgorithms` provides a consistent interface to use many well-established sorting algorithms.
   対照的に、 `SortingAlgorithms`は、多くの確立されたソートアルゴリズムを使用する一貫したインターフェースを提供します。
6. 
  # Packages that wrap external libraries or programs should be named after those libraries or programs.
    外部ライブラリまたはプログラムをラップするパッケージには、それらのライブラリまたはプログラムの名前を付ける必要があります。
    *
  #   `CPLEX.jl` wraps the `CPLEX` library, which can be identified easily in a web search.
    *
    `CPLEX.jl`はウェブ検索で簡単に識別できる` CPLEX`ライブラリをラップします。
  #   `MATLAB.jl` provides an interface to call the MATLAB engine from within Julia.
    `MATLAB.jl`はJulia内からMATLABエンジンを呼び出すためのインターフェイスを提供します。

### Generating the package
###パッケージの生成

#Suppose you want to create a new Julia package called `FooBar`. 
`FooBar`という新しいJuliaパッケージを作成したいとします。
#To get started, do `PkgDev.generate(pkg,license)` where `pkg` is the new package name and `license` is the name of a license that the package generator knows about:
始めに `PkgDev.generate（pkg、license）`を実行してください。 `pkg`は新しいパッケージ名で、` license`はパッケージジェネレータが知っているライセンスの名前です：

```julia-repl
julia> PkgDev.generate("FooBar","MIT")
INFO: Initializing FooBar repo: /Users/stefan/.julia/v0.6/FooBar
INFO: Origin: git://github.com/StefanKarpinski/FooBar.jl.git
INFO: Generating LICENSE.md
INFO: Generating README.md
INFO: Generating src/FooBar.jl
INFO: Generating test/runtests.jl
INFO: Generating REQUIRE
INFO: Generating .travis.yml
INFO: Generating appveyor.yml
INFO: Generating .gitignore
INFO: Committing FooBar generated files
```

#This creates the directory `~/.julia/v0.6/FooBar`, initializes it as a git repository, generates a bunch of files that all packages should have, and commits them to the repository:
`〜/ .julia / v0.6 / FooBar`ディレクトリが作成され、gitリポジトリとして初期化され、すべてのパッケージに必要な一連のファイルが生成され、リポジトリにコミットされます。

```
$ cd ~/.julia/v0.6/FooBar && git show --stat

commit 84b8e266dae6de30ab9703150b3bf771ec7b6285
Author: Stefan Karpinski <stefan@karpinski.org>
Date:   Wed Oct 16 17:57:58 2013 -0400

    FooBar.jl generated files.

        license: MIT
        authors: Stefan Karpinski
        years:   2013
        user:    StefanKarpinski

    Julia Version 0.3.0-prerelease+3217 [5fcfb13*]

 .gitignore       |  2 ++
 .travis.yml      | 13 +++++++++++++
 LICENSE.md       | 22 +++++++++++++++++++++++
 README.md        |  3 +++
 REQUIRE          |  1 +
 appveyor.yml     | 34 ++++++++++++++++++++++++++++++++++
 src/FooBar.jl    |  5 +++++
 test/runtests.jl |  5 +++++
 8 files changed, 85 insertions(+)
```


#At the moment, the package manager knows about the MIT "Expat" License, indicated by `"MIT"`, the Simplified BSD License, indicated by `"BSD"`, and version 2.0 of the Apache Software License, indicated by `"ASL"`. 
現時点では、パッケージマネージャーは、MITの "Expat"ライセンス、 "MIT"で示されている簡体字BSDライセンス、BSDで表示されているApacheソフトウェアライセンスのバージョン2.0、 ASL "。
#If you want to use a different license, you can ask us to add it to the package generator, or just pick one of these three and then modify the `~/.julia/v0.6/PACKAGE/LICENSE.md` file after it has been generated.
別のライセンスを使用したい場合は、パッケージジェネレータにパッケージを追加するか、これら3つのうちの1つを選択してから、 `〜/ .julia / v0.6 / PACKAGE / LICENSE.md`ファイルを変更してください生成されています。

#If you created a GitHub account and configured git to know about it, `PkgDev.generate()` will set an appropriate origin URL for you. 
GitHubアカウントを作成し、それについて知るようにgitを設定した場合、 `PkgDev.generate（）`は適切なオリジンURLを設定します。
#It will also automatically generate a `.travis.yml` file for using the [Travis](https://travis-ci.org) automated testing service, and an `appveyor.yml` file for using [AppVeyor](https://www.appveyor.com). 
また、[Travis]（https://travis-ci.org）自動テストサービスを使用するための `.travis.yml`ファイルと、[AppVeyor]（https：//travis-ci.org）を使用するための` appveyor.yml`ファイルを自動的に生成します。 //www.appveyor.com）。
#You will have to enable testing on the Travis and AppVeyor websites for your package repository, but once you've done that, it will already have working tests. 
パッケージリポジトリ用のTravisとAppVeyorのWebサイトでテストを有効にする必要がありますが、これを済ませたら既に動作テストが行​​われています。
#Of course, all the default testing does is verify that `using FooBar` in Julia works.
もちろん、すべてのデフォルトテストは、Juliaで `FooBarを使う 'が動作することを確認しています。

#### Loading Static Non-Julia Files
###静的非ジュリアファイルを読み込む

#If your package code needs to load static files which are not Julia code, e.g. an external library or data files, and are located within the package directory, use the `@__DIR__` macro to determine the directory of the current source file. 
あなたのパッケージコードがジュリアコードではない静的ファイルをロードする必要がある場合、例えば、外部のライブラリやデータファイルであり、パッケージディレクトリ内にある場合は、 `@__ DIR__`マクロを使用して現在のソースファイルのディレクトリを決定します。
#For example if `FooBar/src/FooBar.jl` needs to load `FooBar/data/foo.csv`, use the following code:
たとえば `FooBar / src / FooBar.jl`が` FooBar / data / foo.csv`をロードする必要がある場合、次のコードを使用します：

```julia
datapath = joinpath(@__DIR__, "..", "data")
foo = readcsv(joinpath(datapath, "foo.csv"))
```

### Making Your Package Available
###あなたのパッケージを利用可能にする

#Once you've made some commits and you're happy with how `FooBar` is working, you may want to get some other people to try it out. 
いくつかのコミットを行い、 `FooBar`がどのように動作しているかに満足すれば、他の人に試してもらうことができます。
#First you'll need to create the remote repository and push your code to it; we don't yet automatically do this for you, but we will in the future and it's not too hard to figure out [^3]. 
まず、リモートリポジトリを作成してコードをプッシュする必要があります。 私たちはまだあなたのためにこれを自動的には行いませんが、私たちは将来的にそれを理解することが難しくありません。
#Once you've done this, letting people try out your code is as simple as sending them the URL of the published repo – in this case:
これを済ませたら、コードを試してみましょう。この場合、公開されたレポのURLを送信するだけです。

```
git://github.com/StefanKarpinski/FooBar.jl.git
```

#For your package, it will be your GitHub user name and the name of your package, but you get the idea. 
あなたのパッケージでは、あなたのGitHubユーザ名とあなたのパッケージの名前ですが、あなたはそのアイデアを得ます。
#People you send this URL to can use [`Pkg.clone()`](@ref) to install the package and try it out:
このURLを送る人は[`Pkg.clone（）`]（@ ref）を使ってパッケージをインストールして試してみることができます：

```julia-repl
julia> Pkg.clone("git://github.com/StefanKarpinski/FooBar.jl.git")
INFO: Cloning FooBar from git@github.com:StefanKarpinski/FooBar.jl.git
```

[^3]:
   # Installing and using GitHub's ["hub" tool](https://github.com/github/hub) is highly recommended.
    GitHubの["hub"ツール（https://github.com/github/hub）のインストールと使用を強くお勧めします。
   # It allows you to do things like run `hub create` in the package repo and have it automatically created via GitHub's API.
    パッケージレポで `hub create`を実行し、GitHubのAPI経由で自動的に作成されるようなことをすることができます。

### Tagging and Publishing Your Package
###あなたのパッケージにタグをつけて公開する



!!! tip
!!! 先端
   # If you are hosting your package on GitHub, you can use the [attobot integration](https://github.com/attobot/attobot) to handle package registration, tagging and publishing.
    GitHubでパッケージをホスティングしている場合は、[attobot integration]（https://github.com/attobot/attobot）を使用して、パッケージの登録、タグ付け、公開を処理できます。

#Once you've decided that `FooBar` is ready to be registered as an official package, you can add it to your local copy of `METADATA` using `PkgDev.register()`:
`FooBar`が公式パッケージとして登録される準備が整ったら、` PkgDev.register（） `を使って` METADATA`のローカルコピーに追加することができます：

```julia-repl
julia> PkgDev.register("FooBar")
INFO: Registering FooBar at git://github.com/StefanKarpinski/FooBar.jl.git
INFO: Committing METADATA for FooBar
```

#This creates a commit in the `~/.julia/v0.6/METADATA` repo:
これは `〜/ .julia / v0.6 / METADATA`レポでコミットを作成します：

```
$ cd ~/.julia/v0.6/METADATA && git show

commit 9f71f4becb05cadacb983c54a72eed744e5c019d
Author: Stefan Karpinski <stefan@karpinski.org>
Date:   Wed Oct 16 18:46:02 2013 -0400

    Register FooBar

diff --git a/FooBar/url b/FooBar/url
new file mode 100644
index 0000000..30e525e
--- /dev/null
+++ b/FooBar/url
@@ -0,0 +1 @@
+git://github.com/StefanKarpinski/FooBar.jl.git
```

#This commit is only locally visible, however.
ただし、このコミットはローカルでのみ表示されます。
#To make it visible to the Julia community, you need to merge your local `METADATA` upstream into the official repo. 
Juliaコミュニティに表示させるには、ローカルの `METADATA`上流を公式リポジトリにマージする必要があります。
#The `PkgDev.publish()` command will fork the `METADATA` repository on GitHub, push your changes to your fork, and open a pull request:
`PkgDev.publish（）`コマンドは、GitHub上の `METADATA`リポジトリをフォークし、あなたの変更をあなたのフォークにプッシュし、プルリクエストをオープンします：

```julia-repl
julia> PkgDev.publish()
INFO: Validating METADATA
INFO: No new package versions to publish
INFO: Submitting METADATA changes
INFO: Forking JuliaLang/METADATA.jl to StefanKarpinski
INFO: Pushing changes as branch pull-request/ef45f54b
INFO: To create a pull-request open:

  https://github.com/StefanKarpinski/METADATA.jl/compare/pull-request/ef45f54b
```

!!! tip
   # If `PkgDev.publish()` fails with error:
   `PkgDev.publish()` がエラーで失敗した場合：

    ```
    ERROR: key not found: "token"
    ```

  # then you may have encountered an issue from using the GitHub API on multiple systems. 
   複数のシステムでGitHub APIを使用する際に問題が発生した可能性があります。
  # The solution is to delete the "Julia Package Manager" personal access token [from your Github account](https://github.com/login?return_to=https%3A%2F%2Fgithub.com%2Fsettings%2Ftokens) and try again.
   解決方法は、[Julia Package Manager]個人アクセストークン（Githubアカウントから）（https://github.com/login?return_to=https%3A%2F%2Fgithub.com%2Fsettings%2Ftokens）を削除してもう一度お試しください 。

  # Other failures may require you to circumvent `PkgDev.publish()` by [creating a pull request on GitHub](https://help.github.com/articles/creating-a-pull-request/).
   他の失敗は、[GitHubでプルリクエストを作成する]（https://help.github.com/articles/creating-a-pull-request/）で `PkgDev.publish（）`を回避する必要があります。
  # See: [Publishing METADATA manually](@ref) below.
    以下を参照してください。[手動でMETADATAを公開する]（@ ref）

#Once the package URL for `FooBar` is registered in the official `METADATA` repo, people know where to clone the package from, but there still aren't any registered versions available. 
`FooBar`のパッケージURLが公式の` METADATA`リポジトリに登録されると、人々はパッケージをどこからクローンするかを知っていますが、まだ登録されているバージョンはありません。
#You can tag and register it with the `PkgDev.tag()` command:
`PkgDev.tag（）`コマンドでタグ付けして登録することができます：

```julia-repl
julia> PkgDev.tag("FooBar")
INFO: Tagging FooBar v0.0.1
INFO: Committing METADATA for FooBar
```

#This tags `v0.0.1` in the `FooBar` repo:
これは `FooBar`リポジトリに` v0.0.1`のタグを付けます：

```
$ cd ~/.julia/v0.6/FooBar && git tag
v0.0.1
```

#It also creates a new version entry in your local `METADATA` repo for `FooBar`:
また、ローカルの `METADATA`リポジトリに` FooBar`の新しいバージョンエントリを作成します：

```
$ cd ~/.julia/v0.6/FooBar && git show
commit de77ee4dc0689b12c5e8b574aef7f70e8b311b0e
Author: Stefan Karpinski <stefan@karpinski.org>
Date:   Wed Oct 16 23:06:18 2013 -0400

    Tag FooBar v0.0.1

diff --git a/FooBar/versions/0.0.1/sha1 b/FooBar/versions/0.0.1/sha1
new file mode 100644
index 0000000..c1cb1c1
--- /dev/null
+++ b/FooBar/versions/0.0.1/sha1
@@ -0,0 +1 @@
+84b8e266dae6de30ab9703150b3bf771ec7b6285
```

#The `PkgDev.tag()` command takes an optional second argument that is either an explicit version number object like `v"0.0.1"` or one of the symbols `:patch`, `:minor` or `:major`. 
`PkgDev.tag（）`コマンドは、 `v" 0.0.1 "`のような明示的なバージョン番号オブジェクトか、 `：patch`、`：minor`、 `：major`のいずれかのシンボル。
#These increment the patch, minor or major version number of your package intelligently.
これらは、パッケージのパッチ、マイナーバージョンまたはメジャーバージョン番号をインテリジェントにインクリメントします。

#Adding a tagged version of your package will expedite the official registration into METADATA.jl by collaborators. 
タグ付きバージョンのパッケージを追加すると、公式登録が共同作業者によってMETADATA.jlに迅速化されます。
#It is strongly recommended that you complete this process, regardless if your package is completely ready for an official release.
パッケージが公式リリースのために完全に準備されているかどうかにかかわらず、このプロセスを完了することを強くお勧めします。

#As a general rule, packages should be tagged `0.0.1` first.
一般的な規則として、パッケージには最初に `0.0.1`というタグが付けられます。
#Since Julia itself hasn't achieved `1.0` status, it's best to be conservative in your package's tagged versions.
Julia自体が `1.0 'ステータスに達していないので、パッケージのタグ付きバージョンでは控えめな方がいいです。

#As with `PkgDev.register()`, these changes to `METADATA` aren't available to anyone else until they've been included upstream. 
`PkgDev.register（）`と同様に、 `METADATA`に対するこれらの変更は、上流に含まれるまで他人には利用できません。
#Again, use the `PkgDev.publish()` command, which first makes sure that individual package repos have been tagged, pushes them if they haven't already been, and then opens a pull request to `METADATA`:
ここでも `PkgDev.publish（）`コマンドを使います。これは、個々のパッケージreposがタグ付けされていることを最初に確認し、まだreppされていなければpushし、 `METADATA`にプルリクエストを開きます。

```julia-repl
julia> PkgDev.publish()
INFO: Validating METADATA
INFO: Pushing FooBar permanent tags: v0.0.1
INFO: Submitting METADATA changes
INFO: Forking JuliaLang/METADATA.jl to StefanKarpinski
INFO: Pushing changes as branch pull-request/3ef4f5c4
INFO: To create a pull-request open:

  https://github.com/StefanKarpinski/METADATA.jl/compare/pull-request/3ef4f5c4
```

#### Publishing METADATA manually
####手動でMETADATAを公開する

#If `PkgDev.publish()` fails you can follow these instructions to manually publish your package.
`PkgDev.publish（）`が失敗した場合、以下の指示に従って手動でパッケージを公開することができます。

#By "forking" the main METADATA repository, you can create a personal copy (of METADATA.jl) under your GitHub account. 
主要なMETADATAリポジトリを「フォーク」することで、GitHubアカウントの下に（METADATA.jlの）個人用コピーを作成することができます。
#Once that copy exists, you can push your local changes to your copy (just like any other GitHub project).
そのコピーが存在すると、（他のGitHubプロジェクトと同様に）ローカルの変更をコピーにプッシュできます。

1. 
  # go to [https://github.com/login?return_to=https%3A%2F%2Fgithub.com%2FJuliaLang%2FMETADATA.jl%2Ffork](https://github.com/login?return_to=https%3A%2F%2Fgithub.com%2FJuliaLang%2FMETADATA.jl%2Ffork) and create your own fork.
    [https://github.com/login?return_to=https%3A%2F%2Fgithub.com%2FJuliaLang%2FMETADATA.jl%2Ffork](https://github.com/login?return_to=https%3A%）にアクセスしてください。 2F％2Fgithub.com％2FJuliaLang％2FMETADATA.jl％2Ffork）自分のフォークを作成します。

2. 
  # add your fork as a remote repository for the METADATA repository on your local computer (in the terminal where USERNAME is your github username):
   あなたのローカルコンピュータ上のMETADATAリポジトリのリモートリポジトリとしてフォークを追加します（USERNAMEはあなたのgithubユーザ名です）。

```
cd ~/.julia/v0.6/METADATA
git remote add USERNAME https://github.com/USERNAME/METADATA.jl.git
```

3. 
  # push your changes to your fork:
   あなたの変更をあなたのフォークにプッシュする：

   ```
   git push USERNAME metadata-v2
   ```

4. 
  # If all of that works, then go back to the GitHub page for your fork, and click the "pull request" link.
   それがすべて機能している場合は、フォークのGitHubページに戻り、「プルリクエスト」リンクをクリックします。

    `` ``
    git push USERNAMEメタデータ-v2

##パッケージ要件の修正
## Fixing Package Requirements

#If you need to fix the registered requirements of an already-published package version, you can do so just by editing the metadata for that version, which will still have the same commit hash – the hash associated with a version is permanent:
すでに公開されているパッケージバージョンの登録済みの要件を修正する必要がある場合は、同じバージョンのメタデータを編集するだけで済みます。コミットハッシュは同じですが、バージョンに関連付けられたハッシュは永続的です。

```
$ cd ~/.julia/v0.6/METADATA/FooBar/versions/0.0.1 && cat requires
julia 0.3-
$ vi requires
```

# Since the commit hash stays the same, the contents of the `REQUIRE` file that will be checked out in the repo will **not** match the requirements in `METADATA` after such a change; this is unavoidable. 
コミットハッシュは同じままであるので、リポジトリでチェックアウトされる `REQUIRE`ファイルの内容は、そのような変更後に` METADATA`の要件に合致しません**。 これはやむを得ないことです。
# When you fix the requirements in `METADATA` for a previous version of a package, however, you should also fix the `REQUIRE` file in the current version of the package.
しかし、以前のバージョンのパッケージのために `METADATA`の要件を修正した場合は、現在のバージョンのパッケージで` REQUIRE`ファイルを修正するべきです。

## Requirements Specification
##要求仕様

# The `~/.julia/v0.6/REQUIRE` file, the `REQUIRE` file inside packages, and the `METADATA` package `requires` files use a simple line-based format to express the ranges of package versions which need to be installed. 
パッケージ内の `〜/ .julia / v0.6 / REQUIRE`ファイル、` REQUIRE`ファイル、 `METADATA`パッケージ` requires`ファイルは、必要なパッケージバージョンの範囲を表現するための単純な行ベースの形式を使用します インストールする。
# Package `REQUIRE` and `METADATA requires` files should also include the range of versions of `julia` the package is expected to work with. 
パッケージ `REQUIRE`と` METADATA requires`ファイルには、パッケージが動作すると予想される `julia`のバージョンの範囲も含まれていなければなりません。
# Additionally, packages can include a `test/REQUIRE` file to specify additional packages which are only required for testing.
さらに、パッケージには、テストに必要な追加パッケージを指定する `test / REQUIRE`ファイルを含めることができます。

# Here's how these files are parsed and interpreted.
これらのファイルを解析して解釈する方法は次のとおりです。

  * 
  # verything after a `#` mark is stripped from each line as a comment.
     `＃ 'マークのあとはコメントとして各行から削除されます。
  * 
  # If nothing but whitespace is left, the line is ignored.
     空白以外のものが残っていれば、その行は無視されます。
  * 
  # If there are non-whitespace characters remaining, the line is a requirement and the is split on whitespace into words.
     白文字以外の文字が残っている場合、その行は必須であり、空白文字で分割されて単語になります。

# The simplest possible requirement is just the name of a package name on a line by itself:
可能な限り単純な要求は、パッケージ名の名前だけです。

```julia
Distributions
```

#This requirement is satisfied by any version of the `Distributions` package. 
この要件は `Distributions`パッケージのどのバージョンでも満たされます。
#The package name can be followed by zero or more version numbers in ascending order, indicating acceptable intervals of versions of that package. 
パッケージ名の後ろに、ゼロまたはそれ以上のバージョン番号を昇順で続けて、そのパッケージのバージョンの許容可能な間隔を示すことができます。
#One version opens an interval, while the next closes it, and the next opens a new interval, and so on; if an odd number of version numbers are given, then arbitrarily large versions will satisfy; if an even number of version numbers are given, the last one is an upper limit on acceptable version numbers. 
あるバージョンは間隔を開き、次のバージョンはそれを閉じ、次のバージョンは新しい間隔を開きます。 奇数のバージョン番号が与えられた場合、任意に大きなバージョンが満足されます。 偶数のバージョン番号が与えられた場合、最後のバージョン番号は許容バージョン番号の上限です。
#For example, the line:
たとえば、次の行：

```
Distributions 0.1
```

#is satisfied by any version of `Distributions` greater than or equal to `0.1.0`. Suffixing a version with `-` allows any pre-release versions as well. 
「0.1.0」以上の「分布」の任意のバージョンによって満たされる。 `--`でバージョンに接尾辞を付けると、リリース前のバージョンも可能になります。 
#For example:
例えば：

```
Distributions 0.1-
```

# is satisfied by pre-release versions such as `0.1-dev` or `0.1-rc1`, or by any version greater than or equal to `0.1.0`.
0.1-dev」または「0.1-rc1」のようなプレリリースバージョン、または「0.1.0」以上の任意のバージョンによって満たされる。

# This requirement entry:
この要件エントリ：

```
Distributions 0.1 0.2.5
```

#is satisfied by versions from `0.1.0` up to, but not including `0.2.5`.
「0.1.0」から「0.2.5」までのバージョンによって満足される。
#If you want to indicate that any `0.1.x` version will do, you will want to write:
`0.1.x`のバージョンが何をするかを示したいなら、あなたは次のように書いておきます：

```
Distributions 0.1 0.2-
```

#If you want to start accepting versions after `0.2.7`, you can write:
`0.2.7`の後でバージョンの受け入れを開始したい場合、以下のように書くことができます：

```
Distributions 0.1 0.2- 0.2.7
```

#If a requirement line has leading words that begin with `@`, it is a system-dependent requirement.
要件行に `@ 'で始まる先頭の単語がある場合、これはシステム依存の要件です。
#If your system matches these system conditionals, the requirement is included, if not, the requirement is ignored. 
システムがこれらのシステム条件と一致する場合、要件が含まれます。要件が含まれていない場合、要件は無視されます。
#For example:
例えば：

```
@osx Homebrew
```

#will require the `Homebrew` package only on systems where the operating system is OS X. 
オペレーティングシステムがOS Xのシステムでのみ `Homebrew`パッケージが必要になります。
#The system conditions that are currently supported are (hierarchically):
現在サポートされているシステム条件は（階層的に）次のとおりです。

  * `@unix`

      * `@linux`
      * `@bsd`

          * `@osx`
  * `@windows`

#The `@unix` condition is satisfied on all UNIX systems, including Linux and BSD. 
`@unix`条件は、LinuxやBSDを含むすべてのUNIXシステムで満たされます。
#Negated system conditionals are also supported by adding a `!` after the leading `@`. 
ネゲートされたシステム条件は、先頭の `@`の後ろに `！`を追加することによってもサポートされます。
例：
#Examples:

```
@!windows
@unix @!osx
```

#The first condition applies to any system but Windows and the second condition applies to any UNIX system besides OS X.
最初の条件はWindowsを除くすべてのシステムに適用され、2番目の条件はOS X以外のUNIXシステムに適用されます。

#Runtime checks for the current version of Julia can be made using the built-in `VERSION` variable, which is of type `VersionNumber`. 
Juliaの現在のバージョンのランタイムチェックは、 `VersionNumber`タイプの組み込みの` VERSION`変数を使用して行うことができます。
#Such code is occasionally necessary to keep track of new or deprecated functionality between various releases of Julia. 
このようなコードは、Juliaのさまざまなリリース間で新しい機能または非推奨の機能を追跡するために必要な場合があります。
#Examples of runtime checks:
ランタイムチェックの例：

```julia
VERSION < v"0.3-" #exclude all pre-release versions of 0.3

v"0.2-" <= VERSION < v"0.3-" #get all 0.2 versions, including pre-releases, up to the above

v"0.2" <= VERSION < v"0.3-" #To get only stable 0.2 versions (Note v"0.2" == v"0.2.0")

VERSION >= v"0.2.1" #get at least version 0.2.1
```

#See the section on [version number literals](@ref man-version-number-literals) for a more complete description.
より完全な説明は、[バージョン番号リテラル]（@ ref man-version-number-literals）の節を参照してください。
