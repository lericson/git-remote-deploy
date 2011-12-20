## Automatic Deploy with Git

![git-remot-deploy example](http://bsg.lericson.se/git-remote-deploy.gif)

Small and large teams alike, there is no question people want Git for deployment. I do not condone this as a method of release management. If your development team has any kind of resources, you should look into release tools like Capistrano, Phing, Fabric, or whatever the flavor of the month is.

For us, the 99%, there are dirt jobs, campaign sites, and quick hacks. These are better off with a simple, straight-forward solution. This is my take.

Let's get started!

    $ git clone https://github.com/lericson/git-remote-deploy ~/devel/src/git-remote-deploy
    $ cd some_project
    $ ~/git-remote-deploy/setup-repo
    [follow instructions]
    $ git push deploy
    BADABING!

### Retrying a Deploy

If something went wonky, you might want to attempt deploy again. To your distress you notice that because the ref didn't change, no hooks are run and no deploy is retried! Calm down. There are a couple of solutions, like pushing the previous `HEAD` in between:

    git push deploy HEAD^:deploy
    git push deploy

The more sensible thing to do, which doesn't deploy the previous version, is to simply delete the ref first:

    git push deploy :deploy
    git push deploy

### Manual setup

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
