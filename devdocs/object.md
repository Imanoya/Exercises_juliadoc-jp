# Memory layout of Julia Objects

## Object layout (jl_value_t)

The `jl_value_t` struct is the name for a block of memory owned by the Julia Garbage Collector, representing the data associated with a Julia object in memory.
> `jl_value_t` 構造体はJuliaガベージコレクタが所有するメモリブロックの名前で、メモリ内のJuliaオブジェクトに関連するデータを表します。
Absent any type information, it is simply an opaque pointer:
> 任意の型情報がない場合、それは単なる不透明なポインタです：

```c
typedef struct jl_value_t* jl_pvalue_t;
```

Each `jl_value_t` struct is contained in a `jl_typetag_t` struct that contains metadata information about the Julia object, such as its type and garbage collector (gc) reachability:
> それぞれの `jl_value_t` 構造体は、Juliaオブジェクトに関するメタデータ情報を含む `jl_typetag_t` 構造体に含まれています。その型やガベージコレクタ（gc）の到達可能性：

```c
typedef struct {
    opaque metadata;
    jl_value_t value;
} jl_typetag_t;
```

The type of any Julia object is an instance of a leaf `jl_datatype_t` object.
> 任意のJuliaオブジェクトの型は、葉 `jl_datatype_t` オブジェクトのインスタンスです。
The `jl_typeof()` function can be used to query for it:
> `jl_typeof（）`関数を使ってそれを問い合わせることができます：

```c
jl_value_t *jl_typeof(jl_value_t *v);
```

The layout of the object depends on its type.
> オブジェクトのレイアウトは、そのタイプによって異なります。
Reflection methods can be used to inspect that layout.
> リフレクションメソッドを使用してそのレイアウトを検査することができます。
A field can be accessed by calling one of the get-field methods:
> フィールドは、get-fieldメソッドの1つを呼び出すことでアクセスできます。

```c
jl_value_t *jl_get_nth_field_checked(jl_value_t *v, size_t i);
jl_value_t *jl_get_field(jl_value_t *o, char *fld);
```

If the field types are known, a priori, to be all pointers, the values can also be extracted directly as an array access:
> フィールドの型がわかっていれば、先験的に、すべてのポインタになるため、配列アクセスとして値を直接抽出することもできます。

```c
jl_value_t *v = value->fieldptr[n];
```

As an example, a "boxed" `uint16_t` is stored as follows:
> 一例として、「ボックス化された」 `uint16_t` は以下のように格納される。

```c
struct {
    opaque metadata;
    struct {
        uint16_t data;        // -- 2 bytes
    } jl_value_t;
};
```

This object is created by `jl_box_uint16()`.
> このオブジェクトは `jl_box_uint16()` によって作成されます。
Note that the `jl_value_t` pointer references the data portion, not the metadata at the top of the struct.
> `jl_value_t` ポインタは、構造体の先頭のメタデータではなく、データ部分を参照することに注意してください。

A value may be stored "unboxed" in many circumstances (just the data, without the metadata, and possibly not even stored but just kept in registers), so it is unsafe to assume that the address of a box is a unique identifier.
> 値は、多くの状況（データのみ、メタデータなし、場合によっては格納されずにレジスタに保存されている）で「アンボックス化」されて格納されている可能性があるので、ボックスのアドレスが一意の識別子であると仮定することは危険です。
The "egal" test (corresponding to the `===` function in Julia), should instead be used to compare two unknown objects for equivalence:
> 代わりに、 "egal"テスト（ジュリアの `===` 関数に相当）を使用して、2つの未知の等価物を比較する必要があります。

```c
int jl_egal(jl_value_t *a, jl_value_t *b);
```

This optimization should be relatively transparent to the API, since the object will be "boxed" on-demand, whenever a `jl_value_t` pointer is needed.
> この最適化は、 `jl_value_t` ポインタが必要なときはいつでも、オブジェクトがオンデマンドで「ボックス化」されるので、APIに対して比較的透過的でなければなりません。

Note that modification of a `jl_value_t` pointer in memory is permitted only if the object is mutable.
> メモリ内の `jl_value_t` ポインタの変更は、オブジェクトが変更可能な場合にのみ許可されることに注意してください。
Otherwise, modification of the value may corrupt the program and the result will be undefined.
> そうしないと、値を変更するとプログラムが破壊され、結果は不定になります。
The mutability property of a value can be queried for with:
> 値の可変性プロパティは、次のように照会できます。

