# git

```
$ git stash save "sso-mock users"
$ git stash list
$ git stash pop stash@{0}
```

```
$ git diff -U<n> - controls the amount of context lines shown around each change in a diff. It modifies the _unified diff_ output by including `<n>` unchanged lines before and after each modified block. In practice this affects how much surrounding code you see, not the actual changes.
```

# worktree

- `git worktree list` - list worktrees
- `git worktree remove ../project-feature-a` - remove worktree
- `git worktree add ../project-feature-a -b feature-a` - create a worktree with a new branch
- `git worktree add ../project-bugfix bugfix-123` - create a worktree with an existing branchg
- `git worktree prune` - cleans up the stale worktree references in `.git/worktrees/` after you manually delete the directory