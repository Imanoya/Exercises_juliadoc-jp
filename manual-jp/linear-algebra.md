<!-- Start -->

# Linear algebra

> # 線形代数

<!-- End -->
In addition to (and as part of) its support for multi-dimensional arrays, Julia provides native implementations of many common and useful linear algebra operations. 
> Juliaは、多次元配列に対するサポートに加えて、その一部として、多くの一般的かつ有用な線形代数演算のネイティブ実装を提供しています。
<!-- End -->
<!-- Start -->
Basic operations, such as [`trace`](@ref), [`det`](@ref), and [`inv`](@ref) are all supported:
> [trace]（@ ref）、[det]（@ ref）、[`inv`]（@ ref）などの基本的な操作はすべてサポートされています：
<!-- End -->

```jldoctest
julia> A = [1 2 3; 4 1 6; 7 8 1]
3×3 Array{Int64,2}:
 1  2  3
 4  1  6
 7  8  1

julia> trace(A)
3

julia> det(A)
104.0

julia> inv(A)
3×3 Array{Float64,2}:
 -0.451923   0.211538    0.0865385
  0.365385  -0.192308    0.0576923
  0.240385   0.0576923  -0.0673077
```

<!-- Start -->
As well as other useful operations, such as finding eigenvalues or eigenvectors:
> 固有値や固有ベクトルの発見などの他の有用な演算と同様に、
<!-- End -->

```jldoctest
julia> A = [1.5 2 -4; 3 -1 -6; -10 2.3 4]
3×3 Array{Float64,2}:
   1.5   2.0  -4.0
   3.0  -1.0  -6.0
 -10.0   2.3   4.0

julia> eigvals(A)
3-element Array{Complex{Float64},1}:
  9.31908+0.0im
 -2.40954+2.72095im
 -2.40954-2.72095im

julia> eigvecs(A)
3×3 Array{Complex{Float64},2}:
 -0.488645+0.0im  0.182546-0.39813im   0.182546+0.39813im
 -0.540358+0.0im  0.692926+0.0im       0.692926-0.0im
   0.68501+0.0im  0.254058-0.513301im  0.254058+0.513301im
```

<!-- Start -->
In addition, Julia provides many [factorizations](@ref man-linalg-factorizations) which can be used to speed up problems such as linear solve or matrix exponentiation by pre-factorizing a matrix into a form more amenable (for performance or memory reasons) to the problem. 
> さらに、Juliaは行列をより適切な形にプリファクトリ化することで、線形解法や行列のべき乗などの問題を高速化するために使用できる多くの[因数分解]（@ ref man-linalg-factorizations）を提供しています ）問題に。
<!-- End -->
<!-- Start -->
See the documentation on [`factorize`](@ref) for more information.
> 詳細は、[`factorize`]（@ ref）のドキュメントを参照してください。
<!-- End -->
<!-- Start -->
As an example:
>例として：
<!-- End -->

```jldoctest
julia> A = [1.5 2 -4; 3 -1 -6; -10 2.3 4]
3×3 Array{Float64,2}:
   1.5   2.0  -4.0
   3.0  -1.0  -6.0
 -10.0   2.3   4.0

julia> factorize(A)
Base.LinAlg.LU{Float64,Array{Float64,2}} with factors L and U:
[1.0 0.0 0.0; -0.15 1.0 0.0; -0.3 -0.132196 1.0]
[-10.0 2.3 4.0; 0.0 2.345 -3.4; 0.0 0.0 -5.24947]
```

<!-- Start -->
Since `A` is not Hermitian, symmetric, triangular, tridiagonal, or bidiagonal, an LU factorization may be the best we can do. 
> `A` はエルミート、対称、三角形、三重対角、二重対角線ではないので、LU分解は可能な限り最善の方法かもしれません。
<!-- End -->
<!-- Start -->
Compare with:
> と比べて：
<!-- End -->

