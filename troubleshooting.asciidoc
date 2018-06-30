// NOTE: This is AsciiDoc (mostly for the TOC), see: http://asciidoctor.org/docs/asciidoc-syntax-quick-reference/
// NO EMPTY LINES UNTIL THE END OF THE HEADER
// Quickly: bold and italics are the same
// Checkmarks: [ ] or [x]
// Lists: instead of spaces at the beginning (which are allowed), it's number of marks:
// * first level unnumbered
// ** second level unnumbered
// . first level numbered
// .. second level numbered
// Links: http://url[Descriptive Text That's Visible]
// WikiLinks: link:other-page[Other Page]
// Images: image:path/to/image[]
// Note that because of the :imagesdir: below images/ will be prepended if there's no /
// Settings:
:idprefix:
:idseparator: -
ifndef::env-github[:icons: font]
ifdef::env-github,env-browser[]
:toc: macro
:toclevels: 1
endif::[]
ifdef::env-github[]
:branch: master
:status:
:outfilesuffix: .adoc
:!toc-title:
:caution-caption: :fire:
:important-caption: :exclamation:
:note-caption: :notebook:
:tip-caption: :bulb:
:warning-caption: :warning:
endif::[]
:imagesdir: images
// END OF THE HEADER -- You may resume having empty lines

This page has some issues people have reported whist getting started with g2core. Please feel free to add to this page if you have guidance that you think would help.

toc::[]

# Motors Do Not Move
The board seems fine, but the motors don't move.

##
