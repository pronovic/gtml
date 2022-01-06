# GTML Reference

This is the reference page for the HTML pre-processor GTML. It describes all the GTML features and commands in detail.

GTML features fall into four main areas, and we'll look at them in this order:

- GTML commands embedded among HTML text.
- The optional GTML project file, which allows you to manage a set of web pages together.
- Special _named constants_ generated or used by GTML.
- Command-line options

Before we dive in, there's also a quick summary to get you started.

## Quick Summary

GTML source files end in `.gtm` (or `.gtml`), not `.html`. If you're using GTML on existing HTML files, simply rename them with the appropriate ending.

GTML is run from the command line, like this:

```
$ gtml fred.gtm harry.gtm bill.gtm
```

The output of this command will be in `fred.html`, `harry.html` and `bill.html`.

If you have a GTML project file, you include this on the command line.  In this case, it's not necessary to list any of the files in the project as well.

You can use `-D` on the command line to create named constants. You can have as many `-D` options as you like. Make sure they appear before the file names to which they apply. For example, if you say:

```
$ gtml -DNAME=Fred fred.gtm harry.gtm -DTYPE=car bill.gtm
```

then `NAME` is defined for all three files and `TYPE` is defined for `bill.gtm` only.

By default, GTML will try to process some project file. It will look at these configuration files in this order:

- `$HOME/.gtmlrc`
- `$HOME/gtml.conf`
- `.gtmlrc`
- `gtml.conf`

Those files, if they exist, are parsed before command line is processed.

## Commands

The original syntax of GTML was stolen shamelessly from the C language pre-processor, and has been adapted to suit web site management. It supports the following commands:

- `include` - Include the contents of another file.
- `define` and `newdefine` - Create _named constants_ which can be used as shorthand definitions of frequently-used HTML segments.
- `undef` - Remove a named constant.
- `timestamp` and `mtimestamp` - Insert a formatted date/time stamp.
- `literal` - Turn off GTML processing.
- `definechar` - Define characters (or strings) translations.
- `entities` - Convert special characters to HTML entities.
- `if`, `ifdef`, `ifndef`, `elsif`, `elsifdef`, `elsifndef`, `else`, `endif` - Select sections of the source based on certain conditions.
- `compress` - Turn on file compression, eliminating HTML useless segments.
- `sitemap` and `toc` - Insert a map of all your site.

### Command syntax

There are two ways of writing commands:

You can insert the commands directly, in which case they are preceded by `#`, and must appear at the start of a separate line (no preceeding spaces). Here's an example:  

```
#include "header.txt"
```

The other method is to embed the command inside an HTML comment. In this case, the line must begin with `<!-- ###` followed by the command name, followed by the close-comment tag `-->`. Here's the same example in this format:  

```
<!-- ###include "header.txt" -->
```

The first method is simpler, and is the recommended way of writing GTML , if possible. However, if you're using an HTML authoring tool which complains about GTML commands, you can use the second method to hide the GTML commands from the tool.

### `#include`

If you have some common text which you want inserted in more than one web page, put it in a separate file - say, `common.txt`. Then, at the place where you want the text in each file, put the following line:

```
#include "common.txt"
```

GTML replaces this line with the entire contents of the file `common.txt`.

The name of the file can be defined in a named constant, as described next.

The directories in which the file may be looked for can be defined in a special named constant.

### `#includeliteral`

Works the same as `#include` but also turns on `#literal` so GTML does not interpret a given file, just includes it.  This is equivalent to putting `#literal ON` as the first line of the included file and then `#literal OFF` as the last line of the included file.

### `#define`

This is a simple way to create shorthand definitions - what we call _named constants_, or _macros_ - for your frequently-used text. For some text which you use often - say, an e-mail address - include a line like this:

```
#define MY_EMAIL gihan@pobox.com
```

This defines a _named constant_ `MY_EMAIL` and its _value_ `gihan@pobox.com`. The value can be any text, including spaces and HTML markup (leading spaces and tabs are ignored), as long as **it's all on one line** or **it's all on multiple consecutives lines with a trailing \ at the end of each lines, except the last**.

To use this named constant, whenever you want your e-mail address, specify it like this:

```
<<MY_EMAIL>>
```

The double angle-brackets tell GTML to substitute this with its definition (You can specify your own choice of delimiters instead of the double-angle brackets.)

There are a few other things you should know about `#define`:

#### **Named constants** 

Named constants can be set from the command line when you run GTML.

#### **Nested definition values**  

The value of a named constant can itself contain other named constants, like this:

```
#define MAILTO <a href="mailto:<<MY_EMAIL>>">
```

GTML will happily do both substitutions.  

_Note_: Definitions are evaluated at time of use, not time of definition. This allows you to change the nested value to get a different result. To get definitions body evaluated at time of definition, you must use the `#define!` command.<a name="define!"></a>

#### **Nested definition names**  

If you want the _name_ of a named constant to contain other named constants, use `#define!` instead of `#define`. For example, this sets the named constant A to fred:

```
#define BLAH  A
#define! <<BLAH>> fred
```

(This doesn't achieve much here, but typically the second line is in a separate file, which is included by the file with the first line).<a name="define+"></a>

#### **Added values**  

If you want to add something to the definition of a named constant, use `#define+`. For example, this sets the named constant A to foobar:

```
#define A foo
#define+ A bar
```

#### **Undefined values**  

If a named constant is not defined before it's used, its value is a blank string.

#### **Re-defining values**  

It's perfectly OK to have more than one `#define` for the same named constant. You can use this to change the value of a named constant part-way through the processing.

#### **Pre-defined values**  

GTML creates some named constants for you automatically, such as the file name and links to the previous and next files.

#### **Variables values**  

Sometimes the frequently-used text you want to shorthand may embed some variable pieces. GTML allows you to specify what the pieces are which may vary, the same way the C language preprocessor does for macros with arguments. Parameters are enclosed in parenthesis and seperated by a comma. For instance, this creates a shortand to make HTML links:

``
#define url(x,y) <a href="mailto:x">y</a>
``
    
Now the use of `<<url(<<MY_EMAIL>>,Gihan)>>` will give:

``
<a href="mailto:gigan@pobox.com">Gihan</a>
``

#### **Added variables values**  

If you add a variable valued definition with `#define+`, then you add the parameter to the initial constant. For instance, this adds a third parameter to the `url` macro:

```
#define+ url(x) <a href="http://x">Perera</a>
```
    
Now guess what the following line will give:

``
<<url(<<MY_EMAIL>>,Gihan,www.pobox.com/~gihan)>>
``

Yeah, you find:

``
<a href="mailto:gigan@pobox.com">Gihan</a><a href="http://www.pobox.com/~gihan">Perera</a>
``

If you want to use commas in your arguments you just have to quote the complete argument with single or double quotes.

### `#newdefine`

This is identical to `#define`, except that the definition applies only if the named constant has not been defined already. If it has been defined, the old definition is unchanged. `#newdefine!` is identical to `#define!` with the same exception.

### `#undef`

Use this to remove the definition of a named constant:

If the named constant didn't exist previously, this does nothing.

### `#if`, `#ifdef`, `#ifndef`, `#elsif`, `#elsifdef`, `#elsifndef`, `#else`, `#endif`

These commands are used together for conditional output:

#### `#if <<FRED>> == foo`  

Process the following lines only if after substitution, the left and right values are equal. Here if value of named constant FRED is equal to foo.

#### `#if foo != <<FRED>>`  

Process the following lines only if after substitution, the left and right values are different. Here if value of named constant FRED is different from foo.

#### `#ifdef FRED`  

Process the following lines only if the named constant FRED is defined.

#### `#ifndef FRED`  

Process the following lines only if the named constant FRED is _not_ defined.

#### `#elsif <<FRED>> == foo`  

If previous lines were not processed, then process the following lines only if after substitution, the left and right values are equal.

#### `#elsif <<FRED>> != foo`  

If previous lines were not processed, then process the following lines only if after substitution, the left and right values are different.

#### `#elsifdef FRED`  

If previous lines were not processed, then process the following lines only if the named constant FRED is defined.

#### `#elsifndef FRED`  

If previous lines were not processed, then process the following lines only if the named constant FRED is not defined.

#### `#else`  

The opposite effect of the most recent matching `#if`, `#ifdef`, `#ifndef`, `#elsif`, `#elsifdef` or `#elsifndef`.

#### `#endif`  

End a block of conditional output. Every `#if`, `#ifdef` or `#ifndef` must have a matching `#endif`

Conditional blocks can be nested.

### `#timestamp`

The special named constant `TIMESTAMP` evaluates to the current date and time. But to use it, you must tell GTML what format to use to print the date/time.

The format string is specified in the `#timestamp` statement like this:

```
#timestamp   $dd/$MM/$yy at $hh:$mm:$ss
```

The value of `<<TIMESTAMP>>` will then be: `8/06/96 at 11:45:03`.

As you can see, certain strings (like `$dd`) are replaced with values. The full set of substitution strings is as follows (everything else is left unchanged in the format string):

<table cellpadding="5" border="1">
<tbody>
    <tr>
        <td>$hh</td>
        <td>Hour (00 to 23)</td>
    </tr>
    <tr>
        <td>$mm</td>
        <td>Minute (00 to 59)</td>
    </tr>
    <tr>
        <td>$ss</td>
        <td>Seconds (00 to 59)</td>
    </tr>
    <tr>
        <td>$Day</td>
        <td>Full weekday name (Sunday to Saturday)</td>
    </tr>
    <tr>
        <td>$Ddd</td>
        <td>Short weekday name (Sun to Sat)</td>
    </tr>
    <tr>
        <td>$dd</td>
        <td>Day of the month (1 to 31)</td>
    </tr>
    <tr>
        <td>$ddth</td>
        <td>Day of the month with ordinal extension (1st to 31th)</td>
    </tr>
    <tr>
        <td>$MM</td>
        <td>Month number (1 to 12)</td>
    </tr>
    <tr>
        <td>$Month</td>
        <td>Full month name (January to December)</td>
    </tr>
    <tr>
        <td>$Mmm</td>
        <td>Short month name (Jan to Dec)</td>
    </tr>
    <tr>
        <td>$yyyy</td>
        <td>Year (e.g. 1996)</td>
    </tr>
    <tr>
        <td>$yy</td>
        <td>Short year (e.g. 96)</td>
    </tr>
</tbody>
</table>

Monthnames are output in English by default; but they can be output in other languages, according to the `LANGUAGE` special definition.

### `#mtimestamp`

This is identical to `timestamp`, except that the time used is not the current one, but the time the file was last modified.

### `#literal`

The command `#literal ON` tells GTML to stop interpreting lines beginning with `#` as GTML commands, until the next `#literal OFF` line. Defined constants are still substituted, and entities are translated if desired.

For example, this is useful for bracketing blocks of C code, which might have lines beginning with `#`.

### `#entities`

The command `#entities ON` tells GTML to convert the characters `<`, `>` and `&` into HTML entities so they aren't treated as part of HTML commands. Use `#entities OFF` to turn off this conversion.

This is useful for bracketing blocks of program code, which often contain these characters.

### `#definechar`

Basic HTML authorized characters may only be ASCII characters. Accentuated characters are coded in HTML in a certain way. For instance the `é` character is coded `&eacute;`. You may want to input `é` in your source file, and have the right code used in your HTML file. This character's translation may be defined with the `#definechar` command, like in this example:

```
#definechar é &eacute;
```

Sometimes you might want to input special characters that are not available on your keyboard, but do not want to input its HTML code (Think of the German `ß` on an English keyboard). For instance if you want all occurrances of `ss` in your source file to be translated into `ß`, you can use the `#definechar` command:

```
#definechar ss &szlig;
```

### `#compress`

The command `#compress ON` tells GTML to begin producing compressed HTML code, i.e. stripping multiple spaces, removing newlines, HTML comments, and all other stuff useless for an HTML browser to render a page. This compression is done on the output file until the next `#compress OFF`.

### `#sitemap`, `#toc`

When you specify a hierarchy of pages in your project file you may insert a table of contents, or site map, of those pages, using the command `#sitemap`, or `#toc`.

See **Document hierarchy** specifications for more information on how the list is created.

## GTML project files

Because GTML is most useful for managing multiple files, it's quite common to change something in a `#include`'d file, and then run GTML on all the `.gtm` files which use it.

To make this procedure easier, GTML supports a concept of a _project file_. This is a simple text file with the extension `.gtp`. It can contain:

#### `define ...`  

As for `#define` in a source file.

#### `define! ...`  

As for `#define!` in a source file.

#### `define+ ...`  

As for `#define+` in a source file.

#### `newdefine ...`  

As for `#newdefine` in a source file.

#### `newdefine! ...`  

As for `#newdefine!` in a source file.

#### `undef ...`  

As for `#undef` in a source file.

#### `timestamp ...`  

As for `#timestamp` in a source file.

#### `mtimestamp ...`  

As for `#mtimestamp` in a source file.

#### `if ...`, etc   

As for `#if`, `#ifdef`, `#ifndef`, `#elsif`, `#elsifdef`, `#elsifndef`, `#else`, `#endif` in a source file.

#### `definechar ...`  

As for `#definechar in a source file.

#### `compress ...`  

As for `#compress` in a source file.

#### `include ...`  

As for `#include` in a source file, but only on project files.

#### `// ...`  

Comment lines (the `//` must be the first characters on the line), which are ignored. Blank lines are also ignored.

#### `allsource`  

This is a shorthand way of specifying all source files from the current directory and sub-directories (recursively).

#### `filename alias file`  

This is used to make an alias to a file, so that you may use the alias, as a defined constant, instead of the filename itself in source files. GTML will compute the right relative path to the file in each source file. This way you may move your files wherever you want, GTML will be able to recreate your pages without having to modify the source files.

#### `filename.gtm`  

All other lines are names of source files, which are processed by GTML in turn.

These files can be in other directories below the current directory. Simply specify the file name relative to the current directory (e.g. `sub/fred/index.gtm`).

The file can be a defined alias filename. In this case use it as a filenamed constant, e.g.

```
filename FOO bar/foo
<<FOO>>
```

The file name can be followed by a level number and a title, to be used in [creating a hierarchy of documents](#hierarchy). In this case the file is processed after the project file has been completely read (in order to have the complete document hirearchy).

#### `hierarchy`  
    
This is used to process all files of the declared hierarchy so far, so that you may process those files more than once, if you want to change some named constant values between each process. If not used files of the hierarchy are processed after the project file has been entirely read.

When you run GTML with the project file as a command-line argument, it will process all the source files in the project file. They will all inherit the `define`, `definechar` and `timestamp` values, if any, in the project file. The `mtimestamp` value will change according to the modification date of the appropriate source file.

You may use a project file, but process only selected source files (declared in the project) with the `-F` command line argument.

Note that `#define`, `#newdefine` and `#undef` commands inside a file are local to that file - they don't carry over to the next file in the project. However, named constants defined in the project file are inherited by all files in the project.

### Document hierarchy

GTML allows you to create a hierarchy of web pages, with links between them. Each web page can have a link to the previous page, the next page, or one level up in the hierarchy. Obviously, some of these links don't apply to some pages - GTML generates only those that apply to each page.

#### Describing the hierarchy

You describe the document hierarchy to GTML by listing the file names in the project file in a certain order, with a document level and title for each. Level 1 is for top-level documents, and 2, 3, 4, and so on are lower levels. File names without a level have no hierarchical information attached to them.

When GTML processes a file, it defines special named constants which can be used in exactly the same way as other named constants.

For each file, GTML generates the named constants, `LINK_PREV`, `LINK_NEXT` and `LINK_UP`. These correspond to the file names of the previous file, next file and one level up in the hierarchy. In addition, it also generates the corresponding named constants `TITLE_PREV`, `TITLE_NEXT`, `TITLE_UP` and `TITLE_CURRENT` to be the titles of these files (As stated above, the title follows the level number in the project file).

Some of these named constants may not be applicable to some files, in which case the named constant is not defined for that file.

#### Example

Here's an extract from a hypothetical GTML project file:

```
contents.gtm 1 Table of Contents
chapter1.gtm 2 Introduction
sec11.gtm    3 What's the Problem
sec12.gtm    3 Old Approaches
sec13.gtm    3 New Idea
chapter2.gtm 2 Historical Evidence
sec21.gtm    3 Early 1900's
sec22.gtm    3 Recent Findings
chapter3.gtm 2 Our Bright Idea
```

To take a simple case, the file `sec21.gtm` will have `LINK_NEXT` set to `sec22.html` (and `TITLE_NEXT` set to _Recent Findings_) and `LINK_UP` set to `chapter2.html` (and `TITLE_UP` set to _Historical Evidence_). `LINK_PREV` and `TITLE_PREV` will be undefined.

#### Using the links

The links can be used to create _navigation links_ between the documents. In other words, each document can have links up the hierarchy and to the next and previous documents.

Typically, you would place the navigation information in a common file and `#include` it into each GTML source file. The GTML `#ifdef` command can be used to exclude links which don't apply to a particular file.

Here's a simple example:

```
#ifdef LINK_NEXT
   <p>Next document: <a href="<<LINK_NEXT>>"><<TITLE_NEXT>></a>
#endif
#ifdef LINK_PREV
   <p>Previous document: <a href="<<LINK_PREV>>"><<TITLE_PREV>></a>;
#endif
#ifdef LINK_UP
   <p>Up one level: <a href="<<LINK_UP>>"><<TITLE_UP>></a>;
#endif
```

#### Creation of table of contents

When you have described a document hierarchy, and you use a `#toc` or `#sitemap` command into your source file, GTML includes a sitemap generated as a list with the help of the `__TOC_#__(x)`, and `__TOC_#_ITEM__(x,y)` special named constants (`#` being a file level).

With the previous example it gives:

```
<<__TOC_1__('
  <<__TOC_1_ITEM__('contents.html','Table of Contents')>>
  <<__TOC_2__('
    <<__TOC_2_ITEM__('chapter1.html','Introduction')>>
    <<__TOC_3__('
      <<__TOC_3_ITEM__('sec11.html','What's the Problem')>>
      <<__TOC_3_ITEM__('sec12.html','Old Approaches')>>
      <<__TOC_3_ITEM__('sec13.html','New Idea')>>
    ')>>
    <<__TOC_2_ITEM__('chapter2.html','Historical Evidence')>>
    <<__TOC_3__('
      <<__TOC_3_ITEM__('sec21.html','Early 1900's')>>
      <<__TOC_3_ITEM__('sec22.html','Recent Findings')>>
    ')>>
  ')>>
')>>
```

`__TOC_#__(x)`, and `__TOC_#_ITEM__(x,y)` have the following default values:

```
#define __TOC_#__(x)        <ul>x</ul>
#define __TOC_#_ITEM__(x,y) <li><a href="x">y</a>
```

You may redefine this constant to whatever you want as long as you respect the number of variables.

## Special definitions

#### Environment

All environment variables are defined as named constants by GTML.

#### Current file names

The special named constants `ROOT_PATH`, `BASENAME`, `FILENAME` and `PATHNAME` are set to the current path to root of the project (where the project file resides), output file name without any extension and excluding any directory path information, output file name excluding any directory path information, and directory path information relative to the path to the root of the project.

#### Search path for include files

GTML always searches for include files in the directory of the processed source file first, then the current directory (where the GTML command is executed).

In addition, if you define the named constant `INCLUDE_PATH`, GTML will interpret it as a list of directories (separated by colons), to search for include files. Those directories may be absolute, or relative to the root path of the project.

#### Output directory

By default, GTML writes its output files to the same directory as the corresponding source file. You can override this by defining the named constant `OUTPUT_DIR` as the name of the output directory.

If you are doing this with a project file, specify `OUTPUT_DIR` as the top-level output directory. GTML will create the same directory hierarchy for the output files as for the input files (It creates sub-directories as required).

#### Output suffix

By default all created files are created by changing the `.gtm`, or `.gtml` extension of the source file to the `.html` one.

You may change this behavior by defining the special constant `EXTENSION` to whatever extension you want.

The definition of this constant does not have sense in a source file, since the output file is already created the moment GTML starts to parse the source file. It makes sense however to define it in the project file or on the command line.

If the suffix is preceded by two dots as in `gtml.css..gtm`, then the source suffix is not replaced, but just removed, as in `gtml.css`.

#### Fast generation

GTML only processes files which are younger than the output files which they might produce. This is very useful, as it increases the generation speed of web sites with a big number of pages, when only one of them has been changed. Same kind of the way `make` works.

To enable this feature you just have to define the special constant `FAST_GENERATION`. The use of this constant will work only in GTML project files or on the command line.

This feature does not take into account included files, but only the main gtml source, and the wanted output. To deal with more complex file dependencies you may use the `make` tool in conjunction with the `-F` and `-M` command line arguments.

#### Language localisation

By default all timestamps produce monthnames and daynames which are output in English. You can output them in other languages by defining the constant `LANGUAGE` to a value corresponding to an available language.

As of today seven languages are available. The default one is English. Following is the list of those language with the corresponding value for `LANGUAGE` constant:

- `fr` for French
- `de` for German
- `ie` for Irish
- `it` for Italian
- `nl` for Dutch
- `no` for Norvegian
- `se` for Swedish

If you can send me month and day names in other languages, just do it, I will integrate it in following versions of GTML.

#### Substitution delimiters

The default delimiters for substituting named constants are `<<` and `>>`. You can change these to any other strings by defining the named constants `OPEN_DELIMITER` and `CLOSE_DELIMITER` respectively.

For example, if you had the following lines:

```
#define OPEN_DELIMITER {{
#define CLOSE_DELIMTER }}

#define MY_EMAIL  gihan@pobox.com
```

then GTML substitutes `MY_EMAIL` when it finds the text `{{MY_EMAIL}}` instead of the default `<<MY_EMAIL>>`.

#### Document hierarchy links

As described above, GTML defines several named constants for links between documents.

#### Embedded Perl code

You may embed Perl code into your GTML source file, and have the result of the last evaluated expression inserted in your output file, with the help of the one argument macro __PERL__.

Here is an example inserting the size of the source file:

``
<<__PERL__(return (stat(<<ROOT_PATH>><<BASENAME>>.".gtm"))[7];)>>
``

#### Embedded system command code

You may embed system command output into your GTML source file, with the help of the one argument macro `__SYSTEM__`.

Here is an example inserting the list of files in the current directory:

``
<<__SYSTEM__(dir)>>
``

## Command-line arguments

### `-D`

When you run GTML from the command line, you can define named constants like this:

```
-DMY_EMAIL=fred
```

This is the same as `#define MY_EMAIL fred` in the file.

These definitions can occur anywhere within the command-line options, but only affect the files _after_ them. For example, if your command line is:

``
perl gtml.pl fred.gtm -DMY_EMAIL=fred bill.gtm harry.gtm
``

then the MY_EMAIL definition doesn't apply to `fred.gtm`.

### `-F`

When you run GTML from the command line on a project file you may want to process only some of the files used in it. This may be very useful in conjunction with `make` for page regeneration based on complex files dependencies (induced by `#include` commands).

To process only one file of a project you can use the `-F` argument followed by the name of the file to process. File to be processed must appear, on the command line, before the project file it appears in.

Let us suppose we have this project file, called `foo.gtp`:

```
// Beginning of foo.gtp
define MAIL beaufils@lifl.fr

foo.gtm
bar.gtm
// End of foo.gtp
```

If you just want to regenerate the `bar.html` file your command line will be:

```
perl gtml -Fbar.gtm foo.gtp
```

List of files to processed is cleared after each project file treatment.

### `-M`

When you run GTML with the `-M` command line argument, GTML process project and source files, but do not produce ouput files. GTML generates a makefile ready to create output files.

If you do not specify a filename, the generated makefile is created under the `GNUmakefile` name.

To specify a filename, just add it after the `-M` argument, with a colon between the option and the file name.

### `-h`, `--help`

To get a small command line usage description you can use the `-h`, or `--help` command line argument.

### `--silent`

If you specify the `--silent` command line argument, GTML will produce no processing information during its work.

### `--version`

To get the version number of the GTML you are currently using you can use the `--version` command line argument.