```jldoctest
julia> B = [1.5 2 -4; 2 -1 -3; -4 -3 5]
3×3 Array{Float64,2}:
  1.5   2.0  -4.0
  2.0  -1.0  -3.0
 -4.0  -3.0   5.0

julia> factorize(B)
Base.LinAlg.BunchKaufman{Float64,Array{Float64,2}}
D factor:
3×3 Tridiagonal{Float64}:
 -1.64286   0.0   ⋅
  0.0      -2.8  0.0
   ⋅        0.0  5.0
U factor:
3×3 Base.LinAlg.UnitUpperTriangular{Float64,Array{Float64,2}}:
 1.0  0.142857  -0.8
 0.0  1.0       -0.6
 0.0  0.0        1.0
permutation:
3-element Array{Int64,1}:
 1
 2
 3
successful: true
```

<!-- Start -->
Here, Julia was able to detect that `B` is in fact symmetric, and used a more appropriate factorization.
> ここで、Juliaは、「B」が実際に対称であることを検出し、より適切な分解を使用した。
<!-- End -->
<!-- Start -->
Often it's possible to write more efficient code for a matrix that is known to have certain properties e.g. it is symmetric, or tridiagonal. 
> 例えば、特定の特性を有することが知られているマトリックスに対して、より効率的なコードを書くことができる。 それは対称または三重対角です。
<!-- End -->
<!-- Start -->
Julia provides some special types so that you can "tag" matrices as having these properties. 
> Juliaにはいくつかの特殊な型が用意されているため、行列にこれらのプロパティを持つものとして「タグ付け」することができます。
<!-- End -->
<!-- Start -->
For instance:
> 例えば：
<!-- End -->

```jldoctest
julia> B = [1.5 2 -4; 2 -1 -3; -4 -3 5]
3×3 Array{Float64,2}:
  1.5   2.0  -4.0
  2.0  -1.0  -3.0
 -4.0  -3.0   5.0

julia> sB = Symmetric(B)
3×3 Symmetric{Float64,Array{Float64,2}}:
  1.5   2.0  -4.0
  2.0  -1.0  -3.0
 -4.0  -3.0   5.0
```

<!-- Start -->
`sB` has been tagged as a matrix that's (real) symmetric, so for later operations we might perform on it, such as eigenfactorization or computing matrix-vector products, efficiencies can be found by only referencing half of it.
> `sB`は（実際の）対称の行列としてタグ付けされていますので、後の演算で固有値化や行列ベクトル積の計算などの演算を実行することがあります。効率は半分を参照するだけで見つけることができます。
<!-- End -->
<!-- Start -->
For example:
> 例えば：
<!-- End -->

```jldoctest
julia> B = [1.5 2 -4; 2 -1 -3; -4 -3 5]
3×3 Array{Float64,2}:
  1.5   2.0  -4.0
  2.0  -1.0  -3.0
 -4.0  -3.0   5.0

julia> sB = Symmetric(B)
3×3 Symmetric{Float64,Array{Float64,2}}:
  1.5   2.0  -4.0
  2.0  -1.0  -3.0
 -4.0  -3.0   5.0

julia> x = [1; 2; 3]
3-element Array{Int64,1}:
 1
 2
 3

julia> sB\x
3-element Array{Float64,1}:
 -1.73913
 -1.1087
 -1.45652
```

