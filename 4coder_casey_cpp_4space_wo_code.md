    /* NOTE(casey): This code is _extremely_ bad and is mostly just me hacking things // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
       around to put in features I want in advance of 4coder having them properly. // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
       Most of the time I haven&#39;t even taken enough time to read the 4coder API // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
       to know what I&#39;m actually even doing.  So if you decide to use the code in // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
       here, be advised that it might be super crashy or break something or cause you  // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
       to lose work or who knows what else! // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>

       DON&#39;T SAY I WE DIDN&#39;T WARN YA: This custom extension provided &quot;as is&quot; without // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
       warranty of any kind, either express or implied, including without // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
       limitation any implied warranties of condition, uninterrupted use, // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
       merchantability, fitness for a particular purpose, or non-infringement. // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    */

    /* TODO(casey): Here are our current issues // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>

       - High priority: // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
         - Buffer switching still seems a little bit broken.  I find I can&#39;t reliably hit switch-return // <a href="https://hero.handmade.network/episode/game-architecture/day265">265</a>
           and switch to the most recently viewed file that wasn&#39;t one of the two currently viewed buffers? // <a href="https://hero.handmade.network/episode/game-architecture/day265">265</a>
           But maybe I&#39;m imagining things? // <a href="https://hero.handmade.network/episode/game-architecture/day265">265</a>
         - High-DPI settings break rendering and all fonts just show up as solid squares // <a href="https://hero.handmade.network/episode/game-architecture/day265">265</a>
         - Pretty sure auto-indent has some bugs.  Things that should be pretty easy to indent // <a href="https://hero.handmade.network/episode/game-architecture/day265">265</a>
           properly even from only a few surrounding lines seem to be indented improperly at the moment // <a href="https://hero.handmade.network/episode/game-architecture/day265">265</a>
         - Multi-line comments should default to indenting to the indentation of the line prior? // <a href="https://hero.handmade.network/episode/game-architecture/day256">256</a>
         - Would like the option to indent to hanging parentheses, equals signs, etc. instead of // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
           always just &quot;one tab in from the previous line&quot;. // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
           - Actually, maybe just expose the dirty state, so that the user can decide whether to // <a href="https://hero.handmade.network/episode/game-architecture/day256">256</a>
             save or not?  Not sure... // <a href="https://hero.handmade.network/episode/game-architecture/day256">256</a>
         - Replace: // <a href="https://hero.handmade.network/episode/game-architecture/day256">256</a>
           - Needs to be case-insensitive, or at least have the option to be // <a href="https://hero.handmade.network/episode/game-architecture/day256">256</a>
           - Needs to replace using the case of the thing being replaced, or at least have the option to do so // <a href="https://hero.handmade.network/episode/game-architecture/day256">256</a>
         - Auto-complete doesn&#39;t pick nearby words first, it seems, which makes it much slower to use? // <a href="https://hero.handmade.network/episode/game-architecture/day256">256</a>
         - Bug with not being able to switch-to-corresponding-file in another buffer // <a href="https://hero.handmade.network/episode/game-architecture/day252">252</a>
           without accidentally bringing up the file open dialog? // <a href="https://hero.handmade.network/episode/game-architecture/day252">252</a>
         - Up/down arrows and mouse clicks on wrapped lines don&#39;t seem to work properly with several wraps. // <a href="https://hero.handmade.network/episode/game-architecture/day265">265</a>
           (eg., a line wrapped to more than 2 physical lines on the screen often doesn&#39;t work anymore, // <a href="https://hero.handmade.network/episode/game-architecture/day265">265</a>
            with up or down jumping to totally wrong places, and mouse clicking jumping to wrong places // <a href="https://hero.handmade.network/episode/game-architecture/day265">265</a>
            as well - similarly, scrolling breaks, in that it thinks it has &quot;hit the end&quot; of the buffer // <a href="https://hero.handmade.network/episode/game-architecture/day265">265</a>
            when you cursor down, but the cursor and the rest of the wrapped lines are actually off // <a href="https://hero.handmade.network/episode/game-architecture/day265">265</a>
            the bottom of the screen) // <a href="https://hero.handmade.network/episode/game-architecture/day265">265</a>

       - Display: // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
         - There are often repaint bugs with 4coder coming to the front / unminimizing, etc. // <a href="https://hero.handmade.network/episode/game-architecture/day265">265</a>
           I think this might have something to do with the way you&#39;re doing lots of semaphore // <a href="https://hero.handmade.network/episode/game-architecture/day265">265</a>
           locking but I haven&#39;t investigated yet. // <a href="https://hero.handmade.network/episode/game-architecture/day265">265</a>
         - Need a word-wrap mode that wraps at word boundaries instead of characters // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
         - Need to be able to set a word wrap length at something other than the window // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    ?FIXED First go-to-line for a file seems to still just go to the beginning of the buffer? // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
           Not sure Allen&#39;s right about the slash problem, but either way, we need some // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
           way to fix it. // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
         - NOTE / IMPORTANT / TODO highlighting?  Ability to customize?  Whatever. // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
         - Some kind of parentheses highlighting?  I can write this myself, but I // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
           would need some way of adding highlight information to the buffer. // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
         - Need a way of highlighting the current line like Emacs does for the benefit // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
           of people on The Stream(TM) // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
         - Some kind of matching brace display so in long ifs, etc., you can see // <a href="https://hero.handmade.network/episode/game-architecture/day265">265</a>
           what they match (maybe draw it directly into the buffer?) // <a href="https://hero.handmade.network/episode/game-architecture/day265">265</a>

       - Indentation: // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
         - Multiple // lines don&#39;t seem to indent properly.  The first one will go to the correct place, but the subsequent ones will go to the first column regardless? // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
         - Need to have better indentation / wrapping control for typing in comments.  // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
           Right now it&#39;s a bit worse than Emacs, which does automatically put you at // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
           the same margin as the prev. line (4coder just goes back to column 1).  It&#39;d // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
           be nice if it go _better_ than Emacs, with no need to manually flow comments, // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
           etc. // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>

       - Buffer management:  // <a href="https://hero.handmade.network/episode/game-architecture/day256">256</a>
         - Seems like there&#39;s no way to switch to buffers whose names are substrings of other // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
           buffers&#39; names without using the mouse? // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
           - Also, mouse-clicking on buffers doesn&#39;t seem to work reliably?  Often it just goes to a  // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
             blank window? // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>

       - File system // <a href="https://hero.handmade.network/episode/game-architecture/day256">256</a>
         - When switching to a buffer that has changed on disk, notify?  Really this can just // <a href="https://hero.handmade.network/episode/game-architecture/day265">265</a>
           be some way to query the modification flag and then the customization layer can do it? // <a href="https://hero.handmade.network/episode/game-architecture/day265">265</a>
         - Still can&#39;t seem to open a zero-length file? // <a href="https://hero.handmade.network/episode/game-architecture/day265">265</a>
         - I&#39;d prefer it if file-open could create new files, and that I could get called on that // <a href="https://hero.handmade.network/episode/game-architecture/day265">265</a>
           so I can insert my boilerplate headers on new files // <a href="https://hero.handmade.network/episode/game-architecture/day265">265</a>
         - I&#39;d prefer it if file-open deleted per-character instead of deleting entire path sections // <a href="https://hero.handmade.network/episode/game-architecture/day256">256</a>

       - Need auto-complete for things like &quot;arbitrary command&quot;, with options listed, etc., // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
         so this should either be built into 4ed, or the custom DLL should have the ability // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
         to display possible completions and iterate over internal cmdid&#39;s, etc.  Possibly // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
         the latter, for maximal ability of customizers to add their own commands? // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>

       - Macro recording/playback // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>

       - Arbitrary cool features: // <a href="https://hero.handmade.network/episode/game-architecture/day256">256</a>
         - LOC count for the buffer and for all buffers summed shown in the title bar? // <a href="https://hero.handmade.network/episode/game-architecture/day265">265</a>
         - Show auto-parsed #if/if/for/while/etc. statements at else and closing places. // <a href="https://hero.handmade.network/episode/game-architecture/day265">265</a>
         - Automatic highlighting of the region in side the parentheses / etc. // <a href="https://hero.handmade.network/episode/game-architecture/day265">265</a>
         - You should just implement a shell inside 4coder which can call all the 4coder // <a href="https://hero.handmade.network/episode/game-architecture/day265">265</a>
           stuff as well as execute system stuff, so that from now on you just write // <a href="https://hero.handmade.network/episode/game-architecture/day265">265</a>
           scripts &quot;in 4coder&quot;, etc., so they are always portable everywhere 4coder runs? // <a href="https://hero.handmade.network/episode/game-architecture/day265">265</a>

       - Things I should write: // <a href="https://hero.handmade.network/episode/game-architecture/day265">265</a>
         - Ability to do &quot;file open from same directory as the current buffer&quot; // <a href="https://hero.handmade.network/episode/game-architecture/day265">265</a>
         - Spell-checker // <a href="https://hero.handmade.network/episode/game-architecture/day265">265</a>
         - To-do list dependent on project? // <a href="https://hero.handmade.network/episode/game-architecture/day265">265</a>
         - Repeat last replace? // <a href="https://hero.handmade.network/episode/game-architecture/day265">265</a>
         - Maybe search could be a permanent thing, so instead of initiating a search, // <a href="https://hero.handmade.network/episode/game-architecture/day265">265</a>
           you&#39;re just _changing_ the search term with MODAL-S, and then there&#39;s _always_ // <a href="https://hero.handmade.network/episode/game-architecture/day265">265</a>
           a next-of-these-in... and that could go through buffers in order, to... // <a href="https://hero.handmade.network/episode/game-architecture/day265">265</a>
    */

    // NOTE(casey): Microsoft/Windows is poopsauce. // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>

    #include &lt;math.h&gt; // <a href="https://hero.handmade.network/episode/game-architecture/day266">266</a> <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a> <a href="https://hero.handmade.network/episode/game-architecture/day170">170</a> <a href="https://hero.handmade.network/episode/game-architecture/day029">029</a> <a href="https://hero.handmade.network/episode/win32-platform/day021">021</a> <a href="https://hero.handmade.network/episode/win32-platform/day009">009</a>
    #include &lt;stdio.h&gt; // <a href="https://hero.handmade.network/episode/game-architecture/day266">266</a> <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a> <a href="https://hero.handmade.network/episode/game-architecture/day206">206</a> <a href="https://hero.handmade.network/episode/game-architecture/day180">180</a> <a href="https://hero.handmade.network/episode/game-architecture/day177">177</a> <a href="https://hero.handmade.network/episode/game-architecture/day170">170</a> <a href="https://hero.handmade.network/episode/win32-platform/day010">010</a>

    #include &quot;4coder_default_include.cpp&quot; // <a href="https://hero.handmade.network/episode/game-architecture/day285">285</a>

    enum maps{ // <a href="https://hero.handmade.network/episode/game-architecture/day256">256</a>
        my_code_map // <a href="https://hero.handmade.network/episode/game-architecture/day256">256</a>
    };

    #ifndef Assert // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    #define internal static // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a> <a href="https://hero.handmade.network/episode/game-architecture/day039">039</a> <a href="https://hero.handmade.network/episode/win32-platform/day021">021</a> <a href="https://hero.handmade.network/episode/win32-platform/day003">003</a>
    #define Assert assert  // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    #endif

    struct Parsed_Error // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    {
        int exists; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>

        String target_file_name; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        int target_line_number; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        int target_column_number; // <a href="https://hero.handmade.network/episode/game-architecture/day265">265</a>

        int source_buffer_id; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        int source_position; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    };

    static bool GlobalEditMode; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    static char *GlobalCompilationBufferName = &quot;*compilation*&quot;; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>

    // TODO(casey): If 4coder gets variables at some point, this would go in a variable. // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    static char BuildDirectory[4096] = &quot;./&quot;; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>

    enum token_type // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a> <a href="https://hero.handmade.network/episode/game-architecture/day206">206</a>
    {
        Token_Unknown, // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a> <a href="https://hero.handmade.network/episode/game-architecture/day206">206</a>

        Token_OpenParen,     // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a> <a href="https://hero.handmade.network/episode/game-architecture/day206">206</a>
        Token_CloseParen,     // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a> <a href="https://hero.handmade.network/episode/game-architecture/day206">206</a>
        Token_Asterisk, // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a> <a href="https://hero.handmade.network/episode/game-architecture/day206">206</a>
        Token_Minus, // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        Token_Plus, // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        Token_ForwardSlash, // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        Token_Percent, // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        Token_Colon, // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a> <a href="https://hero.handmade.network/episode/game-architecture/day206">206</a>
        Token_Number, // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        Token_Comma, // <a href="https://hero.handmade.network/episode/game-architecture/day265">265</a>

        Token_EndOfStream, // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a> <a href="https://hero.handmade.network/episode/game-architecture/day206">206</a>
    };
    struct token // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a> <a href="https://hero.handmade.network/episode/game-architecture/day206">206</a>
    {
        token_type Type; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a> <a href="https://hero.handmade.network/episode/game-architecture/day206">206</a>

        size_t TextLength; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a> <a href="https://hero.handmade.network/episode/game-architecture/day206">206</a>
        char *Text; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a> <a href="https://hero.handmade.network/episode/game-architecture/day206">206</a>
    };

    struct tokenizer // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a> <a href="https://hero.handmade.network/episode/game-architecture/day206">206</a>
    {
        char *At; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a> <a href="https://hero.handmade.network/episode/game-architecture/day206">206</a>
    };

    inline bool
    IsEndOfLine(char C) // <a href="https://hero.handmade.network/episode/game-architecture/day255">255</a> <a href="https://hero.handmade.network/episode/game-architecture/day249">249</a> <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a> <a href="https://hero.handmade.network/episode/game-architecture/day242">242</a> <a href="https://hero.handmade.network/episode/game-architecture/day206">206</a>
    {
        bool Result = ((C == &#39;n&#39;) ||
                       (C == &#39;r&#39;));

        return(Result);
    }

    inline bool
    IsWhitespace(char C) // <a href="https://hero.handmade.network/episode/game-architecture/day255">255</a> <a href="https://hero.handmade.network/episode/game-architecture/day249">249</a> <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a> <a href="https://hero.handmade.network/episode/game-architecture/day242">242</a> <a href="https://hero.handmade.network/episode/game-architecture/day206">206</a>
    {
        bool Result = ((C == &#39; &#39;) || // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a> <a href="https://hero.handmade.network/episode/game-architecture/day206">206</a>
                       (C == &#39;t&#39;) ||
                       (C == &#39;v&#39;) ||
                       (C == &#39;f&#39;) ||
                       IsEndOfLine(C)); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a> <a href="https://hero.handmade.network/episode/game-architecture/day206">206</a>

        return(Result);
    }

    inline bool
    IsAlpha(char C) // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a> <a href="https://hero.handmade.network/episode/game-architecture/day206">206</a>
    {
        bool Result = (((C &gt;= &#39;a&#39;) &amp;&amp; (C &lt;= &#39;z&#39;)) || // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a> <a href="https://hero.handmade.network/episode/game-architecture/day206">206</a>
                       ((C &gt;= &#39;A&#39;) &amp;&amp; (C &lt;= &#39;Z&#39;))); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a> <a href="https://hero.handmade.network/episode/game-architecture/day206">206</a>

        return(Result);
    }

    inline bool
    IsNumeric(char C) // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    {
        bool Result = ((C &gt;= &#39;0&#39;) &amp;&amp; (C &lt;= &#39;9&#39;)); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a> <a href="https://hero.handmade.network/episode/game-architecture/day206">206</a>

        return(Result);
    }

    static void // <a href="https://hero.handmade.network/episode/game-architecture/day266">266</a> <a href="https://hero.handmade.network/episode/game-architecture/day255">255</a> <a href="https://hero.handmade.network/episode/game-architecture/day249">249</a> <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a> <a href="https://hero.handmade.network/episode/game-architecture/day220">220</a> <a href="https://hero.handmade.network/episode/game-architecture/day206">206</a> <a href="https://hero.handmade.network/episode/game-architecture/day170">170</a>
    EatAllWhitespace(tokenizer *Tokenizer) // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a> <a href="https://hero.handmade.network/episode/game-architecture/day206">206</a>
    {
        for(;;)
        {
            if(IsWhitespace(Tokenizer-&gt;At[0])) // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a> <a href="https://hero.handmade.network/episode/game-architecture/day206">206</a>
            {
                ++Tokenizer-&gt;At; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a> <a href="https://hero.handmade.network/episode/game-architecture/day206">206</a>
            }
            else if((Tokenizer-&gt;At[0] == &#39;/&#39;) &amp;&amp; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a> <a href="https://hero.handmade.network/episode/game-architecture/day206">206</a>
                    (Tokenizer-&gt;At[1] == &#39;/&#39;)) // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a> <a href="https://hero.handmade.network/episode/game-architecture/day206">206</a>
            {
                Tokenizer-&gt;At += 2; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a> <a href="https://hero.handmade.network/episode/game-architecture/day206">206</a>
                while(Tokenizer-&gt;At[0] &amp;&amp; !IsEndOfLine(Tokenizer-&gt;At[0])) // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a> <a href="https://hero.handmade.network/episode/game-architecture/day206">206</a>
                {
                    ++Tokenizer-&gt;At; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a> <a href="https://hero.handmade.network/episode/game-architecture/day206">206</a>
                }
            }
            else if((Tokenizer-&gt;At[0] == &#39;/&#39;) &amp;&amp; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a> <a href="https://hero.handmade.network/episode/game-architecture/day206">206</a>
                    (Tokenizer-&gt;At[1] == &#39;*&#39;)) // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a> <a href="https://hero.handmade.network/episode/game-architecture/day206">206</a>
            {
                Tokenizer-&gt;At += 2; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a> <a href="https://hero.handmade.network/episode/game-architecture/day206">206</a>
                while(Tokenizer-&gt;At[0] &amp;&amp; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a> <a href="https://hero.handmade.network/episode/game-architecture/day206">206</a>
                      !((Tokenizer-&gt;At[0] == &#39;*&#39;) &amp;&amp; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a> <a href="https://hero.handmade.network/episode/game-architecture/day206">206</a>
                        (Tokenizer-&gt;At[1] == &#39;/&#39;))) // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a> <a href="https://hero.handmade.network/episode/game-architecture/day206">206</a>
                {
                    ++Tokenizer-&gt;At; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a> <a href="https://hero.handmade.network/episode/game-architecture/day206">206</a>
                }

                if(Tokenizer-&gt;At[0] == &#39;*&#39;) // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a> <a href="https://hero.handmade.network/episode/game-architecture/day206">206</a>
                {
                    Tokenizer-&gt;At += 2; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a> <a href="https://hero.handmade.network/episode/game-architecture/day206">206</a>
                }
            }
            else
            {
                break;
            }
        }
    }

    static token // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a> <a href="https://hero.handmade.network/episode/game-architecture/day206">206</a>
    GetToken(tokenizer *Tokenizer) // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a> <a href="https://hero.handmade.network/episode/game-architecture/day206">206</a>
    {
        EatAllWhitespace(Tokenizer); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a> <a href="https://hero.handmade.network/episode/game-architecture/day206">206</a>

        token Token = {}; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a> <a href="https://hero.handmade.network/episode/game-architecture/day206">206</a>
        Token.TextLength = 1; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a> <a href="https://hero.handmade.network/episode/game-architecture/day206">206</a>
        Token.Text = Tokenizer-&gt;At; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a> <a href="https://hero.handmade.network/episode/game-architecture/day206">206</a>
        char C = Tokenizer-&gt;At[0]; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a> <a href="https://hero.handmade.network/episode/game-architecture/day206">206</a>
        ++Tokenizer-&gt;At; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a> <a href="https://hero.handmade.network/episode/game-architecture/day206">206</a>
        switch(C) // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a> <a href="https://hero.handmade.network/episode/game-architecture/day206">206</a>
        {
            case 0: {--Tokenizer-&gt;At; Token.Type = Token_EndOfStream;} break; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>

            case &#39;(&#39;: {Token.Type = Token_OpenParen;} break; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a> <a href="https://hero.handmade.network/episode/game-architecture/day206">206</a>
            case &#39;)&#39;: {Token.Type = Token_CloseParen;} break; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a> <a href="https://hero.handmade.network/episode/game-architecture/day206">206</a>
            case &#39;*&#39;: {Token.Type = Token_Asterisk;} break; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a> <a href="https://hero.handmade.network/episode/game-architecture/day206">206</a>
            case &#39;-&#39;: {Token.Type = Token_Minus;} break; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
            case &#39;+&#39;: {Token.Type = Token_Plus;} break; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
            case &#39;/&#39;: {Token.Type = Token_ForwardSlash;} break; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
            case &#39;%&#39;: {Token.Type = Token_Percent;} break; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
            case &#39;:&#39;: {Token.Type = Token_Colon;} break; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a> <a href="https://hero.handmade.network/episode/game-architecture/day206">206</a>
            case &#39;,&#39;: {Token.Type = Token_Comma;} break; // <a href="https://hero.handmade.network/episode/game-architecture/day265">265</a>

            default:
            {
                if(IsNumeric(C)) // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
                {
                    // TODO(casey): Real number // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
                    Token.Type = Token_Number; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
                    while(IsNumeric(Tokenizer-&gt;At[0]) || // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
                          (Tokenizer-&gt;At[0] == &#39;.&#39;) || // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
                          (Tokenizer-&gt;At[0] == &#39;f&#39;)) // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
                    {
                        ++Tokenizer-&gt;At; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a> <a href="https://hero.handmade.network/episode/game-architecture/day206">206</a>
                        Token.TextLength = Tokenizer-&gt;At - Token.Text; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
                    }
                }
                else
                {
                    Token.Type = Token_Unknown; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a> <a href="https://hero.handmade.network/episode/game-architecture/day206">206</a>
                }
            } break;         // <a href="https://hero.handmade.network/episode/game-architecture/day255">255</a> <a href="https://hero.handmade.network/episode/game-architecture/day249">249</a> <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a> <a href="https://hero.handmade.network/episode/game-architecture/day212">212</a> <a href="https://hero.handmade.network/episode/game-architecture/day206">206</a>
        }

        return(Token); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a> <a href="https://hero.handmade.network/episode/game-architecture/day206">206</a>
    }

    static token // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a> <a href="https://hero.handmade.network/episode/game-architecture/day206">206</a>
    PeekToken(tokenizer *Tokenizer) // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    {
        tokenizer Tokenizer2 = *Tokenizer; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        token Result = GetToken(&amp;Tokenizer2); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        return(Result);
    }

    inline bool
    IsH(String extension) // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    {
        bool Result = (match(extension, make_lit_string(&quot;h&quot;)) || // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
                       match(extension, make_lit_string(&quot;hpp&quot;)) || // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
                       match(extension, make_lit_string(&quot;hin&quot;))); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>

        return(Result);
    }

    inline bool
    IsCPP(String extension) // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    {
        bool Result = (match(extension, make_lit_string(&quot;c&quot;)) || // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
                       match(extension, make_lit_string(&quot;cpp&quot;)) || // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
                       match(extension, make_lit_string(&quot;cin&quot;))); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>

        return(Result);
    }

    inline bool
    IsINL(String extension) // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    {
        bool Result = (match(extension, make_lit_string(&quot;inl&quot;))); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>

        return(Result);
    }

    inline bool
    IsCode(String extension) // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    {
        bool Result = (IsH(extension) || IsCPP(extension) || IsINL(extension)); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>

        return(Result);
    }


    CUSTOM_COMMAND_SIG(casey_open_in_other) // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    {
        exec_command(app, cmdid_change_active_panel); // <a href="https://hero.handmade.network/episode/game-architecture/day252">252</a> <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        exec_command(app, cmdid_interactive_open); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    }

    CUSTOM_COMMAND_SIG(casey_clean_and_save) // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    {
        exec_command(app, cmdid_clean_all_lines); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        exec_command(app, cmdid_eol_nixify); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        exec_command(app, cmdid_save); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    }

    CUSTOM_COMMAND_SIG(casey_newline_and_indent) // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    {
        exec_command(app, cmdid_write_character); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        exec_command(app, auto_tab_line_at_cursor); // <a href="https://hero.handmade.network/episode/game-architecture/day256">256</a> <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    }

    CUSTOM_COMMAND_SIG(casey_open_file_other_window) // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    {
        exec_command(app, cmdid_change_active_panel); // <a href="https://hero.handmade.network/episode/game-architecture/day252">252</a> <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        exec_command(app, cmdid_interactive_open); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    }

    CUSTOM_COMMAND_SIG(casey_switch_buffer_other_window) // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    {
        exec_command(app, cmdid_change_active_panel); // <a href="https://hero.handmade.network/episode/game-architecture/day252">252</a> <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        exec_command(app, cmdid_interactive_switch_buffer); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    }

    internal void
    DeleteAfterCommand(struct Application_Links *app, unsigned long long CommandID) // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    {
        View_Summary view = app-&gt;get_active_view(app); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>

        int pos2 = view.cursor.pos; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        if (CommandID &lt; cmdid_count){ // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
            exec_command(app, (Command_ID)CommandID); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        }
        else{ // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
            exec_command(app, (Custom_Command_Function*)CommandID); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        }
        app-&gt;refresh_view(app, &amp;view); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        int pos1 = view.cursor.pos; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>

        Range range = make_range(pos1, pos2); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>

        Buffer_Summary buffer = app-&gt;get_buffer(app, view.buffer_id); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        app-&gt;buffer_replace_range(app, &amp;buffer, range.min, range.max, 0, 0); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    }

    CUSTOM_COMMAND_SIG(casey_delete_token_left) // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    {
        DeleteAfterCommand(app, (unsigned long long)seek_white_or_token_left); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    }

    CUSTOM_COMMAND_SIG(casey_delete_token_right) // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    {
        DeleteAfterCommand(app, (unsigned long long)seek_white_or_token_right); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    }

    CUSTOM_COMMAND_SIG(casey_kill_to_end_of_line) // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    {
        View_Summary view = app-&gt;get_active_view(app); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>

        int pos2 = view.cursor.pos; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        exec_command(app, cmdid_seek_end_of_line); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        app-&gt;refresh_view(app, &amp;view); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        int pos1 = view.cursor.pos; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>

        Range range = make_range(pos1, pos2); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        if(pos1 == pos2) // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        {
            range.max += 1; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        }

        Buffer_Summary buffer = app-&gt;get_buffer(app, view.buffer_id); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        app-&gt;buffer_replace_range(app, &amp;buffer, range.min, range.max, 0, 0); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        exec_command(app, auto_tab_line_at_cursor); // <a href="https://hero.handmade.network/episode/game-architecture/day256">256</a> <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    }

    CUSTOM_COMMAND_SIG(casey_paste_and_tab) // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    {
        // NOTE(allen): Paste puts the mark at the beginning and the cursor at // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        // the end of the pasted chunk, so it is all set for cmdid_auto_tab_range // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        exec_command(app, cmdid_paste); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        exec_command(app, cmdid_auto_tab_range); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    }

    CUSTOM_COMMAND_SIG(casey_seek_beginning_of_line_and_tab) // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    {
        exec_command(app, cmdid_seek_beginning_of_line); // <a href="https://hero.handmade.network/episode/game-architecture/day256">256</a> <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        exec_command(app, auto_tab_line_at_cursor); // <a href="https://hero.handmade.network/episode/game-architecture/day256">256</a> <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    }

    CUSTOM_COMMAND_SIG(casey_seek_beginning_of_line) // <a href="https://hero.handmade.network/episode/game-architecture/day256">256</a>
    {
        exec_command(app, auto_tab_line_at_cursor); // <a href="https://hero.handmade.network/episode/game-architecture/day256">256</a> <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        exec_command(app, cmdid_seek_beginning_of_line); // <a href="https://hero.handmade.network/episode/game-architecture/day256">256</a> <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    }

    struct switch_to_result // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    {
        bool Switched; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        bool Loaded; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        View_Summary view; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        Buffer_Summary buffer; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    };

    inline void
    SanitizeSlashes(String Value) // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    {
        for(int At = 0; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
            At &lt; Value.size; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
            ++At) // <a href="https://hero.handmade.network/episode/game-architecture/day255">255</a> <a href="https://hero.handmade.network/episode/game-architecture/day249">249</a> <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a> <a href="https://hero.handmade.network/episode/game-architecture/day143">143</a>
        {
            if(Value.str[At] == &#39;\&#39;)
            {
                Value.str[At] = &#39;/&#39;; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
            }
        }
    }

    inline switch_to_result // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    SwitchToOrLoadFile(struct Application_Links *app, String FileName, bool CreateIfNotFound = false) // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    {
        switch_to_result Result = {}; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>

        SanitizeSlashes(FileName); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>

        View_Summary view = app-&gt;get_active_view(app); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        Buffer_Summary buffer = app-&gt;get_buffer_by_name(app, FileName.str, FileName.size); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>

        Result.view = view; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        Result.buffer = buffer; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>

        if(buffer.exists) // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        {
            app-&gt;view_set_buffer(app, &amp;view, buffer.buffer_id); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
            Result.Switched = true; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        }
        else
        {
            if(app-&gt;file_exists(app, FileName.str, FileName.size) || CreateIfNotFound) // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
            {
                push_parameter(app, par_name, expand_str(FileName)); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
                // TODO(casey): Do I have to check for existence, or can I pass a parameter // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
                // to interactive open to tell it to fail if the file isn&#39;t there? // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
                exec_command(app, cmdid_interactive_open); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>

                Result.buffer = app-&gt;get_buffer_by_name(app, FileName.str, FileName.size);             // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>

                Result.Loaded = true; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
                Result.Switched = true; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
            }
        }

        return(Result);
    }

    CUSTOM_COMMAND_SIG(casey_load_todo) // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    {
        String ToDoFileName = make_lit_string(&quot;w:/handmade/code/todo.txt&quot;); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        SwitchToOrLoadFile(app, ToDoFileName, true); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    }

    CUSTOM_COMMAND_SIG(casey_build_search) // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    {
        int keep_going = 1; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        int old_size; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        // TODO(allen): It&#39;s fine to get memory this way for now, eventually // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        // we should properly suballocating from app-&gt;memory. // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        String dir = make_string(app-&gt;memory, 0, app-&gt;memory_size); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        dir.size = app-&gt;directory_get_hot(app, dir.str, dir.memory_size); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>

        while (keep_going) // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        {
            old_size = dir.size; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
            append(&amp;dir, &quot;build.bat&quot;); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>

            if (app-&gt;file_exists(app, dir.str, dir.size)) // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
            {
                dir.size = old_size; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
                memcpy(BuildDirectory, dir.str, dir.size); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
                BuildDirectory[dir.size] = 0; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>

                return; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
            }

            dir.size = old_size; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>

            if (app-&gt;directory_cd(app, dir.str, &amp;dir.size, dir.memory_size, literal(&quot;..&quot;)) == 0) // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
            {
                keep_going = 0; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
            }
        }

        // TODO(casey): How do I print out that it found or didn&#39;t find something? // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    }

    CUSTOM_COMMAND_SIG(casey_find_corresponding_file) // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    {
        View_Summary view = app-&gt;get_active_view(app); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        Buffer_Summary buffer = app-&gt;get_buffer(app, view.buffer_id); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>

        String extension = file_extension(make_string(buffer.file_name, buffer.file_name_len)); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        if (extension.str) // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        {       
            char *HExtensions[] = // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
            {
                &quot;hpp&quot;, // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
                &quot;hin&quot;, // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
                &quot;h&quot;, // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
            };

            char *CExtensions[] = // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
            {
                &quot;c&quot;, // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
                &quot;cin&quot;, // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
                &quot;cpp&quot;, // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
            };

            int ExtensionCount = 0; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
            char **Extensions = 0; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
            if(IsH(extension)) // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
            {
                ExtensionCount = ArrayCount(CExtensions); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
                Extensions = CExtensions; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
            }
            else if(IsCPP(extension) || IsINL(extension)) // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
            {
                ExtensionCount = ArrayCount(HExtensions); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
                Extensions = HExtensions; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
            }

            int MaxExtensionLength = 3; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
            int Space = (int)(buffer.file_name_len + MaxExtensionLength); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
            String FileNameStem = make_string(buffer.file_name, (int)(extension.str - buffer.file_name), 0); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
            String TestFileName = make_string(app-&gt;push_memory(app, Space), 0, Space);    // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
            for(int ExtensionIndex = 0; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
                ExtensionCount; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
                ++ExtensionIndex) // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
            {
                TestFileName.size = 0; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
                append(&amp;TestFileName, FileNameStem); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
                append(&amp;TestFileName, Extensions[ExtensionIndex]); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>

                if(SwitchToOrLoadFile(app, TestFileName, ((ExtensionIndex + 1) == ExtensionCount)).Switched) // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
                {
                    break;
                }
            }
        }
    }

    CUSTOM_COMMAND_SIG(casey_find_corresponding_file_other_window) // <a href="https://hero.handmade.network/episode/game-architecture/day252">252</a>
    {
        View_Summary old_view = app-&gt;get_active_view(app); // <a href="https://hero.handmade.network/episode/game-architecture/day252">252</a>
        Buffer_Summary buffer = app-&gt;get_buffer(app, old_view.buffer_id); // <a href="https://hero.handmade.network/episode/game-architecture/day252">252</a>

        exec_command(app, cmdid_change_active_panel); // <a href="https://hero.handmade.network/episode/game-architecture/day252">252</a> <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        View_Summary new_view = app-&gt;get_active_view(app); // <a href="https://hero.handmade.network/episode/game-architecture/day252">252</a>
        app-&gt;view_set_buffer(app, &amp;new_view, buffer.buffer_id); // <a href="https://hero.handmade.network/episode/game-architecture/day252">252</a>

    //    exec_command(app, casey_find_corresponding_file); // <a href="https://hero.handmade.network/episode/game-architecture/day252">252</a>
    }

    CUSTOM_COMMAND_SIG(casey_save_and_make_without_asking) // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    {
        exec_command(app, cmdid_change_active_panel); // <a href="https://hero.handmade.network/episode/game-architecture/day252">252</a> <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>

        Buffer_Summary buffer = {}; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>

        for(buffer = app-&gt;get_buffer_first(app); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
            buffer.exists; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
            app-&gt;get_buffer_next(app, &amp;buffer)) // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        {
            push_parameter(app, par_name, buffer.file_name, buffer.file_name_len); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
            push_parameter(app, par_buffer_id, buffer.buffer_id); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
            exec_command(app, cmdid_save); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        }

        String dir = make_string(app-&gt;memory, 0, app-&gt;memory_size); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        append(&amp;dir, BuildDirectory); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        for(int At = 0; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
            At &lt; dir.size; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
            ++At) // <a href="https://hero.handmade.network/episode/game-architecture/day255">255</a> <a href="https://hero.handmade.network/episode/game-architecture/day249">249</a> <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a> <a href="https://hero.handmade.network/episode/game-architecture/day143">143</a>
        {
            if(dir.str[At] == &#39;/&#39;) // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
            {
                dir.str[At] = &#39;\&#39;;
            }
        }


        push_parameter(app, par_flags, CLI_OverlapWithConflict); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        push_parameter(app, par_name, GlobalCompilationBufferName, (int)strlen(GlobalCompilationBufferName)); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        push_parameter(app, par_cli_path, dir.str, dir.size); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>

        if(append(&amp;dir, &quot;build.bat&quot;)) // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        {
            push_parameter(app, par_cli_command, dir.str, dir.size); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
            exec_command(app, cmdid_command_line); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
            exec_command(app, cmdid_change_active_panel); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        }
        else{ // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
            app-&gt;clear_parameters(app); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        }
    }

    internal bool
    casey_errors_are_the_same(Parsed_Error a, Parsed_Error b) // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    {
        bool result = ((a.exists == b.exists) &amp;&amp; compare(a.target_file_name, b.target_file_name) &amp;&amp; (a.target_line_number == b.target_line_number)); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>

        return(result); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    }

    internal void
    casey_goto_error(Application_Links *app, Parsed_Error e) // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    {
        if(e.exists) // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        {
            switch_to_result Switch = SwitchToOrLoadFile(app, e.target_file_name, false); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
            if(Switch.Switched) // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
            {
                app-&gt;view_set_cursor(app, &amp;Switch.view, seek_line_char(e.target_line_number, e.target_column_number), 1); // <a href="https://hero.handmade.network/episode/game-architecture/day265">265</a>
            }

            View_Summary compilation_view = get_first_view_with_buffer(app, e.source_buffer_id); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
            if(compilation_view.exists) // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
            {
                app-&gt;view_set_cursor(app, &amp;compilation_view, seek_pos(e.source_position), 1); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
            }
        }
    }

    internal Parsed_Error // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    casey_parse_error(Application_Links *app, Buffer_Summary buffer, View_Summary view) // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    {
        Parsed_Error result = {}; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>

        app-&gt;refresh_view(app, &amp;view); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        int restore_pos = view.cursor.pos; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>

        app-&gt;view_set_cursor(app, &amp;view, seek_line_char(view.cursor.line, 1), 1); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        app-&gt;refresh_view(app, &amp;view); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        int start = view.cursor.pos; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>

        app-&gt;view_set_cursor(app, &amp;view, seek_line_char(view.cursor.line, 65536), 1); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        app-&gt;refresh_view(app, &amp;view); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        int end = view.cursor.pos; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>

        app-&gt;view_set_cursor(app, &amp;view, seek_pos(restore_pos), 1); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        app-&gt;refresh_view(app, &amp;view); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>

        int size = end - start; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>

        char *ParsingRegion = (char *)malloc(size + 1); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    //    char *ParsingRegion = (char *)app-&gt;push_memory(app, size + 1); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        app-&gt;buffer_read_range(app, &amp;buffer, start, end, ParsingRegion); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        ParsingRegion[size] = 0; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        tokenizer Tokenizer = {ParsingRegion}; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        for(;;)
        {
            token Token = GetToken(&amp;Tokenizer); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a> <a href="https://hero.handmade.network/episode/game-architecture/day206">206</a>
            if(Token.Type == Token_OpenParen) // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
            {
                token LineToken = GetToken(&amp;Tokenizer); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
                if(LineToken.Type == Token_Number) // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
                {
                    token CloseToken = GetToken(&amp;Tokenizer); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>

                    int column_number = 0; // <a href="https://hero.handmade.network/episode/game-architecture/day265">265</a>
                    if(CloseToken.Type == Token_Comma) // <a href="https://hero.handmade.network/episode/game-architecture/day265">265</a>
                    {
                        token ColumnToken = GetToken(&amp;Tokenizer); // <a href="https://hero.handmade.network/episode/game-architecture/day265">265</a>
                        if(ColumnToken.Type == Token_Number) // <a href="https://hero.handmade.network/episode/game-architecture/day265">265</a>
                        {
                            column_number = atoi(ColumnToken.Text); // <a href="https://hero.handmade.network/episode/game-architecture/day265">265</a>
                            CloseToken = GetToken(&amp;Tokenizer); // <a href="https://hero.handmade.network/episode/game-architecture/day265">265</a>
                        }
                    }

                    if(CloseToken.Type == Token_CloseParen) // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
                    {
                        token ColonToken = GetToken(&amp;Tokenizer); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
                        if(ColonToken.Type == Token_Colon) // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
                        {
                            // NOTE(casey): We maybe found an error! // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
                            int line_number = atoi(LineToken.Text); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>

                            char *Seek = Token.Text; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
                            while(Seek != ParsingRegion) // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
                            {
                                if(IsEndOfLine(*Seek)) // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
                                {
                                    while(IsWhitespace(*Seek)) // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
                                    {
                                        ++Seek; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
                                    }
                                    break; // <a href="https://hero.handmade.network/episode/game-architecture/day270">270</a> <a href="https://hero.handmade.network/episode/game-architecture/day255">255</a> <a href="https://hero.handmade.network/episode/game-architecture/day249">249</a> <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a> <a href="https://hero.handmade.network/episode/game-architecture/day226">226</a> <a href="https://hero.handmade.network/episode/game-architecture/day086">086</a>
                                }

                                --Seek; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
                            }

                            result.exists = true; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
                            result.target_file_name = make_string(Seek, (int)(Token.Text - Seek));; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
                            result.target_line_number = line_number; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
                            result.target_column_number = column_number; // <a href="https://hero.handmade.network/episode/game-architecture/day265">265</a>
                            result.source_buffer_id = buffer.buffer_id; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
                            result.source_position = start + (int)(ColonToken.Text - ParsingRegion); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>

                            break;
                        }
                    }
                }
            }
            else if(Token.Type == Token_EndOfStream) // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
            {
                break;
            }
        }

        return(result); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    }

    internal void
    casey_seek_error_dy(Application_Links *app, int dy) // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    {
        Buffer_Summary Buffer = app-&gt;get_buffer_by_name(app, GlobalCompilationBufferName, (int)strlen(GlobalCompilationBufferName)); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        View_Summary compilation_view = get_first_view_with_buffer(app, Buffer.buffer_id); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>

        // NOTE(casey): First get the current error (which may be none, if we&#39;ve never parsed before) // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        Parsed_Error StartingError = casey_parse_error(app, Buffer, compilation_view); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>

        // NOTE(casey): Now hunt for the previous distinct error // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        for(;;)
        {
            int prev_pos = compilation_view.cursor.pos; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
            app-&gt;view_set_cursor(app, &amp;compilation_view, seek_line_char(compilation_view.cursor.line + dy, 0), 1); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
            app-&gt;refresh_view(app, &amp;compilation_view); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
            if(compilation_view.cursor.pos != prev_pos) // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
            {
                Parsed_Error Error = casey_parse_error(app, Buffer, compilation_view); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
                if(Error.exists &amp;&amp; !casey_errors_are_the_same(StartingError, Error)) // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
                {
                    casey_goto_error(app, Error); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
                    break;
                }
            }
            else
            {
                break;
            }
        }
    }

    CUSTOM_COMMAND_SIG(casey_goto_previous_error) // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    {
        casey_seek_error_dy(app, -1); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    }

    CUSTOM_COMMAND_SIG(casey_goto_next_error) // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    {
        casey_seek_error_dy(app, 1); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    }

    CUSTOM_COMMAND_SIG(casey_imenu) // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    {
        // TODO(casey): Implement! // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a> <a href="https://hero.handmade.network/episode/game-architecture/day210">210</a> <a href="https://hero.handmade.network/episode/game-architecture/day209">209</a>
    }

    //
    // TODO(casey): Everything below this line probably isn&#39;t possible yet // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    //

    CUSTOM_COMMAND_SIG(casey_call_keyboard_macro) // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    {
        // TODO(casey): Implement! // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a> <a href="https://hero.handmade.network/episode/game-architecture/day210">210</a> <a href="https://hero.handmade.network/episode/game-architecture/day209">209</a>
    }

    CUSTOM_COMMAND_SIG(casey_begin_keyboard_macro_recording) // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    {
        // TODO(casey): Implement! // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a> <a href="https://hero.handmade.network/episode/game-architecture/day210">210</a> <a href="https://hero.handmade.network/episode/game-architecture/day209">209</a>
    }

    CUSTOM_COMMAND_SIG(casey_end_keyboard_macro_recording) // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    {
        // TODO(casey): Implement! // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a> <a href="https://hero.handmade.network/episode/game-architecture/day210">210</a> <a href="https://hero.handmade.network/episode/game-architecture/day209">209</a>
    }

    CUSTOM_COMMAND_SIG(casey_fill_paragraph) // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    {
        // TODO(casey): Implement! // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a> <a href="https://hero.handmade.network/episode/game-architecture/day210">210</a> <a href="https://hero.handmade.network/episode/game-architecture/day209">209</a>
    }

    enum calc_node_type // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    {
        CalcNode_UnaryMinus, // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>

        CalcNode_Add, // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        CalcNode_Subtract, // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        CalcNode_Multiply, // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        CalcNode_Divide, // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        CalcNode_Mod, // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>

        CalcNode_Constant, // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    };
    struct calc_node // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    {
        calc_node_type Type; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        double Value; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        calc_node *Left; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        calc_node *Right; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    };

    internal double // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    ExecCalcNode(calc_node *Node) // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    {
        double Result = 0.0f; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>

        if(Node) // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        {
            switch(Node-&gt;Type) // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
            {
                case CalcNode_UnaryMinus: {Result = -ExecCalcNode(Node-&gt;Left);} break; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
                case CalcNode_Add: {Result = ExecCalcNode(Node-&gt;Left) + ExecCalcNode(Node-&gt;Right);} break; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
                case CalcNode_Subtract: {Result = ExecCalcNode(Node-&gt;Left) - ExecCalcNode(Node-&gt;Right);} break; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
                case CalcNode_Multiply: {Result = ExecCalcNode(Node-&gt;Left) * ExecCalcNode(Node-&gt;Right);} break; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
                case CalcNode_Divide: {/*TODO(casey): Guard 0*/Result = ExecCalcNode(Node-&gt;Left) / ExecCalcNode(Node-&gt;Right);} break; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
                case CalcNode_Mod: {/*TODO(casey): Guard 0*/Result = fmod(ExecCalcNode(Node-&gt;Left), ExecCalcNode(Node-&gt;Right));} break; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
                case CalcNode_Constant: {Result = Node-&gt;Value;} break; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
                default: {Assert(!&quot;AHHHHH!&quot;);} // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
            }
        }

        return(Result);
    }

    internal void
    FreeCalcNode(calc_node *Node) // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    {
        if(Node) // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        {
            FreeCalcNode(Node-&gt;Left); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
            FreeCalcNode(Node-&gt;Right); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
            free(Node); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        }
    }

    internal calc_node * // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    AddNode(calc_node_type Type, calc_node *Left = 0, calc_node *Right = 0) // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    {
        calc_node *Node = (calc_node *)malloc(sizeof(calc_node)); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        Node-&gt;Type = Type; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        Node-&gt;Value = 0; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        Node-&gt;Left = Left; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        Node-&gt;Right = Right;     // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        return(Node); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    }

    internal calc_node * // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    ParseNumber(tokenizer *Tokenizer) // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    {
        calc_node *Result = AddNode(CalcNode_Constant); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>

        token Token = GetToken(Tokenizer); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a> <a href="https://hero.handmade.network/episode/game-architecture/day206">206</a>
        Result-&gt;Value = atof(Token.Text); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>

        return(Result);
    }

    internal calc_node * // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    ParseConstant(tokenizer *Tokenizer) // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    {
        calc_node *Result = 0; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>

        token Token = PeekToken(Tokenizer); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        if(Token.Type == Token_Minus) // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        {
            Token = GetToken(Tokenizer); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
            Result = AddNode(CalcNode_UnaryMinus); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
            Result-&gt;Left = ParseNumber(Tokenizer); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        }
        else
        {
            Result = ParseNumber(Tokenizer); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        }

        return(Result);
    }

    internal calc_node * // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    ParseMultiplyExpression(tokenizer *Tokenizer) // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    {
        calc_node *Result = 0; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>

        token Token = PeekToken(Tokenizer); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        if((Token.Type == Token_Minus) || // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
           (Token.Type == Token_Number)) // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        {
            Result = ParseConstant(Tokenizer); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
            token Token = PeekToken(Tokenizer); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
            if(Token.Type == Token_ForwardSlash) // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
            {
                GetToken(Tokenizer); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
                Result = AddNode(CalcNode_Divide, Result, ParseNumber(Tokenizer)); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
            }
            else if(Token.Type == Token_Asterisk) // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
            {
                GetToken(Tokenizer); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
                Result = AddNode(CalcNode_Multiply, Result, ParseNumber(Tokenizer)); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
            }
        }

        return(Result);
    }

    internal calc_node * // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    ParseAddExpression(tokenizer *Tokenizer) // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    {
        calc_node *Result = 0; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>

        token Token = PeekToken(Tokenizer); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        if((Token.Type == Token_Minus) || // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
           (Token.Type == Token_Number)) // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        {
            Result = ParseMultiplyExpression(Tokenizer); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
            token Token = PeekToken(Tokenizer); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
            if(Token.Type == Token_Plus) // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
            {
                GetToken(Tokenizer); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
                Result = AddNode(CalcNode_Add, Result, ParseMultiplyExpression(Tokenizer)); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
            }
            else if(Token.Type == Token_Minus) // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
            {
                GetToken(Tokenizer); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
                Result = AddNode(CalcNode_Subtract, Result, ParseMultiplyExpression(Tokenizer)); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
            }
        }

        return(Result);
    }

    internal calc_node * // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    ParseCalc(tokenizer *Tokenizer) // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    {
        calc_node *Node = ParseAddExpression(Tokenizer); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>

        return(Node); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    }

    CUSTOM_COMMAND_SIG(casey_quick_calc) // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    {
        View_Summary view = app-&gt;get_active_view(app); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>

        Range range = get_range(&amp;view); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>

        size_t Size = range.max - range.min; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        char *Stuff = (char *)malloc(Size + 1); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        Stuff[Size] = 0; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>

        Buffer_Summary buffer = app-&gt;get_buffer(app, view.buffer_id); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        app-&gt;buffer_read_range(app, &amp;buffer, range.min, range.max, Stuff); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>

        tokenizer Tokenizer = {Stuff}; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        calc_node *CalcTree = ParseCalc(&amp;Tokenizer); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        double ComputedValue = ExecCalcNode(CalcTree); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        FreeCalcNode(CalcTree); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>

        char ResultBuffer[256]; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        int ResultSize = sprintf(ResultBuffer, &quot;%f&quot;, ComputedValue); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>

        app-&gt;buffer_replace_range(app, &amp;buffer, range.min, range.max, ResultBuffer, ResultSize); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>

        free(Stuff); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    }

    internal void
    OpenProject(Application_Links *app, char *ProjectFileName) // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    {
        int TotalOpenAttempts = 0; // <a href="https://hero.handmade.network/episode/game-architecture/day256">256</a>

        FILE *ProjectFile = fopen(ProjectFileName, &quot;r&quot;); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        if(ProjectFile) // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        {
            fgets(BuildDirectory, sizeof(BuildDirectory) - 1, ProjectFile); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
            size_t BuildDirSize = strlen(BuildDirectory); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
            if((BuildDirSize) &amp;&amp; (BuildDirectory[BuildDirSize - 1] == &#39;n&#39;))
            {
                --BuildDirSize; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
            }

            if((BuildDirSize) &amp;&amp; (BuildDirectory[BuildDirSize - 1] != &#39;/&#39;)) // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
            {
                BuildDirectory[BuildDirSize++] = &#39;/&#39;; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
                BuildDirectory[BuildDirSize] = 0; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
            }

            char SourceFileDirectoryName[4096]; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
            char FileDirectoryName[4096]; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
            while(fgets(SourceFileDirectoryName, sizeof(SourceFileDirectoryName) - 1, ProjectFile)) // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
            {
                // NOTE(allen|a3.4.4): Here we get the list of files in this directory. // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
                // Notice that we free_file_list at the end. // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
                String dir = make_string(FileDirectoryName, 0, sizeof(FileDirectoryName)); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
                append(&amp;dir, SourceFileDirectoryName); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
                if(dir.size &amp;&amp; dir.str[dir.size-1] == &#39;n&#39;)
                {
                    --dir.size; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
                }

                if(dir.size &amp;&amp; dir.str[dir.size-1] != &#39;/&#39;) // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
                {
                    dir.str[dir.size++] = &#39;/&#39;; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
                }

                File_List list = app-&gt;get_file_list(app, dir.str, dir.size); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
                int dir_size = dir.size; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>

                for (int i = 0; i &lt; list.count; ++i) // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
                {
                    File_Info *info = list.infos + i; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
                    if (!info-&gt;folder) // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
                    {
                        String extension = file_extension(info-&gt;filename); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
                        if (IsCode(extension)) // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
                        {
                            // NOTE(allen): There&#39;s no way in the 4coder API to use relative // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
                            // paths at the moment, so everything should be full paths.  Which is // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
                            // managable.  Here simply set the dir string size back to where it // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
                            // was originally, so that new appends overwrite old ones. // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
                            dir.size = dir_size; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
                            append(&amp;dir, info-&gt;filename); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
                            push_parameter(app, par_name, dir.str, dir.size); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
                            push_parameter(app, par_do_in_background, 1); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
                            exec_command(app, cmdid_interactive_open); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
                            ++TotalOpenAttempts; // <a href="https://hero.handmade.network/episode/game-architecture/day256">256</a>
                        }
                    }
                }

                app-&gt;free_file_list(app, list); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
            }

            fclose(ProjectFile); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        }
    }    

    CUSTOM_COMMAND_SIG(casey_execute_arbitrary_command) // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    {
        Query_Bar bar; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        char space[1024], more_space[1024]; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        bar.prompt = make_lit_string(&quot;Command: &quot;); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        bar.string = make_fixed_width_string(space); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>

        if (!query_user_string(app, &amp;bar)) return; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        app-&gt;end_query_bar(app, &amp;bar, 0); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>

        if(match(bar.string, make_lit_string(&quot;project&quot;))) // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        {
    //        exec_command(app, open_all_code); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        }
        else if(match(bar.string, make_lit_string(&quot;open menu&quot;))) // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        {
            exec_command(app, cmdid_open_menu); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        }
        else
        {
            bar.prompt = make_fixed_width_string(more_space); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
            append(&amp;bar.prompt, make_lit_string(&quot;Unrecognized: &quot;)); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
            append(&amp;bar.prompt, bar.string); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
            bar.string.size = 0; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>

            app-&gt;start_query_bar(app, &amp;bar, 0); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
            app-&gt;get_user_input(app, EventOnAnyKey | EventOnButton, 0); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        }
    }

    internal void
    UpdateModalIndicator(Application_Links *app) // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    {
        Theme_Color normal_colors[] =  // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        {
            {Stag_Cursor, 0x40FF40}, // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
            {Stag_At_Cursor, 0x161616}, // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
            {Stag_Mark, 0x808080}, // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
            {Stag_Margin, 0x262626}, // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
            {Stag_Margin_Hover, 0x333333}, // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
            {Stag_Margin_Active, 0x404040}, // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
            {Stag_Bar, 0xCACACA} // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        };

        Theme_Color edit_colors[] =  // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        {
            {Stag_Cursor, 0xFF0000}, // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
            {Stag_At_Cursor, 0x00FFFF}, // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
            {Stag_Mark, 0xFF6F1A}, // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
            {Stag_Margin, 0x33170B}, // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
            {Stag_Margin_Hover, 0x49200F}, // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
            {Stag_Margin_Active, 0x934420}, // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
            {Stag_Bar, 0x934420} // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        };

        if (GlobalEditMode){ // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
            app-&gt;set_theme_colors(app, edit_colors, ArrayCount(edit_colors)); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        }
        else{ // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
            app-&gt;set_theme_colors(app, normal_colors, ArrayCount(normal_colors)); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        }
    }
    CUSTOM_COMMAND_SIG(begin_free_typing) // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    {
        GlobalEditMode = false; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        UpdateModalIndicator(app); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    }

    CUSTOM_COMMAND_SIG(end_free_typing) // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    {
        GlobalEditMode = true; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        UpdateModalIndicator(app); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    }

    #define DEFINE_FULL_BIMODAL_KEY(binding_name,edit_code,normal_code) CUSTOM_COMMAND_SIG(binding_name) {     if(GlobalEditMode)     {         edit_code;                }     else     {         normal_code;     } }

    #define DEFINE_BIMODAL_KEY(binding_name,edit_code,normal_code) DEFINE_FULL_BIMODAL_KEY(binding_name,exec_command(app,edit_code),exec_command(app,normal_code)) // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    #define DEFINE_MODAL_KEY(binding_name,edit_code) DEFINE_BIMODAL_KEY(binding_name,edit_code,cmdid_write_character) // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>

    //    cmdid_paste_next ? // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    //    cmdid_timeline_scrub ? // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    //    cmdid_history_backward, // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    //    cmdid_history_forward, // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    //    cmdid_toggle_line_wrap, // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    //    cmdid_close_minor_view, // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>

    DEFINE_MODAL_KEY(modal_space, cmdid_set_mark); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    DEFINE_MODAL_KEY(modal_back_slash, casey_clean_and_save); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    DEFINE_MODAL_KEY(modal_single_quote, casey_call_keyboard_macro); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    DEFINE_MODAL_KEY(modal_comma, casey_goto_previous_error); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    DEFINE_MODAL_KEY(modal_period, casey_fill_paragraph); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    DEFINE_MODAL_KEY(modal_forward_slash, cmdid_change_active_panel); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    DEFINE_MODAL_KEY(modal_semicolon, cmdid_cursor_mark_swap); // TODO(casey): Maybe cmdid_history_backward? // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    DEFINE_BIMODAL_KEY(modal_open_bracket, casey_begin_keyboard_macro_recording, write_and_auto_tab); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    DEFINE_BIMODAL_KEY(modal_close_bracket, casey_end_keyboard_macro_recording, write_and_auto_tab); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    DEFINE_MODAL_KEY(modal_a, cmdid_write_character); // TODO(casey): Arbitrary command + casey_quick_calc // <a href="https://hero.handmade.network/episode/game-architecture/day256">256</a>
    DEFINE_MODAL_KEY(modal_b, cmdid_interactive_switch_buffer); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    DEFINE_MODAL_KEY(modal_c, casey_find_corresponding_file); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    DEFINE_MODAL_KEY(modal_d, casey_kill_to_end_of_line); // <a href="https://hero.handmade.network/episode/game-architecture/day256">256</a>
    DEFINE_MODAL_KEY(modal_e, cmdid_write_character); // TODO(casey): Available // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    DEFINE_MODAL_KEY(modal_f, casey_paste_and_tab); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    DEFINE_MODAL_KEY(modal_g, goto_line); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    DEFINE_MODAL_KEY(modal_h, cmdid_auto_tab_range); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    DEFINE_MODAL_KEY(modal_i, cmdid_move_up); // <a href="https://hero.handmade.network/episode/game-architecture/day256">256</a>
    DEFINE_MODAL_KEY(modal_j, seek_white_or_token_left); // <a href="https://hero.handmade.network/episode/game-architecture/day256">256</a>
    DEFINE_MODAL_KEY(modal_k, cmdid_move_down); // <a href="https://hero.handmade.network/episode/game-architecture/day256">256</a>
    DEFINE_MODAL_KEY(modal_l, seek_white_or_token_right); // <a href="https://hero.handmade.network/episode/game-architecture/day256">256</a>
    DEFINE_MODAL_KEY(modal_m, casey_save_and_make_without_asking); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    DEFINE_MODAL_KEY(modal_n, casey_goto_next_error); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    DEFINE_MODAL_KEY(modal_o, query_replace); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    DEFINE_MODAL_KEY(modal_p, replace_in_range); // <a href="https://hero.handmade.network/episode/game-architecture/day256">256</a>
    DEFINE_MODAL_KEY(modal_q, cmdid_copy); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    DEFINE_MODAL_KEY(modal_r, reverse_search); // NOTE(allen): I&#39;ve modified my default search so you can use it now. // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    DEFINE_MODAL_KEY(modal_s, search); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    DEFINE_MODAL_KEY(modal_t, casey_load_todo); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    DEFINE_MODAL_KEY(modal_u, cmdid_undo); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    DEFINE_MODAL_KEY(modal_v, casey_switch_buffer_other_window); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    DEFINE_MODAL_KEY(modal_w, cmdid_cut); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    DEFINE_MODAL_KEY(modal_x, casey_find_corresponding_file_other_window); // <a href="https://hero.handmade.network/episode/game-architecture/day252">252</a>
    DEFINE_MODAL_KEY(modal_y, cmdid_redo); // <a href="https://hero.handmade.network/episode/game-architecture/day256">256</a>
    DEFINE_MODAL_KEY(modal_z, cmdid_interactive_open); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>

    DEFINE_MODAL_KEY(modal_1, casey_build_search); // TODO(casey): Shouldn&#39;t need to bind a key for this? // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    DEFINE_MODAL_KEY(modal_2, cmdid_write_character); // TODO(casey): Available // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    DEFINE_MODAL_KEY(modal_3, cmdid_write_character); // TODO(casey): Available // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    DEFINE_MODAL_KEY(modal_4, cmdid_write_character); // TODO(casey): Available // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    DEFINE_MODAL_KEY(modal_5, cmdid_write_character); // TODO(casey): Available // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    DEFINE_MODAL_KEY(modal_6, cmdid_write_character); // TODO(casey): Available // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    DEFINE_MODAL_KEY(modal_7, cmdid_write_character); // TODO(casey): Available // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    DEFINE_MODAL_KEY(modal_8, cmdid_write_character); // TODO(casey): Available // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    DEFINE_MODAL_KEY(modal_9, cmdid_write_character); // TODO(casey): Available // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    DEFINE_MODAL_KEY(modal_0, cmdid_kill_buffer); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    DEFINE_MODAL_KEY(modal_minus, cmdid_write_character); // TODO(casey): Available // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    DEFINE_MODAL_KEY(modal_equals, casey_execute_arbitrary_command); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>

    DEFINE_BIMODAL_KEY(modal_backspace, casey_delete_token_left, cmdid_backspace); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    DEFINE_BIMODAL_KEY(modal_up, cmdid_move_up, cmdid_move_up); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    DEFINE_BIMODAL_KEY(modal_down, cmdid_move_down, cmdid_move_down); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    DEFINE_BIMODAL_KEY(modal_left, seek_white_or_token_left, cmdid_move_left);  // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    DEFINE_BIMODAL_KEY(modal_right, seek_white_or_token_right, cmdid_move_right); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    DEFINE_BIMODAL_KEY(modal_delete, casey_delete_token_right, cmdid_delete); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    DEFINE_BIMODAL_KEY(modal_home, casey_seek_beginning_of_line, casey_seek_beginning_of_line_and_tab); // <a href="https://hero.handmade.network/episode/game-architecture/day256">256</a>
    DEFINE_BIMODAL_KEY(modal_end, cmdid_seek_end_of_line, cmdid_seek_end_of_line); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    DEFINE_BIMODAL_KEY(modal_page_up, cmdid_page_up, cmdid_seek_whitespace_up); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    DEFINE_BIMODAL_KEY(modal_page_down, cmdid_page_down, cmdid_seek_whitespace_down); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    DEFINE_BIMODAL_KEY(modal_tab, cmdid_word_complete, cmdid_word_complete); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>

    HOOK_SIG(casey_file_settings) // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    {
        // NOTE(allen|a4): As of alpha 4 hooks can have parameters which are // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        // received through functions like this app-&gt;get_parameter_buffer. // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        // This is different from the past where this hook got a buffer // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        // from app-&gt;get_active_buffer. // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        Buffer_Summary buffer = app-&gt;get_parameter_buffer(app, 0); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>

        int treat_as_code = 0; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        int treat_as_project = 0; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>

        if (buffer.file_name &amp;&amp; buffer.size &lt; (16 &lt;&lt; 20)) // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        {
            String ext = file_extension(make_string(buffer.file_name, buffer.file_name_len)); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
            treat_as_code = IsCode(ext); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
            treat_as_project = match(ext, make_lit_string(&quot;prj&quot;)); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        }

        push_parameter(app, par_buffer_id, buffer.buffer_id); // <a href="https://hero.handmade.network/episode/game-architecture/day285">285</a> <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        push_parameter(app, par_lex_as_cpp_file, treat_as_code); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        push_parameter(app, par_wrap_lines, !treat_as_code); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        push_parameter(app, par_key_mapid, (treat_as_code)?((int)my_code_map):((int)mapid_file)); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        exec_command(app, cmdid_set_settings); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>

        if(treat_as_project) // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        {
            OpenProject(app, buffer.file_name); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
            // NOTE(casey): Don&#39;t actually want to kill this, or you can never edit the project. // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    //        exec_command(app, cmdid_kill_buffer); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>

        }

        return(0);
    }

    bool
    CubicUpdateFixedDuration1(float *P0, float *V0, float P1, float V1, float Duration, float dt) // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    {
        bool Result = false; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>

        if(dt &gt; 0) // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        {
            if(Duration &lt; dt) // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
            {
                *P0 = P1 + (dt - Duration)*V1; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
                *V0 = V1; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
                Result = true; // <a href="https://hero.handmade.network/episode/game-architecture/day255">255</a> <a href="https://hero.handmade.network/episode/game-architecture/day249">249</a> <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a> <a href="https://hero.handmade.network/episode/game-architecture/day227">227</a> <a href="https://hero.handmade.network/episode/game-architecture/day212">212</a> <a href="https://hero.handmade.network/episode/game-architecture/day161">161</a> <a href="https://hero.handmade.network/episode/game-architecture/day074">074</a> <a href="https://hero.handmade.network/episode/game-architecture/day070">070</a>
            }
            else
            {
                float t = (dt / Duration); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
                float u = (1.0f - t); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>

                float C0 = 1*u*u*u; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
                float C1 = 3*u*u*t; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
                float C2 = 3*u*t*t; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
                float C3 = 1*t*t*t; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>

                float dC0 = -3*u*u; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
                float dC1 = -6*u*t + 3*u*u; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
                float dC2 =  6*u*t - 3*t*t; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
                float dC3 =  3*t*t; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>

                float B0 = *P0; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
                float B1 = *P0 + (Duration / 3.0f) * *V0; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
                float B2 = P1 - (Duration / 3.0f) * V1; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
                float B3 = P1; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>

                *P0 = C0*B0 + C1*B1 + C2*B2 + C3*B3; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
                *V0 = (dC0*B0 + dC1*B1 + dC2*B2 + dC3*B3) * (1.0f / Duration); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
            }
        }

        return(Result);
    }

    struct Casey_Scroll_Velocity // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    {
        float x, y, t; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    };

    Casey_Scroll_Velocity casey_scroll_velocity_[16] = {0}; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    Casey_Scroll_Velocity *casey_scroll_velocity = casey_scroll_velocity_; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>

    SCROLL_RULE_SIG(casey_smooth_scroll_rule){ // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        float dt = 1.0f/60.0f; // TODO(casey): Why do I not get the timestep here? // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        Casey_Scroll_Velocity *velocity = casey_scroll_velocity + view_id; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        int result = 0; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        if(is_new_target) // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        {
            if((*scroll_x != target_x) || // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
                (*scroll_y != target_y)) // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
            {
                velocity-&gt;t = 0.1f; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
            }
        }

        if(velocity-&gt;t &gt; 0) // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        {
            result = !(CubicUpdateFixedDuration1(scroll_x, &amp;velocity-&gt;x, target_x, 0.0f, velocity-&gt;t, dt) || // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
                    CubicUpdateFixedDuration1(scroll_y, &amp;velocity-&gt;y, target_y, 0.0f, velocity-&gt;t, dt)); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        }

        velocity-&gt;t -= dt; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        if(velocity-&gt;t &lt; 0) // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        {
            velocity-&gt;t = 0; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
            *scroll_x = target_x; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
            *scroll_y = target_y; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        }

        return(result); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    }

    #include &lt;windows.h&gt; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a> <a href="https://hero.handmade.network/episode/game-architecture/day174">174</a> <a href="https://hero.handmade.network/episode/game-architecture/day170">170</a> <a href="https://hero.handmade.network/episode/win32-platform/day001">001</a>
    #pragma comment(lib, &quot;user32.lib&quot;) // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    static HWND GlobalWindowHandle; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    static WINDOWPLACEMENT GlobalWindowPosition = {sizeof(GlobalWindowPosition)}; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    internal BOOL CALLBACK win32_find_4coder_window(HWND Window, LPARAM LParam) // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    {
        BOOL Result = TRUE; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>

        char TestClassName[256]; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        GetClassName(Window, TestClassName, sizeof(TestClassName)); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        if((strcmp(&quot;4coder-win32-wndclass&quot;, TestClassName) == 0) &amp;&amp;  // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
           ((HINSTANCE)GetWindowLongPtr(Window, GWLP_HINSTANCE) == GetModuleHandle(0))) // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        {
            GlobalWindowHandle = Window; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
            Result = FALSE; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        }

        return(Result);
    }

    internal void
    win32_toggle_fullscreen(void) // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    {
    #if 0
        // NOTE(casey): This follows Raymond Chen&#39;s prescription // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a> <a href="https://hero.handmade.network/episode/game-architecture/day040">040</a>
        // for fullscreen toggling, see: // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a> <a href="https://hero.handmade.network/episode/game-architecture/day040">040</a>
        // http://blogs.msdn.com/b/oldnewthing/archive/2010/04/12/9994016.aspx // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a> <a href="https://hero.handmade.network/episode/game-architecture/day040">040</a>

        HWND Window = GlobalWindowHandle; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        DWORD Style = GetWindowLong(Window, GWL_STYLE); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a> <a href="https://hero.handmade.network/episode/game-architecture/day040">040</a>
        if(Style &amp; WS_OVERLAPPEDWINDOW) // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a> <a href="https://hero.handmade.network/episode/game-architecture/day040">040</a>
        {
            MONITORINFO MonitorInfo = {sizeof(MonitorInfo)}; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a> <a href="https://hero.handmade.network/episode/game-architecture/day040">040</a>
            if(GetWindowPlacement(Window, &amp;GlobalWindowPosition) &amp;&amp; // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a> <a href="https://hero.handmade.network/episode/game-architecture/day040">040</a>
                GetMonitorInfo(MonitorFromWindow(Window, MONITOR_DEFAULTTOPRIMARY), &amp;MonitorInfo)) // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
            {
                SetWindowLong(Window, GWL_STYLE, Style &amp; ~WS_OVERLAPPEDWINDOW); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a> <a href="https://hero.handmade.network/episode/game-architecture/day040">040</a>
                SetWindowPos(Window, HWND_TOP, // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a> <a href="https://hero.handmade.network/episode/game-architecture/day040">040</a>
                                MonitorInfo.rcMonitor.left, MonitorInfo.rcMonitor.top, // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
                                MonitorInfo.rcMonitor.right - MonitorInfo.rcMonitor.left, // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
                                MonitorInfo.rcMonitor.bottom - MonitorInfo.rcMonitor.top, // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
                                SWP_NOOWNERZORDER | SWP_FRAMECHANGED); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
            }
        }
        else
        {
            SetWindowLong(Window, GWL_STYLE, Style | WS_OVERLAPPEDWINDOW); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a> <a href="https://hero.handmade.network/episode/game-architecture/day040">040</a>
            SetWindowPlacement(Window, &amp;GlobalWindowPosition); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a> <a href="https://hero.handmade.network/episode/game-architecture/day040">040</a>
            SetWindowPos(Window, 0, 0, 0, 0, 0, // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a> <a href="https://hero.handmade.network/episode/game-architecture/day040">040</a>
                            SWP_NOMOVE | SWP_NOSIZE | SWP_NOZORDER | // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
                            SWP_NOOWNERZORDER | SWP_FRAMECHANGED); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a> <a href="https://hero.handmade.network/episode/game-architecture/day040">040</a>
        }
    #else
        ShowWindow(GlobalWindowHandle, SW_MAXIMIZE); // <a href="https://hero.handmade.network/episode/game-architecture/day256">256</a>
    #endif
    }

    HOOK_SIG(casey_start) // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
    {
        exec_command(app, cmdid_open_panel_vsplit); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        app-&gt;change_theme(app, literal(&quot;Handmade Hero&quot;)); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        app-&gt;change_font(app, literal(&quot;liberation mono&quot;)); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>

        Theme_Color colors[] = // <a href="https://hero.handmade.network/episode/game-architecture/day256">256</a>
        {
            {Stag_Default, 0xA08563}, // <a href="https://hero.handmade.network/episode/game-architecture/day256">256</a>
            // {Stag_Bar, }, // <a href="https://hero.handmade.network/episode/game-architecture/day256">256</a>
            // {Stag_Bar_Active, }, // <a href="https://hero.handmade.network/episode/game-architecture/day256">256</a>
            // {Stag_Base, }, // <a href="https://hero.handmade.network/episode/game-architecture/day256">256</a>
            // {Stag_Pop1, }, // <a href="https://hero.handmade.network/episode/game-architecture/day256">256</a>
            // {Stag_Pop2, }, // <a href="https://hero.handmade.network/episode/game-architecture/day256">256</a>
            // {Stag_Back, }, // <a href="https://hero.handmade.network/episode/game-architecture/day256">256</a>
            // {Stag_Margin, }, // <a href="https://hero.handmade.network/episode/game-architecture/day256">256</a>
            // {Stag_Margin_Hover, }, // <a href="https://hero.handmade.network/episode/game-architecture/day256">256</a>
            // {Stag_Margin_Active, }, // <a href="https://hero.handmade.network/episode/game-architecture/day256">256</a>
            // {Stag_Cursor, }, // <a href="https://hero.handmade.network/episode/game-architecture/day256">256</a>
            // {Stag_At_Cursor, }, // <a href="https://hero.handmade.network/episode/game-architecture/day256">256</a>
            // {Stag_Highlight, }, // <a href="https://hero.handmade.network/episode/game-architecture/day256">256</a>
            // {Stag_At_Highlight, }, // <a href="https://hero.handmade.network/episode/game-architecture/day256">256</a>
            {Stag_Comment, 0x7D7D7D}, // <a href="https://hero.handmade.network/episode/game-architecture/day256">256</a>
            {Stag_Keyword, 0xCD950C}, // <a href="https://hero.handmade.network/episode/game-architecture/day256">256</a>
            // {Stag_Str_Constant, }, // <a href="https://hero.handmade.network/episode/game-architecture/day256">256</a>
            // {Stag_Char_Constant, }, // <a href="https://hero.handmade.network/episode/game-architecture/day256">256</a>
            // {Stag_Int_Constant, }, // <a href="https://hero.handmade.network/episode/game-architecture/day256">256</a>
            // {Stag_Float_Constant, }, // <a href="https://hero.handmade.network/episode/game-architecture/day256">256</a>
            // {Stag_Bool_Constant, }, // <a href="https://hero.handmade.network/episode/game-architecture/day256">256</a>
            {Stag_Preproc, 0xDAB98F}, // <a href="https://hero.handmade.network/episode/game-architecture/day256">256</a>
            {Stag_Include, 0x6B8E23}, // <a href="https://hero.handmade.network/episode/game-architecture/day256">256</a>
            // {Stag_Special_Character, }, // <a href="https://hero.handmade.network/episode/game-architecture/day256">256</a>
            // {Stag_Highlight_Junk, }, // <a href="https://hero.handmade.network/episode/game-architecture/day256">256</a>
            // {Stag_Highlight_White, }, // <a href="https://hero.handmade.network/episode/game-architecture/day256">256</a>
            // {Stag_Paste, }, // <a href="https://hero.handmade.network/episode/game-architecture/day256">256</a>
            // {Stag_Undo, }, // <a href="https://hero.handmade.network/episode/game-architecture/day256">256</a>
            // {Stag_Next_Undo, }, // <a href="https://hero.handmade.network/episode/game-architecture/day256">256</a>
        };
        app-&gt;set_theme_colors(app, colors, ArrayCount(colors)); // <a href="https://hero.handmade.network/episode/game-architecture/day256">256</a>

        win32_toggle_fullscreen(); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>

        return(0);
    }

    extern &quot;C&quot; GET_BINDING_DATA(get_bindings) // <a href="https://hero.handmade.network/episode/game-architecture/day256">256</a>
    {
        Bind_Helper context_actual = begin_bind_helper(data, size); // <a href="https://hero.handmade.network/episode/game-architecture/day256">256</a>
        Bind_Helper *context = &amp;context_actual; // <a href="https://hero.handmade.network/episode/game-architecture/day256">256</a>

        set_hook(context, hook_start, casey_start); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        set_hook(context, hook_open_file, casey_file_settings); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        set_scroll_rule(context, casey_smooth_scroll_rule); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>

        EnumWindows(win32_find_4coder_window, 0); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>

        begin_map(context, mapid_global); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        {
            bind(context, &#39;z&#39;, MDFR_NONE, cmdid_interactive_open); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
            bind(context, &#39;x&#39;, MDFR_NONE, casey_open_in_other); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
            bind(context, &#39;t&#39;, MDFR_NONE, casey_load_todo); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
            bind(context, &#39;/&#39;, MDFR_NONE, cmdid_change_active_panel); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
            bind(context, &#39;b&#39;, MDFR_NONE, cmdid_interactive_switch_buffer); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
            bind(context, key_page_up, MDFR_NONE, search); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
            bind(context, key_page_down, MDFR_NONE, reverse_search); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
            bind(context, &#39;m&#39;, MDFR_NONE, casey_save_and_make_without_asking); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        }        
        end_map(context); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>

        begin_map(context, mapid_file); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>

        bind_vanilla_keys(context, cmdid_write_character); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>

        bind(context, key_insert, MDFR_NONE, begin_free_typing); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        bind(context, &#39;`&#39;, MDFR_NONE, begin_free_typing); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        bind(context, key_esc, MDFR_NONE, end_free_typing); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        bind(context, &#39;n&#39;, MDFR_NONE, casey_newline_and_indent);
        bind(context, &#39;n&#39;, MDFR_SHIFT, casey_newline_and_indent);

        // NOTE(casey): Modal keys come here. // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        bind(context, &#39; &#39;, MDFR_NONE, modal_space); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        bind(context, &#39; &#39;, MDFR_SHIFT, modal_space); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>

        bind(context, &#39;\&#39;, MDFR_NONE, modal_back_slash);
        bind(context, &#39;&#39;&#39;, MDFR_NONE, modal_single_quote);
        bind(context, &#39;,&#39;, MDFR_NONE, modal_comma); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        bind(context, &#39;.&#39;, MDFR_NONE, modal_period); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        bind(context, &#39;/&#39;, MDFR_NONE, modal_forward_slash); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        bind(context, &#39;;&#39;, MDFR_NONE, modal_semicolon); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        bind(context, &#39;[&#39;, MDFR_NONE, modal_open_bracket); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        bind(context, &#39;]&#39;, MDFR_NONE, modal_close_bracket); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        bind(context, &#39;{&#39;, MDFR_NONE, write_and_auto_tab); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        bind(context, &#39;}&#39;, MDFR_NONE, write_and_auto_tab); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        bind(context, &#39;a&#39;, MDFR_NONE, modal_a); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        bind(context, &#39;b&#39;, MDFR_NONE, modal_b); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        bind(context, &#39;c&#39;, MDFR_NONE, modal_c); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        bind(context, &#39;d&#39;, MDFR_NONE, modal_d); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        bind(context, &#39;e&#39;, MDFR_NONE, modal_e); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        bind(context, &#39;f&#39;, MDFR_NONE, modal_f); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        bind(context, &#39;g&#39;, MDFR_NONE, modal_g); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        bind(context, &#39;h&#39;, MDFR_NONE, modal_h); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        bind(context, &#39;i&#39;, MDFR_NONE, modal_i); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        bind(context, &#39;j&#39;, MDFR_NONE, modal_j); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        bind(context, &#39;k&#39;, MDFR_NONE, modal_k); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        bind(context, &#39;l&#39;, MDFR_NONE, modal_l); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        bind(context, &#39;m&#39;, MDFR_NONE, modal_m); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        bind(context, &#39;n&#39;, MDFR_NONE, modal_n); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        bind(context, &#39;o&#39;, MDFR_NONE, modal_o); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        bind(context, &#39;p&#39;, MDFR_NONE, modal_p); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        bind(context, &#39;q&#39;, MDFR_NONE, modal_q); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        bind(context, &#39;r&#39;, MDFR_NONE, modal_r); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        bind(context, &#39;s&#39;, MDFR_NONE, modal_s); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        bind(context, &#39;t&#39;, MDFR_NONE, modal_t); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        bind(context, &#39;u&#39;, MDFR_NONE, modal_u); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        bind(context, &#39;v&#39;, MDFR_NONE, modal_v); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        bind(context, &#39;w&#39;, MDFR_NONE, modal_w); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        bind(context, &#39;x&#39;, MDFR_NONE, modal_x); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        bind(context, &#39;y&#39;, MDFR_NONE, modal_y); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        bind(context, &#39;z&#39;, MDFR_NONE, modal_z); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>

        bind(context, &#39;1&#39;, MDFR_NONE, modal_1); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        bind(context, &#39;2&#39;, MDFR_NONE, modal_2); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        bind(context, &#39;3&#39;, MDFR_NONE, modal_3); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        bind(context, &#39;4&#39;, MDFR_NONE, modal_4); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        bind(context, &#39;5&#39;, MDFR_NONE, modal_5); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        bind(context, &#39;6&#39;, MDFR_NONE, modal_6); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        bind(context, &#39;7&#39;, MDFR_NONE, modal_7); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        bind(context, &#39;8&#39;, MDFR_NONE, modal_8); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        bind(context, &#39;9&#39;, MDFR_NONE, modal_9); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        bind(context, &#39;0&#39;, MDFR_NONE, modal_0); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        bind(context, &#39;-&#39;, MDFR_NONE, modal_minus); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        bind(context, &#39;=&#39;, MDFR_NONE, modal_equals); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>

        bind(context, key_back, MDFR_NONE, modal_backspace); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        bind(context, key_back, MDFR_SHIFT, modal_backspace); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>

        bind(context, key_up, MDFR_NONE, modal_up); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        bind(context, key_up, MDFR_SHIFT, modal_up); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>

        bind(context, key_down, MDFR_NONE, modal_down); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        bind(context, key_down, MDFR_SHIFT, modal_down); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>

        bind(context, key_left, MDFR_NONE, modal_left); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        bind(context, key_left, MDFR_SHIFT, modal_left); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>

        bind(context, key_right, MDFR_NONE, modal_right); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        bind(context, key_right, MDFR_SHIFT, modal_right); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>

        bind(context, key_del, MDFR_NONE, modal_delete); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        bind(context, key_del, MDFR_SHIFT, modal_delete); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>

        bind(context, key_home, MDFR_NONE, modal_home); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        bind(context, key_home, MDFR_SHIFT, modal_home); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>

        bind(context, key_end, MDFR_NONE, modal_end); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        bind(context, key_end, MDFR_SHIFT, modal_end); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>

        bind(context, key_page_up, MDFR_NONE, modal_page_up); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        bind(context, key_page_up, MDFR_SHIFT, modal_page_up); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>

        bind(context, key_page_down, MDFR_NONE, modal_page_down); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>
        bind(context, key_page_down, MDFR_SHIFT, modal_page_down); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>

        bind(context, &#39;t&#39;, MDFR_NONE, modal_tab);
        bind(context, &#39;t&#39;, MDFR_SHIFT, modal_tab);

        end_map(context); // <a href="https://hero.handmade.network/episode/game-architecture/day247">247</a>

        end_bind_helper(context); // <a href="https://hero.handmade.network/episode/game-architecture/day256">256</a>
        return context-&gt;write_total; // <a href="https://hero.handmade.network/episode/game-architecture/day256">256</a>
    }
