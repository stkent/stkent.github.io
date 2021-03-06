---
title: Pushing To A Differently-Named Branch
author: Stuart Kent
tags: git

---

{% include kramdown_definitions.md %}

Sometimes it’s necessary to push a local git branch (for example, `main`) to a remote branch with a different name (for example, `master`).

In this situation, running `git push` will usually cause an error:

```text
fatal: The upstream branch of your current branch does not match
the name of your current branch.  To push to the upstream branch
on the remote, use

    git push origin HEAD:master
```

If you can, prefer solving this by renaming the remote branch to match your local branch so that `git push` works as expected again.

If you can’t rename the remote branch, you’ll probably get tired of typing out `git push origin HEAD:master` each time you push. I recommend setting the [git push strategy](https://git-scm.com/docs/git-config#Documentation/git-config.txt-pushdefault){:new_tab} for the affected repository to "upstream":

```bash
git config push.default upstream
```

The "upstream" strategy always pushes the current local branch to its upstream branch and does not check that the local and upstream branch names match. The shorter `git push` command will now work again.

One caveat: if you previously relied on `git push` automatically creating upstream branches for you, you’ll have to be a little bit more explicit when using the `upstream` push strategy:

```text
fatal: The current branch new-branch has no upstream branch.
To push the current branch and set the remote as upstream, use

    git push --set-upstream origin new-branch
```

I create branches much less frequently than I update them, so I consider this trade-off acceptable.