<!-- Start -->
The `\` operation here performs the linear solution. Julia's parser provides convenient dispatch to specialized methods for the *transpose* of a matrix left-divided by a vector, or for the various combinations of transpose operations in matrix-matrix solutions. 
> ここでの `\ '操作は線形解を実行します。 Juliaのパーサーは、ベクトルで左分割された行列の*転置*、または行列 - 行列解の転置演算のさまざまな組み合わせに対して、特殊なメソッドへの便利なディスパッチを提供します。
<!-- End -->
<!-- Start -->
Many of these are further specialized for certain special matrix types. 
> これらの多くは特定の特殊マトリックスタイプに特化しています。
<!-- End -->
<!-- Start -->
For example, `A\B` will end up calling [`Base.LinAlg.A_ldiv_B!`](@ref) while `A'\B` will end up calling [`Base.LinAlg.Ac_ldiv_B`](@ref), even though we used the same left-division operator. 
> 例えば、 `A \ B`は` `Base.LinAlg.A_ldiv_B！`（@ ref）を呼び出して終了し、 `A '\ B`は` `Base.LinAlg.Ac_ldiv_B`'（@ ref） たとえ我々が同じ左 - 除算演算子を使用したとしても。
<!-- End -->
<!-- Start -->
This works for matrices too: `A.'\B.'` would call [`Base.LinAlg.At_ldiv_Bt`](@ref). 
> これは行列に対しても機能します： `A。 '\ B.'`は[` Base.LinAlg.At_ldiv_Bt`]（@ ref）を呼び出します。
<!-- End -->
<!-- Start -->
The left-division operator is pretty powerful and it's easy to write compact, readable code that is flexible enough to solve all sorts of systems of linear equations.
> 左辺演算子は非常に強力で、あらゆる種類の線形方程式を解くのに十分柔軟なコンパクトで読みやすいコードを書くのは簡単です。
<!-- End -->
<!-- Start -->

### Special matrices

> ### 特殊な行列

