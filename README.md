# GitHub Team Collaboration Workflow

This guide explains how multiple developers can collaborate safely using Git, GitHub Issues, feature branches, Pull Requests, code reviews, and protected branches.

Repository: https://github.com/patchguard/teamworktest

**Golden rule:** Never push directly to `main`. Each developer works on a separate branch and submits a Pull Request (PR) for review.

---

# 1. Install Git and Clone the Repository

Each team member runs:

##-----------------------------------test step 1 start-------------------------------------------
```bash
git --version
git clone https://github.com/patchguard/teamworktest.git
cd teamworktest
git status
git branch --show-current
```
##-----------------------------------test step 1 end-------------------------------------------

Configure your identity if necessary:

```bash
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
```

Use your own email address.

To push changes, authenticate through Git Credential Manager, GitHub CLI, or an SSH key. GitHub account passwords cannot be used for HTTPS Git pushes.

> **Note:** If the repository is still empty, the owner must create the initial commit on `main` before team members can create feature branches using this workflow.

---

# 2. Start a Task on a Feature Branch

Create or claim a GitHub Issue describing the work.

Always start from an up-to-date `main`:

##-----------------------------------test step 2 start-------------------------------------------

```bash
git switch main
git pull --ff-only origin main
git switch -c feature/42-add-homepage
```
##-----------------------------------test step 2 end-------------------------------------------

## Branch Naming Convention

| Prefix | Example | Use |
|---|---|---|
| `feature/` | `feature/42-add-homepage` | New functionality |
| `fix/` | `fix/43-empty-input` | Bug fix |
| `docs/` | `docs/44-update-guide` | Documentation |
| `test/` | `test/45-add-unit-tests` | Tests |
| `chore/` | `chore/46-configure-ci` | Maintenance |

Each developer should use a separate branch for each task.

**Do not develop directly on `main`.**

---

# 3. Make Small Commits and Push Your Branch

Edit your files, then inspect and stage only the intended changes:

##-----------------------------------test step 3 start-------------------------------------------
{dirname} is fixed folder name, you need to replace it as you owned folder.
For example, hwd is one folder owned by hwd

```bash
mkdir {dirname}
cd {dirname}
echo "test" > README.md
git add README.md
git commit -m "docs: clarify contribution workflow"
git push -u origin feature/42-add-homepage
```
##-----------------------------------test step 3 end-------------------------------------------

The staged files must match your actual work. Replace `README.md` and the example branch name with your own.

For later changes on the same branch:

```bash
git add path/to/changed-file
git commit -m "fix: handle empty input"
git push
```

Prefer focused commits.

Never commit:

- Passwords or API keys
- `.env` files containing secrets
- Build artifacts that should be ignored
- Unrelated local changes

---

# 4. Open a Pull Request

1. Open https://github.com/patchguard/teamworktest/pulls.
2. Click **New pull request** (or **Compare & pull request** after pushing).
3. Select:
   - **Base:** `main`
   - **Compare:** your feature branch
4. Write a clear title and description.
5. Link the relevant Issue using `Closes #42` if the PR completes it.
6. Request review from a teammate other than the author.
7. Wait for approval and any required checks.

Do not merge your own PR unless the team has explicitly agreed on an exception.

## Example Pull Request Description

```text
Title: Add homepage

Summary:
- Add homepage layout
- Add navigation
- Add basic styling

Testing:
- Opened homepage locally
- Verified navigation links
- Checked browser console

Closes #42
```

---

# 5. Review a Teammate's PR

Reviewers should inspect **Files changed**, verify the behavior, and check for:

- Correctness
- Readability
- Regressions
- Security issues
- Unrelated modifications

## Test the Branch Locally

```bash
git fetch origin
git switch --track origin/feature/42-add-homepage
```

Run the project's tests or start the application.

If you already have a local branch with that name, use:

```bash
git switch feature/42-add-homepage
```

After testing, return to your own branch or `main`.

## Review Actions

On GitHub, choose one of:

- **Comment:** Provide feedback without approving or blocking.
- **Request changes:** Require modifications before approval.
- **Approve:** Accept the proposed changes.

If changes are requested, the author commits fixes to the same branch and pushes again.

The PR updates automatically.

Approval should apply to the latest changes.

