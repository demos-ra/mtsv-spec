---
title: "Multi-Sheet Tab-Separated Values (MTSV)"
abbrev: "MTSV"
category: info

docname: draft-demosra-mtsv-01
submissiontype: IETF
number:
date:
v: 3
keyword:
 - tsv
 - tabular data
 - media type
venue:
  github: demos-ra/mtsv-spec

author:
 -
    fullname: Demos Ra
    email: demos_ra@hotmail.com

normative:
  RFC20:
  RFC3629:
  RFC5234:
  RFC6838:
  TSV:
    target: https://www.iana.org/assignments/media-types/text/tab-separated-values
    title: "text/tab-separated-values"
    author:
      org: University of Minnesota Internet Gopher Team

informative:
  RFC2046:
  RFC2277:
  RFC4180:
  RFC6657:
  RFC8259:
  CSVW:
    target: https://www.w3.org/TR/2015/REC-tabular-data-model-20151217/
    title: "Model for Tabular Data and Metadata on the Web"
    author:
      org: W3C
    date: 2015-12-17
  UAX14:
    target: https://www.unicode.org/reports/tr14/
    title: "Unicode Line Breaking Algorithm"
    author:
      org: Unicode Consortium
  UCD:
    target: https://www.unicode.org/ucd/
    title: "Unicode Character Database"
    author:
      org: Unicode Consortium
  XML:
    target: https://www.w3.org/TR/2008/REC-xml-20081126/
    title: "Extensible Markup Language (XML) 1.0 (Fifth Edition)"
    author:
      org: W3C
    date: 2008-11-26
  ODF:
    target: https://docs.oasis-open.org/office/OpenDocument/v1.3/
    title: "Open Document Format for Office Applications (OpenDocument) Version 1.3"
    author:
      org: OASIS
    date: 2021-04-27
  OOXML:
    title: "Information technology - Document description and processing languages - Office Open XML File Formats - Part 1: Fundamentals and Markup Language Reference"
    author:
      org: ISO/IEC
    seriesinfo:
      ISO/IEC: 29500-1:2016
    date: 2016

...

--- abstract

This document defines Multi-Sheet Tab-Separated Values (MTSV), a text
format that carries one or more sheets of tab-separated values in a
single file. MTSV is TSV with one additional dimension: sheets are
separated by the ASCII form feed (FF) character. A TSV file that
contains no FF, and no CR other than in CRLF line breaks, is an MTSV
file. This document also registers the text/prs.mtsv media type.


--- middle

# Introduction {#intro}

The tab-separated values format {{TSV}} encodes one set of records per
file. Fields are separated by a tab, and records are separated by line
breaks.

Both separators are ASCII format effectors {{RFC20}}. HT moves "to the
next in a series of predetermined positions along the printing line",
and LF moves "to the next printing line". MTSV adds FF, the format
effector for the next larger unit, which moves "to the first
pre-determined printing line on the next form or page". Tab gives the
next field, line feed gives the next record, and form feed gives the
next sheet.

## Relationship to TSV {#relationship}

MTSV keeps the separators and structure of TSV, and adds only one
separator, FF, and a name for each sheet.

MTSV does not carry over these restrictions of the TSV grammar:

* A field can be empty, and a record can consist of a single field, as
  in {{RFC4180}}.

* A sheet can consist of a header with no records, or of no lines at
  all, as a sheet can have no rows in {{OOXML}}.

A TSV file that contains no FF, and no CR other than in CRLF line
breaks, is an MTSV file ({{data-model}}).

## Out of Scope

This document does not define data types, formulas, formatting, cell
references, or metadata. Conversion to and from spreadsheet formats such
as {{ODF}} and {{OOXML}} is also out of scope.


# Conventions and Definitions

{::boilerplate bcp14-tagged}

The grammar in this document uses ABNF {{RFC5234}}, including its core
rules HTAB, LF, and CRLF.