```c
int jl_is_mutable(jl_value_t *v);
```

If the object being stored is a `jl_value_t`, the Julia garbage collector must be notified also:
> 格納されているオブジェクトが `jl_value_t`である場合、Juliaガベージコレクタにも通知する必要があります。

```c
void jl_gc_wb(jl_value_t *parent, jl_value_t *ptr);
```

However, the [Embedding Julia](@ref) section of the manual is also required reading at this point, for covering other details of boxing and unboxing various types, and understanding the gc interactions.
> しかし、マニュアルの[Embedding Julia](@ref)セクションでは、ボクシングのさまざまな詳細やさまざまなタイプのアンボクシング、gcインタラクションの理解について、この時点で読むことが必要です。

Mirror structs for some of the built-in types are [defined in `julia.h`](https://github.com/JuliaLang/julia/blob/master/src/julia.h).
> いくつかの組み込み型のミラー構造体は[julia.h`で定義されています（https://github.com/JuliaLang/julia/blob/master/src/julia.h）。
The corresponding global `jl_datatype_t` objects are created by [`jl_init_types` in `jltypes.c`](https://github.com/JuliaLang/julia/blob/master/src/jltypes.c).
> 対応するグローバルな `jl_datatype_t` オブジェクトは [`jltypes.c` の  `jl_init_types`]（https://github.com/JuliaLang/julia/blob/master/src/jltypes.c）によって作成されます。

## Garbage collector mark bits

The garbage collector uses several bits from the metadata portion of the `jl_typetag_t` to track each object in the system.
> ガベージコレクタは、 `jl_typetag_t`のメタデータ部分からいくつかのビットを使用して、システム内の各オブジェクトを追跡します。
Further details about this algorithm can be found in the comments of the [garbage collector implementation in `gc.c`](https://github.com/JuliaLang/julia/blob/master/src/gc.c).
> このアルゴリズムの詳細は、[`gc.c`のガベージコレクタ実装]（https://github.com/JuliaLang/julia/blob/master/src/gc.c）のコメントにあります。

## Object allocation

Most new objects are allocated by `jl_new_structv()`:
> ほとんどの新しいオブジェクトは `jl_new_structv()` によって割り当てられます：

```c
jl_value_t *jl_new_struct(jl_datatype_t *type, ...);
jl_value_t *jl_new_structv(jl_datatype_t *type, jl_value_t **args, uint32_t na);
```

Although, [`isbits`](@ref) objects can be also constructed directly from memory:
しかし、[`isbits`](@ref) オブジェクトはメモリーから直接構築することもできますが、

```c
jl_value_t *jl_new_bits(jl_value_t *bt, void *data)
```

And some objects have special constructors that must be used instead of the above functions:
> また、一部のオブジェクトには、上記の関数の代わりに使用する必要がある特別なコンストラクタがあります。

Types:

```c
jl_datatype_t *jl_apply_type(jl_datatype_t *tc, jl_tuple_t *params);
jl_datatype_t *jl_apply_array_type(jl_datatype_t *type, size_t dim);
jl_uniontype_t *jl_new_uniontype(jl_tuple_t *types);
```

While these are the most commonly used options, there are more low-level constructors too, which you can find declared in [`julia.h`](https://github.com/JuliaLang/julia/blob/master/src/julia.h).
> これらは最も一般的に使用されるオプションですが、より低レベルのコンストラクタもあります。これは[`julia.h`]で宣言されています（https://github.com/JuliaLang/julia/blob/master/src/ julia.h）。
These are used in `jl_init_types()` to create the initial types needed to bootstrap the creation of the Julia system image.
> これらは `jl_init_types()` で使用され、Juliaシステムイメージの作成をブートストラップするのに必要な初期型を作成します。

Tuples:

```c
jl_tuple_t *jl_tuple(size_t n, ...);
jl_tuple_t *jl_tuplev(size_t n, jl_value_t **v);
jl_tuple_t *jl_alloc_tuple(size_t n);
```

The representation of tuples is highly unique in the Julia object representation ecosystem.
> タプルの表現は、Juliaオブジェクト表現エコシステムにおいて極めてユニークです。
In some cases, a [`Base.tuple()`](@ref) object may be an array of pointers to the objects contained by the tuple equivalent to:
> 場合によっては、[`Base.tuple()`](@ref) オブジェクトは、次のものに相当するタプルに含まれるオブジェクトへのポインタの配列であるかもしれません：

```c
typedef struct {
    size_t length;
    jl_value_t *data[length];
} jl_tuple_t;
```

However, in other cases, the tuple may be converted to an anonymous [`isbits`](@ref) type and stored unboxed, or it may not stored at all (if it is not being used in a generic context as a `jl_value_t*`).
> しかし、他のケースでは、タプルは匿名の[`isbits`](@ref)型に変換され、ボックス化されていない状態で格納されるか、まったく格納されないことがあります。 （ジェネリックコンテキストで `jl_value_t*`として使用されていない場合）。

Symbols:

```c
jl_sym_t *jl_symbol(const char *str);
```

Functions and MethodInstance:
> 関数とMethodInstance：

```c
jl_function_t *jl_new_generic_function(jl_sym_t *name);
jl_method_instance_t *jl_new_method_instance(jl_value_t *ast, jl_tuple_t *sparams);
```

Arrays:

```c
jl_array_t *jl_new_array(jl_value_t *atype, jl_tuple_t *dims);
jl_array_t *jl_new_arrayv(jl_value_t *atype, ...);
jl_array_t *jl_alloc_array_1d(jl_value_t *atype, size_t nr);
jl_array_t *jl_alloc_array_2d(jl_value_t *atype, size_t nr, size_t nc);
jl_array_t *jl_alloc_array_3d(jl_value_t *atype, size_t nr, size_t nc, size_t z);
jl_array_t *jl_alloc_vec_any(size_t n);
```

Note that many of these have alternative allocation functions for various special-purposes.
> これらの多くは、さまざまな特殊目的のための代替割り当て関数を持っていることに注意してください。
The list here reflects the more common usages, but a more complete list can be found by reading the [`julia.h` header file](https://github.com/JuliaLang/julia/blob/master/src/julia.h).
> ここのリストはより一般的な使用法を反映していますが、より完全なリストは[`julia.h`ヘッダファイル]（https://github.com/JuliaLang/julia/blob/master/src/julia）を参照してください。 h）。

Internal to Julia, storage is typically allocated by `newstruct()` (or `newobj()` for the special types):
> Juliaの内部では、通常、ストレージは `newstruct()` （または特別な型の `newobj()` によって割り当てられます）：

```c
jl_value_t *newstruct(jl_value_t *type);
jl_value_t *newobj(jl_value_t *type, size_t nfields);
```

And at the lowest level, memory is getting allocated by a call to the garbage collector (in `gc.c`), then tagged with its type:
> そして最も低いレベルでは、メモリはガベージコレクタ（ `gc.c` 内）の呼び出しによって割り当てられ、その型でタグ付けされます：

```c
jl_value_t *jl_gc_allocobj(size_t nbytes);
void jl_set_typeof(jl_value_t *v, jl_datatype_t *type);
```

Note that all objects are allocated in multiples of 4 bytes and aligned to the platform pointer size.
> すべてのオブジェクトが4バイトの倍数で割り当てられ、プラットフォームのポインタサイズにアライメントされることに注意してください。
Memory is allocated from a pool for smaller objects, or directly with `malloc()` for large objects.
> メモリは、小さいオブジェクトの場合はプールから、大きなオブジェクトの場合は `malloc()` で直接割り当てられます。

!!! sidebar "Singleton Types"
   Singleton types have only one instance and no data fields. Singleton instances have a size of 0 bytes, and consist only of their metadata. e.g. `nothing::Nothing`.
   > シングルトンタイプはインスタンスが1つしかなく、データフィールドもありません。 シングルトンインスタンスのサイズは0バイトで、メタデータのみで構成されています。 例えば `nothing::Nothing`。

   See [Singleton Types](@ref man-singleton-types) and [Nothingness and missing values](@ref)
   > [Singleton Types](@ref man-singleton-types) と [Nothingness and missing values](@ref) を参照してください。