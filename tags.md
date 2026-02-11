## Tags

In the UI, you'll notice that there is a new field called tags, which is a comma separated list of tags. These are arbitrary list of comma separated tags that are associated with each phase. In addition to manually entered tags, the name of the phase is automatically added as a tag.

## Usage

Tags are used to filter sections in or out of a generated form. For example, if you have a phase called "academic" and you want to include or exclude sections of the template depending on which tags are specified, you can use the tag `{{"include": "academic"}}` or `{{"exclude": "academic"}}` in your form template.

If you want to include all sections associated with a tag, you can use the tag `{{"include": "academic"}}` in your form template.

If you want to exclude all sections associated with a tag, you can use the tag `{{"exclude": "academic"}}` in your form template.

## multiple tags

You can specify multiple tags in a form template by using the `{{"include": "academic,flourishing"}}` and `{{"exclude": "demographics"}}` tags. The tags are comma separated and the tags are evaluated in order. If a section has multiple tags, it will be included if any of the tags are included and excluded if any of the tags are excluded. A section with no tags will always be included.

## priority

Exclude tags take priority over include tags. If a section is excluded, it will not be included even if it is included by another tag.

## usage with blocks

You can combine exclude and include tags with blocks. For example {{"block": "demographics", "include":"pre"}} will include the block demographics only for phases with the tag "pre".

## placement of tags

include/exclude tags can be placed once in any section of the form template. If used in conjuction with block tags, then the entire section will be replaced as normal using the block approach. If used emmbedded in some text and there is no block specified, then the text remain intact but with the tags marker removed.