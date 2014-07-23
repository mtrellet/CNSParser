CNSParser
=========

NAME
----

cnsparser - Translate CNS files into Python datastructures

cnstojson - Generate a JSON model description based on parsed CNS data

jsontocns - Rewrite a CNS file with modified sections and parameter
values

SYNOPSIS
--------

    cnstojson.py -o model.json run.cns
    jsontocns.py -o run.cns template.cns model.json

DESCRIPTION
-----------

The scripts in this repository form a layer between
[HADDOCK](http://www.nmr.chem.uu.nl/haddock/) and a user interface for
submitting runs to HADDOCK.

CNSParser reads a CNS file, as used by HADDOCK and subsequently CNS,
and generates generalized model descriptions.

These model descriptions can be parsed by a program that provides an
interface for filling in parameters. This allows for flexibility in
implementing the user interface to HADDOCK and greatly simplifies the
process of adding and removing parameters and form structure.

This project consists of one Python module and two Python scripts:

### cnsparser

This is the module that contains the CNSParser class. This class is used
both for parsing CNS files, converting them to Python datastructures,
and for writing parameters back to a CNS file based on a template.

The syntax of CNS files is defined in this class as a list of regular
expressions. Adding or replacing syntax can be done by editing this list
of patterns and the pattern handler list that binds the patterns to
callbacks.

### cnstojson

This script uses CNSParser to generate a python datastructure and saves
the result in a JSON model file.

### jsontocns

This script uses CNSParser to loop through a CNS file and fill in
parameter values based on a JSON model file that contains a user's
input.

FEATURES
--------

Besides parsing parameters and documentation paragraphs, CNSParser has
support for (and requires) certain additional information to be
specified in the CNS file.

See the SYNTAX section for details on how to implement these features in
your CNS file.

### Sections

CNSParser allows for parameters to be grouped into nestable sections.

This feature can be used to structure forms, to repeat multiple
parameters (see below), and to support inheritance of certain parameter
attributes.

### Repeating sections and parameters

Sections and parameters can be multiplied. CNSParser understands
attributes that specify a minimum, a maximum and a string that is used
to determine how to name repeated parameters and (sub)sections.

### Access levels

To allow for more than one interface to HADDOCK, CNSParser supports the
concept of access levels.
Access levels are meant to range from easiest to most complex, but can
be filled in in any way you like. They define what attributes are
visible to different groups of users.

### Backwards-compatibility with inp2form and form2inp CGI scripts

All new syntax and keywords are tested for backwards-compatibility with
the perl/CGI scripts as included with CNS.

An attempt was made to avoid adding too much new syntax to the already
complex CNS format. CNSParser tries to be as consistent and predictable
as possible.

SYNTAX
------

All information that is CNSParser-specific is added in what the CNS
software recognizes as line or block comments to avoid parsing errors.

**NOTE**: CNSParser can not parse brace-enclosed blocks that span
multiple lines. These can be easily avoided.

CNSParser expects a CNS file to be structured in the following manner:

1. A list of access levels
2. A list of sections and parameters

### Access levels

All access levels must be specified before the first section or parameter.

    {!accesslevel name "User-friendly label"}

The accesslevel name may only contain alphanumeric characters and
underscores. This string is used as an identifier throughout the file
and may be used as such in interfaces. For example, an HTML frontend
should be able to expect the name to be a valid HTML class attribute.

Accesslevels are saved in the order they are defined.

### Paragraphs

Paragraphs are pieces of documentation that are not bound to a specific
parameter. They are saved as any other component and should be shown in
place by interfaces.

    {* This bit of text might appear somewhere on a form. *}
    
    {* Paragraphs can span multiple lines, but note that *}
    {* separate brace-asterisk blocks must be used. *}
    
    {* A blank line ends a paragraph. *}

When paragraphs are not closed with a blank line as shown above, they
will be used as a label for the next parameter, see below.

### Parameter labels

Parameters can be labeled by adding a paragraph on the line before the
parameter definition.

It is also possible to specify parameter attributes after a label, but
care must be taken not to leave any blank lines between the paragraph
and the parameter, to avoid it being parsed as a separate paragraph.

Technically, label definitions can span multiple lines just like regular
paragraphs. This isn't recommended as long labels may be truncated by
the interface.

### Attributes

Attributes are certain characteristics that can be applied to sections
and parameters. They are applied to the first parameter or section
definition after the line specifying the attribute.

Attributes take the form of a line comment with one ore more key=value
pairs:

    ! #type=integer #level-min=easy

It is possible to specify attributes over multiple lines like so:

    ! #type=integer
    ! #level-min=easy
    ...
    (parameter definition)

### Parameters

Parameters are defined as follows:

    {===>} name="default value";

Any attributes specified before a parameter definition are applied to
the parameter.

### Attributes

CNSParser supports the following parameter attributes:

#### type

Specifies the datatype. Can be one of 'integer', 'float', 'string' and
'file'.

This attribute is optional. When no type is specified, CNSParser tries
to deduce it from the default value.

#### hidden

This valueless attribute can be used to define parameters that should
not be included in an interface.


Additionally, the following attributes can be used on parameters as well
as sections:

#### multi-index

Parameters and sections that have this attribute are marked as
repeatable. The string value for this attribute is used as a placeholder
in parameter names, default parameter values, paragraphs and section
headers, and will be replaced by an index number in the interface.

Parameter children of repeatable sections **must** contain all
placeholders specified by their parents in their name. Likewise, the
names of repeatable parameters must contain their own placeholder.

For example, consider the following hierarchy of sections (refer to the
"Sections" section for an explanation of section syntax):

    {==== Parent block ====}

    ! #multi-index=NN
    {====== Repeatable child block NN ====}

    {===>} normal_parameter_NN = 1;

    ! #multi-index=MM
    {===>} repeatable_parameter_NN_MM = 1;

    ! The following generates a parser error: The parameter name does
    ! not contain the 'NN' placeholder of its parent block

    ! #multi-index=MM
    {===>} repeatable_parameter_MM = 1;

Naturally, the same placeholder string can't be reused within a single
hierarchy line (but it can be used by siblings).

#### multi-min

Specifies the minimum number of repetitions. Interfaces should show this
much parameters / section by default, and allow the user to add more
themselves (if multi-max allows it).

#### multi-max

Specifies the maximum number of repetitions for a parameter or section

#### level-min

Sets the lowest access level at which this section or parameter should
be visible to users.

See also "Notes on access level inheritance".

#### level-max

Sets the highest access level at which this section or parameter should
be visible to users.

See also "Notes on access level inheritance".

#### level-include

Explicitly allow certain access levels after denying them with level-min
or level-max.

See also "Notes on access level inheritance".

#### level-exclude

Explicitly deny certain access levels.

See also "Notes on access level inheritance".

### Sections

Sections are defined as follows:

    {==== Some block ====}

The amount of equals signs before the section name signifies te depth
of the section, as shown by this example:

    {==== Level 1 ====}
    {===>} level1_parameter = 1;
    
    {====== Level 2 ====}
    {===>} level2_parameter = 2;
    
    {====== Level 2 ====}
    {======== Level 3 ====}
    {===>} level3_parameter = 3;
    
    {====== Level 2 ====}
    
    {==== Level 1 ====}
    {====== Level 2 ====}

The amount of equals signs does not need to be divisible by 2. In fact,
any amount of equals signs can be used, as long as they are used
consistently for sections of the same depth.

To be compatible with CNS however, at least 4 equals signs should be
used on both sides of the section header.

### Notes on access level inheritance

_TODO_

For reference, see the function `squash_accesslevels()` in cnsparser.py.

EXAMPLES
--------

_TODO_

SEE ALSO
--------

- [HADDOCK-WebUI](https://github.com/csmeele/HADDOCK-WebUI), a HADDOCK
  interface that makes use of data models generated by CNSParser

LICENSE
-------

To be decided.

Do not copy or redistribute this development version.

AUTHOR
------

[Chris Smeele](https://github.com/cjsmeele)
