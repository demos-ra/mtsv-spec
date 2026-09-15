---
title: "Multi-Sheet Tab-Separated Values (MTSV)"
abbrev: "MTSV"
category: info

docname: draft-demos-ra-mtsv-latest
submissiontype: IETF
number:
date:
v: 3
keyword:
 - tsv
 - tabular data
 - media type

author:
 -
    fullname: demos-ra
    email: demos-ra@hotmail.com

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
    target: https://www.w3.org/TR/tabular-data-model/
    title: "Model for Tabular Data and Metadata on the Web"
    author:
      org: W3C
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
  OOXML:
    title: "Office Open XML File Formats"
    author:
      org: ISO/IEC
    seriesinfo:
      ISO/IEC: 29500

--- abstract

This document defines Multi-Sheet Tab-Separated Values (MTSV), a text
format that carries one or more sheets of tab-separated values in a
single file. MTSV is TSV with one additional dimension: sheets are
separated by the ASCII form feed (FF) character. Every TSV file that
contains no FF is an MTSV file. This document also registers the
text/prs.mtsv media type.


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

MTSV does not carry over three restrictions of the TSV grammar:

* A field can be empty, and a record can consist of a single field, as
  in {{RFC4180}}.

* A sheet can consist of a header with no records, or of no lines at
  all, as a sheet can have no rows in {{OOXML}}.

Every TSV file that contains no FF is an MTSV file ({{data-model}}).

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
: The text that follows an FF on the same line.

unnamed sheet:
: The sheet formed by the lines before the first FF, or by all lines of
  a file that contains no FF. It exists only if there is at least one
  such line.

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

Every sheet has a sheet name, except the unnamed sheet. An MTSV file has
an unnamed sheet only if the file contains at least one line before the
first FF. An empty sheet name is permitted. Sheet names are not required
to be unique.

A TSV file that contains no FF is an MTSV file that consists of exactly
one unnamed sheet.


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
| line break | LF (%x0A) or CRLF (%x0D.0A) | records | {{TSV}} |
| form feed | FF (%x0C) | sheets | {{RFC20}} |

An FF appears only at the start of a line.

## Sheet Name

A sheet name is written on the line that begins with an FF, directly
after the FF, and ends at the line break. A sheet name follows the same
rules as a field: it cannot contain HT, LF, FF, or CR.

## Grammar

~~~ abnf
mtsv-file     = unnamed-sheet *named-sheet
unnamed-sheet = sheet-body
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
unnamed sheet, and each line that begins with an FF as the start of a
new sheet.

A parser MUST accept both LF and CRLF as line breaks, consistent with
the default line terminators in {{CSVW}}. A parser MAY accept a final
record that is not followed by a line break, consistent with {{RFC4180}}.

A parser MAY ignore a byte order mark at the start of a file, consistent
with {{Section 8.1 of RFC8259}}.

A parser MAY accept input that does not conform to this document.

An implementation MAY set limits on the size of files, the number of
sheets, records, and fields, and the length of fields and sheet names,
consistent with {{Section 9 of RFC8259}}.


# Generators {#generators}

An MTSV generator MUST produce MTSV files that conform to {{syntax}}.

A generator MUST end every record with a line break. A generator
SHOULD encode MTSV files in UTF-8, consistent with {{RFC2277}}, and MUST
NOT add a byte order mark, consistent with {{Section 8.1 of RFC8259}}.

A field or sheet name that contains HT, LF, FF, or CR cannot be
represented in MTSV, as with fields that contain a tab in {{TSV}}. A
generator MUST NOT write such a value. How a generator handles such
values is out of scope.


# Examples

## Single Sheet

This is the example from {{TSV}}. It is both a TSV file and an MTSV file
with one unnamed sheet.

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

This MTSV file contains an unnamed sheet, an empty sheet named "Empty",
and a sheet named "Animals".

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
  Otherwise, the unnamed sheet of a later file becomes part of the last
  sheet of the file before it.

Unchanged structure:
: Printing, Unicode normalization, and conversion between character
  sets do not change the structure of an MTSV file.

Spreadsheets:
: Spreadsheet applications apply their own rules to sheet names, such
  as uniqueness and length, and to data types and sizes. These rules are
  outside the scope of this document.


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
: None

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
: None

Additional information:
: Deprecated alias names for this type:
  : None

  Magic number(s):
  : None

  File extension(s):
  : .mtsv

  Macintosh file type code(s):
  : None

Person & email address to contact for further information:
: demos-ra <demos-ra@hotmail.com>

Intended usage:
: COMMON

Restrictions on usage:
: None

Author:
: demos-ra

Change controller:
: demos-ra


--- back

# Change Log
{:removeInRFC="true"}

draft-demos-ra-mtsv-00:
: Initial version.
