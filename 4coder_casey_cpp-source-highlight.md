<pre><i><font color="#9A1900">/* NOTE(casey): This code is _extremely_ bad and is mostly just me hacking things</font></i>
<i><font color="#9A1900">   around to put in features I want in advance of 4coder having them properly.</font></i>
<i><font color="#9A1900">   Most of the time I haven't even taken enough time to read the 4coder API</font></i>
<i><font color="#9A1900">   to know what I'm actually even doing.  So if you decide to use the code in</font></i>
<i><font color="#9A1900">   here, be advised that it might be super crashy or break something or cause you </font></i>
<i><font color="#9A1900">   to lose work or who knows what else!</font></i>

<i><font color="#9A1900">   DON'T SAY I WE DIDN'T WARN YA: This custom extension provided "as is" without</font></i>
<i><font color="#9A1900">   warranty of any kind, either express or implied, including without</font></i>
<i><font color="#9A1900">   limitation any implied warranties of condition, uninterrupted use,</font></i>
<i><font color="#9A1900">   merchantability, fitness for a particular purpose, or non-infringement.</font></i>
<i><font color="#9A1900">*/</font></i>

<i><font color="#9A1900">/* </font></i><b>TODO</b><i><font color="#9A1900">(casey): Here are our current issues</font></i>

<i><font color="#9A1900">   - High priority:</font></i>
<i><font color="#9A1900">     - Buffer switching still seems a little bit broken.  I find I can't reliably hit switch-return</font></i>
<i><font color="#9A1900">       and switch to the most recently viewed file that wasn't one of the two currently viewed buffers?</font></i>
<i><font color="#9A1900">       But maybe I'm imagining things?</font></i>
<i><font color="#9A1900">     - High-DPI settings break rendering and all fonts just show up as solid squares</font></i>
<i><font color="#9A1900">     - Pretty sure auto-indent has some bugs.  Things that should be pretty easy to indent</font></i>
<i><font color="#9A1900">       properly even from only a few surrounding lines seem to be indented improperly at the moment</font></i>
<i><font color="#9A1900">     - Multi-line comments should default to indenting to the indentation of the line prior?</font></i>
<i><font color="#9A1900">     - Would like the option to indent to hanging parentheses, equals signs, etc. instead of</font></i>
<i><font color="#9A1900">       always just "one tab in from the previous line".</font></i>
<i><font color="#9A1900">       - Actually, maybe just expose the dirty state, so that the user can decide whether to</font></i>
<i><font color="#9A1900">         save or not?  Not sure...</font></i>
<i><font color="#9A1900">     - Replace:</font></i>
<i><font color="#9A1900">       - Needs to be case-insensitive, or at least have the option to be</font></i>
<i><font color="#9A1900">       - Needs to replace using the case of the thing being replaced, or at least have the option to do so</font></i>
<i><font color="#9A1900">     - Auto-complete doesn't pick nearby words first, it seems, which makes it much slower to use?</font></i>
<i><font color="#9A1900">     - Bug with not being able to switch-to-corresponding-file in another buffer</font></i>
<i><font color="#9A1900">       without accidentally bringing up the file open dialog?</font></i>
<i><font color="#9A1900">     - Up/down arrows and mouse clicks on wrapped lines don't seem to work properly with several wraps.</font></i>
<i><font color="#9A1900">       (eg., a line wrapped to more than 2 physical lines on the screen often doesn't work anymore,</font></i>
<i><font color="#9A1900">        with up or down jumping to totally wrong places, and mouse clicking jumping to wrong places</font></i>
<i><font color="#9A1900">        as well - similarly, scrolling breaks, in that it thinks it has "hit the end" of the buffer</font></i>
<i><font color="#9A1900">        when you cursor down, but the cursor and the rest of the wrapped lines are actually off</font></i>
<i><font color="#9A1900">        the bottom of the screen)</font></i>

<i><font color="#9A1900">   - Display:</font></i>
<i><font color="#9A1900">     - There are often repaint bugs with 4coder coming to the front / unminimizing, etc.</font></i>
<i><font color="#9A1900">       I think this might have something to do with the way you're doing lots of semaphore</font></i>
<i><font color="#9A1900">       locking but I haven't investigated yet.</font></i>
<i><font color="#9A1900">     - Need a word-wrap mode that wraps at word boundaries instead of characters</font></i>
<i><font color="#9A1900">     - Need to be able to set a word wrap length at something other than the window</font></i>
<i><font color="#9A1900">?FIXED First go-to-line for a file seems to still just go to the beginning of the buffer?</font></i>
<i><font color="#9A1900">       Not sure Allen's right about the slash problem, but either way, we need some</font></i>
<i><font color="#9A1900">       way to fix it.</font></i>
<i><font color="#9A1900">     - NOTE / IMPORTANT / </font></i><b>TODO</b><i><font color="#9A1900"> highlighting?  Ability to customize?  Whatever.</font></i>
<i><font color="#9A1900">     - Some kind of parentheses highlighting?  I can write this myself, but I</font></i>
<i><font color="#9A1900">       would need some way of adding highlight information to the buffer.</font></i>
<i><font color="#9A1900">     - Need a way of highlighting the current line like Emacs does for the benefit</font></i>
<i><font color="#9A1900">       of people on The Stream(TM)</font></i>
<i><font color="#9A1900">     - Some kind of matching brace display so in long ifs, etc., you can see</font></i>
<i><font color="#9A1900">       what they match (maybe draw it directly into the buffer?)</font></i>

<i><font color="#9A1900">   - Indentation:</font></i>
<i><font color="#9A1900">     - Multiple // lines don't seem to indent properly.  The first one will go to the correct place, but the subsequent ones will go to the first column regardless?</font></i>
<i><font color="#9A1900">     - Need to have better indentation / wrapping control for typing in comments. </font></i>
<i><font color="#9A1900">       Right now it's a bit worse than Emacs, which does automatically put you at</font></i>
<i><font color="#9A1900">       the same margin as the prev. line (4coder just goes back to column 1).  It'd</font></i>
<i><font color="#9A1900">       be nice if it go _better_ than Emacs, with no need to manually flow comments,</font></i>
<i><font color="#9A1900">       etc.</font></i>

<i><font color="#9A1900">   - Buffer management: </font></i>
<i><font color="#9A1900">     - Seems like there's no way to switch to buffers whose names are substrings of other</font></i>
<i><font color="#9A1900">       buffers' names without using the mouse?</font></i>
<i><font color="#9A1900">       - Also, mouse-clicking on buffers doesn't seem to work reliably?  Often it just goes to a </font></i>
<i><font color="#9A1900">         blank window?</font></i>

<i><font color="#9A1900">   - File system</font></i>
<i><font color="#9A1900">     - When switching to a buffer that has changed on disk, notify?  Really this can just</font></i>
<i><font color="#9A1900">       be some way to query the modification flag and then the customization layer can do it?</font></i>
<i><font color="#9A1900">     - Still can't seem to open a zero-length file?</font></i>
<i><font color="#9A1900">     - I'd prefer it if file-open could create new files, and that I could get called on that</font></i>
<i><font color="#9A1900">       so I can insert my boilerplate headers on new files</font></i>
<i><font color="#9A1900">     - I'd prefer it if file-open deleted per-character instead of deleting entire path sections</font></i>

<i><font color="#9A1900">   - Need auto-complete for things like "arbitrary command", with options listed, etc.,</font></i>
<i><font color="#9A1900">     so this should either be built into 4ed, or the custom DLL should have the ability</font></i>
<i><font color="#9A1900">     to display possible completions and iterate over internal cmdid's, etc.  Possibly</font></i>
<i><font color="#9A1900">     the latter, for maximal ability of customizers to add their own commands?</font></i>

<i><font color="#9A1900">   - Macro recording/playback</font></i>

<i><font color="#9A1900">   - Arbitrary cool features:</font></i>
<i><font color="#9A1900">     - LOC count for the buffer and for all buffers summed shown in the title bar?</font></i>
<i><font color="#9A1900">     - Show auto-parsed #if/if/for/while/etc. statements at else and closing places.</font></i>
<i><font color="#9A1900">     - Automatic highlighting of the region in side the parentheses / etc.</font></i>
<i><font color="#9A1900">     - You should just implement a shell inside 4coder which can call all the 4coder</font></i>
<i><font color="#9A1900">       stuff as well as execute system stuff, so that from now on you just write</font></i>
<i><font color="#9A1900">       scripts "in 4coder", etc., so they are always portable everywhere 4coder runs?</font></i>

<i><font color="#9A1900">   - Things I should write:</font></i>
<i><font color="#9A1900">     - Ability to do "file open from same directory as the current buffer"</font></i>
<i><font color="#9A1900">     - Spell-checker</font></i>
<i><font color="#9A1900">     - To-do list dependent on project?</font></i>
<i><font color="#9A1900">     - Repeat last replace?</font></i>
<i><font color="#9A1900">     - Maybe search could be a permanent thing, so instead of initiating a search,</font></i>
<i><font color="#9A1900">       you're just _changing_ the search term with MODAL-S, and then there's _always_</font></i>
<i><font color="#9A1900">       a next-of-these-in... and that could go through buffers in order, to...</font></i>
<i><font color="#9A1900">*/</font></i>

<i><font color="#9A1900">// NOTE(casey): Microsoft/Windows is poopsauce.</font></i>

<b><font color="#000080">#include</font></b> <font color="#FF0000">&lt;math.h&gt;</font>
<b><font color="#000080">#include</font></b> <font color="#FF0000">&lt;stdio.h&gt;</font>

<b><font color="#000080">#include</font></b> <font color="#FF0000">"4coder_default_include.cpp"</font>

<b><font color="#0000FF">enum</font></b> maps<font color="#FF0000">{</font>
    my_code_map
<font color="#FF0000">}</font><font color="#990000">;</font>

<b><font color="#000080">#ifndef</font></b> <span class="pl-c1">Assert</span>
<b><font color="#000080">#define</font></b> internal <b><font color="#0000FF">static</font></b>
<b><font color="#000080">#define</font></b> Assert assert 
<b><font color="#000080">#endif</font></b>

<b><font color="#0000FF">struct</font></b> <font color="#008080">Parsed_Error</font>
<font color="#FF0000">{</font>
    <font color="#009900">int</font> exists<font color="#990000">;</font>

    <font color="#008080">String</font> target_file_name<font color="#990000">;</font>
    <font color="#009900">int</font> target_line_number<font color="#990000">;</font>
    <font color="#009900">int</font> target_column_number<font color="#990000">;</font>

    <font color="#009900">int</font> source_buffer_id<font color="#990000">;</font>
    <font color="#009900">int</font> source_position<font color="#990000">;</font>
<font color="#FF0000">}</font><font color="#990000">;</font>

<b><font color="#0000FF">static</font></b> <font color="#009900">bool</font> GlobalEditMode<font color="#990000">;</font>
<b><font color="#0000FF">static</font></b> <font color="#009900">char</font> <font color="#990000">*</font>GlobalCompilationBufferName <font color="#990000">=</font> <font color="#FF0000">"*compilation*"</font><font color="#990000">;</font>

<i><font color="#9A1900">// TODO(casey): If 4coder gets variables at some point, this would go in a variable.</font></i>
<b><font color="#0000FF">static</font></b> <font color="#009900">char</font> BuildDirectory<font color="#990000">[</font><font color="#993399">4096</font><font color="#990000">]</font> <font color="#990000">=</font> <font color="#FF0000">"./"</font><font color="#990000">;</font>

<b><font color="#0000FF">enum</font></b> token_type
<font color="#FF0000">{</font>
    Token_Unknown<font color="#990000">,</font>

    Token_OpenParen<font color="#990000">,</font>    
    Token_CloseParen<font color="#990000">,</font>    
    Token_Asterisk<font color="#990000">,</font>
    Token_Minus<font color="#990000">,</font>
    Token_Plus<font color="#990000">,</font>
    Token_ForwardSlash<font color="#990000">,</font>
    Token_Percent<font color="#990000">,</font>
    Token_Colon<font color="#990000">,</font>
    Token_Number<font color="#990000">,</font>
    Token_Comma<font color="#990000">,</font>

    Token_EndOfStream<font color="#990000">,</font>
<font color="#FF0000">}</font><font color="#990000">;</font>
<b><font color="#0000FF">struct</font></b> <font color="#008080">token</font>
<font color="#FF0000">{</font>
    <font color="#008080">token_type</font> Type<font color="#990000">;</font>

    <font color="#008080">size_t</font> TextLength<font color="#990000">;</font>
    <font color="#009900">char</font> <font color="#990000">*</font>Text<font color="#990000">;</font>
<font color="#FF0000">}</font><font color="#990000">;</font>

<b><font color="#0000FF">struct</font></b> <font color="#008080">tokenizer</font>
<font color="#FF0000">{</font>
    <font color="#009900">char</font> <font color="#990000">*</font>At<font color="#990000">;</font>
<font color="#FF0000">}</font><font color="#990000">;</font>

<b><font color="#0000FF">inline</font></b> <font color="#009900">bool</font>
<b><font color="#000000">IsEndOfLine</font></b><font color="#990000">(</font><font color="#009900">char</font> C<font color="#990000">)</font>
<font color="#FF0000">{</font>
    <font color="#009900">bool</font> Result <font color="#990000">=</font> <font color="#990000">((</font>C <font color="#990000">==</font> <font color="#FF0000">'</font><font color="#CC33CC">\n</font><font color="#FF0000">'</font><font color="#990000">)</font> <font color="#990000">||</font>
                   <font color="#990000">(</font>C <font color="#990000">==</font> <font color="#FF0000">'</font><font color="#CC33CC">\r</font><font color="#FF0000">'</font><font color="#990000">));</font>

    <b><font color="#0000FF">return</font></b><font color="#990000">(</font>Result<font color="#990000">);</font>
<font color="#FF0000">}</font>

<b><font color="#0000FF">inline</font></b> <font color="#009900">bool</font>
<b><font color="#000000">IsWhitespace</font></b><font color="#990000">(</font><font color="#009900">char</font> C<font color="#990000">)</font>
<font color="#FF0000">{</font>
    <font color="#009900">bool</font> Result <font color="#990000">=</font> <font color="#990000">((</font>C <font color="#990000">==</font> <font color="#FF0000">' '</font><font color="#990000">)</font> <font color="#990000">||</font>
                   <font color="#990000">(</font>C <font color="#990000">==</font> <font color="#FF0000">'</font><font color="#CC33CC">\t</font><font color="#FF0000">'</font><font color="#990000">)</font> <font color="#990000">||</font>
                   <font color="#990000">(</font>C <font color="#990000">==</font> <font color="#FF0000">'</font><font color="#CC33CC">\v</font><font color="#FF0000">'</font><font color="#990000">)</font> <font color="#990000">||</font>
                   <font color="#990000">(</font>C <font color="#990000">==</font> <font color="#FF0000">'</font><font color="#CC33CC">\f</font><font color="#FF0000">'</font><font color="#990000">)</font> <font color="#990000">||</font>
                   <b><font color="#000000">IsEndOfLine</font></b><font color="#990000">(</font>C<font color="#990000">));</font>

    <b><font color="#0000FF">return</font></b><font color="#990000">(</font>Result<font color="#990000">);</font>
<font color="#FF0000">}</font>

<b><font color="#0000FF">inline</font></b> <font color="#009900">bool</font>
<b><font color="#000000">IsAlpha</font></b><font color="#990000">(</font><font color="#009900">char</font> C<font color="#990000">)</font>
<font color="#FF0000">{</font>
    <font color="#009900">bool</font> Result <font color="#990000">=</font> <font color="#990000">(((</font>C <font color="#990000">&gt;=</font> <font color="#FF0000">'a'</font><font color="#990000">)</font> <font color="#990000">&amp;&amp;</font> <font color="#990000">(</font>C <font color="#990000">&lt;=</font> <font color="#FF0000">'z'</font><font color="#990000">))</font> <font color="#990000">||</font>
                   <font color="#990000">((</font>C <font color="#990000">&gt;=</font> <font color="#FF0000">'A'</font><font color="#990000">)</font> <font color="#990000">&amp;&amp;</font> <font color="#990000">(</font>C <font color="#990000">&lt;=</font> <font color="#FF0000">'Z'</font><font color="#990000">)));</font>

    <b><font color="#0000FF">return</font></b><font color="#990000">(</font>Result<font color="#990000">);</font>
<font color="#FF0000">}</font>

<b><font color="#0000FF">inline</font></b> <font color="#009900">bool</font>
<b><font color="#000000">IsNumeric</font></b><font color="#990000">(</font><font color="#009900">char</font> C<font color="#990000">)</font>
<font color="#FF0000">{</font>
    <font color="#009900">bool</font> Result <font color="#990000">=</font> <font color="#990000">((</font>C <font color="#990000">&gt;=</font> <font color="#FF0000">'0'</font><font color="#990000">)</font> <font color="#990000">&amp;&amp;</font> <font color="#990000">(</font>C <font color="#990000">&lt;=</font> <font color="#FF0000">'9'</font><font color="#990000">));</font>

    <b><font color="#0000FF">return</font></b><font color="#990000">(</font>Result<font color="#990000">);</font>
<font color="#FF0000">}</font>

<b><font color="#0000FF">static</font></b> <font color="#009900">void</font>
<b><font color="#000000">EatAllWhitespace</font></b><font color="#990000">(</font><font color="#008080">tokenizer</font> <font color="#990000">*</font>Tokenizer<font color="#990000">)</font>
<font color="#FF0000">{</font>
    <b><font color="#0000FF">for</font></b><font color="#990000">(;;)</font>
    <font color="#FF0000">{</font>
        <b><font color="#0000FF">if</font></b><font color="#990000">(</font><b><font color="#000000">IsWhitespace</font></b><font color="#990000">(</font>Tokenizer<font color="#990000">-&gt;</font>At<font color="#990000">[</font><font color="#993399">0</font><font color="#990000">]))</font>
        <font color="#FF0000">{</font>
            <font color="#990000">++</font>Tokenizer<font color="#990000">-&gt;</font>At<font color="#990000">;</font>
        <font color="#FF0000">}</font>
        <b><font color="#0000FF">else</font></b> <b><font color="#0000FF">if</font></b><font color="#990000">((</font>Tokenizer<font color="#990000">-&gt;</font>At<font color="#990000">[</font><font color="#993399">0</font><font color="#990000">]</font> <font color="#990000">==</font> <font color="#FF0000">'/'</font><font color="#990000">)</font> <font color="#990000">&amp;&amp;</font>
                <font color="#990000">(</font>Tokenizer<font color="#990000">-&gt;</font>At<font color="#990000">[</font><font color="#993399">1</font><font color="#990000">]</font> <font color="#990000">==</font> <font color="#FF0000">'/'</font><font color="#990000">))</font>
        <font color="#FF0000">{</font>
            Tokenizer<font color="#990000">-&gt;</font>At <font color="#990000">+=</font> <font color="#993399">2</font><font color="#990000">;</font>
            <b><font color="#0000FF">while</font></b><font color="#990000">(</font>Tokenizer<font color="#990000">-&gt;</font>At<font color="#990000">[</font><font color="#993399">0</font><font color="#990000">]</font> <font color="#990000">&amp;&amp;</font> <font color="#990000">!</font><b><font color="#000000">IsEndOfLine</font></b><font color="#990000">(</font>Tokenizer<font color="#990000">-&gt;</font>At<font color="#990000">[</font><font color="#993399">0</font><font color="#990000">]))</font>
            <font color="#FF0000">{</font>
                <font color="#990000">++</font>Tokenizer<font color="#990000">-&gt;</font>At<font color="#990000">;</font>
            <font color="#FF0000">}</font>
        <font color="#FF0000">}</font>
        <b><font color="#0000FF">else</font></b> <b><font color="#0000FF">if</font></b><font color="#990000">((</font>Tokenizer<font color="#990000">-&gt;</font>At<font color="#990000">[</font><font color="#993399">0</font><font color="#990000">]</font> <font color="#990000">==</font> <font color="#FF0000">'/'</font><font color="#990000">)</font> <font color="#990000">&amp;&amp;</font>
                <font color="#990000">(</font>Tokenizer<font color="#990000">-&gt;</font>At<font color="#990000">[</font><font color="#993399">1</font><font color="#990000">]</font> <font color="#990000">==</font> <font color="#FF0000">'*'</font><font color="#990000">))</font>
        <font color="#FF0000">{</font>
            Tokenizer<font color="#990000">-&gt;</font>At <font color="#990000">+=</font> <font color="#993399">2</font><font color="#990000">;</font>
            <b><font color="#0000FF">while</font></b><font color="#990000">(</font>Tokenizer<font color="#990000">-&gt;</font>At<font color="#990000">[</font><font color="#993399">0</font><font color="#990000">]</font> <font color="#990000">&amp;&amp;</font>
                  <font color="#990000">!((</font>Tokenizer<font color="#990000">-&gt;</font>At<font color="#990000">[</font><font color="#993399">0</font><font color="#990000">]</font> <font color="#990000">==</font> <font color="#FF0000">'*'</font><font color="#990000">)</font> <font color="#990000">&amp;&amp;</font>
                    <font color="#990000">(</font>Tokenizer<font color="#990000">-&gt;</font>At<font color="#990000">[</font><font color="#993399">1</font><font color="#990000">]</font> <font color="#990000">==</font> <font color="#FF0000">'/'</font><font color="#990000">)))</font>
            <font color="#FF0000">{</font>
                <font color="#990000">++</font>Tokenizer<font color="#990000">-&gt;</font>At<font color="#990000">;</font>
            <font color="#FF0000">}</font>

            <b><font color="#0000FF">if</font></b><font color="#990000">(</font>Tokenizer<font color="#990000">-&gt;</font>At<font color="#990000">[</font><font color="#993399">0</font><font color="#990000">]</font> <font color="#990000">==</font> <font color="#FF0000">'*'</font><font color="#990000">)</font>
            <font color="#FF0000">{</font>
                Tokenizer<font color="#990000">-&gt;</font>At <font color="#990000">+=</font> <font color="#993399">2</font><font color="#990000">;</font>
            <font color="#FF0000">}</font>
        <font color="#FF0000">}</font>
        <b><font color="#0000FF">else</font></b>
        <font color="#FF0000">{</font>
            <b><font color="#0000FF">break</font></b><font color="#990000">;</font>
        <font color="#FF0000">}</font>
    <font color="#FF0000">}</font>
<font color="#FF0000">}</font>

