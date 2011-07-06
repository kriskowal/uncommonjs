
1.  Specifications must be in Markdown format.
1.  Specifications must be physically wrapped at 72 columns.
1.  Specifications must use visually equivalent spaces instead of tabs.
1.  Specifications must align tab stops every 4 spaces.
1.  Specifications must use an outline of numbered sentences so that
    they can be easily referenced.
    1.  Within Markdown format, all numbered points must be number 1
        such that they are easy to edit, diff, and patch.
    1.  The markdown ``1.`` must start on and be followed by spaces
        until the next tab stop.
1.  Non-normative text (*statements that cannot be tested*) must be
    marked down with emphasis or in non-numbered paragraphs following
    the specification.
1.  The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT",
    "SHOULD", "SHOULD NOT", "RECOMMENDED",  "MAY", and "OPTIONAL" in
    this document are to be interpreted as described in RFC 2119.

In Vim, use ``:set tw=72`` or ``textwidth=72`` to configure automatic
physical wrapping at the 72nd column.  Select a region and type `gq` to
reflow a paragraph or bullet point.  Use ``ts=4`` and ``sw=4``
(``tabstop=4`` and ``shiftwidth=4``) to align text at tab stops.

