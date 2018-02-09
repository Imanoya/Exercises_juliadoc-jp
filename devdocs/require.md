# Module loading

[`Base.require`](@ref) is responsible for loading modules and it also manages the precompilation cache.
> [`Base.require`](@ref) はモジュールのロードを担当し、プリコンパイルキャッシュも管理します。
It is the implementation of the `import` statement.
> それは `import`ステートメントの実装です。

## Experimental features
The features below are experimental and not part of the stable Julia API.
> 以下の機能は実験的なものであり、安定したJulia APIの一部ではありません。
Before building upon them inform yourself about the current thinking and whether they might change soon.
> それらを構築する前に、現在の考え方や、すぐに変化するかどうかを自分自身に知らせてください。

### Module loading callbacks

It is possible to listen to the modules loaded by `Base.require`, by registering a callback.
> `Base.require`によってロードされたモジュールをコールバックを登録することで聴くことができます。

```julia
loaded_packages = Channel{Symbol}()
callback = (mod::Symbol) -> put!(loaded_packages, mod)
push!(Base.package_callbacks, callback)
```

Please note that the symbol given to the callback is a non-unique identifier and it is the responsibility of the callback provider to walk the module chain to determine the fully qualified name of the loaded binding.
> コールバックに与えられたシンボルは一意ではない識別子であり、ロードされたバインディングの完全修飾名を決定するためにモジュールチェーンを歩くのはコールバックプロバイダの責任です。

The callback below is an example of how to do that:
> 以下のコールバックはそれを行う方法の例です：

```julia
# Get the fully-qualified name of a module.
function module_fqn(name::Symbol)
    fqn = fullname(Base.root_module(name))
    return join(fqn, '.')
end
```
