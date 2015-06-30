# Introduction #

# Directory structure #

Use the following directory structure for your changes:

```
 svn-changes
     |
     +----my-change-1
     |        |
     |        +----google-caja
     |                 |
     |                 +----src
     |
     +----my-change-2
              |
              +----google-caja
                       |
                       +----src
```

In this way, the myvn script you will use for creating and submitting changelists can provide a changelist name for you for common commands.  For example, `myvn change` will behave like `gvn change -c my-change-1` if run under `svn-changes/my-change-1/google-caja` as will other gvn commands like `mail`, `describe` and `snapshot` unless a changelist name is explicitly provided.

# Life of a Changelist #

  1. Create a client directory
```
$ mkdir -p svn-changes/my-change-name
```
  1. Checkout the src
```
$ cd svn-changes/my-change-name
$ myvn checkout
$ cd google-caja/
```
  1. Muck around with some source files
```
      See Eclipse steps below
```
  1. Prepare a changelist
```
$ myvn change
```
> > Should pop up an editor.  Check the `EDITOR`, or `SVNEDITOR` environment
> > variables if it doesn't. This is a purely local operation.
  1. Send it off for review
```
$ myvn mail
```
> > This will upload to codereview.appspot.com, send mail to the reviewers specified using `myvn change`, and automatically CC google-caja-discuss (or, if the change's Private flag is set, then it will instead be CC'd to caja-discuss-undisclosed and the core Caja team).
  1. Update your changes based on feedback and snapshot for review
```
$ myvn snapshot
```
  1. Maybe add or remove files
```
$ myvn change
```
  1. Commit the change
```
$ myvn submit
```

> This will also cause your change to be tested before submission. You'll need a recent Firefox to run the tests.

> Submitting a change places it in the public repository regardless of whether the Private flag is set. Security patches should **not** be submitted until they are to be publicly disclosed.
  1. Show the change description
```
$ myvn describe
```
  1. Delete the change branch
```
$ myvn reset
```

# Eclipse #

  1. Generate the project
```
$ myvn eclipse
```
  1. Open Eclipse
  1. Choose New Java Project
  1. From Existing Source
  1. Enter the .../svn-changes/my-change-1/google-caja/ path
  1. Finish
  1. Build some stuff
```
$ ant
```
  1. Test stuff
> > Right click on project and select "Run As" > "JUnit Test"
```
$ ant runtests
```

# Other Commands #

  * `changed`      -  files changed in the snapshotted CL.  Pipeable to xargs
  * `diffstats`    -  number of lines added/changed/removed
  * `files`        -  files changed in svn.  Pipeable to xargs
  * `filetypes`    -  sets `svn:mime-type` of modified files based on file suffix

# Legal #

See also the Contributor Licence Agreement (either the
[individual CLA](http://code.google.com/legal/individual-cla-v1.0.html)
or the [corporate CLA](http://code.google.com/legal/corporate-cla-v1.0.html)).