In examples, `<TAB>` denotes the tab character (%x09), as in {{TSV}},
and `<FF>` denotes the form feed character (%x0C). Line breaks are
shown as line breaks; every line shown, including the last, ends with a
line break.

This document uses the following terms:

field:
: A text value, as defined in {{TSV}}.

record:
: A sequence of fields on one line, as defined in {{TSV}}.

header:
: The first record of a sheet, which contains the name of each field,
  as defined in {{TSV}}.

sheet:
: A header followed by zero or more records, or no lines at all (an
  empty sheet). The term matches the corresponding structure in
  {{OOXML}}.

sheet name:
: The text that follows an FF on the same line. The first sheet of a
  file can be written without its FF line; its sheet name is then
  empty.

MTSV file:
: A sequence of sheets, conforming to {{syntax}}.

TSV file:
: A file conforming to {{TSV}}.

generator:
: An implementation that writes MTSV files.

parser:
: An implementation that reads MTSV files.


# Data Model {#data-model}

A field is text. A record is an ordered sequence of fields. A sheet is a
header and an ordered sequence of zero or more records. Every record in
a sheet has as many fields as the header of that sheet. A sheet with no
lines is an empty sheet; it has neither a header nor records.

An MTSV file is an ordered sequence of sheets. The order of the sheets is
the order in which they appear in the file.

Every sheet has a sheet name. An empty sheet name is permitted. Sheet
names are not required to be unique.

The lines before the first FF, if there are any, form the first sheet,
written without its FF line; its sheet name is empty.

A TSV file that contains no FF, and no CR other than in CRLF line
breaks, is an MTSV file that consists of exactly one sheet, whose sheet
name is empty.


# Syntax {#syntax}

## Character Encoding {#encoding}

An MTSV file is text in the character set identified by the charset
parameter ({{media-type}}). If the parameter is absent, the character set
is UTF-8 {{RFC3629}}. The grammar below is expressed in terms of
characters after decoding.

## Separators

MTSV uses three separators, all of which are ASCII format effectors
{{RFC20}}:

| Separator | Character | Separates | Source |
|---|---|---|---|
| tab | HT (%x09) | fields | {{TSV}} |
| line break | LF (%x0A) or CRLF (%x0D.0A) | records | {{TSV}}, {{CSVW}} |
| form feed | FF (%x0C) | sheets | {{RFC20}} |

An FF appears only at the start of a line.

## Sheet Name

A sheet name is written on the line that begins with an FF, directly
after the FF, and ends at the line break. A sheet name follows the same
rules as a field: it cannot contain HT, LF, FF, or CR.

## Grammar

~~~ abnf
mtsv-file     = first-sheet *named-sheet
first-sheet   = sheet-body
named-sheet   = FF sheet-name eol sheet-body
sheet-body    = [header *record]
header        = record
record        = field *(HTAB field) eol
field         = *field-char
sheet-name    = *field-char
field-char    = %x00-08 / %x0B / %x0E-10FFFF
                ; any character except HTAB, LF, FF, and CR
FF            = %x0C
eol           = LF / CRLF
~~~

In addition to matching the grammar, each record in a sheet MUST have
the same number of fields as the header of that sheet, as required by
{{TSV}}.


# Parsers {#parsers}

An MTSV parser MUST accept every MTSV file that conforms to {{syntax}}.

A parser MUST treat the lines before the first FF, if any, as the
first sheet, with an empty sheet name, and each line that begins with
an FF as the start of a new sheet.

A parser MUST accept both LF and CRLF as line breaks, consistent with
the default line terminators in {{CSVW}}.

A parser SHOULD treat a U+FEFF character at the start of a file as an
encoding signature and not as part of the first field, consistent with
{{Section 6 of RFC3629}}.

A parser MAY accept input that does not conform to this document.

An implementation MAY set limits on the size of files, the number of
sheets, records, and fields, and the length of fields and sheet names,
consistent with {{Section 9 of RFC8259}}.


