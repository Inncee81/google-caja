# Cajoler Pipeline Configuration #

## Background ##

The Cajoler parses some input files to parse trees, transforms those parse
trees to other parse trees, and then renders those parse trees back to source
code.

The Cajoler's main method looks like:
```
      void cajole(inputStreams, outputStreams) {
        inputParseTrees = parseInputs(inputStreams);
        outputParseTrees = pipeline.apply(inputParseTrees);
        emit(outputParseTrees, outputStreams);
      }
```

> ![http://google-caja.googlecode.com/svn/trunk/doc/images/Cajoler-Arch.png](http://google-caja.googlecode.com/svn/trunk/doc/images/Cajoler-Arch.png)

The input files on the left are classified into CSS, JS, and HTML, and various
template files (`*.jsp`, `*.csst`) represented as triangles on the left.
(The template files here are compiled to functions that produce trademarked
strings).

Those parse trees are then fed into the boxes in the middle which can transform
one parse tree into another, break one parse tree into pieces, or combine
multiple parse trees together.

Finally, there is one JS and one CSS parse tree left, and those are written out.


## Why Configurable Pipelines? ##

The diagram above shows that the Cajoler architecture decomposes well into
separable tasks, but it still depends on its inputs containing the right
information, and its outputs being used in the right way.

Clients have requested the ability to replace some stages with their own
implementations, or add chunks onto the process.  Examples of changes they'd
like to make include:
  * Replacing our HTML parser and/or renderer with their own scheme.
  * Adding additional HTML transforms, such as converting `<embed>` tags with safe `<object>` tags.
  * Adding instrumentation to JS code.
  * JS macro expansion.
  * Lint style checks.
Additionally, some groups like the Lively Kernel folks are interested in working
with us to possibly support safe SVG generation, similar to our mechanisms for
safe HTML generation.  If we want to encourage container maintainers to include
experimental stages such us SVG, it'd be nicer if we could just distribute a
jar and a configuration file, as maintainers are less leery of configuration
changes than code changes.


## Pipelines: The Junk → (Pipeline) → Bicycle Design Pattern ##

The Cajoler is organized as a series of parse tree transformations that take
heterogeneous inputs and produce a (hopefully) working web application.  This is
somewhat analogous to building a bicycle from a box of junk that you found in
your attic.  The parsing stage above, fills the box of junk with parse trees.
The cajoler then shoves the box into the pipeline, and hopefully a working web
application (bicycle).  Finally it emits the parse-trees (sends the box with the
bicycle) to the client.

The Junk → (Pipeline) → Bicycle design pattern specifies a
function from a box of junk to a bicycle, or more precisely, from a box of junk
and no (NULL) bicycle to a box of less junk and a bicycle.  So the type of
a pipeline is `(Junk ∗ Bicycle) ⇨ (Junk ∗ Bicycle)`.

The diagram above has lines running all over the place.  We could be careful to
route all the parse trees to the right place, but that would require that a
stage either know where their inputs are coming from, or know where their
outputs are going to, or both.

Instead, we can just put the inputs in a box.  Then each stage takes from the
box things that it can operate on.  It can do any combination of the following:
  * Take parts from the box, combine them and put them back in the box.
  * Take a part from the box, and break it into pieces.
  * Attach a part from the box to the bike.
  * Disassemble the bike and put parts into the box.

Pipeline stages can be composed in this way since a stage has the type
`(Junk ∗ Bicycle) ⇨ (Junk ∗ Bicycle)` and
`((α ⇨ α) ∘ (α ⇨ α)) ⇨ (α ⇨ α)`.

The person designing a pipeline stage doesn't care how the parts get into the
box or how they end up getting used.  Whomever configures the pipeline just has
to make sure that all the parts get used.


## Design ##

The goal is to allow a container maintainer to specify a wiring diagram like the
one in the diagram.

Since pipelines compose, the main thing we need to specify is which stages depend on the outputs of other stages, so that we can order stages.  This need not, and should not, be
a total ordering, since a total ordering would be over-specifying the pipeline -- such a
configuration would be less extensible than one that doesn't over-specify.

We represent the wiring diagram explicitly as compositions using a subset of javascript, since JS makes it easy to specify input/output relationships, we already have a lot of infrastructure for parsing it, and it will be immediately apparent to our users.

We will recognize the following structure:
```
    Config       ::==  (<ParserSpec> | <Relation>)*
    ParserSpec   ::==  <Reference> '=' <JavaClassName> '(' <Glob> ')' ';'
    Relation     ::==  <Reference> '=' <JavaClassName>
                       '(' (<Reference> (',' <Reference>)*)? ')' ';'
    Glob         ::==  <StringLiteral>                                    # like '*.html'
```

The class named in a `<ParserSpec>` must implement the class `com.google.caja.plugin.Parser` which consumes files and produces parse trees.

The class named in a `<Relation>` must implement `com.google.caja.util.Pipeline.Stage<Jobs>`, unless it is the `com.google.caja.plugin.Emitter` rule which specifies that the results are meant
to be outputs.

This scheme can be extended to routing configuration files to pipeline stages if that
becomes necessary.  We could add a construct like:
```
    StageConfig  ::==  <Reference> = <JavaClassName> '(' <JsonRef> ')' ';'
    JsonRef      ::==  <Uri>
                    |  <JsonLiteral>
    Uri          ::==  <StringLiteral>
```
where the class has to be an instance of `com.google.caja.plugin.Config` and has a constructor that takes a JSON parse tree.  The reference to the left can then be used in
the parameter list to any of the other constructs to get passed to their constructors.


## Example Configuration ##

Let's consider the case where a client wants to add JS
[macro expansion](http://google-caja.googlecode.com/svn/changes/mikesamuel/string-interpolation-29-Jan-2008/trunk/src/NOT-FOR-TRUNK/interp/index.html#optimizer).
We implement a `com.google.caja.util.Pipeline.Stage` that pulls each
JavaScript parse tree out of the box, looks for a macro and does the expansion
server side.

Let's put that "Rewrite Macros" box in between the parsed JS inputs and the
"Cajole JS" box thus:

> ![http://google-caja.googlecode.com/svn/trunk/doc/images/Cajoler-Arch-Tweaked.png](http://google-caja.googlecode.com/svn/trunk/doc/images/Cajoler-Arch-Tweaked.png)

The part of the old configuration file into which we want to inject
"Rewrite Macros" looks like:
The part of the old configuration file into which we want to inject
"Rewrite Macros" looks like:
```
    ...

    htmlInputs = new com.google.caja.parser.JsParser('*.html');
    parseTrees = new com.google.caja.plugin.HtmlExtractor(htmlInputs);
    safeHtml = new com.google.caja.plugin.HtmlValidator(
        filter(parseTrees, '*.html'));

    ...

    outputJs = new com.google.caja.plugin.Emitter(cajoledJs);

    ...
```

Let's rewrite that thusly:
```
    ...

    htmlInputs = new com.google.caja.parser.JsParser('*.html');
    parseTrees = new com.google.caja.plugin.HtmlExtractor(htmlInputs);
    html = new com.foo.RewriteEmbeds(filter(parseTrees, '*.html'));     // ADDED
    safeHtml = new com.google.caja.plugin.HtmlValidator(html);          // CHANGED

    ...

    outputJs = new com.google.caja.plugin.Emitter(cajoledJs);

    ...
```