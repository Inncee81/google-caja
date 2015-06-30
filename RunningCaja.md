# Getting Started with Caja #

## Prerequisites ##

Basic familiarity with Java, including
  * how to manage a CLASSPATH
  * how to run a Java app from the command line
  * how to run a Java app from within an IDE like [Eclipse](http://www.eclipse.org/downloads/).
For more information on those topics, see [PATH and CLASSPATH](http://java.sun.com/docs/books/tutorial/essential/environment/paths.html), [the command line](http://java.sun.com/docs/books/tutorial/essential/environment/cmdLineArgs.html).

Caja requires
  * [JDK](http://java.sun.com/javase/downloads/index.jsp) 6
  * The [subversion](http://subversion.tigris.org/project_packages.html) version manager
  * The [Apache Ant](http://ant.apache.org/manual/install.html) build system.

If you intend to contribute patches to Caja (and we would love you to do that, see [ContributingCode](ContributingCode.md) for how), you need to sign a Contributor Licence Agreement (either the [individual CLA](http://code.google.com/legal/individual-cla-v1.0.html) or the [corporate CLA](http://code.google.com/legal/corporate-cla-v1.0.html), depending on your situation). Note that patches are accepted at the discretion of the development team. Membership of the development team will be offered to those who consistently make positive contributions.

## Building Caja via Ant ##

Follow the instructions at http://code.google.com/p/google-caja/source/checkout to checkout the Caja source.

You should see a lot of lines like this:
```
A  google-caja
A  google-caja/docs
...
```

Then, from the command line:
```
$ cd google-caja
$ ant
```

If Ant is properly installed, you should see something like this:
```
Buildfile: build.xml

dirs:
    [mkdir] Created dir: ...
...

jars:
     [copy] Copying 13 files to ant-jars
      [jar] Building jar: ant-jars/pluginc.jar
      [jar] Building jar: ant-jars/pluginc-src.jar

default:

BUILD SUCCESSFUL
Total time: 55 seconds
```

You can see the set of ant targets by running `ant -projecthelp`.
  * `ant runtests` will build and run tests
  * `ant clean` will wipe out all generated files, so you can rebuild from scratch

If you look at the directory structure after building, you'll see a
number of `ant-*` directories
  * `ant-lib` - compiled class files and resources
  * `ant-jars` - all the jars needed to run the Cajoler.  To run the Cajoler from a command-line, use the `bin/cajole_html` script.
  * `ant-reports/tests/index.html` - unit test status and logs
  * `ant-reports/coverage/index.html` - EMMA test coverage reports.  See `ant emma runtests`.
  * `ant-www` - demo output.  See `ant demos`
  * `ant-docs` - javadoc (and eventually jsdoc) output.

### Problems running Ant ###

If you see the following error when running ant, try upgrading to Ant 1.7:
```
google-caja/build.xml:144: java.lang.NoClassDefFoundError: org/apache/tools/ant/types/ResourceCollection
        at org.apache.tools.ant.IntrospectionHelper$Creator.create(IntrospectionHelper.java:1166)
        at org.apache.tools.ant.UnknownElement.handleChild(UnknownElement.java:549)
        at org.apache.tools.ant.UnknownElement.handleChildren(UnknownElement.java:326)
...
```

If you get a complaint about Java running out of heap memory, set `ANT_OPTS` to the additional option to be passed to your Java for increasing heap memory. For example, if you use bash and Sun's JDK, placing the following in your `~/.bashrc` file increase your maximum Java heap size to 300MB:

```
export ANT_OPTS=-Xmx300M
```

## Building Caja via Eclipse ##

Caja includes a script for creating a Eclipse project file.  Eclipse relies on some files created by the ant build. Once you have checked out the caja source, run ant as follows:

```
$ cd google-caja
$ ant
```

Check that your ANT\_HOME environment variable is set correctly.  If it isn't, set it. For example, if you use bash and your ant is installed at `/usr/share/ant`, place the following in your `~/.bashrc` file:

```
export ANT_HOME=/usr/share/ant
```

Now create an eclipse project:

```
$ ./tools/myvn eclipse
```

From inside eclipse, import the project from the File -> Import -> Existing Project into Workspace option.  Select the Caja checkout directory as the root directory and import the project.

Run the JUnit tests to ensure everything was checked out correctly.

## Code Layout ##

```
google-caja
|
+--docs : documentation files.
|
+--src : source code (java and javascript)
|  |
|  +--com
|     |
|     +--google
|        |
|        +--caja
|           |
|           +--lexer : Tokenization and escaping
|           |
|           +--parser : Parsers and tree implementations
|           |  |
|           |  +--ParseTreeNode.java : Main parse tree interface
|           |  |
|           |  +--quasiliteral : Syntactic sugar for parse tree xforms
|           |
|           +--opensocial : Dealing with Gadget specs
|           |
|           +--plugin : Transformations
|           |  |
|           |  +--PluginCompilerMain.java : main class
|           |  |
|           |  +--stages : Parse tree transforms
|           |
|           +--reporting : Error and warning messaging.
|
+--tests : test files and resources
```

## Other Resources ##

http://code.google.com/p/google-caja/ : The source code repository, bug
tracker, and wiki.

http://groups.google.com/group/google-caja-discuss/ : The public
discussion list.

[Cajadores.com](http://www.cajadores.com/) periodically generates test
and coverage status reports, and includes a snapshot of generated
code, including the
[Testbed applet](http://www.cajadores.com/demos/testbed/index.html) and other demos that can be linked to.