<b><font color="#0000FF">static</font></b> token
<b><font color="#000000">GetToken</font></b><font color="#990000">(</font><font color="#008080">tokenizer</font> <font color="#990000">*</font>Tokenizer<font color="#990000">)</font>
<font color="#FF0000">{</font>
    <b><font color="#000000">EatAllWhitespace</font></b><font color="#990000">(</font>Tokenizer<font color="#990000">);</font>

    <font color="#008080">token</font> Token <font color="#990000">=</font> <font color="#FF0000">{}</font><font color="#990000">;</font>
    Token<font color="#990000">.</font>TextLength <font color="#990000">=</font> <font color="#993399">1</font><font color="#990000">;</font>
    Token<font color="#990000">.</font>Text <font color="#990000">=</font> Tokenizer<font color="#990000">-&gt;</font>At<font color="#990000">;</font>
    <font color="#009900">char</font> C <font color="#990000">=</font> Tokenizer<font color="#990000">-&gt;</font>At<font color="#990000">[</font><font color="#993399">0</font><font color="#990000">];</font>
    <font color="#990000">++</font>Tokenizer<font color="#990000">-&gt;</font>At<font color="#990000">;</font>
    <b><font color="#0000FF">switch</font></b><font color="#990000">(</font>C<font color="#990000">)</font>
    <font color="#FF0000">{</font>
        <b><font color="#0000FF">case</font></b> <font color="#993399">0</font><font color="#990000">:</font> <font color="#FF0000">{</font><font color="#990000">--</font>Tokenizer<font color="#990000">-&gt;</font>At<font color="#990000">;</font> Token<font color="#990000">.</font>Type <font color="#990000">=</font> Token_EndOfStream<font color="#990000">;</font><font color="#FF0000">}</font> <b><font color="#0000FF">break</font></b><font color="#990000">;</font>

        <b><font color="#0000FF">case</font></b> <font color="#FF0000">'('</font><font color="#990000">:</font> <font color="#FF0000">{</font>Token<font color="#990000">.</font>Type <font color="#990000">=</font> Token_OpenParen<font color="#990000">;</font><font color="#FF0000">}</font> <b><font color="#0000FF">break</font></b><font color="#990000">;</font>
        <b><font color="#0000FF">case</font></b> <font color="#FF0000">')'</font><font color="#990000">:</font> <font color="#FF0000">{</font>Token<font color="#990000">.</font>Type <font color="#990000">=</font> Token_CloseParen<font color="#990000">;</font><font color="#FF0000">}</font> <b><font color="#0000FF">break</font></b><font color="#990000">;</font>
        <b><font color="#0000FF">case</font></b> <font color="#FF0000">'*'</font><font color="#990000">:</font> <font color="#FF0000">{</font>Token<font color="#990000">.</font>Type <font color="#990000">=</font> Token_Asterisk<font color="#990000">;</font><font color="#FF0000">}</font> <b><font color="#0000FF">break</font></b><font color="#990000">;</font>
        <b><font color="#0000FF">case</font></b> <font color="#FF0000">'-'</font><font color="#990000">:</font> <font color="#FF0000">{</font>Token<font color="#990000">.</font>Type <font color="#990000">=</font> Token_Minus<font color="#990000">;</font><font color="#FF0000">}</font> <b><font color="#0000FF">break</font></b><font color="#990000">;</font>
        <b><font color="#0000FF">case</font></b> <font color="#FF0000">'+'</font><font color="#990000">:</font> <font color="#FF0000">{</font>Token<font color="#990000">.</font>Type <font color="#990000">=</font> Token_Plus<font color="#990000">;</font><font color="#FF0000">}</font> <b><font color="#0000FF">break</font></b><font color="#990000">;</font>
        <b><font color="#0000FF">case</font></b> <font color="#FF0000">'/'</font><font color="#990000">:</font> <font color="#FF0000">{</font>Token<font color="#990000">.</font>Type <font color="#990000">=</font> Token_ForwardSlash<font color="#990000">;</font><font color="#FF0000">}</font> <b><font color="#0000FF">break</font></b><font color="#990000">;</font>
        <b><font color="#0000FF">case</font></b> <font color="#FF0000">'%'</font><font color="#990000">:</font> <font color="#FF0000">{</font>Token<font color="#990000">.</font>Type <font color="#990000">=</font> Token_Percent<font color="#990000">;</font><font color="#FF0000">}</font> <b><font color="#0000FF">break</font></b><font color="#990000">;</font>
        <b><font color="#0000FF">case</font></b> <font color="#FF0000">':'</font><font color="#990000">:</font> <font color="#FF0000">{</font>Token<font color="#990000">.</font>Type <font color="#990000">=</font> Token_Colon<font color="#990000">;</font><font color="#FF0000">}</font> <b><font color="#0000FF">break</font></b><font color="#990000">;</font>
        <b><font color="#0000FF">case</font></b> <font color="#FF0000">','</font><font color="#990000">:</font> <font color="#FF0000">{</font>Token<font color="#990000">.</font>Type <font color="#990000">=</font> Token_Comma<font color="#990000">;</font><font color="#FF0000">}</font> <b><font color="#0000FF">break</font></b><font color="#990000">;</font>

<b><font color="#008080">        default:</font></b>
        <font color="#FF0000">{</font>
            <b><font color="#0000FF">if</font></b><font color="#990000">(</font><b><font color="#000000">IsNumeric</font></b><font color="#990000">(</font>C<font color="#990000">))</font>
            <font color="#FF0000">{</font>
                <i><font color="#9A1900">// TODO(casey): Real number</font></i>
                Token<font color="#990000">.</font>Type <font color="#990000">=</font> Token_Number<font color="#990000">;</font>
                <b><font color="#0000FF">while</font></b><font color="#990000">(</font><b><font color="#000000">IsNumeric</font></b><font color="#990000">(</font>Tokenizer<font color="#990000">-&gt;</font>At<font color="#990000">[</font><font color="#993399">0</font><font color="#990000">])</font> <font color="#990000">||</font>
                      <font color="#990000">(</font>Tokenizer<font color="#990000">-&gt;</font>At<font color="#990000">[</font><font color="#993399">0</font><font color="#990000">]</font> <font color="#990000">==</font> <font color="#FF0000">'.'</font><font color="#990000">)</font> <font color="#990000">||</font>
                      <font color="#990000">(</font>Tokenizer<font color="#990000">-&gt;</font>At<font color="#990000">[</font><font color="#993399">0</font><font color="#990000">]</font> <font color="#990000">==</font> <font color="#FF0000">'f'</font><font color="#990000">))</font>
                <font color="#FF0000">{</font>
                    <font color="#990000">++</font>Tokenizer<font color="#990000">-&gt;</font>At<font color="#990000">;</font>
                    Token<font color="#990000">.</font>TextLength <font color="#990000">=</font> Tokenizer<font color="#990000">-&gt;</font>At <font color="#990000">-</font> Token<font color="#990000">.</font>Text<font color="#990000">;</font>
                <font color="#FF0000">}</font>
            <font color="#FF0000">}</font>
            <b><font color="#0000FF">else</font></b>
            <font color="#FF0000">{</font>
                Token<font color="#990000">.</font>Type <font color="#990000">=</font> Token_Unknown<font color="#990000">;</font>
            <font color="#FF0000">}</font>
        <font color="#FF0000">}</font> <b><font color="#0000FF">break</font></b><font color="#990000">;</font>        
    <font color="#FF0000">}</font>

    <b><font color="#0000FF">return</font></b><font color="#990000">(</font>Token<font color="#990000">);</font>
<font color="#FF0000">}</font>

<b><font color="#0000FF">static</font></b> token
<b><font color="#000000">PeekToken</font></b><font color="#990000">(</font><font color="#008080">tokenizer</font> <font color="#990000">*</font>Tokenizer<font color="#990000">)</font>
<font color="#FF0000">{</font>
    <font color="#008080">tokenizer</font> Tokenizer2 <font color="#990000">=</font> <font color="#990000">*</font>Tokenizer<font color="#990000">;</font>
    <font color="#008080">token</font> Result <font color="#990000">=</font> <b><font color="#000000">GetToken</font></b><font color="#990000">(&amp;</font>Tokenizer2<font color="#990000">);</font>
    <b><font color="#0000FF">return</font></b><font color="#990000">(</font>Result<font color="#990000">);</font>
<font color="#FF0000">}</font>

<b><font color="#0000FF">inline</font></b> <font color="#009900">bool</font>
<b><font color="#000000">IsH</font></b><font color="#990000">(</font><font color="#008080">String</font> extension<font color="#990000">)</font>
<font color="#FF0000">{</font>
    <font color="#009900">bool</font> Result <font color="#990000">=</font> <font color="#990000">(</font><b><font color="#000000">match</font></b><font color="#990000">(</font>extension<font color="#990000">,</font> <b><font color="#000000">make_lit_string</font></b><font color="#990000">(</font><font color="#FF0000">"h"</font><font color="#990000">))</font> <font color="#990000">||</font>
                   <b><font color="#000000">match</font></b><font color="#990000">(</font>extension<font color="#990000">,</font> <b><font color="#000000">make_lit_string</font></b><font color="#990000">(</font><font color="#FF0000">"hpp"</font><font color="#990000">))</font> <font color="#990000">||</font>
                   <b><font color="#000000">match</font></b><font color="#990000">(</font>extension<font color="#990000">,</font> <b><font color="#000000">make_lit_string</font></b><font color="#990000">(</font><font color="#FF0000">"hin"</font><font color="#990000">)));</font>

    <b><font color="#0000FF">return</font></b><font color="#990000">(</font>Result<font color="#990000">);</font>
<font color="#FF0000">}</font>

<b><font color="#0000FF">inline</font></b> <font color="#009900">bool</font>
<b><font color="#000000">IsCPP</font></b><font color="#990000">(</font><font color="#008080">String</font> extension<font color="#990000">)</font>
<font color="#FF0000">{</font>
    <font color="#009900">bool</font> Result <font color="#990000">=</font> <font color="#990000">(</font><b><font color="#000000">match</font></b><font color="#990000">(</font>extension<font color="#990000">,</font> <b><font color="#000000">make_lit_string</font></b><font color="#990000">(</font><font color="#FF0000">"c"</font><font color="#990000">))</font> <font color="#990000">||</font>
                   <b><font color="#000000">match</font></b><font color="#990000">(</font>extension<font color="#990000">,</font> <b><font color="#000000">make_lit_string</font></b><font color="#990000">(</font><font color="#FF0000">"cpp"</font><font color="#990000">))</font> <font color="#990000">||</font>
                   <b><font color="#000000">match</font></b><font color="#990000">(</font>extension<font color="#990000">,</font> <b><font color="#000000">make_lit_string</font></b><font color="#990000">(</font><font color="#FF0000">"cin"</font><font color="#990000">)));</font>

    <b><font color="#0000FF">return</font></b><font color="#990000">(</font>Result<font color="#990000">);</font>
<font color="#FF0000">}</font>

<b><font color="#0000FF">inline</font></b> <font color="#009900">bool</font>
<b><font color="#000000">IsINL</font></b><font color="#990000">(</font><font color="#008080">String</font> extension<font color="#990000">)</font>
<font color="#FF0000">{</font>
    <font color="#009900">bool</font> Result <font color="#990000">=</font> <font color="#990000">(</font><b><font color="#000000">match</font></b><font color="#990000">(</font>extension<font color="#990000">,</font> <b><font color="#000000">make_lit_string</font></b><font color="#990000">(</font><font color="#FF0000">"inl"</font><font color="#990000">)));</font>

    <b><font color="#0000FF">return</font></b><font color="#990000">(</font>Result<font color="#990000">);</font>
<font color="#FF0000">}</font>

<b><font color="#0000FF">inline</font></b> <font color="#009900">bool</font>
<b><font color="#000000">IsCode</font></b><font color="#990000">(</font><font color="#008080">String</font> extension<font color="#990000">)</font>
<font color="#FF0000">{</font>
    <font color="#009900">bool</font> Result <font color="#990000">=</font> <font color="#990000">(</font><b><font color="#000000">IsH</font></b><font color="#990000">(</font>extension<font color="#990000">)</font> <font color="#990000">||</font> <b><font color="#000000">IsCPP</font></b><font color="#990000">(</font>extension<font color="#990000">)</font> <font color="#990000">||</font> <b><font color="#000000">IsINL</font></b><font color="#990000">(</font>extension<font color="#990000">));</font>

    <b><font color="#0000FF">return</font></b><font color="#990000">(</font>Result<font color="#990000">);</font>
<font color="#FF0000">}</font>


<b><font color="#000000">CUSTOM_COMMAND_SIG</font></b><font color="#990000">(</font>casey_open_in_other<font color="#990000">)</font>
<font color="#FF0000">{</font>
    <b><font color="#000000">exec_command</font></b><font color="#990000">(</font>app<font color="#990000">,</font> cmdid_change_active_panel<font color="#990000">);</font>
    <b><font color="#000000">exec_command</font></b><font color="#990000">(</font>app<font color="#990000">,</font> cmdid_interactive_open<font color="#990000">);</font>
<font color="#FF0000">}</font>

<b><font color="#000000">CUSTOM_COMMAND_SIG</font></b><font color="#990000">(</font>casey_clean_and_save<font color="#990000">)</font>
<font color="#FF0000">{</font>
    <b><font color="#000000">exec_command</font></b><font color="#990000">(</font>app<font color="#990000">,</font> cmdid_clean_all_lines<font color="#990000">);</font>
    <b><font color="#000000">exec_command</font></b><font color="#990000">(</font>app<font color="#990000">,</font> cmdid_eol_nixify<font color="#990000">);</font>
    <b><font color="#000000">exec_command</font></b><font color="#990000">(</font>app<font color="#990000">,</font> cmdid_save<font color="#990000">);</font>
<font color="#FF0000">}</font>

<b><font color="#000000">CUSTOM_COMMAND_SIG</font></b><font color="#990000">(</font>casey_newline_and_indent<font color="#990000">)</font>
<font color="#FF0000">{</font>
    <b><font color="#000000">exec_command</font></b><font color="#990000">(</font>app<font color="#990000">,</font> cmdid_write_character<font color="#990000">);</font>
    <b><font color="#000000">exec_command</font></b><font color="#990000">(</font>app<font color="#990000">,</font> auto_tab_line_at_cursor<font color="#990000">);</font>
<font color="#FF0000">}</font>

<b><font color="#000000">CUSTOM_COMMAND_SIG</font></b><font color="#990000">(</font>casey_open_file_other_window<font color="#990000">)</font>
<font color="#FF0000">{</font>
    <b><font color="#000000">exec_command</font></b><font color="#990000">(</font>app<font color="#990000">,</font> cmdid_change_active_panel<font color="#990000">);</font>
    <b><font color="#000000">exec_command</font></b><font color="#990000">(</font>app<font color="#990000">,</font> cmdid_interactive_open<font color="#990000">);</font>
<font color="#FF0000">}</font>

<b><font color="#000000">CUSTOM_COMMAND_SIG</font></b><font color="#990000">(</font>casey_switch_buffer_other_window<font color="#990000">)</font>
<font color="#FF0000">{</font>
    <b><font color="#000000">exec_command</font></b><font color="#990000">(</font>app<font color="#990000">,</font> cmdid_change_active_panel<font color="#990000">);</font>
    <b><font color="#000000">exec_command</font></b><font color="#990000">(</font>app<font color="#990000">,</font> cmdid_interactive_switch_buffer<font color="#990000">);</font>
<font color="#FF0000">}</font>

internal <font color="#009900">void</font>
<b><font color="#000000">DeleteAfterCommand</font></b><font color="#990000">(</font><b><font color="#0000FF">struct</font></b> <font color="#008080">Application_Links</font> <font color="#990000">*</font>app<font color="#990000">,</font> <font color="#009900">unsigned</font> <font color="#009900">long</font> <font color="#009900">long</font> CommandID<font color="#990000">)</font>
<font color="#FF0000">{</font>
    <font color="#008080">View_Summary</font> view <font color="#990000">=</font> app<font color="#990000">-&gt;</font><b><font color="#000000">get_active_view</font></b><font color="#990000">(</font>app<font color="#990000">);</font>

    <font color="#009900">int</font> pos2 <font color="#990000">=</font> view<font color="#990000">.</font>cursor<font color="#990000">.</font>pos<font color="#990000">;</font>
    <b><font color="#0000FF">if</font></b> <font color="#990000">(</font>CommandID <font color="#990000">&lt;</font> cmdid_count<font color="#990000">)</font><font color="#FF0000">{</font>
        <b><font color="#000000">exec_command</font></b><font color="#990000">(</font>app<font color="#990000">,</font> <font color="#990000">(</font>Command_ID<font color="#990000">)</font>CommandID<font color="#990000">);</font>
    <font color="#FF0000">}</font>
    <b><font color="#0000FF">else</font></b><font color="#FF0000">{</font>
        <b><font color="#000000">exec_command</font></b><font color="#990000">(</font>app<font color="#990000">,</font> <font color="#990000">(</font>Custom_Command_Function<font color="#990000">*)</font>CommandID<font color="#990000">);</font>
    <font color="#FF0000">}</font>
    app<font color="#990000">-&gt;</font><b><font color="#000000">refresh_view</font></b><font color="#990000">(</font>app<font color="#990000">,</font> <font color="#990000">&amp;</font>view<font color="#990000">);</font>
    <font color="#009900">int</font> pos1 <font color="#990000">=</font> view<font color="#990000">.</font>cursor<font color="#990000">.</font>pos<font color="#990000">;</font>

    <font color="#008080">Range</font> range <font color="#990000">=</font> <b><font color="#000000">make_range</font></b><font color="#990000">(</font>pos1<font color="#990000">,</font> pos2<font color="#990000">);</font>

    <font color="#008080">Buffer_Summary</font> buffer <font color="#990000">=</font> app<font color="#990000">-&gt;</font><b><font color="#000000">get_buffer</font></b><font color="#990000">(</font>app<font color="#990000">,</font> view<font color="#990000">.</font>buffer_id<font color="#990000">);</font>
    app<font color="#990000">-&gt;</font><b><font color="#000000">buffer_replace_range</font></b><font color="#990000">(</font>app<font color="#990000">,</font> <font color="#990000">&amp;</font>buffer<font color="#990000">,</font> range<font color="#990000">.</font>min<font color="#990000">,</font> range<font color="#990000">.</font>max<font color="#990000">,</font> <font color="#993399">0</font><font color="#990000">,</font> <font color="#993399">0</font><font color="#990000">);</font>
<font color="#FF0000">}</font>

<b><font color="#000000">CUSTOM_COMMAND_SIG</font></b><font color="#990000">(</font>casey_delete_token_left<font color="#990000">)</font>
<font color="#FF0000">{</font>
    <b><font color="#000000">DeleteAfterCommand</font></b><font color="#990000">(</font>app<font color="#990000">,</font> <font color="#990000">(</font><font color="#009900">unsigned</font> <font color="#009900">long</font> <font color="#009900">long</font><font color="#990000">)</font>seek_white_or_token_left<font color="#990000">);</font>
<font color="#FF0000">}</font>

<b><font color="#000000">CUSTOM_COMMAND_SIG</font></b><font color="#990000">(</font>casey_delete_token_right<font color="#990000">)</font>
<font color="#FF0000">{</font>
    <b><font color="#000000">DeleteAfterCommand</font></b><font color="#990000">(</font>app<font color="#990000">,</font> <font color="#990000">(</font><font color="#009900">unsigned</font> <font color="#009900">long</font> <font color="#009900">long</font><font color="#990000">)</font>seek_white_or_token_right<font color="#990000">);</font>
<font color="#FF0000">}</font>

<b><font color="#000000">CUSTOM_COMMAND_SIG</font></b><font color="#990000">(</font>casey_kill_to_end_of_line<font color="#990000">)</font>
<font color="#FF0000">{</font>
    <font color="#008080">View_Summary</font> view <font color="#990000">=</font> app<font color="#990000">-&gt;</font><b><font color="#000000">get_active_view</font></b><font color="#990000">(</font>app<font color="#990000">);</font>

    <font color="#009900">int</font> pos2 <font color="#990000">=</font> view<font color="#990000">.</font>cursor<font color="#990000">.</font>pos<font color="#990000">;</font>
    <b><font color="#000000">exec_command</font></b><font color="#990000">(</font>app<font color="#990000">,</font> cmdid_seek_end_of_line<font color="#990000">);</font>
    app<font color="#990000">-&gt;</font><b><font color="#000000">refresh_view</font></b><font color="#990000">(</font>app<font color="#990000">,</font> <font color="#990000">&amp;</font>view<font color="#990000">);</font>
    <font color="#009900">int</font> pos1 <font color="#990000">=</font> view<font color="#990000">.</font>cursor<font color="#990000">.</font>pos<font color="#990000">;</font>

    <font color="#008080">Range</font> range <font color="#990000">=</font> <b><font color="#000000">make_range</font></b><font color="#990000">(</font>pos1<font color="#990000">,</font> pos2<font color="#990000">);</font>
    <b><font color="#0000FF">if</font></b><font color="#990000">(</font>pos1 <font color="#990000">==</font> pos2<font color="#990000">)</font>
    <font color="#FF0000">{</font>
        range<font color="#990000">.</font>max <font color="#990000">+=</font> <font color="#993399">1</font><font color="#990000">;</font>
    <font color="#FF0000">}</font>

    <font color="#008080">Buffer_Summary</font> buffer <font color="#990000">=</font> app<font color="#990000">-&gt;</font><b><font color="#000000">get_buffer</font></b><font color="#990000">(</font>app<font color="#990000">,</font> view<font color="#990000">.</font>buffer_id<font color="#990000">);</font>
    app<font color="#990000">-&gt;</font><b><font color="#000000">buffer_replace_range</font></b><font color="#990000">(</font>app<font color="#990000">,</font> <font color="#990000">&amp;</font>buffer<font color="#990000">,</font> range<font color="#990000">.</font>min<font color="#990000">,</font> range<font color="#990000">.</font>max<font color="#990000">,</font> <font color="#993399">0</font><font color="#990000">,</font> <font color="#993399">0</font><font color="#990000">);</font>
    <b><font color="#000000">exec_command</font></b><font color="#990000">(</font>app<font color="#990000">,</font> auto_tab_line_at_cursor<font color="#990000">);</font>
<font color="#FF0000">}</font>

<b><font color="#000000">CUSTOM_COMMAND_SIG</font></b><font color="#990000">(</font>casey_paste_and_tab<font color="#990000">)</font>
<font color="#FF0000">{</font>
    <i><font color="#9A1900">// NOTE(allen): Paste puts the mark at the beginning and the cursor at</font></i>
    <i><font color="#9A1900">// the end of the pasted chunk, so it is all set for cmdid_auto_tab_range</font></i>
    <b><font color="#000000">exec_command</font></b><font color="#990000">(</font>app<font color="#990000">,</font> cmdid_paste<font color="#990000">);</font>
    <b><font color="#000000">exec_command</font></b><font color="#990000">(</font>app<font color="#990000">,</font> cmdid_auto_tab_range<font color="#990000">);</font>
<font color="#FF0000">}</font>

<b><font color="#000000">CUSTOM_COMMAND_SIG</font></b><font color="#990000">(</font>casey_seek_beginning_of_line_and_tab<font color="#990000">)</font>
<font color="#FF0000">{</font>
    <b><font color="#000000">exec_command</font></b><font color="#990000">(</font>app<font color="#990000">,</font> cmdid_seek_beginning_of_line<font color="#990000">);</font>
    <b><font color="#000000">exec_command</font></b><font color="#990000">(</font>app<font color="#990000">,</font> auto_tab_line_at_cursor<font color="#990000">);</font>
<font color="#FF0000">}</font>

<b><font color="#000000">CUSTOM_COMMAND_SIG</font></b><font color="#990000">(</font>casey_seek_beginning_of_line<font color="#990000">)</font>
<font color="#FF0000">{</font>
    <b><font color="#000000">exec_command</font></b><font color="#990000">(</font>app<font color="#990000">,</font> auto_tab_line_at_cursor<font color="#990000">);</font>
    <b><font color="#000000">exec_command</font></b><font color="#990000">(</font>app<font color="#990000">,</font> cmdid_seek_beginning_of_line<font color="#990000">);</font>
<font color="#FF0000">}</font>

<b><font color="#0000FF">struct</font></b> <font color="#008080">switch_to_result</font>
<font color="#FF0000">{</font>
    <font color="#009900">bool</font> Switched<font color="#990000">;</font>
    <font color="#009900">bool</font> Loaded<font color="#990000">;</font>
    <font color="#008080">View_Summary</font> view<font color="#990000">;</font>
    <font color="#008080">Buffer_Summary</font> buffer<font color="#990000">;</font>
<font color="#FF0000">}</font><font color="#990000">;</font>

<b><font color="#0000FF">inline</font></b> <font color="#009900">void</font>
<b><font color="#000000">SanitizeSlashes</font></b><font color="#990000">(</font><font color="#008080">String</font> Value<font color="#990000">)</font>
<font color="#FF0000">{</font>
    <b><font color="#0000FF">for</font></b><font color="#990000">(</font><font color="#009900">int</font> At <font color="#990000">=</font> <font color="#993399">0</font><font color="#990000">;</font>
        At <font color="#990000">&lt;</font> Value<font color="#990000">.</font>size<font color="#990000">;</font>
        <font color="#990000">++</font>At<font color="#990000">)</font>
    <font color="#FF0000">{</font>
        <b><font color="#0000FF">if</font></b><font color="#990000">(</font>Value<font color="#990000">.</font>str<font color="#990000">[</font>At<font color="#990000">]</font> <font color="#990000">==</font> <font color="#FF0000">'</font><font color="#CC33CC">\\</font><font color="#FF0000">'</font><font color="#990000">)</font>
        <font color="#FF0000">{</font>
            Value<font color="#990000">.</font>str<font color="#990000">[</font>At<font color="#990000">]</font> <font color="#990000">=</font> <font color="#FF0000">'/'</font><font color="#990000">;</font>
        <font color="#FF0000">}</font>
    <font color="#FF0000">}</font>
<font color="#FF0000">}</font>

<b><font color="#0000FF">inline</font></b> switch_to_result
<b><font color="#000000">SwitchToOrLoadFile</font></b><font color="#990000">(</font><b><font color="#0000FF">struct</font></b> <font color="#008080">Application_Links</font> <font color="#990000">*</font>app<font color="#990000">,</font> <font color="#008080">String</font> FileName<font color="#990000">,</font> <font color="#009900">bool</font> CreateIfNotFound <font color="#990000">=</font> <b><font color="#0000FF">false</font></b><font color="#990000">)</font>
<font color="#FF0000">{</font>
    <font color="#008080">switch_to_result</font> Result <font color="#990000">=</font> <font color="#FF0000">{}</font><font color="#990000">;</font>

    <b><font color="#000000">SanitizeSlashes</font></b><font color="#990000">(</font>FileName<font color="#990000">);</font>

    <font color="#008080">View_Summary</font> view <font color="#990000">=</font> app<font color="#990000">-&gt;</font><b><font color="#000000">get_active_view</font></b><font color="#990000">(</font>app<font color="#990000">);</font>
    <font color="#008080">Buffer_Summary</font> buffer <font color="#990000">=</font> app<font color="#990000">-&gt;</font><b><font color="#000000">get_buffer_by_name</font></b><font color="#990000">(</font>app<font color="#990000">,</font> FileName<font color="#990000">.</font>str<font color="#990000">,</font> FileName<font color="#990000">.</font>size<font color="#990000">);</font>

    Result<font color="#990000">.</font>view <font color="#990000">=</font> view<font color="#990000">;</font>
    Result<font color="#990000">.</font>buffer <font color="#990000">=</font> buffer<font color="#990000">;</font>

    <b><font color="#0000FF">if</font></b><font color="#990000">(</font>buffer<font color="#990000">.</font>exists<font color="#990000">)</font>
    <font color="#FF0000">{</font>
        app<font color="#990000">-&gt;</font><b><font color="#000000">view_set_buffer</font></b><font color="#990000">(</font>app<font color="#990000">,</font> <font color="#990000">&amp;</font>view<font color="#990000">,</font> buffer<font color="#990000">.</font>buffer_id<font color="#990000">);</font>
        Result<font color="#990000">.</font>Switched <font color="#990000">=</font> <b><font color="#0000FF">true</font></b><font color="#990000">;</font>
    <font color="#FF0000">}</font>
    <b><font color="#0000FF">else</font></b>
    <font color="#FF0000">{</font>
        <b><font color="#0000FF">if</font></b><font color="#990000">(</font>app<font color="#990000">-&gt;</font><b><font color="#000000">file_exists</font></b><font color="#990000">(</font>app<font color="#990000">,</font> FileName<font color="#990000">.</font>str<font color="#990000">,</font> FileName<font color="#990000">.</font>size<font color="#990000">)</font> <font color="#990000">||</font> CreateIfNotFound<font color="#990000">)</font>
        <font color="#FF0000">{</font>
            <b><font color="#000000">push_parameter</font></b><font color="#990000">(</font>app<font color="#990000">,</font> par_name<font color="#990000">,</font> <b><font color="#000000">expand_str</font></b><font color="#990000">(</font>FileName<font color="#990000">));</font>
            <i><font color="#9A1900">// TODO(casey): Do I have to check for existence, or can I pass a parameter</font></i>
            <i><font color="#9A1900">// to interactive open to tell it to fail if the file isn't there?</font></i>
            <b><font color="#000000">exec_command</font></b><font color="#990000">(</font>app<font color="#990000">,</font> cmdid_interactive_open<font color="#990000">);</font>

            Result<font color="#990000">.</font>buffer <font color="#990000">=</font> app<font color="#990000">-&gt;</font><b><font color="#000000">get_buffer_by_name</font></b><font color="#990000">(</font>app<font color="#990000">,</font> FileName<font color="#990000">.</font>str<font color="#990000">,</font> FileName<font color="#990000">.</font>size<font color="#990000">);</font>            

            Result<font color="#990000">.</font>Loaded <font color="#990000">=</font> <b><font color="#0000FF">true</font></b><font color="#990000">;</font>
            Result<font color="#990000">.</font>Switched <font color="#990000">=</font> <b><font color="#0000FF">true</font></b><font color="#990000">;</font>
        <font color="#FF0000">}</font>
    <font color="#FF0000">}</font>

    <b><font color="#0000FF">return</font></b><font color="#990000">(</font>Result<font color="#990000">);</font>
<font color="#FF0000">}</font>