# Generators {#generators}

An MTSV generator MUST produce MTSV files that conform to {{syntax}}.

A generator MUST end every record with a line break. A generator
SHOULD encode MTSV files in UTF-8, consistent with {{RFC2277}}.

A generator MUST write an FF line before every sheet, including the
first, so that generated files can be concatenated ({{interop}}).

A field or sheet name that contains HT, LF, FF, or CR cannot be
represented in MTSV, as with fields that contain a tab in {{TSV}}. A
generator MUST NOT write such a value. How a generator handles such
values is out of scope.


# Examples

## Single Sheet

This is the example from {{TSV}}. It is both a TSV file and an MTSV file
with one sheet, whose sheet name is empty.

~~~
Name<TAB>Age<TAB>Address
Paul<TAB>23<TAB>1115 W Franklin
Bessy the Cow<TAB>5<TAB>Big Farm Way
Zeke<TAB>45<TAB>W Main St
~~~

## Multiple Sheets

This MTSV file contains two sheets, named "People" and "Animals".

~~~
<FF>People
Name<TAB>Age<TAB>Address
Paul<TAB>23<TAB>1115 W Franklin
Zeke<TAB>45<TAB>W Main St
<FF>Animals
Name<TAB>Age<TAB>Address
Bessy the Cow<TAB>5<TAB>Big Farm Way
~~~

## Empty Sheet

This MTSV file contains a first sheet with an empty sheet name, an
empty sheet named "Empty", and a sheet named "Animals".

~~~
Name<TAB>Age
Paul<TAB>23
<FF>Empty
<FF>Animals
Name<TAB>Age
Bessy the Cow<TAB>5
~~~


# Interoperability Considerations {#interop}

{{relationship}} describes which TSV files are MTSV files. This section
describes how common text processing affects MTSV files.

MTSV gives structure to four characters: HT, LF, FF, and CR. These
characters also have other standard properties, and text processing
that acts on those properties can change the structure of an MTSV file:

Line splitting:
: FF has the mandatory break class in {{UAX14}}. An application that
  supports only TSV can present a line that begins with an FF either as a
  record whose first field begins with FF, or as an empty line followed
  by a line that contains the sheet name.

Whitespace:
: HT, LF, FF, and CR have the White_Space property in {{UCD}}. Trimming
  whitespace from a line can remove an FF, which turns a sheet name
  into a record, or remove an HT at either end of a line, which removes
  empty fields. Splitting a line on runs of whitespace removes empty
  fields and splits fields that contain spaces.

Tab expansion:
: Replacing HT with spaces removes the field structure, as it does for
  {{TSV}}.

Control characters:
: FF is a control character. Applications that remove or reject control
  characters remove sheet boundaries. {{XML}} does not permit FF, so an
  MTSV file cannot be carried as XML 1.0 character data without an
  additional encoding.

Line endings:
: Converting between LF and CRLF does not change the structure of an
  MTSV file ({{parsers}}). Converting line breaks to CR alone produces a
  file that does not conform to this document.

Concatenation:
: Concatenating MTSV files produces an MTSV file that contains the
  sheets of each file, in order, only if each non-empty file ends with a
  line break and each non-empty file after the first begins with an FF.
  Otherwise, the first sheet of a later file becomes part of the last
  sheet of the file before it. Files written by a generator begin with
  an FF ({{generators}}).

Unchanged structure:
: Printing, Unicode normalization, and conversion between character
  sets do not change the structure of an MTSV file.

Spreadsheets:
: Spreadsheet applications apply their own rules to sheet names, such
  as uniqueness and length, and to data types and sizes. These rules are
  outside the scope of this document.


# IANA Considerations {#iana}

## Media Type Registration {#media-type}

This document registers the text/prs.mtsv media type in the personal
tree, according to {{Section 3.3 of RFC6838}}, using the template in
{{Section 5.6 of RFC6838}}.

