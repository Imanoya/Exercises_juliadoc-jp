<!-- Start -->

# Unicode Input
> # ユニコード インプット

<!-- End -->
<!-- Start -->
The following table lists Unicode characters that can be entered via tab completion of LaTeX-like abbreviations in the Julia REPL (and in various other editing environments).  
> 次の表に、Julia REPL（およびその他のさまざまな編集環境）でLaTeXのような略語のタブ補完を使用して入力できるUnicode文字を示します。
<!-- End -->
<!-- Start -->
You can also get information on how to type a symbol by entering it in the REPL help, i.e. by typing `?` and then entering the symbol in the REPL (e.g., by copy-paste from somewhere you saw the symbol).
> REPLのヘルプに入力することで、シンボルを入力する方法についての情報を得ることもできます。たとえば、 `?` とタイプしてREPLにシンボルを入力します。（シンボルを見た場所からコピー貼り付けなど）
<!-- End -->

!!! warning

<!-- Start -->
   This table may appear to contain missing characters in the second column, or even show characters that are inconsistent with the characters as they are rendered in the Julia REPL. 
   > このテーブルには、2番目の列に不足している文字が含まれているか、Julia REPLでレンダリングされている文字と一致しない文字が表示されることがあります。
<!-- End -->
<!-- Start -->
   In these cases, users are strongly advised to check their choice of fonts in their browser and REPL environment, as there are known issues with glyphs in many fonts.
   > このような場合、多くのフォントでグリフに関する既知の問題があるため、ブラウザとREPL環境でフォントの選択を確認することを強くお勧めします。
<!-- End -->
<!-- Start -->

```@eval
#
# Generate a table containing all LaTeX and Emoji tab completions available in the REPL.
#

const NBSP = '\u00A0'

function tab_completions(symbols...)
    completions = Dict{String, Vector{String}}()
    for each in symbols, (k, v) in each
        completions[v] = push!(get!(completions, v, String[]), k)
    end
    return completions
end

function unicode_data()
    file = normpath(JULIA_HOME, "..", "..", "doc", "UnicodeData.txt")
    names = Dict{UInt32, String}()
    open(file) do unidata
        for line in readlines(unidata)
            id, name, desc = split(line, ";")[[1, 2, 11]]
            codepoint = parse(UInt32, "0x$id")
            names[codepoint] = titlecase(lowercase(name == "" ? desc : desc == "" ? name : "$name / $desc"))
        end
    end
    return names
end

# Surround combining characters with no-break spaces (i.e '\u00A0'). Follows the same format
# for how unicode is displayed on the unicode.org website:
# http://unicode.org/cldr/utility/character.jsp?a=0300
function fix_combining_chars(char)
    cat = Base.UTF8proc.category_code(char)
    return cat == 6 || cat == 8 ? "$NBSP$char$NBSP" : "$char"
end


function table_entries(completions, unicode_dict)
    entries = [[
        "Code point(s)", "Character(s)",
        "Tab completion sequence(s)", "Unicode name(s)"
    ]]
    for (chars, inputs) in sort!(collect(completions), by = first)
        code_points, unicode_names, characters = String[], String[], String[]
        for char in chars
            push!(code_points, "U+$(uppercase(hex(char, 5)))")
            push!(unicode_names, get(unicode_dict, UInt32(char), "(No Unicode name)"))
            push!(characters, isempty(characters) ? fix_combining_chars(char) : "$char")
        end
        push!(entries, [
            join(code_points, " + "), join(characters),
            join(inputs, ", "), join(unicode_names, " + ")
        ])
    end
    return Markdown.Table(entries, [:l, :l, :l, :l])
end

table_entries(
    tab_completions(
        Base.REPLCompletions.latex_symbols,
        Base.REPLCompletions.emoji_symbols
    ),
    unicode_data()
)
```