<b><font color="#000000">CUSTOM_COMMAND_SIG</font></b><font color="#990000">(</font>casey_load_todo<font color="#990000">)</font>
<font color="#FF0000">{</font>
    <font color="#008080">String</font> ToDoFileName <font color="#990000">=</font> <b><font color="#000000">make_lit_string</font></b><font color="#990000">(</font><font color="#FF0000">"w:/handmade/code/todo.txt"</font><font color="#990000">);</font>
    <b><font color="#000000">SwitchToOrLoadFile</font></b><font color="#990000">(</font>app<font color="#990000">,</font> ToDoFileName<font color="#990000">,</font> <b><font color="#0000FF">true</font></b><font color="#990000">);</font>
<font color="#FF0000">}</font>

<b><font color="#000000">CUSTOM_COMMAND_SIG</font></b><font color="#990000">(</font>casey_build_search<font color="#990000">)</font>
<font color="#FF0000">{</font>
    <font color="#009900">int</font> keep_going <font color="#990000">=</font> <font color="#993399">1</font><font color="#990000">;</font>
    <font color="#009900">int</font> old_size<font color="#990000">;</font>
    <i><font color="#9A1900">// TODO(allen): It's fine to get memory this way for now, eventually</font></i>
    <i><font color="#9A1900">// we should properly suballocating from app-&gt;memory.</font></i>
    <font color="#008080">String</font> dir <font color="#990000">=</font> <b><font color="#000000">make_string</font></b><font color="#990000">(</font>app<font color="#990000">-&gt;</font>memory<font color="#990000">,</font> <font color="#993399">0</font><font color="#990000">,</font> app<font color="#990000">-&gt;</font>memory_size<font color="#990000">);</font>
    dir<font color="#990000">.</font>size <font color="#990000">=</font> app<font color="#990000">-&gt;</font><b><font color="#000000">directory_get_hot</font></b><font color="#990000">(</font>app<font color="#990000">,</font> dir<font color="#990000">.</font>str<font color="#990000">,</font> dir<font color="#990000">.</font>memory_size<font color="#990000">);</font>

    <b><font color="#0000FF">while</font></b> <font color="#990000">(</font>keep_going<font color="#990000">)</font>
    <font color="#FF0000">{</font>
        old_size <font color="#990000">=</font> dir<font color="#990000">.</font>size<font color="#990000">;</font>
        <b><font color="#000000">append</font></b><font color="#990000">(&amp;</font>dir<font color="#990000">,</font> <font color="#FF0000">"build.bat"</font><font color="#990000">);</font>

        <b><font color="#0000FF">if</font></b> <font color="#990000">(</font>app<font color="#990000">-&gt;</font><b><font color="#000000">file_exists</font></b><font color="#990000">(</font>app<font color="#990000">,</font> dir<font color="#990000">.</font>str<font color="#990000">,</font> dir<font color="#990000">.</font>size<font color="#990000">))</font>
        <font color="#FF0000">{</font>
            dir<font color="#990000">.</font>size <font color="#990000">=</font> old_size<font color="#990000">;</font>
            <b><font color="#000000">memcpy</font></b><font color="#990000">(</font>BuildDirectory<font color="#990000">,</font> dir<font color="#990000">.</font>str<font color="#990000">,</font> dir<font color="#990000">.</font>size<font color="#990000">);</font>
            BuildDirectory<font color="#990000">[</font>dir<font color="#990000">.</font>size<font color="#990000">]</font> <font color="#990000">=</font> <font color="#993399">0</font><font color="#990000">;</font>

            <b><font color="#0000FF">return</font></b><font color="#990000">;</font>
        <font color="#FF0000">}</font>

        dir<font color="#990000">.</font>size <font color="#990000">=</font> old_size<font color="#990000">;</font>

        <b><font color="#0000FF">if</font></b> <font color="#990000">(</font>app<font color="#990000">-&gt;</font><b><font color="#000000">directory_cd</font></b><font color="#990000">(</font>app<font color="#990000">,</font> dir<font color="#990000">.</font>str<font color="#990000">,</font> <font color="#990000">&amp;</font>dir<font color="#990000">.</font>size<font color="#990000">,</font> dir<font color="#990000">.</font>memory_size<font color="#990000">,</font> <b><font color="#000000">literal</font></b><font color="#990000">(</font><font color="#FF0000">".."</font><font color="#990000">))</font> <font color="#990000">==</font> <font color="#993399">0</font><font color="#990000">)</font>
        <font color="#FF0000">{</font>
            keep_going <font color="#990000">=</font> <font color="#993399">0</font><font color="#990000">;</font>
        <font color="#FF0000">}</font>
    <font color="#FF0000">}</font>

    <i><font color="#9A1900">// TODO(casey): How do I print out that it found or didn't find something?</font></i>
<font color="#FF0000">}</font>

<b><font color="#000000">CUSTOM_COMMAND_SIG</font></b><font color="#990000">(</font>casey_find_corresponding_file<font color="#990000">)</font>
<font color="#FF0000">{</font>
    <font color="#008080">View_Summary</font> view <font color="#990000">=</font> app<font color="#990000">-&gt;</font><b><font color="#000000">get_active_view</font></b><font color="#990000">(</font>app<font color="#990000">);</font>
    <font color="#008080">Buffer_Summary</font> buffer <font color="#990000">=</font> app<font color="#990000">-&gt;</font><b><font color="#000000">get_buffer</font></b><font color="#990000">(</font>app<font color="#990000">,</font> view<font color="#990000">.</font>buffer_id<font color="#990000">);</font>

    <font color="#008080">String</font> extension <font color="#990000">=</font> <b><font color="#000000">file_extension</font></b><font color="#990000">(</font><b><font color="#000000">make_string</font></b><font color="#990000">(</font>buffer<font color="#990000">.</font>file_name<font color="#990000">,</font> buffer<font color="#990000">.</font>file_name_len<font color="#990000">));</font>
    <b><font color="#0000FF">if</font></b> <font color="#990000">(</font>extension<font color="#990000">.</font>str<font color="#990000">)</font>
    <font color="#FF0000">{</font>       
        <font color="#009900">char</font> <font color="#990000">*</font>HExtensions<font color="#990000">[]</font> <font color="#990000">=</font>
        <font color="#FF0000">{</font>
            <font color="#FF0000">"hpp"</font><font color="#990000">,</font>
            <font color="#FF0000">"hin"</font><font color="#990000">,</font>
            <font color="#FF0000">"h"</font><font color="#990000">,</font>
        <font color="#FF0000">}</font><font color="#990000">;</font>

        <font color="#009900">char</font> <font color="#990000">*</font>CExtensions<font color="#990000">[]</font> <font color="#990000">=</font>
        <font color="#FF0000">{</font>
            <font color="#FF0000">"c"</font><font color="#990000">,</font>
            <font color="#FF0000">"cin"</font><font color="#990000">,</font>
            <font color="#FF0000">"cpp"</font><font color="#990000">,</font>
        <font color="#FF0000">}</font><font color="#990000">;</font>

        <font color="#009900">int</font> ExtensionCount <font color="#990000">=</font> <font color="#993399">0</font><font color="#990000">;</font>
        <font color="#009900">char</font> <font color="#990000">**</font>Extensions <font color="#990000">=</font> <font color="#993399">0</font><font color="#990000">;</font>
        <b><font color="#0000FF">if</font></b><font color="#990000">(</font><b><font color="#000000">IsH</font></b><font color="#990000">(</font>extension<font color="#990000">))</font>
        <font color="#FF0000">{</font>
            ExtensionCount <font color="#990000">=</font> <b><font color="#000000">ArrayCount</font></b><font color="#990000">(</font>CExtensions<font color="#990000">);</font>
            Extensions <font color="#990000">=</font> CExtensions<font color="#990000">;</font>
        <font color="#FF0000">}</font>
        <b><font color="#0000FF">else</font></b> <b><font color="#0000FF">if</font></b><font color="#990000">(</font><b><font color="#000000">IsCPP</font></b><font color="#990000">(</font>extension<font color="#990000">)</font> <font color="#990000">||</font> <b><font color="#000000">IsINL</font></b><font color="#990000">(</font>extension<font color="#990000">))</font>
        <font color="#FF0000">{</font>
            ExtensionCount <font color="#990000">=</font> <b><font color="#000000">ArrayCount</font></b><font color="#990000">(</font>HExtensions<font color="#990000">);</font>
            Extensions <font color="#990000">=</font> HExtensions<font color="#990000">;</font>
        <font color="#FF0000">}</font>

        <font color="#009900">int</font> MaxExtensionLength <font color="#990000">=</font> <font color="#993399">3</font><font color="#990000">;</font>
        <font color="#009900">int</font> Space <font color="#990000">=</font> <font color="#990000">(</font><font color="#009900">int</font><font color="#990000">)(</font>buffer<font color="#990000">.</font>file_name_len <font color="#990000">+</font> MaxExtensionLength<font color="#990000">);</font>
        <font color="#008080">String</font> FileNameStem <font color="#990000">=</font> <b><font color="#000000">make_string</font></b><font color="#990000">(</font>buffer<font color="#990000">.</font>file_name<font color="#990000">,</font> <font color="#990000">(</font><font color="#009900">int</font><font color="#990000">)(</font>extension<font color="#990000">.</font>str <font color="#990000">-</font> buffer<font color="#990000">.</font>file_name<font color="#990000">),</font> <font color="#993399">0</font><font color="#990000">);</font>
        <font color="#008080">String</font> TestFileName <font color="#990000">=</font> <b><font color="#000000">make_string</font></b><font color="#990000">(</font>app<font color="#990000">-&gt;</font><b><font color="#000000">push_memory</font></b><font color="#990000">(</font>app<font color="#990000">,</font> Space<font color="#990000">),</font> <font color="#993399">0</font><font color="#990000">,</font> Space<font color="#990000">);</font>   
        <b><font color="#0000FF">for</font></b><font color="#990000">(</font><font color="#009900">int</font> ExtensionIndex <font color="#990000">=</font> <font color="#993399">0</font><font color="#990000">;</font>
            ExtensionCount<font color="#990000">;</font>
            <font color="#990000">++</font>ExtensionIndex<font color="#990000">)</font>
        <font color="#FF0000">{</font>
            TestFileName<font color="#990000">.</font>size <font color="#990000">=</font> <font color="#993399">0</font><font color="#990000">;</font>
            <b><font color="#000000">append</font></b><font color="#990000">(&amp;</font>TestFileName<font color="#990000">,</font> FileNameStem<font color="#990000">);</font>
            <b><font color="#000000">append</font></b><font color="#990000">(&amp;</font>TestFileName<font color="#990000">,</font> Extensions<font color="#990000">[</font>ExtensionIndex<font color="#990000">]);</font>

            <b><font color="#0000FF">if</font></b><font color="#990000">(</font><b><font color="#000000">SwitchToOrLoadFile</font></b><font color="#990000">(</font>app<font color="#990000">,</font> TestFileName<font color="#990000">,</font> <font color="#990000">((</font>ExtensionIndex <font color="#990000">+</font> <font color="#993399">1</font><font color="#990000">)</font> <font color="#990000">==</font> ExtensionCount<font color="#990000">)).</font>Switched<font color="#990000">)</font>
            <font color="#FF0000">{</font>
                <b><font color="#0000FF">break</font></b><font color="#990000">;</font>
            <font color="#FF0000">}</font>
        <font color="#FF0000">}</font>
    <font color="#FF0000">}</font>
<font color="#FF0000">}</font>

<b><font color="#000000">CUSTOM_COMMAND_SIG</font></b><font color="#990000">(</font>casey_find_corresponding_file_other_window<font color="#990000">)</font>
<font color="#FF0000">{</font>
    <font color="#008080">View_Summary</font> old_view <font color="#990000">=</font> app<font color="#990000">-&gt;</font><b><font color="#000000">get_active_view</font></b><font color="#990000">(</font>app<font color="#990000">);</font>
    <font color="#008080">Buffer_Summary</font> buffer <font color="#990000">=</font> app<font color="#990000">-&gt;</font><b><font color="#000000">get_buffer</font></b><font color="#990000">(</font>app<font color="#990000">,</font> old_view<font color="#990000">.</font>buffer_id<font color="#990000">);</font>

    <b><font color="#000000">exec_command</font></b><font color="#990000">(</font>app<font color="#990000">,</font> cmdid_change_active_panel<font color="#990000">);</font>
    <font color="#008080">View_Summary</font> new_view <font color="#990000">=</font> app<font color="#990000">-&gt;</font><b><font color="#000000">get_active_view</font></b><font color="#990000">(</font>app<font color="#990000">);</font>
    app<font color="#990000">-&gt;</font><b><font color="#000000">view_set_buffer</font></b><font color="#990000">(</font>app<font color="#990000">,</font> <font color="#990000">&amp;</font>new_view<font color="#990000">,</font> buffer<font color="#990000">.</font>buffer_id<font color="#990000">);</font>

<i><font color="#9A1900">//    exec_command(app, casey_find_corresponding_file);</font></i>
<font color="#FF0000">}</font>

<b><font color="#000000">CUSTOM_COMMAND_SIG</font></b><font color="#990000">(</font>casey_save_and_make_without_asking<font color="#990000">)</font>
<font color="#FF0000">{</font>
    <b><font color="#000000">exec_command</font></b><font color="#990000">(</font>app<font color="#990000">,</font> cmdid_change_active_panel<font color="#990000">);</font>

    <font color="#008080">Buffer_Summary</font> buffer <font color="#990000">=</font> <font color="#FF0000">{}</font><font color="#990000">;</font>

    <b><font color="#0000FF">for</font></b><font color="#990000">(</font>buffer <font color="#990000">=</font> app<font color="#990000">-&gt;</font><b><font color="#000000">get_buffer_first</font></b><font color="#990000">(</font>app<font color="#990000">);</font>
        buffer<font color="#990000">.</font>exists<font color="#990000">;</font>
        app<font color="#990000">-&gt;</font><b><font color="#000000">get_buffer_next</font></b><font color="#990000">(</font>app<font color="#990000">,</font> <font color="#990000">&amp;</font>buffer<font color="#990000">))</font>
    <font color="#FF0000">{</font>
        <b><font color="#000000">push_parameter</font></b><font color="#990000">(</font>app<font color="#990000">,</font> par_name<font color="#990000">,</font> buffer<font color="#990000">.</font>file_name<font color="#990000">,</font> buffer<font color="#990000">.</font>file_name_len<font color="#990000">);</font>
        <b><font color="#000000">push_parameter</font></b><font color="#990000">(</font>app<font color="#990000">,</font> par_buffer_id<font color="#990000">,</font> buffer<font color="#990000">.</font>buffer_id<font color="#990000">);</font>
        <b><font color="#000000">exec_command</font></b><font color="#990000">(</font>app<font color="#990000">,</font> cmdid_save<font color="#990000">);</font>
    <font color="#FF0000">}</font>

    <font color="#008080">String</font> dir <font color="#990000">=</font> <b><font color="#000000">make_string</font></b><font color="#990000">(</font>app<font color="#990000">-&gt;</font>memory<font color="#990000">,</font> <font color="#993399">0</font><font color="#990000">,</font> app<font color="#990000">-&gt;</font>memory_size<font color="#990000">);</font>
    <b><font color="#000000">append</font></b><font color="#990000">(&amp;</font>dir<font color="#990000">,</font> BuildDirectory<font color="#990000">);</font>
    <b><font color="#0000FF">for</font></b><font color="#990000">(</font><font color="#009900">int</font> At <font color="#990000">=</font> <font color="#993399">0</font><font color="#990000">;</font>
        At <font color="#990000">&lt;</font> dir<font color="#990000">.</font>size<font color="#990000">;</font>
        <font color="#990000">++</font>At<font color="#990000">)</font>
    <font color="#FF0000">{</font>
        <b><font color="#0000FF">if</font></b><font color="#990000">(</font>dir<font color="#990000">.</font>str<font color="#990000">[</font>At<font color="#990000">]</font> <font color="#990000">==</font> <font color="#FF0000">'/'</font><font color="#990000">)</font>
        <font color="#FF0000">{</font>
            dir<font color="#990000">.</font>str<font color="#990000">[</font>At<font color="#990000">]</font> <font color="#990000">=</font> <font color="#FF0000">'</font><font color="#CC33CC">\\</font><font color="#FF0000">'</font><font color="#990000">;</font>
        <font color="#FF0000">}</font>
    <font color="#FF0000">}</font>


    <b><font color="#000000">push_parameter</font></b><font color="#990000">(</font>app<font color="#990000">,</font> par_flags<font color="#990000">,</font> CLI_OverlapWithConflict<font color="#990000">);</font>
    <b><font color="#000000">push_parameter</font></b><font color="#990000">(</font>app<font color="#990000">,</font> par_name<font color="#990000">,</font> GlobalCompilationBufferName<font color="#990000">,</font> <font color="#990000">(</font><font color="#009900">int</font><font color="#990000">)</font><b><font color="#000000">strlen</font></b><font color="#990000">(</font>GlobalCompilationBufferName<font color="#990000">));</font>
    <b><font color="#000000">push_parameter</font></b><font color="#990000">(</font>app<font color="#990000">,</font> par_cli_path<font color="#990000">,</font> dir<font color="#990000">.</font>str<font color="#990000">,</font> dir<font color="#990000">.</font>size<font color="#990000">);</font>

    <b><font color="#0000FF">if</font></b><font color="#990000">(</font><b><font color="#000000">append</font></b><font color="#990000">(&amp;</font>dir<font color="#990000">,</font> <font color="#FF0000">"build.bat"</font><font color="#990000">))</font>
    <font color="#FF0000">{</font>
        <b><font color="#000000">push_parameter</font></b><font color="#990000">(</font>app<font color="#990000">,</font> par_cli_command<font color="#990000">,</font> dir<font color="#990000">.</font>str<font color="#990000">,</font> dir<font color="#990000">.</font>size<font color="#990000">);</font>
        <b><font color="#000000">exec_command</font></b><font color="#990000">(</font>app<font color="#990000">,</font> cmdid_command_line<font color="#990000">);</font>
        <b><font color="#000000">exec_command</font></b><font color="#990000">(</font>app<font color="#990000">,</font> cmdid_change_active_panel<font color="#990000">);</font>
    <font color="#FF0000">}</font>
    <b><font color="#0000FF">else</font></b><font color="#FF0000">{</font>
        app<font color="#990000">-&gt;</font><b><font color="#000000">clear_parameters</font></b><font color="#990000">(</font>app<font color="#990000">);</font>
    <font color="#FF0000">}</font>
<font color="#FF0000">}</font>

internal <font color="#009900">bool</font>
<b><font color="#000000">casey_errors_are_the_same</font></b><font color="#990000">(</font><font color="#008080">Parsed_Error</font> a<font color="#990000">,</font> <font color="#008080">Parsed_Error</font> b<font color="#990000">)</font>
<font color="#FF0000">{</font>
    <font color="#009900">bool</font> result <font color="#990000">=</font> <font color="#990000">((</font>a<font color="#990000">.</font>exists <font color="#990000">==</font> b<font color="#990000">.</font>exists<font color="#990000">)</font> <font color="#990000">&amp;&amp;</font> <b><font color="#000000">compare</font></b><font color="#990000">(</font>a<font color="#990000">.</font>target_file_name<font color="#990000">,</font> b<font color="#990000">.</font>target_file_name<font color="#990000">)</font> <font color="#990000">&amp;&amp;</font> <font color="#990000">(</font>a<font color="#990000">.</font>target_line_number <font color="#990000">==</font> b<font color="#990000">.</font>target_line_number<font color="#990000">));</font>

    <b><font color="#0000FF">return</font></b><font color="#990000">(</font>result<font color="#990000">);</font>
<font color="#FF0000">}</font>

internal <font color="#009900">void</font>
<b><font color="#000000">casey_goto_error</font></b><font color="#990000">(</font><font color="#008080">Application_Links</font> <font color="#990000">*</font>app<font color="#990000">,</font> <font color="#008080">Parsed_Error</font> e<font color="#990000">)</font>
<font color="#FF0000">{</font>
    <b><font color="#0000FF">if</font></b><font color="#990000">(</font>e<font color="#990000">.</font>exists<font color="#990000">)</font>
    <font color="#FF0000">{</font>
        <font color="#008080">switch_to_result</font> Switch <font color="#990000">=</font> <b><font color="#000000">SwitchToOrLoadFile</font></b><font color="#990000">(</font>app<font color="#990000">,</font> e<font color="#990000">.</font>target_file_name<font color="#990000">,</font> <b><font color="#0000FF">false</font></b><font color="#990000">);</font>
        <b><font color="#0000FF">if</font></b><font color="#990000">(</font>Switch<font color="#990000">.</font>Switched<font color="#990000">)</font>
        <font color="#FF0000">{</font>
            app<font color="#990000">-&gt;</font><b><font color="#000000">view_set_cursor</font></b><font color="#990000">(</font>app<font color="#990000">,</font> <font color="#990000">&amp;</font>Switch<font color="#990000">.</font>view<font color="#990000">,</font> <b><font color="#000000">seek_line_char</font></b><font color="#990000">(</font>e<font color="#990000">.</font>target_line_number<font color="#990000">,</font> e<font color="#990000">.</font>target_column_number<font color="#990000">),</font> <font color="#993399">1</font><font color="#990000">);</font>
        <font color="#FF0000">}</font>

        <font color="#008080">View_Summary</font> compilation_view <font color="#990000">=</font> <b><font color="#000000">get_first_view_with_buffer</font></b><font color="#990000">(</font>app<font color="#990000">,</font> e<font color="#990000">.</font>source_buffer_id<font color="#990000">);</font>
        <b><font color="#0000FF">if</font></b><font color="#990000">(</font>compilation_view<font color="#990000">.</font>exists<font color="#990000">)</font>
        <font color="#FF0000">{</font>
            app<font color="#990000">-&gt;</font><b><font color="#000000">view_set_cursor</font></b><font color="#990000">(</font>app<font color="#990000">,</font> <font color="#990000">&amp;</font>compilation_view<font color="#990000">,</font> <b><font color="#000000">seek_pos</font></b><font color="#990000">(</font>e<font color="#990000">.</font>source_position<font color="#990000">),</font> <font color="#993399">1</font><font color="#990000">);</font>
        <font color="#FF0000">}</font>
    <font color="#FF0000">}</font>
<font color="#FF0000">}</font>

