# NAME

greple - grep with multiple keywords

# SYNOPSIS

__greple__ \[__-M___module_\] \[ __-options__ \] pattern \[ file... \]

    PATTERN
      pattern              'positive +must -negative ?alternative'
      -e pattern           regex pattern match across line boundary
      -r pattern           regex pattern cannot be compromised
      -v pattern           regex pattern not to be matched
      --le pattern         lexical expression (same as bare pattern)
      --re pattern         regular expression
      --fe pattern         fixed expression
      --file file          file contains search pattern
    MATCH
      -i                   ignore case
      --need=[+-]n         required positive match count
      --allow=[+-]n        acceptable negative match count
    STYLE
      -l                   list filename only
      -c                   print count of matched block only
      -n                   print line number
      -h                   do not display filenames
      -H                   always display filenames
      -o                   print only the matching part
      -A[n]                after match context
      -B[n]                before match context
      -C[n]                after and before match context
      --join               delete newline in the matched part
      --joinby=string      replace newline in the matched text by string
      --filestyle=style    how filename printed (once, separate, line)
      --linestyle=style    how line number printed (separate, line)
      --separate           set filestyle and linestyle both "separate"
    FILE
      --glob=glob          glob target files
      --chdir              change directory before search
      --readlist           get filenames from stdin
    COLOR
      --color=when         use terminal color (auto, always, never)
      --nocolor            same as --color=never
      --colormap=color     R, G, B, C, M, Y, W, Standout, Double-struck, Underline
      --colorful           use default multiple colors
      --[no]256            use ANSI 256 colors
      --regioncolor        use different color for inside/outside regions
      --random             random color
      --face
    BLOCK
      -p                   paragraph mode
      --all                print whole data
      --block=pattern      specify the block of records
      --blockend=s         specify the block end mark (Default: "--\n")
    REGION
      --inside=pattern     select matches inside of pattern
      --outside=pattern    select matches outside of pattern
      --include=pattern    reduce matches to the area
      --exclude=pattern    reduce matches to outside of the area
      --strict             strict mode for --inside/outside --block
    CHARACTER CODE
      --icode=name         specify file encoding
      --ocode=name         specify output encoding
    FILTER
      --if=filter          set filter command
      --of=filter          output filter command
      --noif               disable default input filter
    RUNTIME FUNCTION
      --print=func         print function
      --continue           continue after print function
      --call=func          call function before search
    PGP
      --[no]pgp            decrypt and find PGP file (Default: false)
      --pgppass=phrase     pgp passphrase
    OTHER
      --norc               skip reading startup file
      --man                display command or module manual page
      --show               display module file
      -d flags             display info (f:file d:dir c:color m:misc s:stat)

# DESCRIPTION

## MULTIPLE KEYWORDS

