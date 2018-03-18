# High-level Overview of the Native-Code Generation Process

## Representation of Pointers

<!-- EN -->
When emitting code to an object file, pointers will be emitted as relocations.
> オブジェクトファイルにコードを発行する場合、ポインタは再配置として発行されます。
<!-- EN -->
The deserialization code will ensure any object that pointed to one of these constants gets recreated and contains the right runtime pointer.
> デシリアライゼーションコードは、これらの定数の1つを指すオブジェクトが再作成され、正しいランタイムポインタを含むことを保証します。

<!-- EN -->
Otherwise, they will be emitted as literal constants.
> それ以外の場合は、リテラル定数として出力されます。

<!-- EN -->
To emit one of these objects, call `literal_pointer_val`.
> これらのオブジェクトの1つを発行するには、 `literal_pointer_val`を呼び出します。
<!-- EN -->
It'll handle tracking the Julia value and the LLVM global, ensuring they are valid both for the current runtime and after deserialization.
> Julia値とLLVMグローバルのトラッキングを処理し、現在のランタイムとデシリアライズ後の両方で有効であることを保証します。

<!-- EN -->
When emitted into the object file, these globals are stored as references in a large `gvals` table.
> オブジェクトファイルに出力されると、これらのグローバルは大きな `gvals`テーブルに参照として格納されます。
<!-- EN -->
This allows the deserializer to reference them by index, and implement a custom manual mechanism similar to a Global Offset Table (GOT) to restore them.
> これにより、デシリアライザはインデックスでそれらを参照し、GOT（Global Offset Table）に似た独自の手動メカニズムを実装して復元することができます。

<!-- EN -->
Function pointers are handled similarly.
> 関数ポインタも同様に扱われます。
<!-- EN -->
They are stored as values in a large `fvals` table.
> それらは大きな `fvals` テーブルに値として格納されます。
<!-- EN -->
Like globals, this allows the deserializer to reference them by index.
> グローバルと同様に、これにより、デシリアライザはインデックスでそれらを参照できます。

<!-- EN -->
Note that `extern` functions are handled separately, with names, via the usual symbol resolution mechanism in the linker.
> `extern` 関数は、リンカの通常のシンボル解決メカニズムを介して、名前とともに別々に処理されることに注意してください。

<!-- EN -->
Note too that `ccall` functions are also handled separately, via a manual GOT and Procedure Linkage Table (PLT).
> `ccall`関数もまた、マニュアルGOTとプロシージャーリンケージテーブル（PLT）を介して別々に処理されることに注意してください。


## Representation of Intermediate Values

<!-- EN -->
Values are passed around in a `jl_cgval_t` struct.
> 値は `jl_cgval_t`構造体に渡されます。
<!-- EN -->
This represents an R-value, and includes enough information to determine how to assign or pass it somewhere.
> これはR値を表し、それをどこかに割り当てたり渡したりする方法を決定するのに十分な情報を含んでいます。

<!-- EN -->
They are created via one of the helper constructors, usually:
> それらはヘルパーコンストラクタの1つを介して作成されます。
<!-- EN -->
`mark_julia_type` (for immediate values) and `mark_julia_slot` (for pointers to values).
> `mark_julia_type`（即値）と `mark_julia_slot`（値へのポインタ用）です。

<!-- EN -->
The function `convert_julia_type` can transform between any two types.
> `convert_julia_type`関数は任意の2つの型の間で変換できます。
<!-- EN -->
It returns an R-value with `cgval.typ` set to `typ`.
> `cgval.typ`を` typ`に設定してR値を返します。
<!-- EN -->
It'll cast the object to the requested representation, making heap boxes, allocating stack copies, and computing tagged unions as needed to change the representation.
> オブジェクトを要求された表現にキャストし、ヒープボックスを作成し、スタックコピーを割り当て、表現を変更するために必要に応じてタグ付きユニオンを計算します。

<!-- EN -->
By contrast `update_julia_type` will change `cgval.typ` to `typ`, only if it can be done at zero-cost (i.e. without emitting any code).
> 対照的に、 `update_julia_type`はゼロコスト（つまりコードを出さずに）で実行できる場合にのみ、 `cgval.typ` を `typ` に変更します。


## Union representation

<!-- EN -->
Inferred union types may be stack allocated via a tagged type representation.
> 推論された共用体型は、タグ付きの型表現を介してスタックに割り当てられます。

<!-- EN -->
The primitive routines that need to be able to handle tagged unions are:
> タグ付きの共用体を処理できる必要があるプリミティブ・ルーチンは次のとおりです。
- mark-type
- load-local
- store-local
- isa
- is
- emit_typeof
- emit_sizeof
- boxed
- unbox
- specialized cc-ret

<!-- EN -->
Everything else should be possible to handle in inference by using these primitives to implement union-splitting.
> これらのプリミティブを使用してユニオン分割を実装することで、他のすべてが推論で扱うことができるはずです。