<font color="#008080">internal</font> Parsed_Error
<b><font color="#000000">casey_parse_error</font></b><font color="#990000">(</font><font color="#008080">Application_Links</font> <font color="#990000">*</font>app<font color="#990000">,</font> <font color="#008080">Buffer_Summary</font> buffer<font color="#990000">,</font> <font color="#008080">View_Summary</font> view<font color="#990000">)</font>
<font color="#FF0000">{</font>
    <font color="#008080">Parsed_Error</font> result <font color="#990000">=</font> <font color="#FF0000">{}</font><font color="#990000">;</font>

    app<font color="#990000">-&gt;</font><b><font color="#000000">refresh_view</font></b><font color="#990000">(</font>app<font color="#990000">,</font> <font color="#990000">&amp;</font>view<font color="#990000">);</font>
    <font color="#009900">int</font> restore_pos <font color="#990000">=</font> view<font color="#990000">.</font>cursor<font color="#990000">.</font>pos<font color="#990000">;</font>

    app<font color="#990000">-&gt;</font><b><font color="#000000">view_set_cursor</font></b><font color="#990000">(</font>app<font color="#990000">,</font> <font color="#990000">&amp;</font>view<font color="#990000">,</font> <b><font color="#000000">seek_line_char</font></b><font color="#990000">(</font>view<font color="#990000">.</font>cursor<font color="#990000">.</font>line<font color="#990000">,</font> <font color="#993399">1</font><font color="#990000">),</font> <font color="#993399">1</font><font color="#990000">);</font>
    app<font color="#990000">-&gt;</font><b><font color="#000000">refresh_view</font></b><font color="#990000">(</font>app<font color="#990000">,</font> <font color="#990000">&amp;</font>view<font color="#990000">);</font>
    <font color="#009900">int</font> start <font color="#990000">=</font> view<font color="#990000">.</font>cursor<font color="#990000">.</font>pos<font color="#990000">;</font>

    app<font color="#990000">-&gt;</font><b><font color="#000000">view_set_cursor</font></b><font color="#990000">(</font>app<font color="#990000">,</font> <font color="#990000">&amp;</font>view<font color="#990000">,</font> <b><font color="#000000">seek_line_char</font></b><font color="#990000">(</font>view<font color="#990000">.</font>cursor<font color="#990000">.</font>line<font color="#990000">,</font> <font color="#993399">65536</font><font color="#990000">),</font> <font color="#993399">1</font><font color="#990000">);</font>
    app<font color="#990000">-&gt;</font><b><font color="#000000">refresh_view</font></b><font color="#990000">(</font>app<font color="#990000">,</font> <font color="#990000">&amp;</font>view<font color="#990000">);</font>
    <font color="#009900">int</font> end <font color="#990000">=</font> view<font color="#990000">.</font>cursor<font color="#990000">.</font>pos<font color="#990000">;</font>

    app<font color="#990000">-&gt;</font><b><font color="#000000">view_set_cursor</font></b><font color="#990000">(</font>app<font color="#990000">,</font> <font color="#990000">&amp;</font>view<font color="#990000">,</font> <b><font color="#000000">seek_pos</font></b><font color="#990000">(</font>restore_pos<font color="#990000">),</font> <font color="#993399">1</font><font color="#990000">);</font>
    app<font color="#990000">-&gt;</font><b><font color="#000000">refresh_view</font></b><font color="#990000">(</font>app<font color="#990000">,</font> <font color="#990000">&amp;</font>view<font color="#990000">);</font>

    <font color="#009900">int</font> size <font color="#990000">=</font> end <font color="#990000">-</font> start<font color="#990000">;</font>

    <font color="#009900">char</font> <font color="#990000">*</font>ParsingRegion <font color="#990000">=</font> <font color="#990000">(</font><font color="#009900">char</font> <font color="#990000">*)</font><b><font color="#000000">malloc</font></b><font color="#990000">(</font>size <font color="#990000">+</font> <font color="#993399">1</font><font color="#990000">);</font>
<i><font color="#9A1900">//    char *ParsingRegion = (char *)app-&gt;push_memory(app, size + 1);</font></i>
    app<font color="#990000">-&gt;</font><b><font color="#000000">buffer_read_range</font></b><font color="#990000">(</font>app<font color="#990000">,</font> <font color="#990000">&amp;</font>buffer<font color="#990000">,</font> start<font color="#990000">,</font> end<font color="#990000">,</font> ParsingRegion<font color="#990000">);</font>
    ParsingRegion<font color="#990000">[</font>size<font color="#990000">]</font> <font color="#990000">=</font> <font color="#993399">0</font><font color="#990000">;</font>
    <font color="#008080">tokenizer</font> Tokenizer <font color="#990000">=</font> <font color="#FF0000">{</font>ParsingRegion<font color="#FF0000">}</font><font color="#990000">;</font>
    <b><font color="#0000FF">for</font></b><font color="#990000">(;;)</font>
    <font color="#FF0000">{</font>
        <font color="#008080">token</font> Token <font color="#990000">=</font> <b><font color="#000000">GetToken</font></b><font color="#990000">(&amp;</font>Tokenizer<font color="#990000">);</font>
        <b><font color="#0000FF">if</font></b><font color="#990000">(</font>Token<font color="#990000">.</font>Type <font color="#990000">==</font> Token_OpenParen<font color="#990000">)</font>
        <font color="#FF0000">{</font>
            <font color="#008080">token</font> LineToken <font color="#990000">=</font> <b><font color="#000000">GetToken</font></b><font color="#990000">(&amp;</font>Tokenizer<font color="#990000">);</font>
            <b><font color="#0000FF">if</font></b><font color="#990000">(</font>LineToken<font color="#990000">.</font>Type <font color="#990000">==</font> Token_Number<font color="#990000">)</font>
            <font color="#FF0000">{</font>
                <font color="#008080">token</font> CloseToken <font color="#990000">=</font> <b><font color="#000000">GetToken</font></b><font color="#990000">(&amp;</font>Tokenizer<font color="#990000">);</font>

                <font color="#009900">int</font> column_number <font color="#990000">=</font> <font color="#993399">0</font><font color="#990000">;</font>
                <b><font color="#0000FF">if</font></b><font color="#990000">(</font>CloseToken<font color="#990000">.</font>Type <font color="#990000">==</font> Token_Comma<font color="#990000">)</font>
                <font color="#FF0000">{</font>
                    <font color="#008080">token</font> ColumnToken <font color="#990000">=</font> <b><font color="#000000">GetToken</font></b><font color="#990000">(&amp;</font>Tokenizer<font color="#990000">);</font>
                    <b><font color="#0000FF">if</font></b><font color="#990000">(</font>ColumnToken<font color="#990000">.</font>Type <font color="#990000">==</font> Token_Number<font color="#990000">)</font>
                    <font color="#FF0000">{</font>
                        column_number <font color="#990000">=</font> <b><font color="#000000">atoi</font></b><font color="#990000">(</font>ColumnToken<font color="#990000">.</font>Text<font color="#990000">);</font>
                        CloseToken <font color="#990000">=</font> <b><font color="#000000">GetToken</font></b><font color="#990000">(&amp;</font>Tokenizer<font color="#990000">);</font>
                    <font color="#FF0000">}</font>
                <font color="#FF0000">}</font>

                <b><font color="#0000FF">if</font></b><font color="#990000">(</font>CloseToken<font color="#990000">.</font>Type <font color="#990000">==</font> Token_CloseParen<font color="#990000">)</font>
                <font color="#FF0000">{</font>
                    <font color="#008080">token</font> ColonToken <font color="#990000">=</font> <b><font color="#000000">GetToken</font></b><font color="#990000">(&amp;</font>Tokenizer<font color="#990000">);</font>
                    <b><font color="#0000FF">if</font></b><font color="#990000">(</font>ColonToken<font color="#990000">.</font>Type <font color="#990000">==</font> Token_Colon<font color="#990000">)</font>
                    <font color="#FF0000">{</font>
                        <i><font color="#9A1900">// NOTE(casey): We maybe found an error!</font></i>
                        <font color="#009900">int</font> line_number <font color="#990000">=</font> <b><font color="#000000">atoi</font></b><font color="#990000">(</font>LineToken<font color="#990000">.</font>Text<font color="#990000">);</font>

                        <font color="#009900">char</font> <font color="#990000">*</font>Seek <font color="#990000">=</font> Token<font color="#990000">.</font>Text<font color="#990000">;</font>
                        <b><font color="#0000FF">while</font></b><font color="#990000">(</font>Seek <font color="#990000">!=</font> ParsingRegion<font color="#990000">)</font>
                        <font color="#FF0000">{</font>
                            <b><font color="#0000FF">if</font></b><font color="#990000">(</font><b><font color="#000000">IsEndOfLine</font></b><font color="#990000">(*</font>Seek<font color="#990000">))</font>
                            <font color="#FF0000">{</font>
                                <b><font color="#0000FF">while</font></b><font color="#990000">(</font><b><font color="#000000">IsWhitespace</font></b><font color="#990000">(*</font>Seek<font color="#990000">))</font>
                                <font color="#FF0000">{</font>
                                    <font color="#990000">++</font>Seek<font color="#990000">;</font>
                                <font color="#FF0000">}</font>
                                <b><font color="#0000FF">break</font></b><font color="#990000">;</font>
                            <font color="#FF0000">}</font>

                            <font color="#990000">--</font>Seek<font color="#990000">;</font>
                        <font color="#FF0000">}</font>

                        result<font color="#990000">.</font>exists <font color="#990000">=</font> <b><font color="#0000FF">true</font></b><font color="#990000">;</font>
                        result<font color="#990000">.</font>target_file_name <font color="#990000">=</font> <b><font color="#000000">make_string</font></b><font color="#990000">(</font>Seek<font color="#990000">,</font> <font color="#990000">(</font><font color="#009900">int</font><font color="#990000">)(</font>Token<font color="#990000">.</font>Text <font color="#990000">-</font> Seek<font color="#990000">));;</font>
                        result<font color="#990000">.</font>target_line_number <font color="#990000">=</font> line_number<font color="#990000">;</font>
                        result<font color="#990000">.</font>target_column_number <font color="#990000">=</font> column_number<font color="#990000">;</font>
                        result<font color="#990000">.</font>source_buffer_id <font color="#990000">=</font> buffer<font color="#990000">.</font>buffer_id<font color="#990000">;</font>
                        result<font color="#990000">.</font>source_position <font color="#990000">=</font> start <font color="#990000">+</font> <font color="#990000">(</font><font color="#009900">int</font><font color="#990000">)(</font>ColonToken<font color="#990000">.</font>Text <font color="#990000">-</font> ParsingRegion<font color="#990000">);</font>

                        <b><font color="#0000FF">break</font></b><font color="#990000">;</font>
                    <font color="#FF0000">}</font>
                <font color="#FF0000">}</font>
            <font color="#FF0000">}</font>
        <font color="#FF0000">}</font>
        <b><font color="#0000FF">else</font></b> <b><font color="#0000FF">if</font></b><font color="#990000">(</font>Token<font color="#990000">.</font>Type <font color="#990000">==</font> Token_EndOfStream<font color="#990000">)</font>
        <font color="#FF0000">{</font>
            <b><font color="#0000FF">break</font></b><font color="#990000">;</font>
        <font color="#FF0000">}</font>
    <font color="#FF0000">}</font>

    <b><font color="#0000FF">return</font></b><font color="#990000">(</font>result<font color="#990000">);</font>
<font color="#FF0000">}</font>

internal <font color="#009900">void</font>
<b><font color="#000000">casey_seek_error_dy</font></b><font color="#990000">(</font><font color="#008080">Application_Links</font> <font color="#990000">*</font>app<font color="#990000">,</font> <font color="#009900">int</font> dy<font color="#990000">)</font>
<font color="#FF0000">{</font>
    <font color="#008080">Buffer_Summary</font> Buffer <font color="#990000">=</font> app<font color="#990000">-&gt;</font><b><font color="#000000">get_buffer_by_name</font></b><font color="#990000">(</font>app<font color="#990000">,</font> GlobalCompilationBufferName<font color="#990000">,</font> <font color="#990000">(</font><font color="#009900">int</font><font color="#990000">)</font><b><font color="#000000">strlen</font></b><font color="#990000">(</font>GlobalCompilationBufferName<font color="#990000">));</font>
    <font color="#008080">View_Summary</font> compilation_view <font color="#990000">=</font> <b><font color="#000000">get_first_view_with_buffer</font></b><font color="#990000">(</font>app<font color="#990000">,</font> Buffer<font color="#990000">.</font>buffer_id<font color="#990000">);</font>

    <i><font color="#9A1900">// NOTE(casey): First get the current error (which may be none, if we've never parsed before)</font></i>
    <font color="#008080">Parsed_Error</font> StartingError <font color="#990000">=</font> <b><font color="#000000">casey_parse_error</font></b><font color="#990000">(</font>app<font color="#990000">,</font> Buffer<font color="#990000">,</font> compilation_view<font color="#990000">);</font>

    <i><font color="#9A1900">// NOTE(casey): Now hunt for the previous distinct error</font></i>
    <b><font color="#0000FF">for</font></b><font color="#990000">(;;)</font>
    <font color="#FF0000">{</font>
        <font color="#009900">int</font> prev_pos <font color="#990000">=</font> compilation_view<font color="#990000">.</font>cursor<font color="#990000">.</font>pos<font color="#990000">;</font>
        app<font color="#990000">-&gt;</font><b><font color="#000000">view_set_cursor</font></b><font color="#990000">(</font>app<font color="#990000">,</font> <font color="#990000">&amp;</font>compilation_view<font color="#990000">,</font> <b><font color="#000000">seek_line_char</font></b><font color="#990000">(</font>compilation_view<font color="#990000">.</font>cursor<font color="#990000">.</font>line <font color="#990000">+</font> dy<font color="#990000">,</font> <font color="#993399">0</font><font color="#990000">),</font> <font color="#993399">1</font><font color="#990000">);</font>
        app<font color="#990000">-&gt;</font><b><font color="#000000">refresh_view</font></b><font color="#990000">(</font>app<font color="#990000">,</font> <font color="#990000">&amp;</font>compilation_view<font color="#990000">);</font>
        <b><font color="#0000FF">if</font></b><font color="#990000">(</font>compilation_view<font color="#990000">.</font>cursor<font color="#990000">.</font>pos <font color="#990000">!=</font> prev_pos<font color="#990000">)</font>
        <font color="#FF0000">{</font>
            <font color="#008080">Parsed_Error</font> Error <font color="#990000">=</font> <b><font color="#000000">casey_parse_error</font></b><font color="#990000">(</font>app<font color="#990000">,</font> Buffer<font color="#990000">,</font> compilation_view<font color="#990000">);</font>
            <b><font color="#0000FF">if</font></b><font color="#990000">(</font>Error<font color="#990000">.</font>exists <font color="#990000">&amp;&amp;</font> <font color="#990000">!</font><b><font color="#000000">casey_errors_are_the_same</font></b><font color="#990000">(</font>StartingError<font color="#990000">,</font> Error<font color="#990000">))</font>
            <font color="#FF0000">{</font>
                <b><font color="#000000">casey_goto_error</font></b><font color="#990000">(</font>app<font color="#990000">,</font> Error<font color="#990000">);</font>
                <b><font color="#0000FF">break</font></b><font color="#990000">;</font>
            <font color="#FF0000">}</font>
        <font color="#FF0000">}</font>
        <b><font color="#0000FF">else</font></b>
        <font color="#FF0000">{</font>
            <b><font color="#0000FF">break</font></b><font color="#990000">;</font>
        <font color="#FF0000">}</font>
    <font color="#FF0000">}</font>
<font color="#FF0000">}</font>

<b><font color="#000000">CUSTOM_COMMAND_SIG</font></b><font color="#990000">(</font>casey_goto_previous_error<font color="#990000">)</font>
<font color="#FF0000">{</font>
    <b><font color="#000000">casey_seek_error_dy</font></b><font color="#990000">(</font>app<font color="#990000">,</font> <font color="#990000">-</font><font color="#993399">1</font><font color="#990000">);</font>
<font color="#FF0000">}</font>

<b><font color="#000000">CUSTOM_COMMAND_SIG</font></b><font color="#990000">(</font>casey_goto_next_error<font color="#990000">)</font>
<font color="#FF0000">{</font>
    <b><font color="#000000">casey_seek_error_dy</font></b><font color="#990000">(</font>app<font color="#990000">,</font> <font color="#993399">1</font><font color="#990000">);</font>
<font color="#FF0000">}</font>

<b><font color="#000000">CUSTOM_COMMAND_SIG</font></b><font color="#990000">(</font>casey_imenu<font color="#990000">)</font>
<font color="#FF0000">{</font>
    <i><font color="#9A1900">// TODO(casey): Implement!</font></i>
<font color="#FF0000">}</font>

<i><font color="#9A1900">//</font></i>
<i><font color="#9A1900">// TODO(casey): Everything below this line probably isn't possible yet</font></i>
<i><font color="#9A1900">//</font></i>

<b><font color="#000000">CUSTOM_COMMAND_SIG</font></b><font color="#990000">(</font>casey_call_keyboard_macro<font color="#990000">)</font>
<font color="#FF0000">{</font>
    <i><font color="#9A1900">// TODO(casey): Implement!</font></i>
<font color="#FF0000">}</font>

<b><font color="#000000">CUSTOM_COMMAND_SIG</font></b><font color="#990000">(</font>casey_begin_keyboard_macro_recording<font color="#990000">)</font>
<font color="#FF0000">{</font>
    <i><font color="#9A1900">// TODO(casey): Implement!</font></i>
<font color="#FF0000">}</font>

<b><font color="#000000">CUSTOM_COMMAND_SIG</font></b><font color="#990000">(</font>casey_end_keyboard_macro_recording<font color="#990000">)</font>
<font color="#FF0000">{</font>
    <i><font color="#9A1900">// TODO(casey): Implement!</font></i>
<font color="#FF0000">}</font>

<b><font color="#000000">CUSTOM_COMMAND_SIG</font></b><font color="#990000">(</font>casey_fill_paragraph<font color="#990000">)</font>
<font color="#FF0000">{</font>
    <i><font color="#9A1900">// TODO(casey): Implement!</font></i>
<font color="#FF0000">}</font>

<b><font color="#0000FF">enum</font></b> calc_node_type
<font color="#FF0000">{</font>
    CalcNode_UnaryMinus<font color="#990000">,</font>

    CalcNode_Add<font color="#990000">,</font>
    CalcNode_Subtract<font color="#990000">,</font>
    CalcNode_Multiply<font color="#990000">,</font>
    CalcNode_Divide<font color="#990000">,</font>
    CalcNode_Mod<font color="#990000">,</font>

    CalcNode_Constant<font color="#990000">,</font>
<font color="#FF0000">}</font><font color="#990000">;</font>
<b><font color="#0000FF">struct</font></b> <font color="#008080">calc_node</font>
<font color="#FF0000">{</font>
    <font color="#008080">calc_node_type</font> Type<font color="#990000">;</font>
    <font color="#009900">double</font> Value<font color="#990000">;</font>
    <font color="#008080">calc_node</font> <font color="#990000">*</font>Left<font color="#990000">;</font>
    <font color="#008080">calc_node</font> <font color="#990000">*</font>Right<font color="#990000">;</font>
<font color="#FF0000">}</font><font color="#990000">;</font>

internal <font color="#009900">double</font>
<b><font color="#000000">ExecCalcNode</font></b><font color="#990000">(</font><font color="#008080">calc_node</font> <font color="#990000">*</font>Node<font color="#990000">)</font>
<font color="#FF0000">{</font>
    <font color="#009900">double</font> Result <font color="#990000">=</font> <font color="#993399">0</font><font color="#990000">.</font>0f<font color="#990000">;</font>

    <b><font color="#0000FF">if</font></b><font color="#990000">(</font>Node<font color="#990000">)</font>
    <font color="#FF0000">{</font>
        <b><font color="#0000FF">switch</font></b><font color="#990000">(</font>Node<font color="#990000">-&gt;</font>Type<font color="#990000">)</font>
        <font color="#FF0000">{</font>
            <b><font color="#0000FF">case</font></b> CalcNode_UnaryMinus<font color="#990000">:</font> <font color="#FF0000">{</font>Result <font color="#990000">=</font> <font color="#990000">-</font><b><font color="#000000">ExecCalcNode</font></b><font color="#990000">(</font>Node<font color="#990000">-&gt;</font>Left<font color="#990000">);</font><font color="#FF0000">}</font> <b><font color="#0000FF">break</font></b><font color="#990000">;</font>
            <b><font color="#0000FF">case</font></b> CalcNode_Add<font color="#990000">:</font> <font color="#FF0000">{</font>Result <font color="#990000">=</font> <b><font color="#000000">ExecCalcNode</font></b><font color="#990000">(</font>Node<font color="#990000">-&gt;</font>Left<font color="#990000">)</font> <font color="#990000">+</font> <b><font color="#000000">ExecCalcNode</font></b><font color="#990000">(</font>Node<font color="#990000">-&gt;</font>Right<font color="#990000">);</font><font color="#FF0000">}</font> <b><font color="#0000FF">break</font></b><font color="#990000">;</font>
            <b><font color="#0000FF">case</font></b> CalcNode_Subtract<font color="#990000">:</font> <font color="#FF0000">{</font>Result <font color="#990000">=</font> <b><font color="#000000">ExecCalcNode</font></b><font color="#990000">(</font>Node<font color="#990000">-&gt;</font>Left<font color="#990000">)</font> <font color="#990000">-</font> <b><font color="#000000">ExecCalcNode</font></b><font color="#990000">(</font>Node<font color="#990000">-&gt;</font>Right<font color="#990000">);</font><font color="#FF0000">}</font> <b><font color="#0000FF">break</font></b><font color="#990000">;</font>
            <b><font color="#0000FF">case</font></b> CalcNode_Multiply<font color="#990000">:</font> <font color="#FF0000">{</font>Result <font color="#990000">=</font> <b><font color="#000000">ExecCalcNode</font></b><font color="#990000">(</font>Node<font color="#990000">-&gt;</font>Left<font color="#990000">)</font> <font color="#990000">*</font> <b><font color="#000000">ExecCalcNode</font></b><font color="#990000">(</font>Node<font color="#990000">-&gt;</font>Right<font color="#990000">);</font><font color="#FF0000">}</font> <b><font color="#0000FF">break</font></b><font color="#990000">;</font>
            <b><font color="#0000FF">case</font></b> CalcNode_Divide<font color="#990000">:</font> <font color="#FF0000">{</font><i><font color="#9A1900">/*</font></i><b>TODO</b><i><font color="#9A1900">(casey): Guard 0*/</font></i>Result <font color="#990000">=</font> <b><font color="#000000">ExecCalcNode</font></b><font color="#990000">(</font>Node<font color="#990000">-&gt;</font>Left<font color="#990000">)</font> <font color="#990000">/</font> <b><font color="#000000">ExecCalcNode</font></b><font color="#990000">(</font>Node<font color="#990000">-&gt;</font>Right<font color="#990000">);</font><font color="#FF0000">}</font> <b><font color="#0000FF">break</font></b><font color="#990000">;</font>
            <b><font color="#0000FF">case</font></b> CalcNode_Mod<font color="#990000">:</font> <font color="#FF0000">{</font><i><font color="#9A1900">/*</font></i><b>TODO</b><i><font color="#9A1900">(casey): Guard 0*/</font></i>Result <font color="#990000">=</font> <b><font color="#000000">fmod</font></b><font color="#990000">(</font><b><font color="#000000">ExecCalcNode</font></b><font color="#990000">(</font>Node<font color="#990000">-&gt;</font>Left<font color="#990000">),</font> <b><font color="#000000">ExecCalcNode</font></b><font color="#990000">(</font>Node<font color="#990000">-&gt;</font>Right<font color="#990000">));</font><font color="#FF0000">}</font> <b><font color="#0000FF">break</font></b><font color="#990000">;</font>
            <b><font color="#0000FF">case</font></b> CalcNode_Constant<font color="#990000">:</font> <font color="#FF0000">{</font>Result <font color="#990000">=</font> Node<font color="#990000">-&gt;</font>Value<font color="#990000">;</font><font color="#FF0000">}</font> <b><font color="#0000FF">break</font></b><font color="#990000">;</font>
            <b><font color="#0000FF">default</font></b><font color="#990000">:</font> <font color="#FF0000">{</font><b><font color="#000000">Assert</font></b><font color="#990000">(!</font><font color="#FF0000">"AHHHHH!"</font><font color="#990000">);</font><font color="#FF0000">}</font>
        <font color="#FF0000">}</font>
    <font color="#FF0000">}</font>

    <b><font color="#0000FF">return</font></b><font color="#990000">(</font>Result<font color="#990000">);</font>
<font color="#FF0000">}</font>

internal <font color="#009900">void</font>
<b><font color="#000000">FreeCalcNode</font></b><font color="#990000">(</font><font color="#008080">calc_node</font> <font color="#990000">*</font>Node<font color="#990000">)</font>
<font color="#FF0000">{</font>
    <b><font color="#0000FF">if</font></b><font color="#990000">(</font>Node<font color="#990000">)</font>
    <font color="#FF0000">{</font>
        <b><font color="#000000">FreeCalcNode</font></b><font color="#990000">(</font>Node<font color="#990000">-&gt;</font>Left<font color="#990000">);</font>
        <b><font color="#000000">FreeCalcNode</font></b><font color="#990000">(</font>Node<font color="#990000">-&gt;</font>Right<font color="#990000">);</font>
        <b><font color="#000000">free</font></b><font color="#990000">(</font>Node<font color="#990000">);</font>
    <font color="#FF0000">}</font>
<font color="#FF0000">}</font>

<font color="#008080">internal</font> calc_node <font color="#990000">*</font>
<b><font color="#000000">AddNode</font></b><font color="#990000">(</font><font color="#008080">calc_node_type</font> Type<font color="#990000">,</font> <font color="#008080">calc_node</font> <font color="#990000">*</font>Left <font color="#990000">=</font> <font color="#993399">0</font><font color="#990000">,</font> <font color="#008080">calc_node</font> <font color="#990000">*</font>Right <font color="#990000">=</font> <font color="#993399">0</font><font color="#990000">)</font>
<font color="#FF0000">{</font>
    <font color="#008080">calc_node</font> <font color="#990000">*</font>Node <font color="#990000">=</font> <font color="#990000">(</font>calc_node <font color="#990000">*)</font><b><font color="#000000">malloc</font></b><font color="#990000">(</font><b><font color="#0000FF">sizeof</font></b><font color="#990000">(</font>calc_node<font color="#990000">));</font>
    Node<font color="#990000">-&gt;</font>Type <font color="#990000">=</font> Type<font color="#990000">;</font>
    Node<font color="#990000">-&gt;</font>Value <font color="#990000">=</font> <font color="#993399">0</font><font color="#990000">;</font>
    Node<font color="#990000">-&gt;</font>Left <font color="#990000">=</font> Left<font color="#990000">;</font>
    Node<font color="#990000">-&gt;</font>Right <font color="#990000">=</font> Right<font color="#990000">;</font>    
    <b><font color="#0000FF">return</font></b><font color="#990000">(</font>Node<font color="#990000">);</font>
<font color="#FF0000">}</font>

