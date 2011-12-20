# Automatic Deploy with Git

![git-remot-deploy example](http://bsg.lericson.se/git-remote-deploy.gif)

Small and large teams alike, there is no question people want Git for deployment. This glorified Git post-receive hook unpacks whatever tree you push to a remote `deploy` branch.

Let's get started!

    $ git clone https://github.com/lericson/git-remote-deploy ~/devel/src/git-remote-deploy
    $ cd some_project
    $ ~/git-remote-deploy/setup-repo
    [follow instructions]
    $ git push deploy
    BADABING!

While not a wholesale solution for anything but the simplest of projects, it's a decent substrate for making more complicated deployment setups.

## What's in a deploy?

So, you push your `HEAD` to a certain branch and some funky "deploy" happens. What's a deploy?

All the hook does by default is to unpack the code you sent, but--and this is important--the hook doesn't check out your commit as-is, instead it creates a new commit from the same tree. This way, you get a neat and tidy release log:

    $ git log --oneline
    c7f21d6 Deployed commit 774b6f0
    e2a891e Deployed commit 774b6f0
    39ecb9a Deployed commit da822ad

There are other reasons for doing things this way, the bottom-line is that deployment is *always* linear in time: one deploy succeeds the next, whereas your development history is often not so.

## Manual setup

The setup script is a convenience, but it needs to use SCP to put the deploy
hook into place. For that reason it can be convenient to know how to replicate
what the script does.

1. Set up the remote

   Push to the branch named `deploy`, this is what the hook looks for. (Of course, nothing's stopping you from changing that name.)

   You probably want to push your local `master` to the remote `deploy`, and since it's a deploy repository you probably don't want to fetch from it with `git remote update` and always want to overwrite the remote branch, so follow these steps:

       git config remote.deploy.url https://megacorp.net/something.git
       git config remote.deploy.push +refs/heads/master:refs/heads/deploy

2. Copy the deploy hook

   You need a non-bare Git repository, that is one with a work tree and a `.git` subdirectory. Deployed code will end up in this work tree.

   Copy to `repo/.git/hooks/post-receive`, and *make sure it is executable* with `chmod +x post-receive`.

2. **PUSH!**

       git push deploy

## Retrying a Deploy

If something went wonky, you might want to attempt deploy again. To your distress you notice that because the ref didn't change, no hooks are run and no deploy is retried! Calm down. There are a couple of solutions, like pushing the previous `HEAD` in between:

    git push deploy HEAD^:deploy
    git push deploy

The more sensible thing to do, which doesn't deploy the previous version, is to simply delete the ref first:

    git push deploy :deploy
    git push deploy

## Customizing

Open up `src/hook` and go nuts -- there are helpful comments in there. The hook is spartan by intent: the aim is not to tell you how to do your job, but rather to help you get it done.

If you find that your project's architectural needs are outgrowing simple Git pushes, you might want to try [Capistrano](http://https://github.com/capistrano/capistrano/) or something like that.