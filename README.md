# NAME

App::MediaWiki2Git - copy MediaWiki page history into a Git repository# SYNOPSIS

 # Set up
 mkdir pages
 cd pages
 git init
 git commit --allow-empty -m 'initial empty commit'
 printf "---\nmediawiki:\n  api_url: http://example.com/wiki/api.php\npages:\n  - MainPage\n" > mw2git.yaml
 git add mw2git.yaml
 git commit -m 'initial config'

 # Fetch pages
 mw2git

# DESCRIPTION

This is a workaround for the lack of an "annotate" (aka. "blame")
feature in the MediaWiki we use locally.

It operates using configuration in, and upon a Git repository at, the
current directory.  There are no options.



# CONFIGURATION

By default, it expects `mw2git.yaml` to exist in the current
directory.  Via the (Moose-y) OO interface, it can take configuration
from elsewhere.

This should contain one hash (dictionary), whose entries are used to
configure parts of this package.

## Keys used

- mediawiki

This is passed to L<MediaWiki::API/new> and should be another hash.

One entry for `api_url` should be enough.  This is probably
constructed by replacing the `index.php` in
"http://...server.../wiki/index.php" with `api.php` .

See [MediaWiki::API](http://search.cpan.org/perldoc?MediaWiki::API) for other options such as proxy control.

- rvlimit

The MediaWiki `api.php` limits the maximum number of revisions that
can be fetched in one query.  This is likely to be 50, 500 or 5000
depending on context.  This module uses 500 as the default.

- pages

This list defines the pages to be fetched.

TODO: we could populate 'pages' from a category list at the start of each run.

- dns_qual

(Optional, for local use only.)  This key is appended to unqualified
hostnames in `~/.ssh/ssh-config.yaml`, if you have that file.

- _page_revs

For internal use.  This hash of pagename to last fetched revision id
is used to avoid querying the api for previous page revisions.

It is the main reason why the configuration file must be rewritten and
committed along with the copies of the pages.

## Updating

The configuration is extended in-memory, (atomically) replaced on
disk, and committed as fetching progresses.

XXX: errors during a run can leave the config out of sync with the committed pages
so page revisions may get committed again.  One solution would be to
`reset --hard` to the last config save commit.  This could be
automated, at some cost to the principle of least surprise.

# OTHER COMPONENTS

## MediaWiki interface

This is used read-only and anonymously (assumes it does not need to
log in).

## Git interface

Uses [Git::Repository](http://search.cpan.org/perldoc?Git::Repository) to drive Git upon the current directory.
There is no configuration.

It is assumed that the previous requirement for the existence of the
configuration file is enough of a sanity check, to prevent messing
with any other Git repositories' history.

It currently performs only `git add` and `git commit` operations,
but might want to `git reset --hard` later.  This should probably
require permission from the configuration.

## Page tracking

Configuration lists the pages to fetch, and the last revision fetched
per page.

The Git author is constructed from the page information, including
some post-processing to attempt to improve the usefulness of anonymous
(IP address logged) edits.

The committer and commit timestamp are left to be picked up from the
environment.  This means that Git commitids will not be reproducible
between different runs of this code on the same page revisions.

## Hostname lookup

When users do not log in, we get their IP address.  When this is a web
proxy, we learn nothing; but in a company it is often a one-user
desktop machine.

We do a reverse lookup in the DNS (IPv4) to get a hostname.  Results
are cached during the run and errors are written out as warnings.

Beware that looking up historically-recorded IP addresses against the
current DNS is likely to generate surprises.

## Hostname to user lookup

You may safely ignore this part of the code.

If the custom username-to-hostname mapping is present, we include in
the "anonymous" author info the result of a lookup.

This is a mapping I maintain to generate ssh host aliases, to assist
with internal user support.  The tool using it is small and not (yet)
published.

# AUTHOR

Copyright (C) 2011 Genome Research Limited

Author Matthew Astley L<mca@sanger.ac.uk>

This library is free software; you can redistribute it and/or modify
it under the same terms as Perl itself.