<font color="#008080">internal</font> calc_node <font color="#990000">*</font>
<b><font color="#000000">ParseNumber</font></b><font color="#990000">(</font><font color="#008080">tokenizer</font> <font color="#990000">*</font>Tokenizer<font color="#990000">)</font>
<font color="#FF0000">{</font>
    <font color="#008080">calc_node</font> <font color="#990000">*</font>Result <font color="#990000">=</font> <b><font color="#000000">AddNode</font></b><font color="#990000">(</font>CalcNode_Constant<font color="#990000">);</font>

    <font color="#008080">token</font> Token <font color="#990000">=</font> <b><font color="#000000">GetToken</font></b><font color="#990000">(</font>Tokenizer<font color="#990000">);</font>
    Result<font color="#990000">-&gt;</font>Value <font color="#990000">=</font> <b><font color="#000000">atof</font></b><font color="#990000">(</font>Token<font color="#990000">.</font>Text<font color="#990000">);</font>

    <b><font color="#0000FF">return</font></b><font color="#990000">(</font>Result<font color="#990000">);</font>
<font color="#FF0000">}</font>

<font color="#008080">internal</font> calc_node <font color="#990000">*</font>
<b><font color="#000000">ParseConstant</font></b><font color="#990000">(</font><font color="#008080">tokenizer</font> <font color="#990000">*</font>Tokenizer<font color="#990000">)</font>
<font color="#FF0000">{</font>
    <font color="#008080">calc_node</font> <font color="#990000">*</font>Result <font color="#990000">=</font> <font color="#993399">0</font><font color="#990000">;</font>

    <font color="#008080">token</font> Token <font color="#990000">=</font> <b><font color="#000000">PeekToken</font></b><font color="#990000">(</font>Tokenizer<font color="#990000">);</font>
    <b><font color="#0000FF">if</font></b><font color="#990000">(</font>Token<font color="#990000">.</font>Type <font color="#990000">==</font> Token_Minus<font color="#990000">)</font>
    <font color="#FF0000">{</font>
        Token <font color="#990000">=</font> <b><font color="#000000">GetToken</font></b><font color="#990000">(</font>Tokenizer<font color="#990000">);</font>
        Result <font color="#990000">=</font> <b><font color="#000000">AddNode</font></b><font color="#990000">(</font>CalcNode_UnaryMinus<font color="#990000">);</font>
        Result<font color="#990000">-&gt;</font>Left <font color="#990000">=</font> <b><font color="#000000">ParseNumber</font></b><font color="#990000">(</font>Tokenizer<font color="#990000">);</font>
    <font color="#FF0000">}</font>
    <b><font color="#0000FF">else</font></b>
    <font color="#FF0000">{</font>
        Result <font color="#990000">=</font> <b><font color="#000000">ParseNumber</font></b><font color="#990000">(</font>Tokenizer<font color="#990000">);</font>
    <font color="#FF0000">}</font>

    <b><font color="#0000FF">return</font></b><font color="#990000">(</font>Result<font color="#990000">);</font>
<font color="#FF0000">}</font>

<font color="#008080">internal</font> calc_node <font color="#990000">*</font>
<b><font color="#000000">ParseMultiplyExpression</font></b><font color="#990000">(</font><font color="#008080">tokenizer</font> <font color="#990000">*</font>Tokenizer<font color="#990000">)</font>
<font color="#FF0000">{</font>
    <font color="#008080">calc_node</font> <font color="#990000">*</font>Result <font color="#990000">=</font> <font color="#993399">0</font><font color="#990000">;</font>

    <font color="#008080">token</font> Token <font color="#990000">=</font> <b><font color="#000000">PeekToken</font></b><font color="#990000">(</font>Tokenizer<font color="#990000">);</font>
    <b><font color="#0000FF">if</font></b><font color="#990000">((</font>Token<font color="#990000">.</font>Type <font color="#990000">==</font> Token_Minus<font color="#990000">)</font> <font color="#990000">||</font>
       <font color="#990000">(</font>Token<font color="#990000">.</font>Type <font color="#990000">==</font> Token_Number<font color="#990000">))</font>
    <font color="#FF0000">{</font>
        Result <font color="#990000">=</font> <b><font color="#000000">ParseConstant</font></b><font color="#990000">(</font>Tokenizer<font color="#990000">);</font>
        <font color="#008080">token</font> Token <font color="#990000">=</font> <b><font color="#000000">PeekToken</font></b><font color="#990000">(</font>Tokenizer<font color="#990000">);</font>
        <b><font color="#0000FF">if</font></b><font color="#990000">(</font>Token<font color="#990000">.</font>Type <font color="#990000">==</font> Token_ForwardSlash<font color="#990000">)</font>
        <font color="#FF0000">{</font>
            <b><font color="#000000">GetToken</font></b><font color="#990000">(</font>Tokenizer<font color="#990000">);</font>
            Result <font color="#990000">=</font> <b><font color="#000000">AddNode</font></b><font color="#990000">(</font>CalcNode_Divide<font color="#990000">,</font> Result<font color="#990000">,</font> <b><font color="#000000">ParseNumber</font></b><font color="#990000">(</font>Tokenizer<font color="#990000">));</font>
        <font color="#FF0000">}</font>
        <b><font color="#0000FF">else</font></b> <b><font color="#0000FF">if</font></b><font color="#990000">(</font>Token<font color="#990000">.</font>Type <font color="#990000">==</font> Token_Asterisk<font color="#990000">)</font>
        <font color="#FF0000">{</font>
            <b><font color="#000000">GetToken</font></b><font color="#990000">(</font>Tokenizer<font color="#990000">);</font>
            Result <font color="#990000">=</font> <b><font color="#000000">AddNode</font></b><font color="#990000">(</font>CalcNode_Multiply<font color="#990000">,</font> Result<font color="#990000">,</font> <b><font color="#000000">ParseNumber</font></b><font color="#990000">(</font>Tokenizer<font color="#990000">));</font>
        <font color="#FF0000">}</font>
    <font color="#FF0000">}</font>

    <b><font color="#0000FF">return</font></b><font color="#990000">(</font>Result<font color="#990000">);</font>
<font color="#FF0000">}</font>

<font color="#008080">internal</font> calc_node <font color="#990000">*</font>
<b><font color="#000000">ParseAddExpression</font></b><font color="#990000">(</font><font color="#008080">tokenizer</font> <font color="#990000">*</font>Tokenizer<font color="#990000">)</font>
<font color="#FF0000">{</font>
    <font color="#008080">calc_node</font> <font color="#990000">*</font>Result <font color="#990000">=</font> <font color="#993399">0</font><font color="#990000">;</font>

    <font color="#008080">token</font> Token <font color="#990000">=</font> <b><font color="#000000">PeekToken</font></b><font color="#990000">(</font>Tokenizer<font color="#990000">);</font>
    <b><font color="#0000FF">if</font></b><font color="#990000">((</font>Token<font color="#990000">.</font>Type <font color="#990000">==</font> Token_Minus<font color="#990000">)</font> <font color="#990000">||</font>
       <font color="#990000">(</font>Token<font color="#990000">.</font>Type <font color="#990000">==</font> Token_Number<font color="#990000">))</font>
    <font color="#FF0000">{</font>
        Result <font color="#990000">=</font> <b><font color="#000000">ParseMultiplyExpression</font></b><font color="#990000">(</font>Tokenizer<font color="#990000">);</font>
        <font color="#008080">token</font> Token <font color="#990000">=</font> <b><font color="#000000">PeekToken</font></b><font color="#990000">(</font>Tokenizer<font color="#990000">);</font>
        <b><font color="#0000FF">if</font></b><font color="#990000">(</font>Token<font color="#990000">.</font>Type <font color="#990000">==</font> Token_Plus<font color="#990000">)</font>
        <font color="#FF0000">{</font>
            <b><font color="#000000">GetToken</font></b><font color="#990000">(</font>Tokenizer<font color="#990000">);</font>
            Result <font color="#990000">=</font> <b><font color="#000000">AddNode</font></b><font color="#990000">(</font>CalcNode_Add<font color="#990000">,</font> Result<font color="#990000">,</font> <b><font color="#000000">ParseMultiplyExpression</font></b><font color="#990000">(</font>Tokenizer<font color="#990000">));</font>
        <font color="#FF0000">}</font>
        <b><font color="#0000FF">else</font></b> <b><font color="#0000FF">if</font></b><font color="#990000">(</font>Token<font color="#990000">.</font>Type <font color="#990000">==</font> Token_Minus<font color="#990000">)</font>
        <font color="#FF0000">{</font>
            <b><font color="#000000">GetToken</font></b><font color="#990000">(</font>Tokenizer<font color="#990000">);</font>
            Result <font color="#990000">=</font> <b><font color="#000000">AddNode</font></b><font color="#990000">(</font>CalcNode_Subtract<font color="#990000">,</font> Result<font color="#990000">,</font> <b><font color="#000000">ParseMultiplyExpression</font></b><font color="#990000">(</font>Tokenizer<font color="#990000">));</font>
        <font color="#FF0000">}</font>
    <font color="#FF0000">}</font>

    <b><font color="#0000FF">return</font></b><font color="#990000">(</font>Result<font color="#990000">);</font>
<font color="#FF0000">}</font>

<font color="#008080">internal</font> calc_node <font color="#990000">*</font>
<b><font color="#000000">ParseCalc</font></b><font color="#990000">(</font><font color="#008080">tokenizer</font> <font color="#990000">*</font>Tokenizer<font color="#990000">)</font>
<font color="#FF0000">{</font>
    <font color="#008080">calc_node</font> <font color="#990000">*</font>Node <font color="#990000">=</font> <b><font color="#000000">ParseAddExpression</font></b><font color="#990000">(</font>Tokenizer<font color="#990000">);</font>

    <b><font color="#0000FF">return</font></b><font color="#990000">(</font>Node<font color="#990000">);</font>
<font color="#FF0000">}</font>

<b><font color="#000000">CUSTOM_COMMAND_SIG</font></b><font color="#990000">(</font>casey_quick_calc<font color="#990000">)</font>
<font color="#FF0000">{</font>
    <font color="#008080">View_Summary</font> view <font color="#990000">=</font> app<font color="#990000">-&gt;</font><b><font color="#000000">get_active_view</font></b><font color="#990000">(</font>app<font color="#990000">);</font>

    <font color="#008080">Range</font> range <font color="#990000">=</font> <b><font color="#000000">get_range</font></b><font color="#990000">(&amp;</font>view<font color="#990000">);</font>

    <font color="#008080">size_t</font> Size <font color="#990000">=</font> range<font color="#990000">.</font>max <font color="#990000">-</font> range<font color="#990000">.</font>min<font color="#990000">;</font>
    <font color="#009900">char</font> <font color="#990000">*</font>Stuff <font color="#990000">=</font> <font color="#990000">(</font><font color="#009900">char</font> <font color="#990000">*)</font><b><font color="#000000">malloc</font></b><font color="#990000">(</font>Size <font color="#990000">+</font> <font color="#993399">1</font><font color="#990000">);</font>
    Stuff<font color="#990000">[</font>Size<font color="#990000">]</font> <font color="#990000">=</font> <font color="#993399">0</font><font color="#990000">;</font>

    <font color="#008080">Buffer_Summary</font> buffer <font color="#990000">=</font> app<font color="#990000">-&gt;</font><b><font color="#000000">get_buffer</font></b><font color="#990000">(</font>app<font color="#990000">,</font> view<font color="#990000">.</font>buffer_id<font color="#990000">);</font>
    app<font color="#990000">-&gt;</font><b><font color="#000000">buffer_read_range</font></b><font color="#990000">(</font>app<font color="#990000">,</font> <font color="#990000">&amp;</font>buffer<font color="#990000">,</font> range<font color="#990000">.</font>min<font color="#990000">,</font> range<font color="#990000">.</font>max<font color="#990000">,</font> Stuff<font color="#990000">);</font>

    <font color="#008080">tokenizer</font> Tokenizer <font color="#990000">=</font> <font color="#FF0000">{</font>Stuff<font color="#FF0000">}</font><font color="#990000">;</font>
    <font color="#008080">calc_node</font> <font color="#990000">*</font>CalcTree <font color="#990000">=</font> <b><font color="#000000">ParseCalc</font></b><font color="#990000">(&amp;</font>Tokenizer<font color="#990000">);</font>
    <font color="#009900">double</font> ComputedValue <font color="#990000">=</font> <b><font color="#000000">ExecCalcNode</font></b><font color="#990000">(</font>CalcTree<font color="#990000">);</font>
    <b><font color="#000000">FreeCalcNode</font></b><font color="#990000">(</font>CalcTree<font color="#990000">);</font>

    <font color="#009900">char</font> ResultBuffer<font color="#990000">[</font><font color="#993399">256</font><font color="#990000">];</font>
    <font color="#009900">int</font> ResultSize <font color="#990000">=</font> <b><font color="#000000">sprintf</font></b><font color="#990000">(</font>ResultBuffer<font color="#990000">,</font> <font color="#FF0000">"%f"</font><font color="#990000">,</font> ComputedValue<font color="#990000">);</font>

    app<font color="#990000">-&gt;</font><b><font color="#000000">buffer_replace_range</font></b><font color="#990000">(</font>app<font color="#990000">,</font> <font color="#990000">&amp;</font>buffer<font color="#990000">,</font> range<font color="#990000">.</font>min<font color="#990000">,</font> range<font color="#990000">.</font>max<font color="#990000">,</font> ResultBuffer<font color="#990000">,</font> ResultSize<font color="#990000">);</font>

    <b><font color="#000000">free</font></b><font color="#990000">(</font>Stuff<font color="#990000">);</font>
<font color="#FF0000">}</font>

internal <font color="#009900">void</font>
<b><font color="#000000">OpenProject</font></b><font color="#990000">(</font><font color="#008080">Application_Links</font> <font color="#990000">*</font>app<font color="#990000">,</font> <font color="#009900">char</font> <font color="#990000">*</font>ProjectFileName<font color="#990000">)</font>
<font color="#FF0000">{</font>
    <font color="#009900">int</font> TotalOpenAttempts <font color="#990000">=</font> <font color="#993399">0</font><font color="#990000">;</font>

    <font color="#008080">FILE</font> <font color="#990000">*</font>ProjectFile <font color="#990000">=</font> <b><font color="#000000">fopen</font></b><font color="#990000">(</font>ProjectFileName<font color="#990000">,</font> <font color="#FF0000">"r"</font><font color="#990000">);</font>
    <b><font color="#0000FF">if</font></b><font color="#990000">(</font>ProjectFile<font color="#990000">)</font>
    <font color="#FF0000">{</font>
        <b><font color="#000000">fgets</font></b><font color="#990000">(</font>BuildDirectory<font color="#990000">,</font> <b><font color="#0000FF">sizeof</font></b><font color="#990000">(</font>BuildDirectory<font color="#990000">)</font> <font color="#990000">-</font> <font color="#993399">1</font><font color="#990000">,</font> ProjectFile<font color="#990000">);</font>
        <font color="#008080">size_t</font> BuildDirSize <font color="#990000">=</font> <b><font color="#000000">strlen</font></b><font color="#990000">(</font>BuildDirectory<font color="#990000">);</font>
        <b><font color="#0000FF">if</font></b><font color="#990000">((</font>BuildDirSize<font color="#990000">)</font> <font color="#990000">&amp;&amp;</font> <font color="#990000">(</font>BuildDirectory<font color="#990000">[</font>BuildDirSize <font color="#990000">-</font> <font color="#993399">1</font><font color="#990000">]</font> <font color="#990000">==</font> <font color="#FF0000">'</font><font color="#CC33CC">\n</font><font color="#FF0000">'</font><font color="#990000">))</font>
        <font color="#FF0000">{</font>
            <font color="#990000">--</font>BuildDirSize<font color="#990000">;</font>
        <font color="#FF0000">}</font>

        <b><font color="#0000FF">if</font></b><font color="#990000">((</font>BuildDirSize<font color="#990000">)</font> <font color="#990000">&amp;&amp;</font> <font color="#990000">(</font>BuildDirectory<font color="#990000">[</font>BuildDirSize <font color="#990000">-</font> <font color="#993399">1</font><font color="#990000">]</font> <font color="#990000">!=</font> <font color="#FF0000">'/'</font><font color="#990000">))</font>
        <font color="#FF0000">{</font>
            BuildDirectory<font color="#990000">[</font>BuildDirSize<font color="#990000">++]</font> <font color="#990000">=</font> <font color="#FF0000">'/'</font><font color="#990000">;</font>
            BuildDirectory<font color="#990000">[</font>BuildDirSize<font color="#990000">]</font> <font color="#990000">=</font> <font color="#993399">0</font><font color="#990000">;</font>
        <font color="#FF0000">}</font>

        <font color="#009900">char</font> SourceFileDirectoryName<font color="#990000">[</font><font color="#993399">4096</font><font color="#990000">];</font>
        <font color="#009900">char</font> FileDirectoryName<font color="#990000">[</font><font color="#993399">4096</font><font color="#990000">];</font>
        <b><font color="#0000FF">while</font></b><font color="#990000">(</font><b><font color="#000000">fgets</font></b><font color="#990000">(</font>SourceFileDirectoryName<font color="#990000">,</font> <b><font color="#0000FF">sizeof</font></b><font color="#990000">(</font>SourceFileDirectoryName<font color="#990000">)</font> <font color="#990000">-</font> <font color="#993399">1</font><font color="#990000">,</font> ProjectFile<font color="#990000">))</font>
        <font color="#FF0000">{</font>
            <i><font color="#9A1900">// NOTE(allen|a3.4.4): Here we get the list of files in this directory.</font></i>
            <i><font color="#9A1900">// Notice that we free_file_list at the end.</font></i>
            <font color="#008080">String</font> dir <font color="#990000">=</font> <b><font color="#000000">make_string</font></b><font color="#990000">(</font>FileDirectoryName<font color="#990000">,</font> <font color="#993399">0</font><font color="#990000">,</font> <b><font color="#0000FF">sizeof</font></b><font color="#990000">(</font>FileDirectoryName<font color="#990000">));</font>
            <b><font color="#000000">append</font></b><font color="#990000">(&amp;</font>dir<font color="#990000">,</font> SourceFileDirectoryName<font color="#990000">);</font>
            <b><font color="#0000FF">if</font></b><font color="#990000">(</font>dir<font color="#990000">.</font>size <font color="#990000">&amp;&amp;</font> dir<font color="#990000">.</font>str<font color="#990000">[</font>dir<font color="#990000">.</font>size<font color="#990000">-</font><font color="#993399">1</font><font color="#990000">]</font> <font color="#990000">==</font> <font color="#FF0000">'</font><font color="#CC33CC">\n</font><font color="#FF0000">'</font><font color="#990000">)</font>
            <font color="#FF0000">{</font>
                <font color="#990000">--</font>dir<font color="#990000">.</font>size<font color="#990000">;</font>
            <font color="#FF0000">}</font>

            <b><font color="#0000FF">if</font></b><font color="#990000">(</font>dir<font color="#990000">.</font>size <font color="#990000">&amp;&amp;</font> dir<font color="#990000">.</font>str<font color="#990000">[</font>dir<font color="#990000">.</font>size<font color="#990000">-</font><font color="#993399">1</font><font color="#990000">]</font> <font color="#990000">!=</font> <font color="#FF0000">'/'</font><font color="#990000">)</font>
            <font color="#FF0000">{</font>
                dir<font color="#990000">.</font>str<font color="#990000">[</font>dir<font color="#990000">.</font>size<font color="#990000">++]</font> <font color="#990000">=</font> <font color="#FF0000">'/'</font><font color="#990000">;</font>
            <font color="#FF0000">}</font>

            <font color="#008080">File_List</font> list <font color="#990000">=</font> app<font color="#990000">-&gt;</font><b><font color="#000000">get_file_list</font></b><font color="#990000">(</font>app<font color="#990000">,</font> dir<font color="#990000">.</font>str<font color="#990000">,</font> dir<font color="#990000">.</font>size<font color="#990000">);</font>
            <font color="#009900">int</font> dir_size <font color="#990000">=</font> dir<font color="#990000">.</font>size<font color="#990000">;</font>

            <b><font color="#0000FF">for</font></b> <font color="#990000">(</font><font color="#009900">int</font> i <font color="#990000">=</font> <font color="#993399">0</font><font color="#990000">;</font> i <font color="#990000">&lt;</font> list<font color="#990000">.</font>count<font color="#990000">;</font> <font color="#990000">++</font>i<font color="#990000">)</font>
            <font color="#FF0000">{</font>
                <font color="#008080">File_Info</font> <font color="#990000">*</font>info <font color="#990000">=</font> list<font color="#990000">.</font>infos <font color="#990000">+</font> i<font color="#990000">;</font>
                <b><font color="#0000FF">if</font></b> <font color="#990000">(!</font>info<font color="#990000">-&gt;</font>folder<font color="#990000">)</font>
                <font color="#FF0000">{</font>
                    <font color="#008080">String</font> extension <font color="#990000">=</font> <b><font color="#000000">file_extension</font></b><font color="#990000">(</font>info<font color="#990000">-&gt;</font>filename<font color="#990000">);</font>
                    <b><font color="#0000FF">if</font></b> <font color="#990000">(</font><b><font color="#000000">IsCode</font></b><font color="#990000">(</font>extension<font color="#990000">))</font>
                    <font color="#FF0000">{</font>
                        <i><font color="#9A1900">// NOTE(allen): There's no way in the 4coder API to use relative</font></i>
                        <i><font color="#9A1900">// paths at the moment, so everything should be full paths.  Which is</font></i>
                        <i><font color="#9A1900">// managable.  Here simply set the dir string size back to where it</font></i>
                        <i><font color="#9A1900">// was originally, so that new appends overwrite old ones.</font></i>
                        dir<font color="#990000">.</font>size <font color="#990000">=</font> dir_size<font color="#990000">;</font>
                        <b><font color="#000000">append</font></b><font color="#990000">(&amp;</font>dir<font color="#990000">,</font> info<font color="#990000">-&gt;</font>filename<font color="#990000">);</font>
                        <b><font color="#000000">push_parameter</font></b><font color="#990000">(</font>app<font color="#990000">,</font> par_name<font color="#990000">,</font> dir<font color="#990000">.</font>str<font color="#990000">,</font> dir<font color="#990000">.</font>size<font color="#990000">);</font>
                        <b><font color="#000000">push_parameter</font></b><font color="#990000">(</font>app<font color="#990000">,</font> par_do_in_background<font color="#990000">,</font> <font color="#993399">1</font><font color="#990000">);</font>
                        <b><font color="#000000">exec_command</font></b><font color="#990000">(</font>app<font color="#990000">,</font> cmdid_interactive_open<font color="#990000">);</font>
                        <font color="#990000">++</font>TotalOpenAttempts<font color="#990000">;</font>
                    <font color="#FF0000">}</font>
                <font color="#FF0000">}</font>
            <font color="#FF0000">}</font>

            app<font color="#990000">-&gt;</font><b><font color="#000000">free_file_list</font></b><font color="#990000">(</font>app<font color="#990000">,</font> list<font color="#990000">);</font>
        <font color="#FF0000">}</font>

        <b><font color="#000000">fclose</font></b><font color="#990000">(</font>ProjectFile<font color="#990000">);</font>
    <font color="#FF0000">}</font>
<font color="#FF0000">}</font>    