Type name:
: text

Subtype name:
: prs.mtsv

Required parameters:
: N/A

Optional parameters:
: charset. MTSV has no in-band charset information, so a default is
  needed; if charset is absent, UTF-8 is assumed, as
  {{Section 3 of RFC6657}} specifies for text subtypes that define a
  default.

Encoding considerations:
: 8bit. As per {{Section 4.1.1 of RFC2046}}, this media type uses CRLF to
  denote line breaks in transport. Implementations need to be aware
  that files often use LF alone.

Security considerations:
: See {{security}}.

Interoperability considerations:
: See {{interop}}.

Published specification:
: This document.

Applications that use this media type:
: Applications that store or exchange multiple sheets of tabular data
  as text.

Fragment identifier considerations:
: N/A

Additional information:
: Deprecated alias names for this type:
  : N/A

  Magic number(s):
  : N/A

  File extension(s):
  : .mtsv

  Macintosh file type code(s):
  : N/A

Person & email address to contact for further information:
: Demos Ra (demos_ra@hotmail.com)

Intended usage:
: COMMON

Restrictions on usage:
: N/A

Author:
: Demos Ra

Change controller:
: Demos Ra

Files written by a generator begin with FF (%x0C).


# Internationalization Considerations {#i18n}

This section collects the internationalization decisions of this
document, as {{Section 6 of RFC2277}} recommends.

HT, LF, CRLF, and FF are protocol elements. Fields and sheet names are
text ({{data-model}}), as {{Section 2 of RFC2277}} requires a protocol
to distinguish.

An MTSV file carries no in-band charset information. The charset
parameter of the media type identifies the charset ({{media-type}}), and
UTF-8 is assumed when that parameter is absent ({{encoding}}), so that a
stored file remains readable without it, as {{Section 3.2 of RFC2277}}
advises. Any other charset is one registered in the IANA charset
registry, as {{Section 3.1 of RFC2277}} requires.

Sheet names are internationalized. A sheet name is text and follows the
same rules as a field ({{data-model}}); it is not a US-ASCII identifier.
{{Section 2 of RFC2277}} requires a document to state this.

MTSV carries no language information. This document leaves language to
the enclosing protocol or to separate metadata such as {{CSVW}}, which
{{Section 2 of RFC2277}} permits where the responsibility belongs to
another layer.


# Security Considerations {#security}

MTSV files are text and contain no executable content. {{TSV}} lists no
security considerations.

Applications that import MTSV files into spreadsheets can interpret
fields that begin with characters such as "=", "+", "-", or "@" as
formulas. Such applications need to treat imported fields as data.

Sheet names can contain control characters. Applications that display
sheet names need to take care that such characters do not mislead
users.

Parsers that do not set limits ({{parsers}}) can exhaust resources when
reading large files or files with very many sheets.


--- back

# Change Log
{:removeInRFC="true"}

draft-demosra-mtsv-01:
: Gave every sheet a sheet name: the lines before the first FF form a
  first sheet whose sheet name is empty, and the term "unnamed sheet" is
  removed. Required generators to write an FF line before every sheet,
  and removed the rule on a first field that begins with U+FEFF, which
  that makes unnecessary. Used "N/A" in the registration template, as
  Section 5.6 of RFC 6838 asks.

draft-demosra-mtsv-00:
: Replaces draft-demos-ra-mtsv-00, renamed so that the author component
  contains no hyphen. Replaced the byte order mark rules with a parser
  rule that follows Section 6 of RFC 3629, and stated that a first field
  that begins with U+FEFF cannot be represented. Removed the rule on a
  final record without a line break, which the rule on non-conforming
  input already covers. Cited CSVW as the source of LF and CRLF line
  breaks. Added an Internationalization Considerations section, and
  placed the IANA, Internationalization, and Security Considerations
  sections in the order that Section 4 of RFC 7322 recommends.

draft-demos-ra-mtsv-00:
: Initial version.
