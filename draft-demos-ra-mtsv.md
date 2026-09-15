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
separated by the ASCII Form Feed character. Every TSV file is an MTSV
file. This document also registers the text/prs.mtsv media type.


--- middle

# Introduction {#intro}

The tab-separated values format {{TSV}} encodes one sheet of data per
file. Fields are separated by a tab, and records are separated by line
breaks.

Both separators are ASCII format effectors {{RFC20}}. HT moves "to the
next in a series of predetermined positions along the printing line",
and LF moves "to the next printing line". MTSV adds the next format
effector in the same series, FF, which moves "to the first
pre-determined printing line on the next form or page". Tab gives the
next field, line feed gives the next record, and form feed gives the
next sheet.

MTSV inherits TSV unchanged and adds only that one separator and a name
for each sheet.

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
shown as line breaks.

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
: The sheet formed by the content before the first FF, or by the whole
  file if it contains no FF.

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
header and an ordered sequence of zero or more records. A sheet with no
lines is an empty sheet; it has neither a header nor records.

An MTSV file is an ordered sequence of sheets. The order of the sheets is
the order in which they appear in the file.

Every sheet has a sheet name, except the unnamed sheet. An MTSV file has
an unnamed sheet only if the file contains at least one line before the
first FF. An empty sheet name is permitted. Sheet names are not required
to be unique.

A TSV file contains no FF. It is therefore an MTSV file that consists of
exactly one unnamed sheet.


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
| line break | LF (%x0A) or CR LF (%x0D.0A) | records | {{TSV}} |
| form feed | FF (%x0C) | sheets | {{RFC20}} |

An FF appears only at the start of a line.

## Sheet Name

A sheet name is written on the line that begins with an FF, directly
after the FF, and ends at the line break. A sheet name follows the same
rules as a field: it cannot contain a tab, a line break, or an FF.

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

A parser MUST treat the content before the first FF, if any, as the
unnamed sheet, and each FF line as the start of a new sheet.

A parser MUST accept both LF and CR LF as line breaks, consistent with
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

A field or sheet name that contains a tab, a line break, or an FF cannot
be represented in MTSV, as with fields that contain a tab in {{TSV}}. A
generator MUST NOT write such a value. How a generator handles such
values is out of scope.

A generator that writes a single unnamed sheet produces a TSV file.


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


# Security Considerations {#security}

MTSV files are text and contain no executable content. {{TSV}} lists no
security considerations; MTSV adds only a separator and sheet names.

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
  needed; if charset is absent, UTF-8 is assumed, as {{Section 4 of RFC6657}}
  specifies for text subtypes that define a default.

Encoding considerations:
: 8bit. As per {{Section 4.1.1 of RFC2046}}, this media type uses CR LF to
  denote line breaks in transport. Implementations need to be aware
  that files often use LF alone.

Security considerations:
: See {{security}}.

Interoperability considerations:
: Every TSV file {{TSV}} is an MTSV file. How an application that
  supports only TSV presents an MTSV file depends on how it splits
  lines: some applications treat FF as a character within a field,
  while others treat FF as a line break, as in {{UAX14}}. Sheet names
  are not required to be unique; applications that require unique
  names need to handle duplicates.

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