<b><font color="#000000">CUSTOM_COMMAND_SIG</font></b><font color="#990000">(</font>casey_execute_arbitrary_command<font color="#990000">)</font>
<font color="#FF0000">{</font>
    <font color="#008080">Query_Bar</font> bar<font color="#990000">;</font>
    <font color="#009900">char</font> space<font color="#990000">[</font><font color="#993399">1024</font><font color="#990000">],</font> more_space<font color="#990000">[</font><font color="#993399">1024</font><font color="#990000">];</font>
    bar<font color="#990000">.</font>prompt <font color="#990000">=</font> <b><font color="#000000">make_lit_string</font></b><font color="#990000">(</font><font color="#FF0000">"Command: "</font><font color="#990000">);</font>
    bar<font color="#990000">.</font>string <font color="#990000">=</font> <b><font color="#000000">make_fixed_width_string</font></b><font color="#990000">(</font>space<font color="#990000">);</font>

    <b><font color="#0000FF">if</font></b> <font color="#990000">(!</font><b><font color="#000000">query_user_string</font></b><font color="#990000">(</font>app<font color="#990000">,</font> <font color="#990000">&amp;</font>bar<font color="#990000">))</font> <b><font color="#0000FF">return</font></b><font color="#990000">;</font>
    app<font color="#990000">-&gt;</font><b><font color="#000000">end_query_bar</font></b><font color="#990000">(</font>app<font color="#990000">,</font> <font color="#990000">&amp;</font>bar<font color="#990000">,</font> <font color="#993399">0</font><font color="#990000">);</font>

    <b><font color="#0000FF">if</font></b><font color="#990000">(</font><b><font color="#000000">match</font></b><font color="#990000">(</font>bar<font color="#990000">.</font>string<font color="#990000">,</font> <b><font color="#000000">make_lit_string</font></b><font color="#990000">(</font><font color="#FF0000">"project"</font><font color="#990000">)))</font>
    <font color="#FF0000">{</font>
<i><font color="#9A1900">//        exec_command(app, open_all_code);</font></i>
    <font color="#FF0000">}</font>
    <b><font color="#0000FF">else</font></b> <b><font color="#0000FF">if</font></b><font color="#990000">(</font><b><font color="#000000">match</font></b><font color="#990000">(</font>bar<font color="#990000">.</font>string<font color="#990000">,</font> <b><font color="#000000">make_lit_string</font></b><font color="#990000">(</font><font color="#FF0000">"open menu"</font><font color="#990000">)))</font>
    <font color="#FF0000">{</font>
        <b><font color="#000000">exec_command</font></b><font color="#990000">(</font>app<font color="#990000">,</font> cmdid_open_menu<font color="#990000">);</font>
    <font color="#FF0000">}</font>
    <b><font color="#0000FF">else</font></b>
    <font color="#FF0000">{</font>
        bar<font color="#990000">.</font>prompt <font color="#990000">=</font> <b><font color="#000000">make_fixed_width_string</font></b><font color="#990000">(</font>more_space<font color="#990000">);</font>
        <b><font color="#000000">append</font></b><font color="#990000">(&amp;</font>bar<font color="#990000">.</font>prompt<font color="#990000">,</font> <b><font color="#000000">make_lit_string</font></b><font color="#990000">(</font><font color="#FF0000">"Unrecognized: "</font><font color="#990000">));</font>
        <b><font color="#000000">append</font></b><font color="#990000">(&amp;</font>bar<font color="#990000">.</font>prompt<font color="#990000">,</font> bar<font color="#990000">.</font>string<font color="#990000">);</font>
        bar<font color="#990000">.</font>string<font color="#990000">.</font>size <font color="#990000">=</font> <font color="#993399">0</font><font color="#990000">;</font>

        app<font color="#990000">-&gt;</font><b><font color="#000000">start_query_bar</font></b><font color="#990000">(</font>app<font color="#990000">,</font> <font color="#990000">&amp;</font>bar<font color="#990000">,</font> <font color="#993399">0</font><font color="#990000">);</font>
        app<font color="#990000">-&gt;</font><b><font color="#000000">get_user_input</font></b><font color="#990000">(</font>app<font color="#990000">,</font> EventOnAnyKey <font color="#990000">|</font> EventOnButton<font color="#990000">,</font> <font color="#993399">0</font><font color="#990000">);</font>
    <font color="#FF0000">}</font>
<font color="#FF0000">}</font>

internal <font color="#009900">void</font>
<b><font color="#000000">UpdateModalIndicator</font></b><font color="#990000">(</font><font color="#008080">Application_Links</font> <font color="#990000">*</font>app<font color="#990000">)</font>
<font color="#FF0000">{</font>
    <font color="#008080">Theme_Color</font> normal_colors<font color="#990000">[]</font> <font color="#990000">=</font> 
    <font color="#FF0000">{</font>
        <font color="#FF0000">{</font>Stag_Cursor<font color="#990000">,</font> <font color="#993399">0x40FF40</font><font color="#FF0000">}</font><font color="#990000">,</font>
        <font color="#FF0000">{</font>Stag_At_Cursor<font color="#990000">,</font> <font color="#993399">0x161616</font><font color="#FF0000">}</font><font color="#990000">,</font>
        <font color="#FF0000">{</font>Stag_Mark<font color="#990000">,</font> <font color="#993399">0x808080</font><font color="#FF0000">}</font><font color="#990000">,</font>
        <font color="#FF0000">{</font>Stag_Margin<font color="#990000">,</font> <font color="#993399">0x262626</font><font color="#FF0000">}</font><font color="#990000">,</font>
        <font color="#FF0000">{</font>Stag_Margin_Hover<font color="#990000">,</font> <font color="#993399">0x333333</font><font color="#FF0000">}</font><font color="#990000">,</font>
        <font color="#FF0000">{</font>Stag_Margin_Active<font color="#990000">,</font> <font color="#993399">0x404040</font><font color="#FF0000">}</font><font color="#990000">,</font>
        <font color="#FF0000">{</font>Stag_Bar<font color="#990000">,</font> <font color="#993399">0xCACACA</font><font color="#FF0000">}</font>
    <font color="#FF0000">}</font><font color="#990000">;</font>

    <font color="#008080">Theme_Color</font> edit_colors<font color="#990000">[]</font> <font color="#990000">=</font> 
    <font color="#FF0000">{</font>
        <font color="#FF0000">{</font>Stag_Cursor<font color="#990000">,</font> <font color="#993399">0xFF0000</font><font color="#FF0000">}</font><font color="#990000">,</font>
        <font color="#FF0000">{</font>Stag_At_Cursor<font color="#990000">,</font> <font color="#993399">0x00FFFF</font><font color="#FF0000">}</font><font color="#990000">,</font>
        <font color="#FF0000">{</font>Stag_Mark<font color="#990000">,</font> <font color="#993399">0xFF6F1A</font><font color="#FF0000">}</font><font color="#990000">,</font>
        <font color="#FF0000">{</font>Stag_Margin<font color="#990000">,</font> <font color="#993399">0x33170B</font><font color="#FF0000">}</font><font color="#990000">,</font>
        <font color="#FF0000">{</font>Stag_Margin_Hover<font color="#990000">,</font> <font color="#993399">0x49200F</font><font color="#FF0000">}</font><font color="#990000">,</font>
        <font color="#FF0000">{</font>Stag_Margin_Active<font color="#990000">,</font> <font color="#993399">0x934420</font><font color="#FF0000">}</font><font color="#990000">,</font>
        <font color="#FF0000">{</font>Stag_Bar<font color="#990000">,</font> <font color="#993399">0x934420</font><font color="#FF0000">}</font>
    <font color="#FF0000">}</font><font color="#990000">;</font>

    <b><font color="#0000FF">if</font></b> <font color="#990000">(</font>GlobalEditMode<font color="#990000">)</font><font color="#FF0000">{</font>
        app<font color="#990000">-&gt;</font><b><font color="#000000">set_theme_colors</font></b><font color="#990000">(</font>app<font color="#990000">,</font> edit_colors<font color="#990000">,</font> <b><font color="#000000">ArrayCount</font></b><font color="#990000">(</font>edit_colors<font color="#990000">));</font>
    <font color="#FF0000">}</font>
    <b><font color="#0000FF">else</font></b><font color="#FF0000">{</font>
        app<font color="#990000">-&gt;</font><b><font color="#000000">set_theme_colors</font></b><font color="#990000">(</font>app<font color="#990000">,</font> normal_colors<font color="#990000">,</font> <b><font color="#000000">ArrayCount</font></b><font color="#990000">(</font>normal_colors<font color="#990000">));</font>
    <font color="#FF0000">}</font>
<font color="#FF0000">}</font>
<b><font color="#000000">CUSTOM_COMMAND_SIG</font></b><font color="#990000">(</font>begin_free_typing<font color="#990000">)</font>
<font color="#FF0000">{</font>
    GlobalEditMode <font color="#990000">=</font> <b><font color="#0000FF">false</font></b><font color="#990000">;</font>
    <b><font color="#000000">UpdateModalIndicator</font></b><font color="#990000">(</font>app<font color="#990000">);</font>
<font color="#FF0000">}</font>

<b><font color="#000000">CUSTOM_COMMAND_SIG</font></b><font color="#990000">(</font>end_free_typing<font color="#990000">)</font>
<font color="#FF0000">{</font>
    GlobalEditMode <font color="#990000">=</font> <b><font color="#0000FF">true</font></b><font color="#990000">;</font>
    <b><font color="#000000">UpdateModalIndicator</font></b><font color="#990000">(</font>app<font color="#990000">);</font>
<font color="#FF0000">}</font>

<b><font color="#000080">#define</font></b> <b><font color="#000000">DEFINE_FULL_BIMODAL_KEY</font></b><font color="#990000">(</font>binding_name<font color="#990000">,</font>edit_code<font color="#990000">,</font>normal_code<font color="#990000">)</font> <font color="#990000">\</font>
<b><font color="#000000">CUSTOM_COMMAND_SIG</font></b><font color="#990000">(</font>binding_name<font color="#990000">)</font> <font color="#990000">\</font>
<font color="#FF0000">{</font> <font color="#990000">\</font>
    <b><font color="#0000FF">if</font></b><font color="#990000">(</font>GlobalEditMode<font color="#990000">)</font> <font color="#990000">\</font>
    <font color="#FF0000">{</font> <font color="#990000">\</font>
        edit_code<font color="#990000">;</font>            <font color="#990000">\</font>
    <font color="#FF0000">}</font> <font color="#990000">\</font>
    <b><font color="#0000FF">else</font></b> <font color="#990000">\</font>
    <font color="#FF0000">{</font> <font color="#990000">\</font>
        normal_code<font color="#990000">;</font> <font color="#990000">\</font>
    <font color="#FF0000">}</font> <font color="#990000">\</font>
<font color="#FF0000">}</font>

<b><font color="#000080">#define</font></b> <b><font color="#000000">DEFINE_BIMODAL_KEY</font></b><font color="#990000">(</font>binding_name<font color="#990000">,</font>edit_code<font color="#990000">,</font>normal_code<font color="#990000">)</font> <b><font color="#000000">DEFINE_FULL_BIMODAL_KEY</font></b><font color="#990000">(</font>binding_name<font color="#990000">,</font><b><font color="#000000">exec_command</font></b><font color="#990000">(</font>app<font color="#990000">,</font>edit_code<font color="#990000">),</font><b><font color="#000000">exec_command</font></b><font color="#990000">(</font>app<font color="#990000">,</font>normal_code<font color="#990000">))</font>
<b><font color="#000080">#define</font></b> <b><font color="#000000">DEFINE_MODAL_KEY</font></b><font color="#990000">(</font>binding_name<font color="#990000">,</font>edit_code<font color="#990000">)</font> <b><font color="#000000">DEFINE_BIMODAL_KEY</font></b><font color="#990000">(</font>binding_name<font color="#990000">,</font>edit_code<font color="#990000">,</font>cmdid_write_character<font color="#990000">)</font>

<i><font color="#9A1900">//    cmdid_paste_next ?</font></i>
<i><font color="#9A1900">//    cmdid_timeline_scrub ?</font></i>
<i><font color="#9A1900">//    cmdid_history_backward,</font></i>
<i><font color="#9A1900">//    cmdid_history_forward,</font></i>
<i><font color="#9A1900">//    cmdid_toggle_line_wrap,</font></i>
<i><font color="#9A1900">//    cmdid_close_minor_view,</font></i>

<b><font color="#000000">DEFINE_MODAL_KEY</font></b><font color="#990000">(</font>modal_space<font color="#990000">,</font> cmdid_set_mark<font color="#990000">);</font>
<b><font color="#000000">DEFINE_MODAL_KEY</font></b><font color="#990000">(</font>modal_back_slash<font color="#990000">,</font> casey_clean_and_save<font color="#990000">);</font>
<b><font color="#000000">DEFINE_MODAL_KEY</font></b><font color="#990000">(</font>modal_single_quote<font color="#990000">,</font> casey_call_keyboard_macro<font color="#990000">);</font>
<b><font color="#000000">DEFINE_MODAL_KEY</font></b><font color="#990000">(</font>modal_comma<font color="#990000">,</font> casey_goto_previous_error<font color="#990000">);</font>
<b><font color="#000000">DEFINE_MODAL_KEY</font></b><font color="#990000">(</font>modal_period<font color="#990000">,</font> casey_fill_paragraph<font color="#990000">);</font>
<b><font color="#000000">DEFINE_MODAL_KEY</font></b><font color="#990000">(</font>modal_forward_slash<font color="#990000">,</font> cmdid_change_active_panel<font color="#990000">);</font>
<b><font color="#000000">DEFINE_MODAL_KEY</font></b><font color="#990000">(</font>modal_semicolon<font color="#990000">,</font> cmdid_cursor_mark_swap<font color="#990000">);</font> <i><font color="#9A1900">// TODO(casey): Maybe cmdid_history_backward?</font></i>
<b><font color="#000000">DEFINE_BIMODAL_KEY</font></b><font color="#990000">(</font>modal_open_bracket<font color="#990000">,</font> casey_begin_keyboard_macro_recording<font color="#990000">,</font> write_and_auto_tab<font color="#990000">);</font>
<b><font color="#000000">DEFINE_BIMODAL_KEY</font></b><font color="#990000">(</font>modal_close_bracket<font color="#990000">,</font> casey_end_keyboard_macro_recording<font color="#990000">,</font> write_and_auto_tab<font color="#990000">);</font>
<b><font color="#000000">DEFINE_MODAL_KEY</font></b><font color="#990000">(</font>modal_a<font color="#990000">,</font> cmdid_write_character<font color="#990000">);</font> <i><font color="#9A1900">// TODO(casey): Arbitrary command + casey_quick_calc</font></i>
<b><font color="#000000">DEFINE_MODAL_KEY</font></b><font color="#990000">(</font>modal_b<font color="#990000">,</font> cmdid_interactive_switch_buffer<font color="#990000">);</font>
<b><font color="#000000">DEFINE_MODAL_KEY</font></b><font color="#990000">(</font>modal_c<font color="#990000">,</font> casey_find_corresponding_file<font color="#990000">);</font>
<b><font color="#000000">DEFINE_MODAL_KEY</font></b><font color="#990000">(</font>modal_d<font color="#990000">,</font> casey_kill_to_end_of_line<font color="#990000">);</font>
<b><font color="#000000">DEFINE_MODAL_KEY</font></b><font color="#990000">(</font>modal_e<font color="#990000">,</font> cmdid_write_character<font color="#990000">);</font> <i><font color="#9A1900">// TODO(casey): Available</font></i>
<b><font color="#000000">DEFINE_MODAL_KEY</font></b><font color="#990000">(</font>modal_f<font color="#990000">,</font> casey_paste_and_tab<font color="#990000">);</font>
<b><font color="#000000">DEFINE_MODAL_KEY</font></b><font color="#990000">(</font>modal_g<font color="#990000">,</font> goto_line<font color="#990000">);</font>
<b><font color="#000000">DEFINE_MODAL_KEY</font></b><font color="#990000">(</font>modal_h<font color="#990000">,</font> cmdid_auto_tab_range<font color="#990000">);</font>
<b><font color="#000000">DEFINE_MODAL_KEY</font></b><font color="#990000">(</font>modal_i<font color="#990000">,</font> cmdid_move_up<font color="#990000">);</font>
<b><font color="#000000">DEFINE_MODAL_KEY</font></b><font color="#990000">(</font>modal_j<font color="#990000">,</font> seek_white_or_token_left<font color="#990000">);</font>
<b><font color="#000000">DEFINE_MODAL_KEY</font></b><font color="#990000">(</font>modal_k<font color="#990000">,</font> cmdid_move_down<font color="#990000">);</font>
<b><font color="#000000">DEFINE_MODAL_KEY</font></b><font color="#990000">(</font>modal_l<font color="#990000">,</font> seek_white_or_token_right<font color="#990000">);</font>
<b><font color="#000000">DEFINE_MODAL_KEY</font></b><font color="#990000">(</font>modal_m<font color="#990000">,</font> casey_save_and_make_without_asking<font color="#990000">);</font>
<b><font color="#000000">DEFINE_MODAL_KEY</font></b><font color="#990000">(</font>modal_n<font color="#990000">,</font> casey_goto_next_error<font color="#990000">);</font>
<b><font color="#000000">DEFINE_MODAL_KEY</font></b><font color="#990000">(</font>modal_o<font color="#990000">,</font> query_replace<font color="#990000">);</font>
<b><font color="#000000">DEFINE_MODAL_KEY</font></b><font color="#990000">(</font>modal_p<font color="#990000">,</font> replace_in_range<font color="#990000">);</font>
<b><font color="#000000">DEFINE_MODAL_KEY</font></b><font color="#990000">(</font>modal_q<font color="#990000">,</font> cmdid_copy<font color="#990000">);</font>
<b><font color="#000000">DEFINE_MODAL_KEY</font></b><font color="#990000">(</font>modal_r<font color="#990000">,</font> reverse_search<font color="#990000">);</font> <i><font color="#9A1900">// NOTE(allen): I've modified my default search so you can use it now.</font></i>
<b><font color="#000000">DEFINE_MODAL_KEY</font></b><font color="#990000">(</font>modal_s<font color="#990000">,</font> search<font color="#990000">);</font>
<b><font color="#000000">DEFINE_MODAL_KEY</font></b><font color="#990000">(</font>modal_t<font color="#990000">,</font> casey_load_todo<font color="#990000">);</font>
<b><font color="#000000">DEFINE_MODAL_KEY</font></b><font color="#990000">(</font>modal_u<font color="#990000">,</font> cmdid_undo<font color="#990000">);</font>
<b><font color="#000000">DEFINE_MODAL_KEY</font></b><font color="#990000">(</font>modal_v<font color="#990000">,</font> casey_switch_buffer_other_window<font color="#990000">);</font>
<b><font color="#000000">DEFINE_MODAL_KEY</font></b><font color="#990000">(</font>modal_w<font color="#990000">,</font> cmdid_cut<font color="#990000">);</font>
<b><font color="#000000">DEFINE_MODAL_KEY</font></b><font color="#990000">(</font>modal_x<font color="#990000">,</font> casey_find_corresponding_file_other_window<font color="#990000">);</font>
<b><font color="#000000">DEFINE_MODAL_KEY</font></b><font color="#990000">(</font>modal_y<font color="#990000">,</font> cmdid_redo<font color="#990000">);</font>
<b><font color="#000000">DEFINE_MODAL_KEY</font></b><font color="#990000">(</font>modal_z<font color="#990000">,</font> cmdid_interactive_open<font color="#990000">);</font>

<b><font color="#000000">DEFINE_MODAL_KEY</font></b><font color="#990000">(</font>modal_1<font color="#990000">,</font> casey_build_search<font color="#990000">);</font> <i><font color="#9A1900">// TODO(casey): Shouldn't need to bind a key for this?</font></i>
<b><font color="#000000">DEFINE_MODAL_KEY</font></b><font color="#990000">(</font>modal_2<font color="#990000">,</font> cmdid_write_character<font color="#990000">);</font> <i><font color="#9A1900">// TODO(casey): Available</font></i>
<b><font color="#000000">DEFINE_MODAL_KEY</font></b><font color="#990000">(</font>modal_3<font color="#990000">,</font> cmdid_write_character<font color="#990000">);</font> <i><font color="#9A1900">// TODO(casey): Available</font></i>
<b><font color="#000000">DEFINE_MODAL_KEY</font></b><font color="#990000">(</font>modal_4<font color="#990000">,</font> cmdid_write_character<font color="#990000">);</font> <i><font color="#9A1900">// TODO(casey): Available</font></i>
<b><font color="#000000">DEFINE_MODAL_KEY</font></b><font color="#990000">(</font>modal_5<font color="#990000">,</font> cmdid_write_character<font color="#990000">);</font> <i><font color="#9A1900">// TODO(casey): Available</font></i>
<b><font color="#000000">DEFINE_MODAL_KEY</font></b><font color="#990000">(</font>modal_6<font color="#990000">,</font> cmdid_write_character<font color="#990000">);</font> <i><font color="#9A1900">// TODO(casey): Available</font></i>
<b><font color="#000000">DEFINE_MODAL_KEY</font></b><font color="#990000">(</font>modal_7<font color="#990000">,</font> cmdid_write_character<font color="#990000">);</font> <i><font color="#9A1900">// TODO(casey): Available</font></i>
<b><font color="#000000">DEFINE_MODAL_KEY</font></b><font color="#990000">(</font>modal_8<font color="#990000">,</font> cmdid_write_character<font color="#990000">);</font> <i><font color="#9A1900">// TODO(casey): Available</font></i>
<b><font color="#000000">DEFINE_MODAL_KEY</font></b><font color="#990000">(</font>modal_9<font color="#990000">,</font> cmdid_write_character<font color="#990000">);</font> <i><font color="#9A1900">// TODO(casey): Available</font></i>
<b><font color="#000000">DEFINE_MODAL_KEY</font></b><font color="#990000">(</font>modal_0<font color="#990000">,</font> cmdid_kill_buffer<font color="#990000">);</font>
<b><font color="#000000">DEFINE_MODAL_KEY</font></b><font color="#990000">(</font>modal_minus<font color="#990000">,</font> cmdid_write_character<font color="#990000">);</font> <i><font color="#9A1900">// TODO(casey): Available</font></i>
<b><font color="#000000">DEFINE_MODAL_KEY</font></b><font color="#990000">(</font>modal_equals<font color="#990000">,</font> casey_execute_arbitrary_command<font color="#990000">);</font>

<b><font color="#000000">DEFINE_BIMODAL_KEY</font></b><font color="#990000">(</font>modal_backspace<font color="#990000">,</font> casey_delete_token_left<font color="#990000">,</font> cmdid_backspace<font color="#990000">);</font>
<b><font color="#000000">DEFINE_BIMODAL_KEY</font></b><font color="#990000">(</font>modal_up<font color="#990000">,</font> cmdid_move_up<font color="#990000">,</font> cmdid_move_up<font color="#990000">);</font>
<b><font color="#000000">DEFINE_BIMODAL_KEY</font></b><font color="#990000">(</font>modal_down<font color="#990000">,</font> cmdid_move_down<font color="#990000">,</font> cmdid_move_down<font color="#990000">);</font>
<b><font color="#000000">DEFINE_BIMODAL_KEY</font></b><font color="#990000">(</font>modal_left<font color="#990000">,</font> seek_white_or_token_left<font color="#990000">,</font> cmdid_move_left<font color="#990000">);</font> 
<b><font color="#000000">DEFINE_BIMODAL_KEY</font></b><font color="#990000">(</font>modal_right<font color="#990000">,</font> seek_white_or_token_right<font color="#990000">,</font> cmdid_move_right<font color="#990000">);</font>
<b><font color="#000000">DEFINE_BIMODAL_KEY</font></b><font color="#990000">(</font>modal_delete<font color="#990000">,</font> casey_delete_token_right<font color="#990000">,</font> cmdid_delete<font color="#990000">);</font>
<b><font color="#000000">DEFINE_BIMODAL_KEY</font></b><font color="#990000">(</font>modal_home<font color="#990000">,</font> casey_seek_beginning_of_line<font color="#990000">,</font> casey_seek_beginning_of_line_and_tab<font color="#990000">);</font>
<b><font color="#000000">DEFINE_BIMODAL_KEY</font></b><font color="#990000">(</font>modal_end<font color="#990000">,</font> cmdid_seek_end_of_line<font color="#990000">,</font> cmdid_seek_end_of_line<font color="#990000">);</font>
<b><font color="#000000">DEFINE_BIMODAL_KEY</font></b><font color="#990000">(</font>modal_page_up<font color="#990000">,</font> cmdid_page_up<font color="#990000">,</font> cmdid_seek_whitespace_up<font color="#990000">);</font>
<b><font color="#000000">DEFINE_BIMODAL_KEY</font></b><font color="#990000">(</font>modal_page_down<font color="#990000">,</font> cmdid_page_down<font color="#990000">,</font> cmdid_seek_whitespace_down<font color="#990000">);</font>
<b><font color="#000000">DEFINE_BIMODAL_KEY</font></b><font color="#990000">(</font>modal_tab<font color="#990000">,</font> cmdid_word_complete<font color="#990000">,</font> cmdid_word_complete<font color="#990000">);</font>

