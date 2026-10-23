mtsv-spec
=========

A plain-text file format for multi-sheet spreadsheets such as Excel (XLSX)
workbooks: Multi-Sheet Tab-Separated Values (MTSV), specified in an IETF
Internet-Draft. MTSV is TSV with one more dimension: a tab separates
fields, a line break separates records, and a form feed separates sheets,
so a whole workbook is one plain-text file. This repository is the working
area of the draft.

Why It Is Useful
----------------

TSV and CSV hold one table. Excel and ODS hold many sheets, but as
binary packages. MTSV holds many sheets as plain text, so people, AI agents
and ordinary tools such as text editors, diff and grep read the whole
workbook.

Get Started
-----------

* Read the draft: [draft-demosra-mtsv on the IETF Datatracker](https://datatracker.ietf.org/doc/draft-demosra-mtsv/)
* Read its source: [draft-demosra-mtsv.md](draft-demosra-mtsv.md)
* See the examples: [examples/](examples/), each example of the draft as an MTSV file
* Convert files to MTSV: [mtsv, the Python library and command](https://github.com/demos-ra/mtsv)

Details
-------

* Status: individual Internet-Draft
* Media type: [text/prs.mtsv in the IANA Media Types registry](https://www.iana.org/assignments/media-types/text/prs.mtsv), as the draft defines it
* Version: the revision number in the draft's name, on the [Datatracker](https://datatracker.ietf.org/doc/draft-demosra-mtsv/)
* Layout: [draft-demosra-mtsv.md](draft-demosra-mtsv.md), the draft; [examples/](examples/), its examples
* License: [LICENSE.md](LICENSE.md): every contribution is a Contribution to the IETF Standards Process, under BCP 78 and BCP 79, as [CONTRIBUTING.md](CONTRIBUTING.md) states

Get Help
--------

* Issues: [mtsv-spec issue tracker](https://github.com/demos-ra/mtsv-spec/issues)
* Contributing: [CONTRIBUTING.md](CONTRIBUTING.md)
* Maintainer: Demos Ra, author of the draft
* More: [all MTSV tools by Demos Ra](https://github.com/demos-ra)
