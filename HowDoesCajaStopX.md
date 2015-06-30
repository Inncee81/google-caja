# How Does Caja Stop x #

During the security review there were a lot of questions of the form "How does Caja stop x?".  Below we answer that question for select x.

## How Does Caja Stop Gadgets from Creating `<iframe>`s? ##
The tamed `document` object (see `domita.js`) intercepts calls to `document.createElement` and denies any elements not on a whitelist (see `html4-defs.js`).  `"iframe"` is not on that whitelist.