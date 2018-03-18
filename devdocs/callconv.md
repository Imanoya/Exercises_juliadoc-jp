# Calling Conventions

<!-- EN -->
Julia uses three calling conventions for four distinct purposes:
> Juliaは、4つの異なる目的のために3つの呼び出し規則を使用します。

| Name    | Prefix    | Purpose                          |
|:------- |:--------- |:-------------------------------- |
| Native  | `julia_`  | Speed via specialized signatures |
| JL Call | `jlcall_` | Wrapper for generic calls        |
| JL Call | `jl_`     | Builtins                         |
| C ABI   | `jlcapi_` | Wrapper callable from C          |

## Julia Native Calling Convention

<!-- EN -->
The native calling convention is designed for fast non-generic calls.
> ネイティブ呼び出し規約は、高速非総称呼び出し用に設計されています。
<!-- EN -->
It usually uses a specialized signature.
> 通常、特殊な署名を使用します。

  <!-- EN -->
  * LLVM ghosts (zero-length types) are omitted.
  > * LLVMゴースト（長さゼロタイプ）は省略されています。
  <!-- EN -->
  * LLVM scalars and vectors are passed by value.
  > * LLVMスカラーとベクトルは値渡しされます。
  <!-- EN -->
  * LLVM aggregates (arrays and structs) are passed by reference.
  > * LLVM集約（配列と構造体）は参照渡しです。

<!-- EN -->
A small return values is returned as LLVM return values. A large return values is returned via the "structure return" (`sret`) convention, where the caller provides a pointer to a return slot.
> 小さな戻り値がLLVM戻り値として返されます。 大きな戻り値は、呼び出し元が戻りスロットへのポインタを提供する "構造戻り"（ "sret`）規則によって返されます。

<!-- EN -->
An argument or return values thta is a homogeneous tuple is sometimes represented as an LLVM vector instead of an LLVM array.
> 引数または戻り値thtaが同型タプルであることは、LLVM配列の代わりにLLVMベクトルとして表現されることがあります。

## JL Call Convention

<!-- EN -->
The JL Call convention is for builtins and generic dispatch.
> JLコール規約は、ビルトインと汎用ディスパッチのためのものです。
<!-- EN -->
Hand-written functions using this convention are declared via the macro `JL_CALLABLE`.
> この規約を使用する手書き関数は、マクロ `JL_CALLABLE 'を介して宣言されます。
<!-- EN -->
The convention uses exactly 3 parameters:
> 慣例では、3つのパラメータを使用します。

  <!-- EN -->
  * `F`  - Julia representation of function that is being applied
  * > `F` - 適用されている関数のジュリア表現
  <!-- EN -->
  * `args` - pointer to array of pointers to boxes
  * > `args` - 箱へのポインタの配列へのポインタ
  <!-- EN -->
  * `nargs` - length of the array
  * > `nargs` - 配列の長さ

<!-- EN -->
The return value is a pointer to a box.
> 戻り値は、ボックスへのポインタです。

## C ABI

<!-- EN -->
C ABI wrappers enable calling Julia from C.
>C ABIラッパーはCからJuliaを呼び出すことができます。
<!-- EN -->
The wrapper calls a function using the native calling convention.
>ラッパーは、ネイティブ呼び出し規約を使用して関数を呼び出します。

<!-- EN -->
Tuples are always represented as C arrays.
>タプルは常にC配列として表されます。
