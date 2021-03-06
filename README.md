[![Build Status](https://travis-ci.org/dart-lang/markdown.svg?branch=master)](https://travis-ci.org/dart-lang/markdown)

> A fork of [dart-markdown](https://github.com/dart-lang/markdown)
> for easy customization of Markdown syntaxes.

**Differences**

* `LinkMapper` introduced for mapping a *logic* link to a real one, e.g., `#abc` mapped to `https://foo/abc`.
* `parseInlineLink` introduced for parsing URL in inline link.
* `InlineParser.be` and `BlockParser.be` introduced for easy customization. They are used in pair if all parsing shares the same set of syntaxes.
* `Document`'s constructor introduced additional arguments, `options`, `blockParserBuilder` and `inlineParserBuilder` for easy customization.
* `InlineSyntax.match` introduced for easy overriding.
* `LinkSyntax` recognizes a link even if it is not balanced with parentheses.
* `InlineSyntax`'s constructor introduces the caseSensitive argument
* `FencedCodeBlockSyntax.getLanguageClass` introduced for generating custom CSS class
* `TableSyntax.processCellContent` introduced for pre-processing cell's content
& `CondensedHtmlRenderer` introduced for customizing CSS easier.

**Resources**

* [API Reference](http://www.dartdocs.org/documentation/markd/2.1.3+7)
* [Git Repos](https://github.com/tomyeh/markd)

**Customizations**

**Who Uses**

* [Quire](https://quire.io) - a simple, collaborative, multi-level task management tool.
* [Keikai](https://keikai.io) - a sophisticated spreadsheet for big data

## Introduction

A portable Markdown library written in Dart. It can parse Markdown into
HTML on both the client and server.

Play with it at
[dart-lang.github.io/markdown](https://dart-lang.github.io/markdown).

### Usage

```dart
import 'package:markd/markdown.dart';

void main() {
  print(markdownToHtml('Hello *Markdown*'));
  //=> <p>Hello <em>Markdown</em></p>
}
```

### Syntax extensions

A few Markdown extensions, beyond what was specified in the original
[Perl Markdown][] implementation, are supported. By default, the ones supported
in [CommonMark] are enabled. Any individual extension can be enabled by
specifying an Array of extension syntaxes in the `blockSyntaxes` or
`inlineSyntaxes` argument of `markdownToHtml`.

The currently supported inline extension syntaxes are:

* `new InlineHtmlSyntax()` - approximately CommonMark's
  [definition][commonmark-raw-html] of "Raw HTML".

The currently supported block extension syntaxes are:

* `const FencedCodeBlockSyntax()` - Code blocks familiar to Pandoc and PHP
  Markdown Extra users.
* `const HeaderWithIdSyntax()` - ATX-style headers have generated IDs, for link
  anchors (akin to Pandoc's [`auto_identifiers`][pandoc-auto_identifiers]).
* `const SetextHeaderWithIdSyntax()` - Setext-style headers have generated IDs
  for link anchors (akin to Pandoc's
  [`auto_identifiers`][pandoc-auto_identifiers]).
* `const TableSyntax()` - Table syntax familiar to GitHub, PHP Markdown Extra,
  and Pandoc users.

For example:

```dart
import 'package:markd/markdown.dart';

void main() {
  print(markdownToHtml('Hello <span class="green">Markdown</span>',
      inlineSyntaxes: [new InlineHtmlSyntax()]));
  //=> <p>Hello <span class="green">Markdown</span></p>
}
```

### Extension sets

To make extension management easy, you can also just specify an extension set.
Both `markdownToHtml()` and `new Document()` accept an `extensionSet` named
parameter. Right now there are two extension sets:

* `ExtensionSet.none` includes no extensions. With no extensions, Markdown
  documents will be parsed closely to how they might be parsed by the original
  [Perl Markdown][] implementation.
* `ExtensionSet.commonMark` includes two extensions so far, which bring this
  package's Markdown parsing closer to what is found in the [CommonMark] spec:

  * `new InlineHtmlSyntax()`
  * `const FencedCodeBlockSyntax()`

* `ExtensionSet.gitHubWeb` includes seven extensions:

  * `new EmojiSyntax()`
  * `new InlineHtmlSyntax()`
  * `const HeaderWithIdSyntax()`, which adds `id` attributes to ATX-style
    headers, for easy intra-document linking.
  * `const SetextHeaderWithIdSyntax()`, which adds `id` attributes to
    Setext-style headers, for easy intra-document linking.
  * `const FencedCodeBlockSyntax()`
  * `new StrikethroughSyntax()`
  * `const TableSyntax()`

### Custom syntax extensions

You can create and use your own syntaxes.

```dart
import 'package:markd/markdown.dart';

void main() {
  var syntaxes = [new TextSyntax('nyan', sub: '~=[,,_,,]:3')];
  print(markdownToHtml('nyan', inlineSyntaxes: syntaxes));
  //=> <p>~=[,,_,,]:3</p>
}
```

### HTML sanitization

This package offers no features in the way of HTML sanitization. Read Estevão
Soares dos Santos's great article, ["Markdown's XSS Vulnerability (and how to
mitigate it)"], to learn more.

The authors recommend that you perform any necessary sanitization on the
resulting HTML, for example via `dart:html`'s [NodeValidator].

### CommonMark compliance

This package contains a number of files in the `tool` directory for tracking
compliance with [CommonMark].

#### Updating CommonMark stats when changing the implementation

 1. Update the library and test code, making sure that tests still pass.
 2. Run `dart tool/stats.dart --update-files` to update the
    per-test results `tool/common_mark_stats.json` and the test summary
    `tool/common_mark_stats.txt`.
 3. Verify that more tests now pass – or at least, no more tests fail.
 4. Make sure you include the updated stats files in your commit.

#### Updating the CommonMark test file for a spec update

 1. Check out the [CommonMark source]. Make sure you checkout a *major* release.
 2. Dump the test output overwriting the existing tests file.

    ```console
    > cd /path/to/common_mark_dir
    > python3 test/spec_tests.py --dump-tests > \
      /path/to/markdown.dart/tool/common_mark_tests.json
    ```

 3. Update the stats files as described above. Note any changes in the results.
 4. Update any references to the existing spec by search for
    `http://spec.commonmark.org/0.28` in the repository. (Including this one.)
    Verify the updated links are still valid.
 5. Commit changes, including a corresponding note in `CHANGELOG.md`.

[Perl Markdown]: http://daringfireball.net/projects/markdown/
[CommonMark]: http://commonmark.org/
[commonMark-raw-html]: http://spec.commonmark.org/0.28/#raw-html
[CommonMark source]: https://github.com/jgm/CommonMark/
[pandoc-auto_identifiers]: http://pandoc.org/README.html#extension-auto_identifiers
["Markdown's XSS Vulnerability (and how to mitigate it)"]: https://github.com/showdownjs/showdown/wiki/Markdown%27s-XSS-Vulnerability-(and-how-to-mitigate-it)
[NodeValidator]: https://api.dart.dev/stable/dart-html/NodeValidator-class.html
