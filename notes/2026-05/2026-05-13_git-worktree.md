# Git Worktree: Checkout an Existing Branch Into a Separate Directory

Need to work on two branches simultaneously without stashing or cloning? `git worktree` creates a separate working directory that shares the same `.git` — no duplicate objects, no wasted space.

```bash
git worktree add <path> <branch>
```

For example, to checkout `feature/login` into a sibling directory:

```bash
git worktree add ../my-feature feature/login
```

This creates `../my-feature` with the branch checked out and ready to work. Commits, branches, and remotes are shared — it's one repository, multiple workspaces.

Day-to-day operations:

- `git worktree list` — see all linked worktrees
- `git worktree remove <path>` — clean up when done

One restriction: the same branch cannot be checked out in two worktrees at once. Git enforces this to prevent conflicting writes to the ref.
