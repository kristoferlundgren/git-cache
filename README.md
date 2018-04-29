git-cache
=========

Context
-------

If you repeatedly clone a Git repository (for instance, for software you use to hack) you could annoyed by how long
you have to wait for the clone operation, especially for large remote git repositories.
On the other hand, if you have multiple copies of a Git repository on your computer, it can be a waste of space that
you would prefer not to waste.

Git has a feature "--reference"/alternates, which allows you to reference commits from another local repo on the same
computer.

An example use case is:

    $ cd ~/test
    $ git clone https://example.org/git/repo.git normal-repo
      (you wait minutes or hours, and the resulting directory is big)
    $ git clone --reference ~/test/repo https://example.org/git/repo.git referenced-repo
      (you wait seconds, and the resulting directory is small - only the size of the files, no size used by the history)

Both clones can access the whole history of the Git repository.
Internally, the second clone contains a file `.git/objects/info/alternates` containing the path of the first repo, which
tells git to search the referenced `alternates` repo for any commits that can't be found in the current repo.
Obviously, if you delete the referenced (first) repo, the second copy is useless and will display tonnes of errors.
Newer versions of git have a '--dissociate' feature that allows the copy to be independent from the first.


git-cache
---------

`git-cache` creates a central cache directory on a computer, which contains most of the commits of regularly cloned Git
repositories.

If you are a regular hacker of a software, say MediaWiki, you can cache most commits and then only download the most
recent ones.

Example use case:

    $ sudo git cache init  # cache is located in /var/cache/git-cache
    $ git cache add linux https://git.kernel.org/cgit/linux/kernel/git/torvalds/linux.git

      You can now use the cache directory with:

          git cache clone linux

      Or clone a similiar fork:

          git cache clone git://git.kernel.org/pub/scm/linux/kernel/git/rt/linux-stable-rt.git


Usage
-----

__Installation:__
Copy `git-cache` into your git commands directory (e.g. Ubuntu: /usr/lib/git-core) and make sure it is executable by all
users (mode 0755).

__Usage:__
All commands have the format:

    git cache ACTION [SUB-ACTION] PARAMETERS

__Commands:__

    # General maintenance commands
    git cache init [DIR] [TYPE]         initialise the cache directory; TYPE=local|global (default: local)
    git cache delete --force            delete the cache directory

    # Daily commands
    git cache add NAME URL              add a cached git repository
    git cache rm --force NAME           remove a cached git repository
    git cache show                      show all cached git repositories
    git cache update                    fetch all cached git repository
    git cache clone URL/NAME [DIR]      dissociated clone using cache (safe to delete cache or move off machine)
    git cache refclone URL/NAME [DIR]   clone using cache keeping references to cache (minimizes disk space, unsafe to move)

    clone commands accept either the name of an already cached git repository or an arbitrary remote URL
    cloning forks will still result in a signifigant speedup, any objects found in cache will be used rather than downloaded

    (Any other command will be applied to the cache directory.)
     e.g. 'git cache gc' or 'git cache remote show'.)

__Location of the cache directory:__
The default cache directory contains all cached repositories (each Git repository is a remote).
If it is created by the `root` user, the cache directory is `/var/cache/git-cache`, otherwise it is `~/.cache/git-cache`;
it can also be configured to use another directory if you specify this with the `init` command.

__Note:__ This directory could become __big__, be sure you have enough space.

~~You may want to create a cron-job to run `git cache update` to automatically retrieve new commits.~~ (bug)


Development
-----------

This is an updated version by [mexisme](https://github.com/mexisme):
- fairly substantially refactored/restructured
  - moved code into functions
  - used case-blocks (instead of if-elseif trees)
- commands rearranged
- a few new commands
  - a `remote` super-command
  - a `clone` command.
- some bugs fixed
- README language slightly updated

TODO:
- [ ] Rethink some of the sub-commands and names; it would help if they matched common git usages/conventions
- [ ] Figure-out a faster way to do the intial copy-into-cache with a large cache
- [ ] Port away from Bash/Shell; it's a pain to deal effectively with certain types of failures
  - Besides C, it looks like `sh` (*not* `bash`) and `perl` are directly expected/supported by git. There's a single reference to Ruby
  - I'd prefer Ruby, Rust or Go, but Perl wouldn't be a problem
- [ ] Does it make sense to auto-gc the cache?
- [ ] Does it make sense to auto-update the cache contents? How can we make sure to only do this over cheap/fast links?  


__From the original README:__

This is a first version, and feedback is needed to improve its daily usage. Some questions I wonder:
- Are the subcommands sufficiently explicit?
- Should we regularly run `git gc` or even `git gc --aggressive`?
- How git behaves when some commits are both local and in the cache? Does it remove local objects to gain place?
- etc.

Internally the cache directory is a big bare directory containing all remote repositories, even if they do not share
commits (sort of orphans branches).
Should the cache directory be splitted by repository: /var/cache/git-cache/mediawiki, /var/cache/git-cache/visualeditor,
etc.
Particularly in the second case, the name of the remote repository is useless, and I would prefer use unique names,
e.g. md5(URL) and completely hide this to the user (possibly with soft links from the URL to the directory for usability
inside the cache directory).


Licence
-------

- Original author: [Seb35](https://github.com/Seb35)
- Licence: [WTFPL 2.0](http://www.wpfpl.net)


References
----------

This program is a generalisation to arbitrary Git repositories of an idea and implementation by [Randy Fay](http://randyfay.com/content/reference-cache-repositories-speed-clones-git-clone-reference) specifically for Drupal. Hopefully this generalisation is sufficiently simple to stay useable and practical.

Similar program: [git-cached by dvessel](https://github.com/dvessel/git-cached)