__greple__ has almost the same function as Unix command [egrep(1)](http://man.he.net/man1/egrep) but
the search is done in the manner similar to search engine.  For
example, next command print lines those contain all of \`foo' and \`bar'
and \`baz'.

    greple 'foo bar baz' ...

Each word can be found in any order and/or any place in the string.
So this command find all of following texts.

    foo bar baz
    baz bar foo
    the foo, bar and baz

If you want to use OR syntax, prepend question (\`?') mark on each
token, or use regular expression.

    greple 'foo bar baz ?yabba ?dabba ?doo'
    greple 'foo bar baz yabba|dabba|doo'

This command will print the line which contains all of \`foo', \`bar'
and \`baz' and one or more from \`yabba', \`dabba' or \`doo'.

NOT operator can be specified by prefixing the token by minus (\`-')
sign.  Next example will show the line which contain both \`foo' and
bar' but none of \`yabba' or \`dabba' or \`doo'.

    greple 'foo bar -yabba -dabba -doo'

This can be written as this using __-e__ and __-v__ option.

    greple -e foo -e bar -v yabba -v dabba -v doo
    greple -e foo -e bar -v 'yabba|dabba|doo'

If \`+' is placed to positive matching pattern, that pattern is marked
as required, and match required count is automatically set to the
number of required pattern.  So

    greple '+foo bar baz'

commands implicitly set the option `--need 1`, and consequently print
all lines including \`foo'.  If you want to search lines which includes
either or both of \`bar' and \`baz', use like this:

    greple '+foo bar baz' --need 2
    greple '+foo bar baz' --need +1

## LINE ACROSS MATCH

__greple__ also search the pattern across the line boundaries.  This is
especially useful to handle Asian multi-byte text.  Japanese text can
be separated by newline almost any place of the text.  So the search
pattern may spread out on multiple lines.

As for ascii text, space character in the pattern matches any kind of
space including newline.  Next example will search the word sequence
of \`foo', \`bar' and 'baz', even they spread out to multiple lines.

    greple -e 'foo bar baz'

Option __-e__ is necessary because space is taken as a token separator
in the bare or __--le__ pattern.

# OPTIONS

## PATTERNS

If specific option is not provided, __greple__ takes the first argument
as a search pattern specified by __-le__ option.  All of these patterns
can be specified multiple times.

Command itself is written in Perl, and any kind of Perl style regular
expression can be used in patterns.  See [perlre(1)](http://man.he.net/man1/perlre) for detail.

Note that multiple line modifier (`m`) is set when executed, so put
`(?-m)` at the beginning of regex if you want to explicitly disable
it.

Order of capture group in the pattern is not guaranteed.  Please avoid
to use direct index, and use relative or named capture group instead.
For example, repated character can be written as `(\w)\g{-1}`
or `(?<c>\w)\g{c}`.

- __--le__=_pattern_

    Treat the string as a collection of tokens separated by spaces.  Each
    token is interpreted by the first character.  Token start with \`-'
    means negative pattern, \`?' means alternative, and \`+' does required.

    Next example print lines which contains \`foo' and \`bar', and one or
    more of \`yabba' and 'dabba', and none of \`bar' and \`doo'.

        greple --le='foo bar -baz ?yabba ?dabba -doo'

    Multiple \`?' preceded tokens are treated all mixed together.  That
    means \`?A|B ?C|D' is equivalent to \`?A|B|C|D'.  If you want to mean
    \`(A or B) and (C or D)', use AND syntax instead: \`A|B C|D'.

    If the pattern start with ampersand (\`&'), it is treated as a
    function, and the function is called instead of searching pattern.
    Function call interface is same as the one for block/region options.

    If you have a definition of _odd\_line_ function in you `.greplrc`,
    which is described in this manual later, you can print odd number
    lines like this:

        greple -n '&odd_line' file

    This is the summary of start character for __--le__ option:

        +  Required pattern
        -  Negative match pattern
        ?  Alternative pattern
        &  Function call

- __-e__ _pattern_, __--and__=_pattern_

    Specify positive match token.  Next two commands are equivalent.

        greple 'foo bar baz'
        greple -e foo -e bar -e baz

    First character is not interpreted, so next commands will search the
    pattern \`-baz'.

        greple -e -baz

    Space characters are treated specially by __-e__ and __-v__ options.
    They are replaced by the pattern which matches any number of
    white spaces including newline.  So the pattern can be expand to
    multiple lines.  Next commands search the series of word \`foo', \`bar'
    and \`baz' even if they are separated by newlines.

        greple -e 'foo bar baz'

- __-r__ _pattern_, __--must__=_pattern_

    Specify required match token.  Next two commands are equivalent.

        greple '+foo bar baz'
        greple -r foo -e bar -e baz

- __-v__ _pattern_, __--not__=_pattern_

    Specify negative match token.  Because it does not affect to the bare
    pattern argument, you can narrow down the search result like this.

        greple foo pattern file
        greple foo pattern file -v bar
        greple foo pattern file -v bar -v baz

- __--re__=_pattern_

    Specify regular expression.  No special treatment for space and wide
    characters.

- __--fe__=_pattern_

    Specify fixed string pattern, like fgrep.

- __-i__, __--ignore-case__

    Ignore case.

- __--need__=_n_
- __--allow__=_n_

    Option to compromize matching condition.  Option __--need__ specifies
    the required match count, and __--allow__ the number of negative
    condition to be overlooked.

        greple --need=2 --allow=1 'foo bar baz -yabba -dabba -doo'

    Above command prints the line which contains two or more from \`foo',
    \`bar' and \`baz', and does not include more than one of \`yabba',
    \`dabba' or \`doo'.

    Using option __--need__=_1_, __greple__ produces same result as __grep__
    command.

        grep -e foo -e bar -e baz
        greple --need=1 -e foo -e bar -e baz

    When the count _n_ is negative value, it is subtracted from default
    value.

- __-f__ _file_, __--file__=_file_

    Specify the file which contains search pattern.  When file contains
    multiple lines, patterns on each lines are search in OR context.

    Blank line and the line starting with sharp (#) character is ignored.
    Two slashes (//) and following string are taken as a comment and
    removed with preceding spaces.

    Multiple file can be specified, but they will be mixed into single
    pattern.

## STYLES

- __-l__

    List filename only.

- __-c__, __--count__

    Print count of matched block.

- __-n__, __--line-number__

    Show line number.

- __-h__, __--no-filename__

    Do not display filename.

- __-H__

    Display filename always.

- __-o__

    Print matched string only.

- __-A__\[_n_\], __--after-context__\[=_n_\]
- __-B__\[_n_\], __--before-context__\[=_n_\]
- __-C__\[_n_\], __--context__\[=_n_\]

    Print _n_-blocks before/after matched string.  The value _n_ can be
    omitted and the default is 2.  When used with __--paragraph__ or
    __--block__ option, _n_ means number of paragraph or block.

    Actually, these options expand the area of logical operation.  It
    means

        grep -C1 'foo bar baz'

    matches following text.

        foo
        bar
        baz

    Moreover

        greple -C1 'foo baz'

    also matches this text, because matching blocks around \`foo' and \`bar'
    overlaps each other and makes single block.

- __--join__
- __--joinby__=_string_

    Convert newline character found in matched string to empty or specifed
    _string_.  Using __--join__ with __-o__ (only-matching) option, you can
    collect searching sentence list in one per line form.  This is
    sometimes useful for Japanese text processing.  For example, next
    command prints the list of KATAKANA words, including those spread
    across multiple lines.

        greple -ho --join '\p{InKatakana}+(\n\p{InKatakana}+)*'

    Space separated word sequence can be processed with __--joinby__
    option.  Next exapmle prints all \`for \*something\*' pattern in pod
    documents within Perl script.

        greple -Mperl --pod -ioe '\bfor \w+' --joinby ' '

- __--filestyle__=_line_|_once_|_separate_, __--fs__

    Default style is _line_, and __greple__ prints filename at the
    beginning of each line.  Style _once_ prints the filename only once
    at the first time.  Style _separate_ prints filename in the separate
    line before each line or block.

- __--linestyle__=_line_|_separate_, __--ls__

    Default style is _line_, and __greple__ prints line numbers at the
    beginning of each line.  Style _separate_ prints line number in the
    separate line before each line or block.

- __--separate__

    Shortcut for __--filestyle__=_separate_ __--linestyle__=_separate_.
    This is convenient to use block mode search and visiting each location
    from supporting tool, such as Emacs.

## FILES

- __--glob__=_pattern_

    Get files matches to specified pattern and use them as a target files.
    Using __--chdir__ and __--glob__ makes easy to use __greple__ for fixed
    common job.

- __--chdir__=_directory_

    Change directory before processing files.  When multiple directories
    are specified in __--chdir__ option, by using wildcard form or
    repeating option, __--glob__ file expantion will be done for every
    directories.

        greple --chdir '/usr/man/man?' --glob '*.[0-9]' ...

- __--readlist__

    Get filenames from standard input.  Read standard input and use each
    line as a filename for searching.  You can feed the output from other
    command like [find(1)](http://man.he.net/man1/find) for __greple__ with this option.  Next example
    searches string from files modified within 7 days:

        find . -mtime -7 -print | greple --readlist pattern

## COLORS

- __--color__=_auto_|_always_|_never_, __--nocolor__

    Use terminal color capability to emphasize the matched text.  Default
    is \`auto': effective when STDOUT is a terminal and option __-o__ is not
    given, not otherwise.  Option value \`always' and \`never' will work as
    expected.

    Option __--nocolor__ is alias for __--color__=_never_.

- __--colormap__=_RGBCYMKWrgbcymkwSUDF_, __--quote__=_start_,_end_

    Specify color map.  Default is RD: RED and BOLD.

    COLOR is combination of single character representing uppercase
    foreground color :

        R  Red
        G  Green
        B  Blue
        C  Cyan
        M  Magenta
        Y  Yellow
        K  Black
        W  White

    and corresponding lowercase background color :

        r, g, b, c, m, y, k, w

    or RGB value if using ANSI 256 color terminal :

        FORMAT:
            foreground[/background]

        COLOR:
            000 .. 555       : 6 x 6 x 6 216 colors
            000000 .. FFFFFF : 24bit RGB mapped to 216 colors

        Sample:
            005     0000FF        : blue foreground
               /505       /FF00FF : magenta background
            000/555 000000/FFFFFF : black on white
            500/050 FF0000/00FF00 : red on green

    and other effects :

        S  Standout (reverse video)
        U  Underline
        D  Double-struck (boldface)
        F  Flash (blink)

    If the mode string contains colon \`:' character, they are used to
    quote the matched string.  If you want to quote the pattern by angle
    bracket, use like this.

        greple --quote='<:>' pattern

    Option __--quote__ is an alias for __--colormap__, but it set the
    option __--color__=_always_ at the same time.

    Multiple colors can be specified separating by white space or comma,
    or by repeating options.  Those colors will be applied for each
    pattern keywords.  Next command will show word \`foo' in red, \`bar' in
    green and \`baz' in blue.

        greple --colormap='R G B' 'foo bar baz'

        greple --cm R -e foo --cm G -e bar --cm B -e baz

- __--colormap__=_field_=_color_,_field_=_color_,...

    Another form of colormap option to specify the color for fields:

        FILE      File name
        LINE      Line number
        BLOCKEND  Block end mark

- __--\[no\]colorful__

    Shortcut for __--colormap__='_RD GD BD CD MD YD_' in ANSI 16 colors
    mode, and __--colormap__='_D/544 D/454 D/445 D/455 D/454 D/554_' and
    other combination of 3, 4, 5 for 256 colors mode.  Enabled by default.

- __--\[no\]256__

    Set/unset ANSI 256 color mode.  Enabled by default.

- __--regioncolor__

    Use different colors for each __--inside__/__outside__ regions.
    Disabled by default.

- __--face__=\[-+\]_effect_

    Set or unset specified _effect_ for all color specs.  Use \`+'
    (optional) to set, and \`-' to unset.  Effect is a single character
    expressing: S (Standout), U (Underline), D (Double-struck), F (Flash).

    Next example romove D (double-struck) effect.

        greple --face -D

    Multiple effects can be set/unset at once.

        greple --face SF-D

## BLOCKS

- __-p__, __--paragraph__

    Print the paragraph which contains the pattern.  Each paragraph is
    delimited by two or more successive newline characters by default.  Be
    aware that an empty line is not paragraph delimiter if which contains
    space characters.  Example:

        greple -np 'setuid script' /usr/man/catl/perl.l

        greple -pe '^struct sockaddr' /usr/include/sys/socket.h

    It changes the unit of context specified by __-A__, __-B__, __-C__
    options.

- __--all__

    Treat entire file contents as a single block.  This is almost
    identical to following command.

        greple --block='(?s).*'

- __--block__=_pattern_, __--block__=_&sub_

    Specify the record block to display.  Default block is a single line.

    Next example behave almost same as __--paragraph__ option, but is less
    efficient.

        greple --block='(.+\n)+'

    Next command treat the data as a series of 10-line blocks.

        greple -n --block='(.*\n){1,10}'

    When blocks are not continuous and there are gaps between them, the
    match occurred outside blocks are ignored.

    If multiple block options are supplied, overlapping blocks are merged
    into single block.

    Please be aware that this option is sometimes quite time consuming,
    because it finds all blocks before processing.

- __--blockend__=_string_

    Change the end mark displayed after __-pABC__ or __--block__ options.
    Default value is "--\\n".

## REGIONS

- __--inside__=_pattern_
- __--outside__=_pattern_

    Option __--inside__ and __--outside__ limit the text area to be matched.
    For simple example, if you want to find string \`and' not in the word
    \`command', it can be done like this.

        greple --outside=command and

    The block can be larger and expand to multiple lines.  Next command
    searches from C source, excluding comment part.

        greple --outside '(?s)/\*.*?\*/'

    Next command searches only from POD part of the perl script.

        greple --inside='(?s)^=.*?(^=cut|\Z)'

    When multiple __inside__ and __outside__ regions are specified, those
    regions are mixed up in union way.

    In multiple color environment, and if single keyword is specified,
    matches in each __--inside__/__outside__ regions are printed in
    different colors.  Forcing this operation with multiple keywords, use
    __--regioncolor__ option.

- __--inside__=_&function_
- __--outside__=_&function_

    If the pattern name begins by ampersand (&) character, it is treated
    as a name of subroutine which returns a list of blocks.  Using this
    option, user can use arbitrary function to determine from what part of
    the text they want to search.  User defined function can be defined in
    `.greplerc` file or by module option.

- __--include__=_pattern_
- __--exclude__=_pattern_
- __--include__=_&function_
- __--exclude__=_&function_

    __--include__/__exclude__ option behave exactly same as
    __--inside__/__outside__ when used alone.

    When used in combination, __--include__/__exclude__ are mixed in AND
    manner, while __--inside__/__outside__ are in OR.

    Thus, in the next example, first line prints all matches, and second
    does none.

        greple --inside PATTERN --outside PATTERN

        greple --include PATTERN --exclude PATTERN

    You can make up desired matches using __--inside__/__outside__ option,
    then remove unnecessary part by __--include__/__exclude__

- __--strict__

    Limit the match area strictly.

    By default, __--block__, __--inside__/__outside__,
    __--include__/__exclude__ option allows partial match within the
    specified area.  For instance,

        greple --inside and command

    matches pattern `command` because the part of matched string is
    included in specified inside-area.  Partial match failes when option
    __--strict__ provided, and longer string never matches within shorter
    area.

## CHARACTER CODE

- __--icode__=_code_

    Target file is assumed to be encoded in utf8 by default.  Use this
    option to set specific encoding.  When handling Japanese text, you may
    choose from 7bit-jis (jis), euc-jp or shiftjis (sjis).  Multiple code
    can be supplied using multiple option or combined code names with
    space or comma, then file encoding is guessed from those code sets.
    Use encoding name \`guess' for automatic recognition from default code
    list which is euc-jp and 7bit-jis.  Following commands are all
    equivalent.

        greple --icode=guess ...
        greple --icode=euc-jp,7bit-jis ...
        greple --icode=euc-jp --icode=7bit-jis ...

    Default code set are always included suspect code list.  If you have
    just one code adding to suspect list, put + mark before the code name.
    Next example does automatic code detection from euc-kr, ascii, utf8
    and UTF-16/32.

        greple --icode=+euc-kr ...

- __--ocode__=_code_

    Specify output code.  Default is utf8.

## FILTER

- __--if__=_filter_, __--if__=_EXP_:_filter_:_EXP_:_filter_:...

    You can specify filter command which is applied to each files before
    search.  If only one filter command is specified, it is applied to all
    files.  If filter information include multiple fields separated by
    colons, first field will be perl expression to check the filename
    saved in variable $\_.  If it successes, next filter command is pushed.
    These expression and command list can be repeated.

        greple --if=rev perg
        greple --if='/\.tar$/:tar tvf -'

    If the command doesn't accept standard input as processing data, you
    may be able to use special device:

        greple --if='nm /dev/stdin' crypt /usr/lib/lib*

    Filters for compressed and gzipped file is set by default unless
    __--noif__ option is given.  Default action is like this:

        greple --if='s/\.Z$//:zcat:s/\.g?z$//:gunzip -c'

- __--noif__

    Disable default input filter.  Which means compressed files will not
    be decompressed automatically.

- __--of__=_filter_

    Specify output filter commands.

## RUNTIME FUNCTIONS

- __--print__=_function_, __--print__=_sub{...}_

    Specify user defined function executed before data print.  Text to be
    printed is replaced by the result of the funcion.  Arbitrary function
    can be defined in `.greplerc` file.  Matched data is placed in
    variable `$_`.  Other information is passed by key-value pair in the
    arguments.  Filename is passed by `file` key.  Matched informaiton is
    passed by `matched` key, in the form of perl array reference:
    `[[start,end],[start,end]...]`.

    Simplest function is __--print__='_sub{$\_}_'.  Coloring capability can
    be used like this:

        # ~/.greplerc
        __CODE__
        sub print_simple {
            my %attr = @_;
            for my $r (reverse @{$attr{matched}}) {
                my($s, $e) = @$r;
                substr($_, $s, $e - $s, color('B', substr($_, $s, $e - $s)));
            }
            $_;
        }

    Then, you can use this function in the command line.

        greple --print=print_simple ...

    It is possible to use multiple __--print__ options.  In that case,
    second function will get the result of the first function.  The
    command will print the final result of the last funciton.

- __--continue__

    When __--print__ option is given, __greple__ will immediately print the
    result returned from print function and finish the cycle.  Option
    __--continue__ forces to continue normal printing process after print
    function called.  So please be sure that all data being consistent.

- __--call__=_function_(_..._), __--call__=_function_=_..._
- __-M___module_::_function(...)_, __-M___module_::_function=..._

    Option __--call__ specify the function executed at the beginning of
    each file processing.  This _function_ have to be called from __main__
    package.  So if you define the function in the module package, use the
    full package name or export properly.

    It can be set with module option, following module name.  In this
    form, the function will be called with module package name.  So you
    don't have to export it.

For these run-time functions, optional argument list can be set in the
form of `key` or `key=value`, connected by comma.  These arguments
will be passed to the funciton in key => value list.  Sole key will
have the value one.  Also processing file name is passed with "file"
key.  As a result, the option in the next form:

    --print|call function(key1,key=val2)
    --print|call function=key1,key=val2

    -Mmodule::function(key1,key=val2)
    -Mmodule::function=key1,key=val2

will be transformed into following function call:

    function(file => "filename", key1 => 1, key2 => "val2")

The function can be defined in `.greplerc` or modules.  Assign the
arguments into hash, then you can access argument list as member of
the hash.  Content of the target file can be accessed by `$_`.

    sub function {
        my %arg = @_;
        $arg{file};    # "filename"
        $arg{key1};    # 1
        $arg{key2};    # "val2"
        $_;            # contents
    }

## PGP

- __--pgp__

    Invoke PGP decrypt command for files end with _.pgp_, _.gpg_ or
    _.asc_.  PGP passphrase is asked only once at the beginning of
    command execution.

- __--pgppass__=_phrase_

    You can specify PGP passphrase by this option.  Generally, it is not
    recommended to use.

## OTHERS

- __--norc__

    Do not read startup file: `~/.greplerc`.

- __--man__

    Show manual page.
    Display module's manual page when used with __-M__ option.

- __--show__

    Show module file contents.  Use with __-M__ option.

- __--require__=_filename_

    Include arbitrary perl program.

# ENVIRONMENT and STARTUP FILE

Environment variable GREPLEOPTS is used as a default options.  They
are inserted before command line options.

Before starting execution, _greple_ reads the file named `.greplerc`
on user's home directory.  Following directives can be used.

- __option__ _name_ string

    Argument _name_ of \`option' directive is user defined option name.
    The rest are processed by _shellwords_ routine defined in
    Text::ParseWords module.  Be sure that this module sometimes requires
    escape backslashes.

    Any kind of string can be used for option name but it is not combined
    with other options.

        option --fromcode --outside='(?s)\/\*.*?\*\/'
        option --fromcomment --inside='(?s)\/\*.*?\*\/'

    If the option named __default__ is defined, it will be used as a
    default option.

    For the purpose to include following arguments within replaced
    strings, two special notations can be used in option definition.
    String `$<_n_>` is replaced by the _n_th argument after the
    substituted option, where _n_ is number start from one.  String
    `$<shift>` is replaced by following command line argument and
    the argument is removed from option list.

    For example, when

        option --line --le &line=$<shift>

    is defined, command

        greple --line 10,20-30,40

    will be evaluated as this:

        greple --le &line=10,20-30,40

- __help__ _name_

    If \`help' directive is used for same option name, it will be printed
    in usage message.  If the help message is \`ignore', corresponding line
    won't show up in the usage.

- __define__ _name_ string

    Directive \`define' is almost same as \`option', but argument is not
    processed by _shellwords_ and treated just a simple text.
    Metacharacters can be included without escaping.  Defined string
    replacement is done only in definition in option argument.  If you
    want to use the word in command line, use option directive instead.

        define :kana: \p{InKatakana}
        option --kanalist --nocolor -o --join --re ':kana:+(\n:kana:+)*'
        help   --kanalist List up Katakana string

Environment variable substitution is done for string specified by
\`option' and \`define' directivies.  Use Perl syntax __$ENV{NAME}__ for
this purpose.  You can use this to make a portable module.

When _greple_ found `__CODE__` line in `.greplerc` file, the rest
of the file is evaluated as a Perl program.  You can define your own
subroutines which can be used by __--inside__/__outside__,
__--include__/__exclude__, __--block__ options.

For those subroutines, file content will be provided by global
variable `$_`.  Expected response from the subroutine is the list of
array references, which is made up by start and end offset pairs.

For example, suppose that the following function is defined in your
`.greplerc` file.  Start and end offset for each pattern match can be
taken as array element `$-[0]` and `$+[0]`.

    __CODE__
    sub odd_line {
        my @list;
        my $i;
        while (/.*\n/g) {
            push(@list, [ $-[0], $+[0] ]) if ++$i % 2;
        }
        @list;
    }

You can use next command to search pattern included in odd number
lines.

    % greple --inside '&odd_line' patten files...

# MODULE

Modules can be specified only at the beginning of command line by
__-M___module_ option.  Name _module_ is prepended by __App::Greple__,
so place the module file in `App/Greple/` directory in Perl library.

If the package name is declared properly, `__DATA__` section in the
module file will be interpreted same as `.greplerc` file content.

Using __-M__ without module argument will print available module list.
Option __--man__ will display module document when used with __-M__
option.  Use __--show__ option to see the module itself.

See this sample module code.  This sample define options to search
from pod, comment and other segment in Perl script.  Those capability
can be implemented both in function and macro.

    package App::Greple::perl;
    
    BEGIN {
        use Exporter   ();
        our ($VERSION, @ISA, @EXPORT, @EXPORT_OK, %EXPORT_TAGS);
    
        $VERSION = sprintf "%d.%03d", q$Revision: 6.22 $ =~ /(\d+)/g;
    
        @ISA         = qw(Exporter);
        @EXPORT      = qw(&pod &comment &podcomment);
        %EXPORT_TAGS = ( );
        @EXPORT_OK   = qw();
    }
    our @EXPORT_OK;
    
    END { }
    
    my $pod_re = qr{^=\w+(?s:.*?)(?:\Z|^=cut\s*\n)}m;
    my $comment_re = qr{^(?:[ \t]*#.*\n)+}m;
    
    sub pod {
        my @list;
        while (/$pod_re/g) {
            push(@list, [ $-[0], $+[0] ] );
        }
        @list;
    }
    sub comment {
        my @list;
        while (/$comment_re/g) {
            push(@list, [ $-[0], $+[0] ] );
        }
        @list;
    }
    sub podcomment {
        my @list;
        while (/$pod_re|$comment_re/g) {
            push(@list, [ $-[0], $+[0] ] );
        }
        @list;
    }
    
    1;
    
    __DATA__
    
    define :comment: ^(\s*#.*\n)+
    define :pod: ^=(?s:.*?)(?:\Z|^=cut\s*\n)
    
    #option --pod --inside :pod:
    #option --comment --inside :comment:
    #option --code --outside :pod:|:comment:
    
    option --pod --inside '&pod'
    option --comment --inside '&comment'
    option --code --outside '&podcomment'

You can use the module like this:

    greple -Mperl --pod default greple

    greple -Mperl --colorful --code --comment --pod default greple

# HISTORY

Most capability of __greple__ is derived from __mg__ command, which has
been developing from early 1990's by the same author.  Because modern
standard __grep__ family command becomes to have similar capabilities,
it is a time to clean up entire functionarities, totally remodel the
option interfaces, and change the command name. (2013.11)

# AUTHOR

Kazumasa Utashiro

# SEE ALSO

[grep(1)](http://man.he.net/man1/grep), [perl(1)](http://man.he.net/man1/perl)

[github](http://kaz-utashiro.github.io/greple/)

# LICENSE

Copyright (c) 1991-2014 Kazumasa Utashiro

Use and redistribution for ANY PURPOSE are granted as long as all
copyright notices are retained.  Redistribution with modification is
allowed provided that you make your modified version obviously
distinguishable from the original one.  THIS SOFTWARE IS PROVIDED BY
THE AUTHOR \`\`AS IS'' AND ANY EXPRESS OR IMPLIED WARRANTIES ARE
DISCLAIMED.