<b><font color="#000000">HOOK_SIG</font></b><font color="#990000">(</font>casey_file_settings<font color="#990000">)</font>
<font color="#FF0000">{</font>
    <i><font color="#9A1900">// NOTE(allen|a4): As of alpha 4 hooks can have parameters which are</font></i>
    <i><font color="#9A1900">// received through functions like this app-&gt;get_parameter_buffer.</font></i>
    <i><font color="#9A1900">// This is different from the past where this hook got a buffer</font></i>
    <i><font color="#9A1900">// from app-&gt;get_active_buffer.</font></i>
    <font color="#008080">Buffer_Summary</font> buffer <font color="#990000">=</font> app<font color="#990000">-&gt;</font><b><font color="#000000">get_parameter_buffer</font></b><font color="#990000">(</font>app<font color="#990000">,</font> <font color="#993399">0</font><font color="#990000">);</font>

    <font color="#009900">int</font> treat_as_code <font color="#990000">=</font> <font color="#993399">0</font><font color="#990000">;</font>
    <font color="#009900">int</font> treat_as_project <font color="#990000">=</font> <font color="#993399">0</font><font color="#990000">;</font>

    <b><font color="#0000FF">if</font></b> <font color="#990000">(</font>buffer<font color="#990000">.</font>file_name <font color="#990000">&amp;&amp;</font> buffer<font color="#990000">.</font>size <font color="#990000">&lt;</font> <font color="#990000">(</font><font color="#993399">16</font> <font color="#990000">&lt;&lt;</font> <font color="#993399">20</font><font color="#990000">))</font>
    <font color="#FF0000">{</font>
        <font color="#008080">String</font> ext <font color="#990000">=</font> <b><font color="#000000">file_extension</font></b><font color="#990000">(</font><b><font color="#000000">make_string</font></b><font color="#990000">(</font>buffer<font color="#990000">.</font>file_name<font color="#990000">,</font> buffer<font color="#990000">.</font>file_name_len<font color="#990000">));</font>
        treat_as_code <font color="#990000">=</font> <b><font color="#000000">IsCode</font></b><font color="#990000">(</font>ext<font color="#990000">);</font>
        treat_as_project <font color="#990000">=</font> <b><font color="#000000">match</font></b><font color="#990000">(</font>ext<font color="#990000">,</font> <b><font color="#000000">make_lit_string</font></b><font color="#990000">(</font><font color="#FF0000">"prj"</font><font color="#990000">));</font>
    <font color="#FF0000">}</font>

    <b><font color="#000000">push_parameter</font></b><font color="#990000">(</font>app<font color="#990000">,</font> par_buffer_id<font color="#990000">,</font> buffer<font color="#990000">.</font>buffer_id<font color="#990000">);</font>
    <b><font color="#000000">push_parameter</font></b><font color="#990000">(</font>app<font color="#990000">,</font> par_lex_as_cpp_file<font color="#990000">,</font> treat_as_code<font color="#990000">);</font>
    <b><font color="#000000">push_parameter</font></b><font color="#990000">(</font>app<font color="#990000">,</font> par_wrap_lines<font color="#990000">,</font> <font color="#990000">!</font>treat_as_code<font color="#990000">);</font>
    <b><font color="#000000">push_parameter</font></b><font color="#990000">(</font>app<font color="#990000">,</font> par_key_mapid<font color="#990000">,</font> <font color="#990000">(</font>treat_as_code<font color="#990000">)?((</font><font color="#009900">int</font><font color="#990000">)</font>my_code_map<font color="#990000">):((</font><font color="#009900">int</font><font color="#990000">)</font>mapid_file<font color="#990000">));</font>
    <b><font color="#000000">exec_command</font></b><font color="#990000">(</font>app<font color="#990000">,</font> cmdid_set_settings<font color="#990000">);</font>

    <b><font color="#0000FF">if</font></b><font color="#990000">(</font>treat_as_project<font color="#990000">)</font>
    <font color="#FF0000">{</font>
        <b><font color="#000000">OpenProject</font></b><font color="#990000">(</font>app<font color="#990000">,</font> buffer<font color="#990000">.</font>file_name<font color="#990000">);</font>
        <i><font color="#9A1900">// NOTE(casey): Don't actually want to kill this, or you can never edit the project.</font></i>
<i><font color="#9A1900">//        exec_command(app, cmdid_kill_buffer);</font></i>

    <font color="#FF0000">}</font>

    <b><font color="#0000FF">return</font></b><font color="#990000">(</font><font color="#993399">0</font><font color="#990000">);</font>
<font color="#FF0000">}</font>

<font color="#009900">bool</font>
<b><font color="#000000">CubicUpdateFixedDuration1</font></b><font color="#990000">(</font><font color="#009900">float</font> <font color="#990000">*</font>P0<font color="#990000">,</font> <font color="#009900">float</font> <font color="#990000">*</font>V0<font color="#990000">,</font> <font color="#009900">float</font> P1<font color="#990000">,</font> <font color="#009900">float</font> V1<font color="#990000">,</font> <font color="#009900">float</font> Duration<font color="#990000">,</font> <font color="#009900">float</font> dt<font color="#990000">)</font>
<font color="#FF0000">{</font>
    <font color="#009900">bool</font> Result <font color="#990000">=</font> <b><font color="#0000FF">false</font></b><font color="#990000">;</font>

    <b><font color="#0000FF">if</font></b><font color="#990000">(</font>dt <font color="#990000">&gt;</font> <font color="#993399">0</font><font color="#990000">)</font>
    <font color="#FF0000">{</font>
        <b><font color="#0000FF">if</font></b><font color="#990000">(</font>Duration <font color="#990000">&lt;</font> dt<font color="#990000">)</font>
        <font color="#FF0000">{</font>
            <font color="#990000">*</font>P0 <font color="#990000">=</font> P1 <font color="#990000">+</font> <font color="#990000">(</font>dt <font color="#990000">-</font> Duration<font color="#990000">)*</font>V1<font color="#990000">;</font>
            <font color="#990000">*</font>V0 <font color="#990000">=</font> V1<font color="#990000">;</font>
            Result <font color="#990000">=</font> <b><font color="#0000FF">true</font></b><font color="#990000">;</font>
        <font color="#FF0000">}</font>
        <b><font color="#0000FF">else</font></b>
        <font color="#FF0000">{</font>
            <font color="#009900">float</font> t <font color="#990000">=</font> <font color="#990000">(</font>dt <font color="#990000">/</font> Duration<font color="#990000">);</font>
            <font color="#009900">float</font> u <font color="#990000">=</font> <font color="#990000">(</font><font color="#993399">1</font><font color="#990000">.</font>0f <font color="#990000">-</font> t<font color="#990000">);</font>

            <font color="#009900">float</font> C0 <font color="#990000">=</font> <font color="#993399">1</font><font color="#990000">*</font>u<font color="#990000">*</font>u<font color="#990000">*</font>u<font color="#990000">;</font>
            <font color="#009900">float</font> C1 <font color="#990000">=</font> <font color="#993399">3</font><font color="#990000">*</font>u<font color="#990000">*</font>u<font color="#990000">*</font>t<font color="#990000">;</font>
            <font color="#009900">float</font> C2 <font color="#990000">=</font> <font color="#993399">3</font><font color="#990000">*</font>u<font color="#990000">*</font>t<font color="#990000">*</font>t<font color="#990000">;</font>
            <font color="#009900">float</font> C3 <font color="#990000">=</font> <font color="#993399">1</font><font color="#990000">*</font>t<font color="#990000">*</font>t<font color="#990000">*</font>t<font color="#990000">;</font>

            <font color="#009900">float</font> dC0 <font color="#990000">=</font> <font color="#990000">-</font><font color="#993399">3</font><font color="#990000">*</font>u<font color="#990000">*</font>u<font color="#990000">;</font>
            <font color="#009900">float</font> dC1 <font color="#990000">=</font> <font color="#990000">-</font><font color="#993399">6</font><font color="#990000">*</font>u<font color="#990000">*</font>t <font color="#990000">+</font> <font color="#993399">3</font><font color="#990000">*</font>u<font color="#990000">*</font>u<font color="#990000">;</font>
            <font color="#009900">float</font> dC2 <font color="#990000">=</font>  <font color="#993399">6</font><font color="#990000">*</font>u<font color="#990000">*</font>t <font color="#990000">-</font> <font color="#993399">3</font><font color="#990000">*</font>t<font color="#990000">*</font>t<font color="#990000">;</font>
            <font color="#009900">float</font> dC3 <font color="#990000">=</font>  <font color="#993399">3</font><font color="#990000">*</font>t<font color="#990000">*</font>t<font color="#990000">;</font>

            <font color="#009900">float</font> B0 <font color="#990000">=</font> <font color="#990000">*</font>P0<font color="#990000">;</font>
            <font color="#009900">float</font> B1 <font color="#990000">=</font> <font color="#990000">*</font>P0 <font color="#990000">+</font> <font color="#990000">(</font>Duration <font color="#990000">/</font> <font color="#993399">3</font><font color="#990000">.</font>0f<font color="#990000">)</font> <font color="#990000">*</font> <font color="#990000">*</font>V0<font color="#990000">;</font>
            <font color="#009900">float</font> B2 <font color="#990000">=</font> P1 <font color="#990000">-</font> <font color="#990000">(</font>Duration <font color="#990000">/</font> <font color="#993399">3</font><font color="#990000">.</font>0f<font color="#990000">)</font> <font color="#990000">*</font> V1<font color="#990000">;</font>
            <font color="#009900">float</font> B3 <font color="#990000">=</font> P1<font color="#990000">;</font>

            <font color="#990000">*</font>P0 <font color="#990000">=</font> C0<font color="#990000">*</font>B0 <font color="#990000">+</font> C1<font color="#990000">*</font>B1 <font color="#990000">+</font> C2<font color="#990000">*</font>B2 <font color="#990000">+</font> C3<font color="#990000">*</font>B3<font color="#990000">;</font>
            <font color="#990000">*</font>V0 <font color="#990000">=</font> <font color="#990000">(</font>dC0<font color="#990000">*</font>B0 <font color="#990000">+</font> dC1<font color="#990000">*</font>B1 <font color="#990000">+</font> dC2<font color="#990000">*</font>B2 <font color="#990000">+</font> dC3<font color="#990000">*</font>B3<font color="#990000">)</font> <font color="#990000">*</font> <font color="#990000">(</font><font color="#993399">1</font><font color="#990000">.</font>0f <font color="#990000">/</font> Duration<font color="#990000">);</font>
        <font color="#FF0000">}</font>
    <font color="#FF0000">}</font>

    <b><font color="#0000FF">return</font></b><font color="#990000">(</font>Result<font color="#990000">);</font>
<font color="#FF0000">}</font>

<b><font color="#0000FF">struct</font></b> <font color="#008080">Casey_Scroll_Velocity</font>
<font color="#FF0000">{</font>
    <font color="#009900">float</font> x<font color="#990000">,</font> y<font color="#990000">,</font> t<font color="#990000">;</font>
<font color="#FF0000">}</font><font color="#990000">;</font>

<font color="#008080">Casey_Scroll_Velocity</font> casey_scroll_velocity_<font color="#990000">[</font><font color="#993399">16</font><font color="#990000">]</font> <font color="#990000">=</font> <font color="#FF0000">{</font><font color="#993399">0</font><font color="#FF0000">}</font><font color="#990000">;</font>
<font color="#008080">Casey_Scroll_Velocity</font> <font color="#990000">*</font>casey_scroll_velocity <font color="#990000">=</font> casey_scroll_velocity_<font color="#990000">;</font>

<b><font color="#000000">SCROLL_RULE_SIG</font></b><font color="#990000">(</font>casey_smooth_scroll_rule<font color="#990000">)</font><font color="#FF0000">{</font>
    <font color="#009900">float</font> dt <font color="#990000">=</font> <font color="#993399">1</font><font color="#990000">.</font>0f<font color="#990000">/</font><font color="#993399">60</font><font color="#990000">.</font>0f<font color="#990000">;</font> <i><font color="#9A1900">// TODO(casey): Why do I not get the timestep here?</font></i>
    <font color="#008080">Casey_Scroll_Velocity</font> <font color="#990000">*</font>velocity <font color="#990000">=</font> casey_scroll_velocity <font color="#990000">+</font> view_id<font color="#990000">;</font>
    <font color="#009900">int</font> result <font color="#990000">=</font> <font color="#993399">0</font><font color="#990000">;</font>
    <b><font color="#0000FF">if</font></b><font color="#990000">(</font>is_new_target<font color="#990000">)</font>
    <font color="#FF0000">{</font>
        <b><font color="#0000FF">if</font></b><font color="#990000">((*</font>scroll_x <font color="#990000">!=</font> target_x<font color="#990000">)</font> <font color="#990000">||</font>
            <font color="#990000">(*</font>scroll_y <font color="#990000">!=</font> target_y<font color="#990000">))</font>
        <font color="#FF0000">{</font>
            velocity<font color="#990000">-&gt;</font>t <font color="#990000">=</font> <font color="#993399">0</font><font color="#990000">.</font>1f<font color="#990000">;</font>
        <font color="#FF0000">}</font>
    <font color="#FF0000">}</font>

    <b><font color="#0000FF">if</font></b><font color="#990000">(</font>velocity<font color="#990000">-&gt;</font>t <font color="#990000">&gt;</font> <font color="#993399">0</font><font color="#990000">)</font>
    <font color="#FF0000">{</font>
        result <font color="#990000">=</font> <font color="#990000">!(</font><b><font color="#000000">CubicUpdateFixedDuration1</font></b><font color="#990000">(</font>scroll_x<font color="#990000">,</font> <font color="#990000">&amp;</font>velocity<font color="#990000">-&gt;</font>x<font color="#990000">,</font> target_x<font color="#990000">,</font> <font color="#993399">0</font><font color="#990000">.</font>0f<font color="#990000">,</font> velocity<font color="#990000">-&gt;</font>t<font color="#990000">,</font> dt<font color="#990000">)</font> <font color="#990000">||</font>
                <b><font color="#000000">CubicUpdateFixedDuration1</font></b><font color="#990000">(</font>scroll_y<font color="#990000">,</font> <font color="#990000">&amp;</font>velocity<font color="#990000">-&gt;</font>y<font color="#990000">,</font> target_y<font color="#990000">,</font> <font color="#993399">0</font><font color="#990000">.</font>0f<font color="#990000">,</font> velocity<font color="#990000">-&gt;</font>t<font color="#990000">,</font> dt<font color="#990000">));</font>
    <font color="#FF0000">}</font>

    velocity<font color="#990000">-&gt;</font>t <font color="#990000">-=</font> dt<font color="#990000">;</font>
    <b><font color="#0000FF">if</font></b><font color="#990000">(</font>velocity<font color="#990000">-&gt;</font>t <font color="#990000">&lt;</font> <font color="#993399">0</font><font color="#990000">)</font>
    <font color="#FF0000">{</font>
        velocity<font color="#990000">-&gt;</font>t <font color="#990000">=</font> <font color="#993399">0</font><font color="#990000">;</font>
        <font color="#990000">*</font>scroll_x <font color="#990000">=</font> target_x<font color="#990000">;</font>
        <font color="#990000">*</font>scroll_y <font color="#990000">=</font> target_y<font color="#990000">;</font>
    <font color="#FF0000">}</font>

    <b><font color="#0000FF">return</font></b><font color="#990000">(</font>result<font color="#990000">);</font>
<font color="#FF0000">}</font>

<b><font color="#000080">#include</font></b> <font color="#FF0000">&lt;windows.h&gt;</font>
<b><font color="#000080">#pragma</font></b> <b><font color="#000000">comment</font></b><font color="#990000">(</font>lib<font color="#990000">,</font> <font color="#FF0000">"user32.lib"</font><font color="#990000">)</font>
<b><font color="#0000FF">static</font></b> <font color="#008080">HWND</font> GlobalWindowHandle<font color="#990000">;</font>
<b><font color="#0000FF">static</font></b> <font color="#008080">WINDOWPLACEMENT</font> GlobalWindowPosition <font color="#990000">=</font> <font color="#FF0000">{</font><b><font color="#0000FF">sizeof</font></b><font color="#990000">(</font>GlobalWindowPosition<font color="#990000">)</font><font color="#FF0000">}</font><font color="#990000">;</font>
internal BOOL <font color="#008080">CALLBACK</font> <b><font color="#000000">win32_find_4coder_window</font></b><font color="#990000">(</font><font color="#008080">HWND</font> Window<font color="#990000">,</font> <font color="#008080">LPARAM</font> LParam<font color="#990000">)</font>
<font color="#FF0000">{</font>
    <font color="#008080">BOOL</font> Result <font color="#990000">=</font> TRUE<font color="#990000">;</font>

    <font color="#009900">char</font> TestClassName<font color="#990000">[</font><font color="#993399">256</font><font color="#990000">];</font>
    <b><font color="#000000">GetClassName</font></b><font color="#990000">(</font>Window<font color="#990000">,</font> TestClassName<font color="#990000">,</font> <b><font color="#0000FF">sizeof</font></b><font color="#990000">(</font>TestClassName<font color="#990000">));</font>
    <b><font color="#0000FF">if</font></b><font color="#990000">((</font><b><font color="#000000">strcmp</font></b><font color="#990000">(</font><font color="#FF0000">"4coder-win32-wndclass"</font><font color="#990000">,</font> TestClassName<font color="#990000">)</font> <font color="#990000">==</font> <font color="#993399">0</font><font color="#990000">)</font> <font color="#990000">&amp;&amp;</font> 
       <font color="#990000">((</font>HINSTANCE<font color="#990000">)</font><b><font color="#000000">GetWindowLongPtr</font></b><font color="#990000">(</font>Window<font color="#990000">,</font> GWLP_HINSTANCE<font color="#990000">)</font> <font color="#990000">==</font> <b><font color="#000000">GetModuleHandle</font></b><font color="#990000">(</font><font color="#993399">0</font><font color="#990000">)))</font>
    <font color="#FF0000">{</font>
        GlobalWindowHandle <font color="#990000">=</font> Window<font color="#990000">;</font>
        Result <font color="#990000">=</font> FALSE<font color="#990000">;</font>
    <font color="#FF0000">}</font>

    <b><font color="#0000FF">return</font></b><font color="#990000">(</font>Result<font color="#990000">);</font>
<font color="#FF0000">}</font>

internal <font color="#009900">void</font>
<b><font color="#000000">win32_toggle_fullscreen</font></b><font color="#990000">(</font><font color="#009900">void</font><font color="#990000">)</font>
<font color="#FF0000">{</font>
<b><font color="#000080">#if</font></b> <font color="#993399">0</font>
    <i><font color="#9A1900">// NOTE(casey): This follows Raymond Chen's prescription</font></i>
    <i><font color="#9A1900">// for fullscreen toggling, see:</font></i>
    <i><font color="#9A1900">// http://blogs.msdn.com/b/oldnewthing/archive/2010/04/12/9994016.aspx</font></i>

    <font color="#008080">HWND</font> Window <font color="#990000">=</font> GlobalWindowHandle<font color="#990000">;</font>
    <font color="#008080">DWORD</font> Style <font color="#990000">=</font> <b><font color="#000000">GetWindowLong</font></b><font color="#990000">(</font>Window<font color="#990000">,</font> GWL_STYLE<font color="#990000">);</font>
    <b><font color="#0000FF">if</font></b><font color="#990000">(</font>Style <font color="#990000">&amp;</font> WS_OVERLAPPEDWINDOW<font color="#990000">)</font>
    <font color="#FF0000">{</font>
        <font color="#008080">MONITORINFO</font> MonitorInfo <font color="#990000">=</font> <font color="#FF0000">{</font><b><font color="#0000FF">sizeof</font></b><font color="#990000">(</font>MonitorInfo<font color="#990000">)</font><font color="#FF0000">}</font><font color="#990000">;</font>
        <b><font color="#0000FF">if</font></b><font color="#990000">(</font><b><font color="#000000">GetWindowPlacement</font></b><font color="#990000">(</font>Window<font color="#990000">,</font> <font color="#990000">&amp;</font>GlobalWindowPosition<font color="#990000">)</font> <font color="#990000">&amp;&amp;</font>
            <b><font color="#000000">GetMonitorInfo</font></b><font color="#990000">(</font><b><font color="#000000">MonitorFromWindow</font></b><font color="#990000">(</font>Window<font color="#990000">,</font> MONITOR_DEFAULTTOPRIMARY<font color="#990000">),</font> <font color="#990000">&amp;</font>MonitorInfo<font color="#990000">))</font>
        <font color="#FF0000">{</font>
            <b><font color="#000000">SetWindowLong</font></b><font color="#990000">(</font>Window<font color="#990000">,</font> GWL_STYLE<font color="#990000">,</font> Style <font color="#990000">&amp;</font> <font color="#990000">~</font>WS_OVERLAPPEDWINDOW<font color="#990000">);</font>
            <b><font color="#000000">SetWindowPos</font></b><font color="#990000">(</font>Window<font color="#990000">,</font> HWND_TOP<font color="#990000">,</font>
                            MonitorInfo<font color="#990000">.</font>rcMonitor<font color="#990000">.</font>left<font color="#990000">,</font> MonitorInfo<font color="#990000">.</font>rcMonitor<font color="#990000">.</font>top<font color="#990000">,</font>
                            MonitorInfo<font color="#990000">.</font>rcMonitor<font color="#990000">.</font>right <font color="#990000">-</font> MonitorInfo<font color="#990000">.</font>rcMonitor<font color="#990000">.</font>left<font color="#990000">,</font>
                            MonitorInfo<font color="#990000">.</font>rcMonitor<font color="#990000">.</font>bottom <font color="#990000">-</font> MonitorInfo<font color="#990000">.</font>rcMonitor<font color="#990000">.</font>top<font color="#990000">,</font>
                            SWP_NOOWNERZORDER <font color="#990000">|</font> SWP_FRAMECHANGED<font color="#990000">);</font>
        <font color="#FF0000">}</font>
    <font color="#FF0000">}</font>
    <b><font color="#0000FF">else</font></b>
    <font color="#FF0000">{</font>
        <b><font color="#000000">SetWindowLong</font></b><font color="#990000">(</font>Window<font color="#990000">,</font> GWL_STYLE<font color="#990000">,</font> Style <font color="#990000">|</font> WS_OVERLAPPEDWINDOW<font color="#990000">);</font>
        <b><font color="#000000">SetWindowPlacement</font></b><font color="#990000">(</font>Window<font color="#990000">,</font> <font color="#990000">&amp;</font>GlobalWindowPosition<font color="#990000">);</font>
        <b><font color="#000000">SetWindowPos</font></b><font color="#990000">(</font>Window<font color="#990000">,</font> <font color="#993399">0</font><font color="#990000">,</font> <font color="#993399">0</font><font color="#990000">,</font> <font color="#993399">0</font><font color="#990000">,</font> <font color="#993399">0</font><font color="#990000">,</font> <font color="#993399">0</font><font color="#990000">,</font>
                        SWP_NOMOVE <font color="#990000">|</font> SWP_NOSIZE <font color="#990000">|</font> SWP_NOZORDER <font color="#990000">|</font>
                        SWP_NOOWNERZORDER <font color="#990000">|</font> SWP_FRAMECHANGED<font color="#990000">);</font>
    <font color="#FF0000">}</font>
<b><font color="#000080">#else</font></b>
    <b><font color="#000000">ShowWindow</font></b><font color="#990000">(</font>GlobalWindowHandle<font color="#990000">,</font> SW_MAXIMIZE<font color="#990000">);</font>
<b><font color="#000080">#endif</font></b>
<font color="#FF0000">}</font>

<b><font color="#000000">HOOK_SIG</font></b><font color="#990000">(</font>casey_start<font color="#990000">)</font>
<font color="#FF0000">{</font>
    <b><font color="#000000">exec_command</font></b><font color="#990000">(</font>app<font color="#990000">,</font> cmdid_open_panel_vsplit<font color="#990000">);</font>
    app<font color="#990000">-&gt;</font><b><font color="#000000">change_theme</font></b><font color="#990000">(</font>app<font color="#990000">,</font> <b><font color="#000000">literal</font></b><font color="#990000">(</font><font color="#FF0000">"Handmade Hero"</font><font color="#990000">));</font>
    app<font color="#990000">-&gt;</font><b><font color="#000000">change_font</font></b><font color="#990000">(</font>app<font color="#990000">,</font> <b><font color="#000000">literal</font></b><font color="#990000">(</font><font color="#FF0000">"liberation mono"</font><font color="#990000">));</font>

    <font color="#008080">Theme_Color</font> colors<font color="#990000">[]</font> <font color="#990000">=</font>
    <font color="#FF0000">{</font>
        <font color="#FF0000">{</font>Stag_Default<font color="#990000">,</font> <font color="#993399">0xA08563</font><font color="#FF0000">}</font><font color="#990000">,</font>
        <i><font color="#9A1900">// {Stag_Bar, },</font></i>
        <i><font color="#9A1900">// {Stag_Bar_Active, },</font></i>
        <i><font color="#9A1900">// {Stag_Base, },</font></i>
        <i><font color="#9A1900">// {Stag_Pop1, },</font></i>
        <i><font color="#9A1900">// {Stag_Pop2, },</font></i>
        <i><font color="#9A1900">// {Stag_Back, },</font></i>
        <i><font color="#9A1900">// {Stag_Margin, },</font></i>
        <i><font color="#9A1900">// {Stag_Margin_Hover, },</font></i>
        <i><font color="#9A1900">// {Stag_Margin_Active, },</font></i>
        <i><font color="#9A1900">// {Stag_Cursor, },</font></i>
        <i><font color="#9A1900">// {Stag_At_Cursor, },</font></i>
        <i><font color="#9A1900">// {Stag_Highlight, },</font></i>
        <i><font color="#9A1900">// {Stag_At_Highlight, },</font></i>
        <font color="#FF0000">{</font>Stag_Comment<font color="#990000">,</font> <font color="#993399">0x7D7D7D</font><font color="#FF0000">}</font><font color="#990000">,</font>
        <font color="#FF0000">{</font>Stag_Keyword<font color="#990000">,</font> <font color="#993399">0xCD950C</font><font color="#FF0000">}</font><font color="#990000">,</font>
        <i><font color="#9A1900">// {Stag_Str_Constant, },</font></i>
        <i><font color="#9A1900">// {Stag_Char_Constant, },</font></i>
        <i><font color="#9A1900">// {Stag_Int_Constant, },</font></i>
        <i><font color="#9A1900">// {Stag_Float_Constant, },</font></i>
        <i><font color="#9A1900">// {Stag_Bool_Constant, },</font></i>
        <font color="#FF0000">{</font>Stag_Preproc<font color="#990000">,</font> <font color="#993399">0xDAB98F</font><font color="#FF0000">}</font><font color="#990000">,</font>
        <font color="#FF0000">{</font>Stag_Include<font color="#990000">,</font> <font color="#993399">0x6B8E23</font><font color="#FF0000">}</font><font color="#990000">,</font>
        <i><font color="#9A1900">// {Stag_Special_Character, },</font></i>
        <i><font color="#9A1900">// {Stag_Highlight_Junk, },</font></i>
        <i><font color="#9A1900">// {Stag_Highlight_White, },</font></i>
        <i><font color="#9A1900">// {Stag_Paste, },</font></i>
        <i><font color="#9A1900">// {Stag_Undo, },</font></i>
        <i><font color="#9A1900">// {Stag_Next_Undo, },</font></i>
    <font color="#FF0000">}</font><font color="#990000">;</font>
    app<font color="#990000">-&gt;</font><b><font color="#000000">set_theme_colors</font></b><font color="#990000">(</font>app<font color="#990000">,</font> colors<font color="#990000">,</font> <b><font color="#000000">ArrayCount</font></b><font color="#990000">(</font>colors<font color="#990000">));</font>

    <b><font color="#000000">win32_toggle_fullscreen</font></b><font color="#990000">();</font>

    <b><font color="#0000FF">return</font></b><font color="#990000">(</font><font color="#993399">0</font><font color="#990000">);</font>
<font color="#FF0000">}</font>

