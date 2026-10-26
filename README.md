# 1. Install Git and clone the repository
Each team member runs:
git --version
git clone https://github.com/patchguard/teamworktest.git
cd teamworktest
git status
git branch --show-current
Configure your identity if necessary:
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
Use your own email address. To push, authenticate through Git Credential Manager, GitHub CLI, or an SSH key; GitHub account passwords cannot be used for HTTPS Git pushes.

# 2. Start a task on a feature branch
Create or claim a GitHub Issue describing the work. Always start from an up-to-date main:
git switch main
git pull --ff-only origin main
git switch -c feature/42-add-homepage

Branch naming convention:
Prefix	Example	Use
feature/	feature/42-add-homepage	New functionality
fix/	fix/43-empty-input	Bug fix
docs/	docs/44-update-guide	Documentation
test/	test/45-add-unit-tests	Tests
chore/	chore/46-configure-ci	Maintenance


# 3. Make small commits and push your branch
Edit your files, then inspect and stage only the intended changes:
git status
git diff
git add README.md
git commit -m "docs: clarify contribution workflow"
git push -u origin feature/42-add-homepage
The staged files must match your actual work; replace README.md and the example branch name with your own. For later changes on the same branch:
git add path/to/changed-file
git commit -m "fix: handle empty input"
git push
Prefer focused commits. Never commit passwords, API keys, .env files, build artifacts, or unrelated local changes.

# 4. Open a Pull Request
1). Open https://github.com/patchguard/teamworktest/pulls.
2). Click New pull request (or Compare & pull request after pushing).
3). Select base: main and compare: your feature branch.
4). Write a clear title and description, and link the issue using Closes #42 if the PR completes it.
5). Request review from a teammate other than the author.
6). Wait for approval and any required checks. Do not merge your own PR unless the team has explicitly agreed on an exception.

# 5. Review a teammate's PR
Reviewers should inspect Files changed, verify the behavior, and check for correctness, readability, regressions, and security issues.

To test the branch locally:
git fetch origin
git switch --track origin/feature/42-add-homepage

Run the project's tests or start the application.
If you already have a local branch with that name, use git switch feature/42-add-homepage instead. After testing, return to your own branch or main.

On GitHub, choose Comment, Request changes, or Approve. If changes are requested, the author commits fixes to the same branch and pushes again; the PR updates automatically. Approval should apply to the latest changes.

# 6. Keep your branch synchronized with main
Other PRs may merge while you are working. Before requesting final review, update your feature branch:
git switch feature/42-add-homepage
git status  # Commit or stash your work before continuing.
git fetch origin
git merge origin/main
If Git reports a conflict, open the affected files. You might see:
<<<<<<< HEAD
<p>My feature text</p>
=======
<p>New text from main</p>
>>>>>>> origin/main
Decide which content to keep, or combine both. Remove all conflict markers and verify the result. Then:
git add path/to/conflicted-file
git commit -m "merge: sync main and resolve conflict"
git push
If Git reports that there is no merge in progress, the merge may have completed automatically and no extra commit is needed. Test again after resolving conflicts. Prefer fetch + merge for this beginner workflow because it makes the operation explicit.

# 7. Integration lead: merge and verify
Before merging, verify that:
- The PR targets main and has the required independent approval.
- All review comments are resolved and required CI checks pass.
- The PR is free of conflicts and does not contain secrets or unrelated changes.
- Any necessary manual test or preview has been checked.
Click Squash and merge (recommended for a tidy beginner-friendly history), then Delete branch. If your repository does not allow squash merges, use an allowed merge method agreed upon by the team.

Deployment is not automatic unless GitHub Pages or a deployment workflow has actually been configured. If deployment exists, check Actions and the live site after merging.

# 8. Start the next task
Every developer refreshes local main and creates a new branch:
git switch main
git pull --ff-only origin main
git branch -d feature/42-add-homepage
git switch -c feature/47-next-task
Delete the local branch only after it is no longer needed. If you used squash merge, git branch -d may refuse because the original branch commits are not ancestors of main; verify the PR was merged and the work is present before using git branch -D.

# 9. Tag a tested milestone
The integration lead can mark a known-good commit:
git switch main
git pull --ff-only origin main
git tag -a v1-integration -m "First tested integration"
git push origin v1-integration
Tags identify a commit for reference and recovery; they do not automatically deploy or roll back the website.

# 10. Recover from a bad merge
For a squash-merged PR, GitHub's Revert button may be available on the merged PR. If available, use it to create a revert PR, have it reviewed, and merge it. For other merge methods or when the button is unavailable, ask the integration lead to prepare a suitable git revert commit on a new branch and open a PR.

Do not use git reset --hard and force-push to rewrite shared main. Reverting a PR reverses its changes, but if later PRs depend on them, additional fixes may be required. Confirm CI and deployment after the revert.

# 11. Example: two developers working simultaneously
Assume Tom is building a new API helper and Amy is building the UI. These are illustrative file names, not existing files in this initially empty repository.

Tom:
git switch main
git pull --ff-only origin main
git switch -c feature/10-api-tom
mkdir -p js

Create and edit js/api.js
git add js/api.js
git commit -m "feat(api): add API helper"
git push -u origin feature/10-api-tom

Amy:
git switch main
git pull --ff-only origin main
git switch -c feature/11-ui-amy
mkdir -p js

Create and edit js/view.js
git add js/view.js
git commit -m "feat(ui): add view renderer"
git push -u origin feature/11-ui-amy
Both open PRs targeting main. A reviewer approves Tom's PR; the integration lead merges it. Amy then merges the latest origin/main into her own branch, resolves any shared-file conflicts, tests, and requests review. After approval, the integration lead merges Amy's PR. Neither developer needs to push to main.
