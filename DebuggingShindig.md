  1. Check Shindig out.
  1. Build it:
```
mvn clean
mvn
```
  1. Run it:
```
mvn -Prun
```
  1. Once it runs you should see which port it puts itself on (usually 8080).
  1. Browse to http://localhost:8080/gadgets/files/samplecontainer/samplecontainer.html and make sure gadgets work at all.
  1. If that all works ok, then check Caja out.
  1. Each time you modify Caja, build:
```
ant jars
```
  1. Copy Caja into Shindig's Maven repository
```
cp ant-jars/pluginc.jar ~/.m2/repository/caja/caja/r3164/caja-r3164.jar 
```
  1. Rebuild and rerun Shindig:
```
mvn && mvn -Prun
```
> or, optionally, without tests (much faster, but still slow)
```
mvn -Dmaven.test.skip=true && mvn -Prun
```
> or, in offline mode (reasonably fast)
```
mvn -o -Dmaven.test.skip=true && mvn -o -Prun
```
> Just this step is sufficient if all you have changed is Shindig.
  1. Shift-reload the gadget container in your browser - a simple reload doesn't always seem to get your updates.
  1. It also helps to uncheck "Cache".
  1. On the browser, it can be helpful to open the gadget frame in a new tab, add `&debug=1` to the URL (before the `#`) and shift-reload.
  1. With luck, you will now be fully equipped for debugging at all levels.

# Mercurialising Shindig #

Sometimes it might be desirable for several people to work on a checked out version of Shindig to create a patch, rather than just one. Here's how...

  * One person creates a Mercurial repo from the appropriate version of Shindig. In order to avoid merging issues with the Subversion files, don't include them in the Mercurial repo. In the Shindig directory
```
hg init
hg add -X 'glob:**/.svn' -X 'glob:.svn'
hg ci
```
> In the checkin comment, note the Subversion version it was created from. This may come in handy later!
  * Other participants clone the Mercurial repo...
```
hg clone <path to repo> <new directory>
```
  * When any participant has a working change, they first sync with the main repo...
```
hg pull <path to repo>
hg update
```
> then push their own
```
hg ci
hg push <path to repo>
```
> Note that there will be no email notifications, so other participants need to be told to do a pull/update.
  * To produce the Shindig diff, either the original producer of the repo, who still has a Subversion checkout, can do an update (he won't need to do a pull, because it is his repo that everyone is pushing to) and then carry on in the normal way, or you can check out a fresh copy, pull the Mercurial repo over the top of it, and carry on [[TBD](TBD.md)].

  * Mercurial maintains a record of individual changesets that it has applied and lets you backout the last changeset.  This can sometimes be helpful if you have applied a changeset in error and no one has yet submitted another change.  First backout the latest change
```
  hg backout
```
> then apply a new patch and resolve any merge issues
```
  patch -p0 < yourpatch.patch
  hg merge
```
> Once all merge issues are resolved, test your checkout and if everything looks good, commit and push your changes.