<!-- End -->
<!-- Start -->
[Matrices with special symmetries and structures](http://www2.imm.dtu.dk/pubdb/views/publication_details.php?id=3274) arise often in linear algebra and are frequently associated with various matrix factorizations.
> [特別な対称性と構造を持つ行列]（http://www2.imm.dtu.dk/pubdb/views/publication_details.php?id=3274）は、しばしば線形代数で発生し、さまざまな行列分解を頻繁に行います。
<!-- End -->
<!-- Start -->
Julia features a rich collection of special matrix types, which allow for fast computation with specialized routines that are specially developed for particular matrix types.
> Juliaは、特定のマトリックスタイプ用に特別に開発された特殊なルーチンを使用して高速計算を可能にする、特別なマトリックスタイプの豊富なコレクションを備えています。
<!-- End -->

<!-- Start -->
The following tables summarize the types of special matrices that have been implemented in Julia, as well as whether hooks to various optimized methods for them in LAPACK are available.
> 次の表は、Juliaで実装されている特殊な行列のタイプと、LAPACKのさまざまな最適化されたメソッドへのフックが利用可能かどうかをまとめたものです。
<!-- End -->

| Type                      | Description                                                                      |
|:------------------------- |:-------------------------------------------------------------------------------- |
| [`Symmetric`](@ref)       | [Symmetric matrix](https://en.wikipedia.org/wiki/Symmetric_matrix)               |
| [`Hermitian`](@ref)       | [Hermitian matrix](https://en.wikipedia.org/wiki/Hermitian_matrix)               |
| [`UpperTriangular`](@ref) | Upper [triangular matrix](https://en.wikipedia.org/wiki/Triangular_matrix)       |
| [`LowerTriangular`](@ref) | Lower [triangular matrix](https://en.wikipedia.org/wiki/Triangular_matrix)       |
| [`Tridiagonal`](@ref)     | [Tridiagonal matrix](https://en.wikipedia.org/wiki/Tridiagonal_matrix)           |
| [`SymTridiagonal`](@ref)  | Symmetric tridiagonal matrix                                                     |
| [`Bidiagonal`](@ref)      | Upper/lower [bidiagonal matrix](https://en.wikipedia.org/wiki/Bidiagonal_matrix) |
| [`Diagonal`](@ref)        | [Diagonal matrix](https://en.wikipedia.org/wiki/Diagonal_matrix)                 |
| [`UniformScaling`](@ref)  | [Uniform scaling operator](https://en.wikipedia.org/wiki/Uniform_scaling)        |

<!-- Start -->

### Elementary operations

> ### 初等オペレーション

<!-- End -->
| Matrix type               | `+` | `-` | `*` | `\` | Other functions with optimized methods                              |
|:------------------------- |:--- |:--- |:--- |:--- |:------------------------------------------------------------------- |
| [`Symmetric`](@ref)       |     |     |     | MV  | [`inv()`](@ref), [`sqrtm()`](@ref), [`expm()`](@ref)                |
| [`Hermitian`](@ref)       |     |     |     | MV  | [`inv()`](@ref), [`sqrtm()`](@ref), [`expm()`](@ref)                |
| [`UpperTriangular`](@ref) |     |     | MV  | MV  | [`inv()`](@ref), [`det()`](@ref)                                    |
| [`LowerTriangular`](@ref) |     |     | MV  | MV  | [`inv()`](@ref), [`det()`](@ref)                                    |
| [`SymTridiagonal`](@ref)  | M   | M   | MS  | MV  | [`eigmax()`](@ref), [`eigmin()`](@ref)                              |
| [`Tridiagonal`](@ref)     | M   | M   | MS  | MV  |                                                                     |
| [`Bidiagonal`](@ref)      | M   | M   | MS  | MV  |                                                                     |
| [`Diagonal`](@ref)        | M   | M   | MV  | MV  | [`inv()`](@ref), [`det()`](@ref), [`logdet()`](@ref), [`/()`](@ref) |
| [`UniformScaling`](@ref)  | M   | M   | MVS | MVS | [`/()`](@ref)                                                       |

Legend:

| Key        | Description                                                   |
|:---------- |:------------------------------------------------------------- |
| M (matrix) | An optimized method for matrix-matrix operations is available |
| V (vector) | An optimized method for matrix-vector operations is available |
| S (scalar) | An optimized method for matrix-scalar operations is available |

### Matrix factorizations
###行列分解

| Matrix type              | LAPACK | [`eig()`](@ref) | [`eigvals()`](@ref) | [`eigvecs()`](@ref) | [`svd()`](@ref) | [`svdvals()`](@ref) |
|:-------------------------|:------ |:--------------- |:------------------- |:------------------- |:--------------- |:------------------- |
| [`Symmetric`](@ref)      | SY     |                 | ARI                 |                     |                 |                     |
| [`Hermitian`](@ref)      | HE     |                 | ARI                 |                     |                 |                     |
| [`UpperTriangular`](@ref)| TR     | A               | A                   | A                   |                 |                     |
| [`LowerTriangular`](@ref)| TR     | A               | A                   | A                   |                 |                     |
| [`SymTridiagonal`](@ref) | ST     | A               | ARI                 | AV                  |                 |                     |
| [`Tridiagonal`](@ref)    | GT     |                 |                     |                     |                 |                     |
| [`Bidiagonal`](@ref)     | BD     |                 |                     |                     | A               | A                   |
| [`Diagonal`](@ref)       | DI     |                 | A                   |                     |                 |                     |

Legend:

| Key          | Description                                                                                                                     |
                                                                                                                            Example              |
|:------------ |:------------------------------------------------------------------------------------------------------------------------------- |
                                                                                                                           :-------------------- |
| A (all)      | An optimized method to find all the characteristic values and/or vectors is available                                           |
                                                                                                                            e.g. `eigvals(M)`    |
| R (range)    | An optimized method to find the `il`th through the `ih`th characteristic values are available                                   |
                                                                                                                            `eigvals(M, il, ih)` |
| I (interval) | An optimized method to find the characteristic values in the interval [`vl`, `vh`] is available                                 |
                                                                                                                            `eigvals(M, vl, vh)` |
| V (vectors)  | An optimized method to find the characteristic vectors corresponding to the characteristic values `x=[x1, x2,...]` is available |
                                                                                                                             `eigvecs(M, x)`     |


<!-- Start -->

### The uniform scaling operator

> ### 均等スケーリング演算子

<!-- End -->
<!-- Start -->
A [`UniformScaling`](@ref) operator represents a scalar times the identity operator, `λ*I`. 
> [`UniformScaling`]（@ref） 演算子は、スカラーがアイデンティティ演算子「λ* I」を乗じたものを表します。
<!-- End -->
<!-- Start -->
The identity operator `I` is defined as a constant and is an instance of `UniformScaling`. 
> アイデンティティ演算子 `I`は定数として定義され、` UniformScaling`のインスタンスです。
<!-- End -->
<!-- Start -->
The size of these operators are generic and match the other matrix in the binary operations [`+`](@ref), [`-`](@ref), [`*`](@ref) and [`\`](@ref). 
> これらの演算子のサイズは一般的であり、バイナリ演算[`+`]（@ ref）、[` - `]（@ ref）、[`*`]（@ ref）および[`\` ]（@ ref）。
<!-- End -->
<!-- Start -->
For `A+I` and `A-I` this means that `A` must be square. 
> 「A + I」と「A-I」の場合、これは「A」が正方形でなければならないことを意味する。
<!-- End -->
<!-- Start -->
Multiplication with the identity operator `I` is a noop (except for checking that the scaling factor is one) and therefore almost without overhead.
> アイデンティティ演算子「I」を使用した乗算はnoop（スケーリング係数が1であることを確認することを除く）であり、したがってオーバーヘッドはほとんどありません。
<!-- End -->
<!-- Start -->

## [Matrix factorizations](@id man-linalg-factorizations)

## [行列の因数分解]（@ idの人間の因数分解）

<!-- End -->
<!-- Start -->
[Matrix factorizations (a.k.a. matrix decompositions)](https://en.wikipedia.org/wiki/Matrix_decomposition) compute the factorization of a matrix into a product of matrices, and are one of the central concepts in linear algebra.
> [行列分解（a.k.a行列分解）]（https://en.wikipedia.org/wiki/Matrix_decomposition）は、行列の行列分解を計算し、線形代数の中心概念の1つです。
<!-- End -->

<!-- Start -->
The following table summarizes the types of matrix factorizations that have been implemented in Julia. 
> 次の表は、Juliaで実装された行列分解のタイプをまとめたものです。
<!-- End -->
<!-- Start -->
Details of their associated methods can be found in the [Linear Algebra](@ref) section of the standard library documentation.
> 関連するメソッドの詳細は、標準ライブラリドキュメントの[Linear Algebra]（@ ref）セクションにあります。
<!-- End -->

| Type              | Description                                                                                  |
|:----------------- |:-------------------------------------------------------------------------------------------- |
| `Cholesky`        | [Cholesky factorization](https://en.wikipedia.org/wiki/Cholesky_decomposition)               |
| `CholeskyPivoted` | [Pivoted](https://en.wikipedia.org/wiki/Pivot_element) Cholesky factorization                |
| `LU`              | [LU factorization](https://en.wikipedia.org/wiki/LU_decomposition)                           |
| `LUTridiagonal`   | LU factorization for [`Tridiagonal`](@ref) matrices                                          |
| `UmfpackLU`       | LU factorization for sparse matrices (computed by UMFPack)                                   |
| `QR`              | [QR factorization](https://en.wikipedia.org/wiki/QR_decomposition)                           |
| `QRCompactWY`     | Compact WY form of the QR factorization                                                      |
| `QRPivoted`       | Pivoted [QR factorization](https://en.wikipedia.org/wiki/QR_decomposition)                   |
| `Hessenberg`      | [Hessenberg decomposition](http://mathworld.wolfram.com/HessenbergDecomposition.html)        |
| `Eigen`           | [Spectral decomposition](https://en.wikipedia.org/wiki/Eigendecomposition_(matrix))          |
| `SVD`             | [Singular value decomposition](https://en.wikipedia.org/wiki/Singular_value_decomposition)   |
| `GeneralizedSVD`  | [Generalized SVD]
                      (https://en.wikipedia.org/wiki/Generalized_singular_value_decomposition#Higher_order_version) |

