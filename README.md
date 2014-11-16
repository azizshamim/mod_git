
mod_git is an Apache module which serves files from a (possibly bare) git repo.  The client specifies, via cookie or query parameter, which commit to use, and Apache serves the version of the file from that commit.

Configuration is simple.  Include the module by

   LoadModule git_module path/to/mod_git.so
   
and activate it by specifying, in either a directory or virtual server

   SetHandler git

The document root or directory context is assumed to be the git repository, and by default, the working copy will be served -- just as if `mod_git` were not configured.

To change the default version being served, specify

   DefaultVursion branch-or-tag

(Yes, Vursion is spelled with a 'U').  Any valid git commitish will work as the branch-or-tag.

To use from a browser, navigate to the configured directory.  The files you will see will be from the specified DefaultVursion (or `master` if it was not specified).  To see a different version, append the query parameter `vursion=desired-tag` to the URL.  This should have the following effect:

1.  The file served will be from the specified tag (or branch or commitish).
2.  A cookie will be set (`git-tag`) containing the value of the tag requested.
3.  A cookie will be set (`git-commit`) containing the git commit hash of the actual commit that the tag (or branch or commitish) resolved to.  This will be used to serve subsequent requests from the same commit.  This is required, for example, if requesting an HTML page which loads various stylesheets and javascript files.  By setting this cookie, all the subsequent requests will be served from the same commit -- providing a consistently rendered page.

Specifying a value of "-" (or an empty value) will serve the checked out working version from the file system -- as if `git` were not being used.

Tags like `vursion=testing@{3 hours ago}` should work.

A common use case is to have a github hook fire which causes your web server to `git fetch` every time you push to github.  That way, your deployment strategy is "push to github" -- and your changes are visible on your website -- if you are looking at the right branch.  As part of their user profile, users have a "default version" setting -- which sets their `git-tag` cookie to track the appropriate branch (`prod` or `testing` or `beta`).  Each user sees the branch they are tracking.

This is most useful for applications that use Angular or React or other client side frameworks -- such that most of the changes are made to files which are served by Apache.  CGI scripts, for example will not be versioned through this module, since they must be handled by `mod_cgi` instead of `mod_git`.

`mod_git` will set `ETag` to the git hash -- since that should uniquely identify a particularly version, and the `Last-Modified` time will be the time of the commit.

Building
========

You will need `libgit2` (and `apache` of course) installed.

