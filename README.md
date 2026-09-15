# Multi-Sheet Tab-Separated Values (MTSV)

This repository holds the specification for Multi-Sheet Tab-Separated Values
(MTSV), written in Internet-Draft format.

* [Specification source](draft-demos-ra-mtsv.md)

## What it is

MTSV is TSV with one more dimension. TSV uses two ASCII format effectors: tab
moves to the next field, and line feed moves to the next record. MTSV adds the
next larger one, form feed, which moves to the next sheet. The text after a
form feed, on the same line, is the sheet name.

| Axis | Character             | Unit   |
|------|-----------------------|--------|
| X    | tab (HT, 0x09)        | field  |
| Y    | line feed (LF, 0x0A)  | record |
| Z    | form feed (FF, 0x0C)  | sheet  |

Every TSV file that contains no form feed is an MTSV file.

MTSV defines structure only: no data types, formulas, or formatting.

## Example

`<TAB>` is a tab and `<FF>` is a form feed:

```
<FF>People
Name<TAB>Age<TAB>Address
Paul<TAB>23<TAB>1115 W Franklin
Zeke<TAB>45<TAB>W Main St
<FF>Animals
Name<TAB>Age<TAB>Address
Bessy the Cow<TAB>5<TAB>Big Farm Way
```

The same file with the real bytes is
[examples/multi-sheet.mtsv](examples/multi-sheet.mtsv).

## Status

Revision -00. Not yet submitted to the IETF.

* File extension: `.mtsv`
* Media type: `text/prs.mtsv` (proposed, not yet registered with IANA)

## Contributing

Discuss the specification or report problems in
[GitHub Issues](https://github.com/demos-ra/mtsv/issues).

## License

[CC BY 4.0](LICENSE)