<!-- EN -->
The representation of the tagged-union is as a pair of `< void* union, byte selector >`.
> tagged-unionの表現は、 `<void * union、byte selector>のペアです。
<!-- EN -->
The selector is fixed-size as `byte & 0x7f`, and will union-tag the first 126 isbits.
> セレクタは `byte＆0x7f`という固定サイズで、最初の126個のisbitsを結合タグに入れます。
<!-- EN -->
It records the one-based depth-first count into the type-union of the isbits objects inside.
> これは、内部のisbitsオブジェクトのタイプ - ユニオンに1ベースの深さ最初のカウントを記録します。
<!-- EN -->
An index of zero indicates that the `union*` is actually a tagged heap-allocated `jl_value_t*`, and needs to be treated as normal for a boxed object rather than as a tagged union.
> ゼロのインデックスは、 `union*` が実際にタグ付きヒープに割り当てられた `jl_value_t*` であることを示し、タグ付き共用体ではなくボックス化されたオブジェクトに対して通常の扱いをする必要があります。

<!-- EN -->
The high bit of the selector (`byte & 0x80`) can be tested to determine if the `void*` is actually a heap-allocated (`jl_value_t*`) box, thus avoiding the cost of re-allocating a box, while maintaining the ability to efficiently handle union-splitting based on the low bits.
> セレクタ (`byte＆0x80`) の上位ビットは、 `void*` が実際にヒープに割り当てられた(`jl_value_t*`) ボックスであるかどうかを判断するためにテストすることができるので、ボックスを再割り当てするコストを避けることができます。低ビットに基づいて効率的にユニオン分割を処理する能力を維持する。

<!-- EN -->
It is guaranteed that `byte & 0x7f` is an exact test for the type, if the value can be represented by a tag – it will never be marked `byte = 0x80`.
> 値がタグで表現できるならば、 `byte＆0x7f` は型の正確なテストであることが保証されています。 `byte = 0x80` とマークされることはありません。
<!-- EN -->
It is not necessary to also test the type-tag when testing `isa`.
> `isa` テスト時にタイプタグをテストする必要はありません。

<!-- EN -->
The `union*` memory region may be allocated at *any* size.
> `union *`メモリ領域は*任意の*サイズで割り当てられます。
<!-- EN -->
The only constraint is that it is big enough to contain the data currently specified by `selector`.
> 唯一の制約は、 `selector`で現在指定されているデータを格納するのに十分な大きさであるということです。
<!-- EN -->
It might not be big enough to contain the union of all types that could be stored there according to the associated Union type field.
> 関連するUnion型フィールドに従って格納できるすべての型の和集合を格納するのに十分な大きさではないかもしれません。
<!-- EN -->
Use appropriate care when copying.
> コピーするときは適切な注意を払ってください。


## Specialized Calling Convention Signature Representation

<!-- EN -->
A `jl_returninfo_t` object describes the calling convention details of any callable.
> `jl_returninfo_t`オブジェクトは、呼び出し可能な呼び出し規約の詳細を記述します。

<!-- EN -->
If any of the arguments or return type of a method can be represented unboxed, and the method is not varargs, it'll be given an optimized calling convention signature based on its `specTypes` and `rettype` fields.
> メソッドの引数または戻り値の型のいずれかがunboxedで表現され、そのメソッドがvarargsでない場合、 `specTypes`および` rettype`フィールドに基づいて最適化された呼び出し規約シグネチャが与えられます。

<!-- EN -->
The general principles are that:
> 一般的な原則は次のとおりです。

<!-- EN -->
- Primitive types get passed in int/float registers.
- > プリミティブ型はint / floatレジスタで渡されます。
<!-- EN -->
- Tuples of VecElement types get passed in vector registers.
- > VecElement型のタプルはベクトルレジスタに渡されます。
<!-- EN -->
- Structs get passed on the stack.
- > 構造体がスタックに渡されます。
<!-- EN -->
- Return values are handle similarly to arguments,
- > 戻り値は引数と同様に扱われますが、
<!-- EN -->
  with a size-cutoff at which they will instead be returned via a hidden sret argument.
>  それらは代わりに隠されたsret引数を介して返されるサイズカットオフを伴います。

<!-- EN -->
The total logic for this is implemented by `get_specsig_function` and `deserves_sret`.
> これに対する総ロジックは `get_specsig_function`と` deserves_sret`によって実装されています。

<!-- EN -->
Additionally, if the return type is a union, it may be returned as a pair of values (a pointer and a tag).
> さらに、戻り値の型が共用体の場合、値のペア（ポインタおよびタグ）として戻される場合があります。
<!-- EN -->
If the union values can be stack-allocated, then sufficient space to store them will also be passed as a hidden first argument.
> 共用体の値をスタックに割り振ることができる場合、それらを格納するのに十分なスペースも隠された最初の引数として渡されます。
<!-- EN -->
It is up to the callee whether the returned pointer will point to this space, a boxed object, or even other constant memory.
> 返されたポインタがこのスペース、ボックス化されたオブジェクト、または他の定数メモリを指しているかどうかは、呼び出し先に依存します。
