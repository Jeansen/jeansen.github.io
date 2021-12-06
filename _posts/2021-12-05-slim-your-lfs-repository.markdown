---
layout: post
title: Slim your LFS repository
date: 2021-12-05T13:06:21+01:00
tags: [linux, git, lfs]
---


In a [previous post]({{ site.baseurl }}{% post_url 2021-11-24-bcrm-project-update.markdown %}) I wrote about my local test setup for [bcrm](https://github.com/jeansen/bcrm).
Because I have a lot of different images in use during testing, I wanted to be able to have version management in place. This would allow me to play around with
images, restore them as needed or simply jump back in time during testing.

So, I utilized LFS. But my repository grew quite big over time (almost one Terabyte). Being able to switch back and forth in time is nice, but I definitely do not
need images that are older than a year. And with recent updates to the latest Debian stable release, even more images became obsolete. 

The 100 million dollar question now was, how could I clean up my repository from all those old files (local and remote). As you might know [TIMTOWTDI!](https://en.wikipedia.org/wiki/There%27s_more_than_one_way_to_do_it). So, there is a solution that worked best for me.

## Time Travel

The one challenge with LFS is to clean your history. Because, if you simply remove untracked files, they will still exist on the server side. If you delete LFS files
on both ends, you will no longer be able to push simply because LFS complains of missing files. One quick option, after deleting all deprecated files from the
remote repository, could be to set `lfs.allowincompletepush`. This would allow you to push again, as well as clone the repository and checkout the latest working
tree. But every clone would have to set this flag. Additionally, the count of objects would not be correct (although this is a pure cosmetic problem).

## Cleanse the past

The solution above  works, but it is not clean enough to my liking. Instead of deleting old stuff on the server side by hand and pruning local objects, I decided 
to simply create a new repository. and push the latest state with a clean history to it. With "clean" I do not mean to lose my current history. Simply to create 
a state that looks "new" to git lfs.

Therefore, I first created a clean and fresh clone. This would also "smudge" the latest LFS images. With this state, I would have all normal git changes including
the latest blobs (images, no pointes).

I then went through the following steps to clean my history from within the clone folder:

    #Install filter-repo tool used later to clean the git history
    sudo apt install filter-rep

    #Get list of alle ever tracked files
    git lfs ls-files --all > /tmp/files

    #Uninstall LFS support
    git lfs uninstall

    #Clear the cache
    while read -r e; do git filter-repo --invert-paths --path $e; done < /tmp/files.txt

    #Temporarily move tracking information.
    mv  .gitattributes /tmp

    #Remove all files previously tracked by 'git lfs track'
    while read -r e; do git filter-repo --invert-paths --path $e; done < /tmp/files.txt

    #Install git LFS again and restore tracking information
    git lfs install
    mv /tmp/.gitattributes .

Now, I copied over all deleted files from my backup, added them to git (and by that also to LFS) and after a new commit finally pushed everything to a new repository.

There was no harm in running `git push` at this stage because `filter-repo` had removed the upstream and some other date from `.git/config`. After I set the new
upstream repository (and because my instance of gitea is configured to create non-existing repositories on-the-fly), I only had to push and wait.

After some time (and another cup of coffee), I finally had a new repository with all my normal git changes including a fresh start for LFS objects.