---

# 6. Keep Your Branch Synchronized with Main

Other PRs may merge while you are working.

Before requesting final review, update your feature branch:

```bash
git switch feature/42-add-homepage
git status
git fetch origin
git merge origin/main
```

Commit or stash uncommitted work before merging.

## Resolve Merge Conflicts

If Git reports a conflict, open the affected files.

You might see:

```text
<<<<<<< HEAD
<p>My feature text</p>
=======
<p>New text from main</p>
>>>>>>> origin/main
```

Decide which content to keep, or combine both.

Remove all conflict markers and verify the result.

Then run:

```bash
git add path/to/conflicted-file
git commit -m "merge: sync main and resolve conflict"
git push
```

If Git reports that there is no merge in progress, the merge may have completed automatically and no extra commit is needed.

Test again after resolving conflicts.

For this beginner workflow, prefer `git fetch` followed by `git merge` because the operation is explicit.

---

# 7. Integration Lead: Merge and Verify

Before merging, verify that:

- The PR targets `main`.
- The PR has the required independent approval.
- All review comments are resolved.
- Required CI checks pass.
- The PR is free of conflicts.
- No secrets or unrelated changes are included.
- Necessary manual tests or previews have been checked.

## Merge the Pull Request

On GitHub:

1. Open the approved PR.
2. Click **Squash and merge**.
3. Confirm the merge.
4. Click **Delete branch** if the branch is no longer needed.

**Squash and merge** is recommended for a tidy, beginner-friendly history.

If the repository does not allow squash merges, use an allowed merge method agreed upon by the team.

## Verify After Merging

Check:

- The PR shows **Merged**.
- The changes appear on `main`.
- Required GitHub Actions completed successfully.
- The application works as expected.

Deployment is not automatic unless GitHub Pages or a deployment workflow has actually been configured.

If deployment exists, check GitHub Actions and the live site after merging.

---

# 8. Start the Next Task

Every developer refreshes local `main` and creates a new branch:

```bash
git switch main
git pull --ff-only origin main
git branch -d feature/42-add-homepage
git switch -c feature/47-next-task
```

Delete the local branch only after it is no longer needed.

If you used squash merge, `git branch -d` may refuse because the original branch commits are not ancestors of `main`.

Verify that the PR was merged and the work is present before using:

```bash
git branch -D feature/42-add-homepage
```

**Warning:** `git branch -D` force-deletes the local branch, even if it contains unmerged commits.

---

# 9. Tag a Tested Milestone

The integration lead can mark a known-good commit:

```bash
git switch main
git pull --ff-only origin main
git tag -a v1-integration -m "First tested integration"
git push origin v1-integration
```

Tags identify a commit for reference and recovery.

They do not automatically deploy or roll back the website.

---

# 10. Recover from a Bad Merge

For a squash-merged PR, GitHub's **Revert** button may be available on the merged PR.

If available:

1. Open the merged PR.
2. Click **Revert**.
3. Create a revert PR.
4. Request review.
5. Merge the approved revert PR.

For other merge methods, or when the button is unavailable, ask the integration lead to prepare a suitable `git revert` commit on a new branch and open a PR.

**Do not rewrite shared `main` using `git reset --hard` and force-push.**

Reverting a PR reverses its changes, but if later PRs depend on them, additional fixes may be required.

Confirm CI and deployment after the revert.

---

# 11. Example: Two Developers Working Simultaneously

Assume:

- **Tom** is building a new API helper.
- **Amy** is building the user interface.
- Both developers work on separate branches.
- Both developers submit Pull Requests targeting `main`.
- An integration lead merges approved PRs.

These are illustrative file names, not existing files in this initially empty repository.

## Developer 1: Tom

Tom creates a branch for the API helper:

```bash
git switch main
git pull --ff-only origin main
git switch -c feature/10-api-tom
mkdir -p js
```

Tom creates and edits:

```text
js/api.js
```

For example, Tom might write:

```javascript
export function getGreeting(name) {
    return `Hello, ${name}!`;
}
```

Tom commits and pushes his changes:

```bash
git add js/api.js
git commit -m "feat(api): add API helper"
git push -u origin feature/10-api-tom
```

Tom opens a Pull Request:

