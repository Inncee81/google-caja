If you're a tech-writer, and want to help out, we'd love your help.  Please drop us a line at [our mailing list](http://groups.google.com/group/google-caja-discuss/).  Below are some specific topics we think might be useful.

If you're a coder who wants to contribute, please see our ProjectIdeas page.


## How to write code for Caja ##
You might want to cover the differences between Cajita and Valija, features removed such as `with`, common ways `eval` is used that doesn't need to be, a bit on CSS and HTML schemas and the kinds of messages one sees, optimizing to make as much of your HTML and CSS statically analyzable.

## Kinds of attacks that Caja prevents ##
You might want to start talking about the differences between containment vs isolation and why iframes are the latter.  Then the front page talks briefly about the various kinds of ambient authority that caja contains.

## URI Policies ##
There are already wiki pages on the kinds of URLs.  Maybe give examples of each and example scenarios where they might want to be rewritten or proxied, with examples of URI policies that implement these requirements.

## Simplest possible Caja integration into your page using the cajoling service ##
See HostingModules (and talk to Mike Stay for more directions)

## Composable code using the Caja module system ##
Talk to Ihab Awad for more directions

## How to tame container APIs to allow plugins to access your application's objects ##
Talk to Jasvir Nagra and Mark Miller for directions

## Sophisticated Caja integration into your Web application by embedding the Caja codebase  in a Java or C# application ##