<b><font color="#0000FF">extern</font></b> <font color="#FF0000">"C"</font> <b><font color="#000000">GET_BINDING_DATA</font></b><font color="#990000">(</font>get_bindings<font color="#990000">)</font>
<font color="#FF0000">{</font>
    <font color="#008080">Bind_Helper</font> context_actual <font color="#990000">=</font> <b><font color="#000000">begin_bind_helper</font></b><font color="#990000">(</font>data<font color="#990000">,</font> size<font color="#990000">);</font>
    <font color="#008080">Bind_Helper</font> <font color="#990000">*</font>context <font color="#990000">=</font> <font color="#990000">&amp;</font>context_actual<font color="#990000">;</font>

    <b><font color="#000000">set_hook</font></b><font color="#990000">(</font>context<font color="#990000">,</font> hook_start<font color="#990000">,</font> casey_start<font color="#990000">);</font>
    <b><font color="#000000">set_hook</font></b><font color="#990000">(</font>context<font color="#990000">,</font> hook_open_file<font color="#990000">,</font> casey_file_settings<font color="#990000">);</font>
    <b><font color="#000000">set_scroll_rule</font></b><font color="#990000">(</font>context<font color="#990000">,</font> casey_smooth_scroll_rule<font color="#990000">);</font>

    <b><font color="#000000">EnumWindows</font></b><font color="#990000">(</font>win32_find_4coder_window<font color="#990000">,</font> <font color="#993399">0</font><font color="#990000">);</font>

    <b><font color="#000000">begin_map</font></b><font color="#990000">(</font>context<font color="#990000">,</font> mapid_global<font color="#990000">);</font>
    <font color="#FF0000">{</font>
        <b><font color="#000000">bind</font></b><font color="#990000">(</font>context<font color="#990000">,</font> <font color="#FF0000">'z'</font><font color="#990000">,</font> MDFR_NONE<font color="#990000">,</font> cmdid_interactive_open<font color="#990000">);</font>
        <b><font color="#000000">bind</font></b><font color="#990000">(</font>context<font color="#990000">,</font> <font color="#FF0000">'x'</font><font color="#990000">,</font> MDFR_NONE<font color="#990000">,</font> casey_open_in_other<font color="#990000">);</font>
        <b><font color="#000000">bind</font></b><font color="#990000">(</font>context<font color="#990000">,</font> <font color="#FF0000">'t'</font><font color="#990000">,</font> MDFR_NONE<font color="#990000">,</font> casey_load_todo<font color="#990000">);</font>
        <b><font color="#000000">bind</font></b><font color="#990000">(</font>context<font color="#990000">,</font> <font color="#FF0000">'/'</font><font color="#990000">,</font> MDFR_NONE<font color="#990000">,</font> cmdid_change_active_panel<font color="#990000">);</font>
        <b><font color="#000000">bind</font></b><font color="#990000">(</font>context<font color="#990000">,</font> <font color="#FF0000">'b'</font><font color="#990000">,</font> MDFR_NONE<font color="#990000">,</font> cmdid_interactive_switch_buffer<font color="#990000">);</font>
        <b><font color="#000000">bind</font></b><font color="#990000">(</font>context<font color="#990000">,</font> key_page_up<font color="#990000">,</font> MDFR_NONE<font color="#990000">,</font> search<font color="#990000">);</font>
        <b><font color="#000000">bind</font></b><font color="#990000">(</font>context<font color="#990000">,</font> key_page_down<font color="#990000">,</font> MDFR_NONE<font color="#990000">,</font> reverse_search<font color="#990000">);</font>
        <b><font color="#000000">bind</font></b><font color="#990000">(</font>context<font color="#990000">,</font> <font color="#FF0000">'m'</font><font color="#990000">,</font> MDFR_NONE<font color="#990000">,</font> casey_save_and_make_without_asking<font color="#990000">);</font>
    <font color="#FF0000">}</font>        
    <b><font color="#000000">end_map</font></b><font color="#990000">(</font>context<font color="#990000">);</font>

    <b><font color="#000000">begin_map</font></b><font color="#990000">(</font>context<font color="#990000">,</font> mapid_file<font color="#990000">);</font>

    <b><font color="#000000">bind_vanilla_keys</font></b><font color="#990000">(</font>context<font color="#990000">,</font> cmdid_write_character<font color="#990000">);</font>

    <b><font color="#000000">bind</font></b><font color="#990000">(</font>context<font color="#990000">,</font> key_insert<font color="#990000">,</font> MDFR_NONE<font color="#990000">,</font> begin_free_typing<font color="#990000">);</font>
    <b><font color="#000000">bind</font></b><font color="#990000">(</font>context<font color="#990000">,</font> <font color="#FF0000">'`'</font><font color="#990000">,</font> MDFR_NONE<font color="#990000">,</font> begin_free_typing<font color="#990000">);</font>
    <b><font color="#000000">bind</font></b><font color="#990000">(</font>context<font color="#990000">,</font> key_esc<font color="#990000">,</font> MDFR_NONE<font color="#990000">,</font> end_free_typing<font color="#990000">);</font>
    <b><font color="#000000">bind</font></b><font color="#990000">(</font>context<font color="#990000">,</font> <font color="#FF0000">'</font><font color="#CC33CC">\n</font><font color="#FF0000">'</font><font color="#990000">,</font> MDFR_NONE<font color="#990000">,</font> casey_newline_and_indent<font color="#990000">);</font>
    <b><font color="#000000">bind</font></b><font color="#990000">(</font>context<font color="#990000">,</font> <font color="#FF0000">'</font><font color="#CC33CC">\n</font><font color="#FF0000">'</font><font color="#990000">,</font> MDFR_SHIFT<font color="#990000">,</font> casey_newline_and_indent<font color="#990000">);</font>

    <i><font color="#9A1900">// NOTE(casey): Modal keys come here.</font></i>
    <b><font color="#000000">bind</font></b><font color="#990000">(</font>context<font color="#990000">,</font> <font color="#FF0000">' '</font><font color="#990000">,</font> MDFR_NONE<font color="#990000">,</font> modal_space<font color="#990000">);</font>
    <b><font color="#000000">bind</font></b><font color="#990000">(</font>context<font color="#990000">,</font> <font color="#FF0000">' '</font><font color="#990000">,</font> MDFR_SHIFT<font color="#990000">,</font> modal_space<font color="#990000">);</font>

    <b><font color="#000000">bind</font></b><font color="#990000">(</font>context<font color="#990000">,</font> <font color="#FF0000">'</font><font color="#CC33CC">\\</font><font color="#FF0000">'</font><font color="#990000">,</font> MDFR_NONE<font color="#990000">,</font> modal_back_slash<font color="#990000">);</font>
    <b><font color="#000000">bind</font></b><font color="#990000">(</font>context<font color="#990000">,</font> <font color="#FF0000">'</font><font color="#CC33CC">\'</font><font color="#FF0000">'</font><font color="#990000">,</font> MDFR_NONE<font color="#990000">,</font> modal_single_quote<font color="#990000">);</font>
    <b><font color="#000000">bind</font></b><font color="#990000">(</font>context<font color="#990000">,</font> <font color="#FF0000">','</font><font color="#990000">,</font> MDFR_NONE<font color="#990000">,</font> modal_comma<font color="#990000">);</font>
    <b><font color="#000000">bind</font></b><font color="#990000">(</font>context<font color="#990000">,</font> <font color="#FF0000">'.'</font><font color="#990000">,</font> MDFR_NONE<font color="#990000">,</font> modal_period<font color="#990000">);</font>
    <b><font color="#000000">bind</font></b><font color="#990000">(</font>context<font color="#990000">,</font> <font color="#FF0000">'/'</font><font color="#990000">,</font> MDFR_NONE<font color="#990000">,</font> modal_forward_slash<font color="#990000">);</font>
    <b><font color="#000000">bind</font></b><font color="#990000">(</font>context<font color="#990000">,</font> <font color="#FF0000">';'</font><font color="#990000">,</font> MDFR_NONE<font color="#990000">,</font> modal_semicolon<font color="#990000">);</font>
    <b><font color="#000000">bind</font></b><font color="#990000">(</font>context<font color="#990000">,</font> <font color="#FF0000">'['</font><font color="#990000">,</font> MDFR_NONE<font color="#990000">,</font> modal_open_bracket<font color="#990000">);</font>
    <b><font color="#000000">bind</font></b><font color="#990000">(</font>context<font color="#990000">,</font> <font color="#FF0000">']'</font><font color="#990000">,</font> MDFR_NONE<font color="#990000">,</font> modal_close_bracket<font color="#990000">);</font>
    <b><font color="#000000">bind</font></b><font color="#990000">(</font>context<font color="#990000">,</font> <font color="#FF0000">'{'</font><font color="#990000">,</font> MDFR_NONE<font color="#990000">,</font> write_and_auto_tab<font color="#990000">);</font>
    <b><font color="#000000">bind</font></b><font color="#990000">(</font>context<font color="#990000">,</font> <font color="#FF0000">'}'</font><font color="#990000">,</font> MDFR_NONE<font color="#990000">,</font> write_and_auto_tab<font color="#990000">);</font>
    <b><font color="#000000">bind</font></b><font color="#990000">(</font>context<font color="#990000">,</font> <font color="#FF0000">'a'</font><font color="#990000">,</font> MDFR_NONE<font color="#990000">,</font> modal_a<font color="#990000">);</font>
    <b><font color="#000000">bind</font></b><font color="#990000">(</font>context<font color="#990000">,</font> <font color="#FF0000">'b'</font><font color="#990000">,</font> MDFR_NONE<font color="#990000">,</font> modal_b<font color="#990000">);</font>
    <b><font color="#000000">bind</font></b><font color="#990000">(</font>context<font color="#990000">,</font> <font color="#FF0000">'c'</font><font color="#990000">,</font> MDFR_NONE<font color="#990000">,</font> modal_c<font color="#990000">);</font>
    <b><font color="#000000">bind</font></b><font color="#990000">(</font>context<font color="#990000">,</font> <font color="#FF0000">'d'</font><font color="#990000">,</font> MDFR_NONE<font color="#990000">,</font> modal_d<font color="#990000">);</font>
    <b><font color="#000000">bind</font></b><font color="#990000">(</font>context<font color="#990000">,</font> <font color="#FF0000">'e'</font><font color="#990000">,</font> MDFR_NONE<font color="#990000">,</font> modal_e<font color="#990000">);</font>
    <b><font color="#000000">bind</font></b><font color="#990000">(</font>context<font color="#990000">,</font> <font color="#FF0000">'f'</font><font color="#990000">,</font> MDFR_NONE<font color="#990000">,</font> modal_f<font color="#990000">);</font>
    <b><font color="#000000">bind</font></b><font color="#990000">(</font>context<font color="#990000">,</font> <font color="#FF0000">'g'</font><font color="#990000">,</font> MDFR_NONE<font color="#990000">,</font> modal_g<font color="#990000">);</font>
    <b><font color="#000000">bind</font></b><font color="#990000">(</font>context<font color="#990000">,</font> <font color="#FF0000">'h'</font><font color="#990000">,</font> MDFR_NONE<font color="#990000">,</font> modal_h<font color="#990000">);</font>
    <b><font color="#000000">bind</font></b><font color="#990000">(</font>context<font color="#990000">,</font> <font color="#FF0000">'i'</font><font color="#990000">,</font> MDFR_NONE<font color="#990000">,</font> modal_i<font color="#990000">);</font>
    <b><font color="#000000">bind</font></b><font color="#990000">(</font>context<font color="#990000">,</font> <font color="#FF0000">'j'</font><font color="#990000">,</font> MDFR_NONE<font color="#990000">,</font> modal_j<font color="#990000">);</font>
    <b><font color="#000000">bind</font></b><font color="#990000">(</font>context<font color="#990000">,</font> <font color="#FF0000">'k'</font><font color="#990000">,</font> MDFR_NONE<font color="#990000">,</font> modal_k<font color="#990000">);</font>
    <b><font color="#000000">bind</font></b><font color="#990000">(</font>context<font color="#990000">,</font> <font color="#FF0000">'l'</font><font color="#990000">,</font> MDFR_NONE<font color="#990000">,</font> modal_l<font color="#990000">);</font>
    <b><font color="#000000">bind</font></b><font color="#990000">(</font>context<font color="#990000">,</font> <font color="#FF0000">'m'</font><font color="#990000">,</font> MDFR_NONE<font color="#990000">,</font> modal_m<font color="#990000">);</font>
    <b><font color="#000000">bind</font></b><font color="#990000">(</font>context<font color="#990000">,</font> <font color="#FF0000">'n'</font><font color="#990000">,</font> MDFR_NONE<font color="#990000">,</font> modal_n<font color="#990000">);</font>
    <b><font color="#000000">bind</font></b><font color="#990000">(</font>context<font color="#990000">,</font> <font color="#FF0000">'o'</font><font color="#990000">,</font> MDFR_NONE<font color="#990000">,</font> modal_o<font color="#990000">);</font>
    <b><font color="#000000">bind</font></b><font color="#990000">(</font>context<font color="#990000">,</font> <font color="#FF0000">'p'</font><font color="#990000">,</font> MDFR_NONE<font color="#990000">,</font> modal_p<font color="#990000">);</font>
    <b><font color="#000000">bind</font></b><font color="#990000">(</font>context<font color="#990000">,</font> <font color="#FF0000">'q'</font><font color="#990000">,</font> MDFR_NONE<font color="#990000">,</font> modal_q<font color="#990000">);</font>
    <b><font color="#000000">bind</font></b><font color="#990000">(</font>context<font color="#990000">,</font> <font color="#FF0000">'r'</font><font color="#990000">,</font> MDFR_NONE<font color="#990000">,</font> modal_r<font color="#990000">);</font>
    <b><font color="#000000">bind</font></b><font color="#990000">(</font>context<font color="#990000">,</font> <font color="#FF0000">'s'</font><font color="#990000">,</font> MDFR_NONE<font color="#990000">,</font> modal_s<font color="#990000">);</font>
    <b><font color="#000000">bind</font></b><font color="#990000">(</font>context<font color="#990000">,</font> <font color="#FF0000">'t'</font><font color="#990000">,</font> MDFR_NONE<font color="#990000">,</font> modal_t<font color="#990000">);</font>
    <b><font color="#000000">bind</font></b><font color="#990000">(</font>context<font color="#990000">,</font> <font color="#FF0000">'u'</font><font color="#990000">,</font> MDFR_NONE<font color="#990000">,</font> modal_u<font color="#990000">);</font>
    <b><font color="#000000">bind</font></b><font color="#990000">(</font>context<font color="#990000">,</font> <font color="#FF0000">'v'</font><font color="#990000">,</font> MDFR_NONE<font color="#990000">,</font> modal_v<font color="#990000">);</font>
    <b><font color="#000000">bind</font></b><font color="#990000">(</font>context<font color="#990000">,</font> <font color="#FF0000">'w'</font><font color="#990000">,</font> MDFR_NONE<font color="#990000">,</font> modal_w<font color="#990000">);</font>
    <b><font color="#000000">bind</font></b><font color="#990000">(</font>context<font color="#990000">,</font> <font color="#FF0000">'x'</font><font color="#990000">,</font> MDFR_NONE<font color="#990000">,</font> modal_x<font color="#990000">);</font>
    <b><font color="#000000">bind</font></b><font color="#990000">(</font>context<font color="#990000">,</font> <font color="#FF0000">'y'</font><font color="#990000">,</font> MDFR_NONE<font color="#990000">,</font> modal_y<font color="#990000">);</font>
    <b><font color="#000000">bind</font></b><font color="#990000">(</font>context<font color="#990000">,</font> <font color="#FF0000">'z'</font><font color="#990000">,</font> MDFR_NONE<font color="#990000">,</font> modal_z<font color="#990000">);</font>

    <b><font color="#000000">bind</font></b><font color="#990000">(</font>context<font color="#990000">,</font> <font color="#FF0000">'1'</font><font color="#990000">,</font> MDFR_NONE<font color="#990000">,</font> modal_1<font color="#990000">);</font>
    <b><font color="#000000">bind</font></b><font color="#990000">(</font>context<font color="#990000">,</font> <font color="#FF0000">'2'</font><font color="#990000">,</font> MDFR_NONE<font color="#990000">,</font> modal_2<font color="#990000">);</font>
    <b><font color="#000000">bind</font></b><font color="#990000">(</font>context<font color="#990000">,</font> <font color="#FF0000">'3'</font><font color="#990000">,</font> MDFR_NONE<font color="#990000">,</font> modal_3<font color="#990000">);</font>
    <b><font color="#000000">bind</font></b><font color="#990000">(</font>context<font color="#990000">,</font> <font color="#FF0000">'4'</font><font color="#990000">,</font> MDFR_NONE<font color="#990000">,</font> modal_4<font color="#990000">);</font>
    <b><font color="#000000">bind</font></b><font color="#990000">(</font>context<font color="#990000">,</font> <font color="#FF0000">'5'</font><font color="#990000">,</font> MDFR_NONE<font color="#990000">,</font> modal_5<font color="#990000">);</font>
    <b><font color="#000000">bind</font></b><font color="#990000">(</font>context<font color="#990000">,</font> <font color="#FF0000">'6'</font><font color="#990000">,</font> MDFR_NONE<font color="#990000">,</font> modal_6<font color="#990000">);</font>
    <b><font color="#000000">bind</font></b><font color="#990000">(</font>context<font color="#990000">,</font> <font color="#FF0000">'7'</font><font color="#990000">,</font> MDFR_NONE<font color="#990000">,</font> modal_7<font color="#990000">);</font>
    <b><font color="#000000">bind</font></b><font color="#990000">(</font>context<font color="#990000">,</font> <font color="#FF0000">'8'</font><font color="#990000">,</font> MDFR_NONE<font color="#990000">,</font> modal_8<font color="#990000">);</font>
    <b><font color="#000000">bind</font></b><font color="#990000">(</font>context<font color="#990000">,</font> <font color="#FF0000">'9'</font><font color="#990000">,</font> MDFR_NONE<font color="#990000">,</font> modal_9<font color="#990000">);</font>
    <b><font color="#000000">bind</font></b><font color="#990000">(</font>context<font color="#990000">,</font> <font color="#FF0000">'0'</font><font color="#990000">,</font> MDFR_NONE<font color="#990000">,</font> modal_0<font color="#990000">);</font>
    <b><font color="#000000">bind</font></b><font color="#990000">(</font>context<font color="#990000">,</font> <font color="#FF0000">'-'</font><font color="#990000">,</font> MDFR_NONE<font color="#990000">,</font> modal_minus<font color="#990000">);</font>
    <b><font color="#000000">bind</font></b><font color="#990000">(</font>context<font color="#990000">,</font> <font color="#FF0000">'='</font><font color="#990000">,</font> MDFR_NONE<font color="#990000">,</font> modal_equals<font color="#990000">);</font>

    <b><font color="#000000">bind</font></b><font color="#990000">(</font>context<font color="#990000">,</font> key_back<font color="#990000">,</font> MDFR_NONE<font color="#990000">,</font> modal_backspace<font color="#990000">);</font>
    <b><font color="#000000">bind</font></b><font color="#990000">(</font>context<font color="#990000">,</font> key_back<font color="#990000">,</font> MDFR_SHIFT<font color="#990000">,</font> modal_backspace<font color="#990000">);</font>

    <b><font color="#000000">bind</font></b><font color="#990000">(</font>context<font color="#990000">,</font> key_up<font color="#990000">,</font> MDFR_NONE<font color="#990000">,</font> modal_up<font color="#990000">);</font>
    <b><font color="#000000">bind</font></b><font color="#990000">(</font>context<font color="#990000">,</font> key_up<font color="#990000">,</font> MDFR_SHIFT<font color="#990000">,</font> modal_up<font color="#990000">);</font>

    <b><font color="#000000">bind</font></b><font color="#990000">(</font>context<font color="#990000">,</font> key_down<font color="#990000">,</font> MDFR_NONE<font color="#990000">,</font> modal_down<font color="#990000">);</font>
    <b><font color="#000000">bind</font></b><font color="#990000">(</font>context<font color="#990000">,</font> key_down<font color="#990000">,</font> MDFR_SHIFT<font color="#990000">,</font> modal_down<font color="#990000">);</font>

    <b><font color="#000000">bind</font></b><font color="#990000">(</font>context<font color="#990000">,</font> key_left<font color="#990000">,</font> MDFR_NONE<font color="#990000">,</font> modal_left<font color="#990000">);</font>
    <b><font color="#000000">bind</font></b><font color="#990000">(</font>context<font color="#990000">,</font> key_left<font color="#990000">,</font> MDFR_SHIFT<font color="#990000">,</font> modal_left<font color="#990000">);</font>

    <b><font color="#000000">bind</font></b><font color="#990000">(</font>context<font color="#990000">,</font> key_right<font color="#990000">,</font> MDFR_NONE<font color="#990000">,</font> modal_right<font color="#990000">);</font>
    <b><font color="#000000">bind</font></b><font color="#990000">(</font>context<font color="#990000">,</font> key_right<font color="#990000">,</font> MDFR_SHIFT<font color="#990000">,</font> modal_right<font color="#990000">);</font>

    <b><font color="#000000">bind</font></b><font color="#990000">(</font>context<font color="#990000">,</font> key_del<font color="#990000">,</font> MDFR_NONE<font color="#990000">,</font> modal_delete<font color="#990000">);</font>
    <b><font color="#000000">bind</font></b><font color="#990000">(</font>context<font color="#990000">,</font> key_del<font color="#990000">,</font> MDFR_SHIFT<font color="#990000">,</font> modal_delete<font color="#990000">);</font>

    <b><font color="#000000">bind</font></b><font color="#990000">(</font>context<font color="#990000">,</font> key_home<font color="#990000">,</font> MDFR_NONE<font color="#990000">,</font> modal_home<font color="#990000">);</font>
    <b><font color="#000000">bind</font></b><font color="#990000">(</font>context<font color="#990000">,</font> key_home<font color="#990000">,</font> MDFR_SHIFT<font color="#990000">,</font> modal_home<font color="#990000">);</font>

    <b><font color="#000000">bind</font></b><font color="#990000">(</font>context<font color="#990000">,</font> key_end<font color="#990000">,</font> MDFR_NONE<font color="#990000">,</font> modal_end<font color="#990000">);</font>
    <b><font color="#000000">bind</font></b><font color="#990000">(</font>context<font color="#990000">,</font> key_end<font color="#990000">,</font> MDFR_SHIFT<font color="#990000">,</font> modal_end<font color="#990000">);</font>

    <b><font color="#000000">bind</font></b><font color="#990000">(</font>context<font color="#990000">,</font> key_page_up<font color="#990000">,</font> MDFR_NONE<font color="#990000">,</font> modal_page_up<font color="#990000">);</font>
    <b><font color="#000000">bind</font></b><font color="#990000">(</font>context<font color="#990000">,</font> key_page_up<font color="#990000">,</font> MDFR_SHIFT<font color="#990000">,</font> modal_page_up<font color="#990000">);</font>

    <b><font color="#000000">bind</font></b><font color="#990000">(</font>context<font color="#990000">,</font> key_page_down<font color="#990000">,</font> MDFR_NONE<font color="#990000">,</font> modal_page_down<font color="#990000">);</font>
    <b><font color="#000000">bind</font></b><font color="#990000">(</font>context<font color="#990000">,</font> key_page_down<font color="#990000">,</font> MDFR_SHIFT<font color="#990000">,</font> modal_page_down<font color="#990000">);</font>

    <b><font color="#000000">bind</font></b><font color="#990000">(</font>context<font color="#990000">,</font> <font color="#FF0000">'</font><font color="#CC33CC">\t</font><font color="#FF0000">'</font><font color="#990000">,</font> MDFR_NONE<font color="#990000">,</font> modal_tab<font color="#990000">);</font>
    <b><font color="#000000">bind</font></b><font color="#990000">(</font>context<font color="#990000">,</font> <font color="#FF0000">'</font><font color="#CC33CC">\t</font><font color="#FF0000">'</font><font color="#990000">,</font> MDFR_SHIFT<font color="#990000">,</font> modal_tab<font color="#990000">);</font>

    <b><font color="#000000">end_map</font></b><font color="#990000">(</font>context<font color="#990000">);</font>

    <b><font color="#000000">end_bind_helper</font></b><font color="#990000">(</font>context<font color="#990000">);</font>
    <b><font color="#0000FF">return</font></b> context<font color="#990000">-&gt;</font>write_total<font color="#990000">;</font>
<font color="#FF0000">}</font>
</pre>