```text
Base:    main
Compare: feature/10-api-tom
Title:   Add API helper
```

Tom requests a review.

## Developer 2: Amy

At the same time, Amy creates a different branch:

```bash
git switch main
git pull --ff-only origin main
git switch -c feature/11-ui-amy
mkdir -p js
```

Amy creates and edits:

```text
js/view.js
```

For example, Amy might write:

```javascript
export function renderGreeting(message) {
    const element = document.createElement("p");
    element.textContent = message;
    document.body.appendChild(element);
}
```

Amy commits and pushes her changes:

```bash
git add js/view.js
git commit -m "feat(ui): add view renderer"
git push -u origin feature/11-ui-amy
```

Amy opens another Pull Request:

```text
Base:    main
Compare: feature/11-ui-amy
Title:   Add view renderer
```

## Integration Step 1: Merge Tom's PR

A reviewer approves Tom's PR.

The integration lead merges Tom's PR into `main`.

The repository now contains:

```text
main
 |
 +-- README.md
 |
 +-- js/
      |
      +-- api.js
```

Amy's branch is still independent.

## Integration Step 2: Amy Updates Her Branch

Amy incorporates the latest changes from `main`:

```bash
git switch feature/11-ui-amy
git fetch origin
git merge origin/main
```

If there are no conflicts, Git completes the merge automatically.

If there are conflicts, Amy resolves them and completes the merge.

Amy then pushes the updated branch:

```bash
git push
```

Amy tests her changes and requests final review.

## Integration Step 3: Merge Amy's PR

After approval, the integration lead merges Amy's PR.

The repository now contains:

```text
main
 |
 +-- README.md
 |
 +-- js/
      |
      +-- api.js
      |
      +-- view.js
```

Both developers contributed without pushing directly to `main`.

## Final Workflow

```text
                    main
                     |
          +----------+----------+
          |                     |
          v                     v
  feature/10-api-tom    feature/11-ui-amy
          |                     |
          v                     v
     Tom commits           Amy commits
          |                     |
          v                     v
       Tom PR                Amy PR
          |                     |
          v                     |
     Code review                |
          |                     |
          v                     |
      Merge Tom                 |
          |                     |
          +---------------------+
                                |
                                v
                       Amy syncs main
                                |
                                v
                           Code review
                                |
                                v
                            Merge Amy
                                |
                                v
                         Updated main
```

## Key Lessons

1. Each developer uses a separate feature branch.
2. Developers never push directly to `main`.
3. Every change is reviewed through a Pull Request.
4. Feature branches can be developed in parallel.
5. Developers synchronize with `main` when necessary.
6. Merge conflicts are resolved on feature branches.
7. The integration lead merges approved changes.
8. The shared `main` branch remains the team's integration point.

---

# Quick Reference

| Task | Command |
|---|---|
| Check status | `git status` |
| Show current branch | `git branch --show-current` |
| Switch to main | `git switch main` |
| Update main | `git pull --ff-only origin main` |
| Create feature branch | `git switch -c feature/my-task` |
| View changes | `git diff` |
| Stage changes | `git add path/to/file` |
| Commit | `git commit -m "description"` |
| Push new branch | `git push -u origin feature/my-task` |
| Push subsequent commits | `git push` |
| Fetch remote changes | `git fetch origin` |
| Merge main into feature | `git merge origin/main` |
| List branches | `git branch -a` |
| View history | `git log --oneline --graph --all` |

---

# Team Collaboration Checklist

Before starting work:

- [ ] Create or claim a GitHub Issue.
- [ ] Update local `main`.
- [ ] Create a dedicated feature branch.

Before opening a PR:

- [ ] Commit only relevant changes.
- [ ] Push the feature branch.
- [ ] Test the changes locally.
- [ ] Write a clear PR description.
- [ ] Request an independent review.

Before merging:

- [ ] Confirm required approvals.
- [ ] Resolve review comments.
- [ ] Confirm CI checks pass.
- [ ] Resolve merge conflicts.
- [ ] Verify no secrets are included.

After merging:

- [ ] Confirm the changes are on `main`.
- [ ] Verify tests and deployment, if applicable.
- [ ] Delete branches that are no longer needed.
- [ ] Update local `main` before starting the next task.
