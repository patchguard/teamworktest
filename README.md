Teamwork Test — GitHub Collaboration Guide
Repository: https://github.com/patchguard/teamworktest

This repository is a practice space for collaborative software development using Git branches, Pull Requests (PRs), reviews, and protected main. The goal is to let multiple people work independently without accidentally breaking shared code.
Golden rule: Do not push directly to main. Make changes on your own feature branch and submit a PR. The integration lead merges approved work.

1. Team roles
Role	Responsibility
Repository owner / integration lead	Initialize the repository, configure protection, coordinate integration, merge approved PRs, and tag milestones.
Developer	Work on an assigned issue in a feature branch, test changes, and open a PR.
Reviewer	Read the diff, test when appropriate, request fixes, and approve only when ready.


One person may have multiple roles, but the PR author should not be their own reviewer.
2. One-time setup (integration lead)
The repository is initially empty. On GitHub, create a README.md on main (or make an initial commit locally and push it). After the first commit, enable branch protection before inviting the team to start work.

Recommended GitHub settings:
1. Open Settings → Collaborators (or Collaborators and teams) and invite team members. Give only the permissions they need.
2. Open Settings → Rules → Rulesets → New branch ruleset (or Settings → Branches → Add branch protection rule, if available).
3. Target the default branch main and enable Require a pull request before merging.
4. Require at least one approving review, dismiss stale approvals when new commits are pushed, and require conversation resolution before merging.
5. Disable force pushes and branch deletion for main. Do not allow bypasses except for a deliberate emergency policy.
6. Once CI checks exist, require those checks to pass before merging. Do not require nonexistent checks.
7. Optionally enable Automatically delete head branches in repository settings.
GitHub's enforcement options depend on repository ownership and plan. In particular, rules on a public repository owned by a personal account may require an eligible paid plan to be enforced. Verify enforcement by testing with a non-admin collaborator. If rules are unavailable, follow the PR policy voluntarily, but do not claim main is technically protected.

3. Install Git and clone the repository
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
4. Start a task on a feature branch
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


Use descriptive names, preferably including the issue number. Never do feature work directly on main.
5. Make small commits and push your branch
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
6. Open a Pull Request
1. Open https://github.com/patchguard/teamworktest/pulls.
2. Click New pull request (or Compare & pull request after pushing).
3. Select base: main and compare: your feature branch.
4. Write a clear title and description, and link the issue using Closes #42 if the PR completes it.
5. Request review from a teammate other than the author.
6. Wait for approval and any required checks. Do not merge your own PR unless the team has explicitly agreed on an exception.
Suggested PR description:
## Summary
- What changed?
- Why was it needed?

## Testing
- [ ] Tested locally
- [ ] Existing behavior still works
- [ ] No secrets or unrelated changes included

## Screenshots (if UI changes)
Attach screenshots or a short demo.

Closes #42
7. Review a teammate's PR
Reviewers should inspect Files changed, verify the behavior, and check for correctness, readability, regressions, and security issues.

To test the branch locally:
git fetch origin
git switch --track origin/feature/42-add-homepage
# Run the project's tests or start the application.
If you already have a local branch with that name, use git switch feature/42-add-homepage instead. After testing, return to your own branch or main.

On GitHub, choose Comment, Request changes, or Approve. If changes are requested, the author commits fixes to the same branch and pushes again; the PR updates automatically. Approval should apply to the latest changes.
8. Keep your branch synchronized with main
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
9. Integration lead: merge and verify
Before merging, verify that:
- The PR targets main and has the required independent approval.
- All review comments are resolved and required CI checks pass.
- The PR is free of conflicts and does not contain secrets or unrelated changes.
- Any necessary manual test or preview has been checked.
Click Squash and merge (recommended for a tidy beginner-friendly history), then Delete branch. If your repository does not allow squash merges, use an allowed merge method agreed upon by the team.

Deployment is not automatic unless GitHub Pages or a deployment workflow has actually been configured. If deployment exists, check Actions and the live site after merging.
10. Start the next task
Every developer refreshes local main and creates a new branch:
git switch main
git pull --ff-only origin main
git branch -d feature/42-add-homepage
git switch -c feature/47-next-task
Delete the local branch only after it is no longer needed. If you used squash merge, git branch -d may refuse because the original branch commits are not ancestors of main; verify the PR was merged and the work is present before using git branch -D.
11. Tag a tested milestone
The integration lead can mark a known-good commit:
git switch main
git pull --ff-only origin main
git tag -a v1-integration -m "First tested integration"
git push origin v1-integration
Tags identify a commit for reference and recovery; they do not automatically deploy or roll back the website.
12. Recover from a bad merge
For a squash-merged PR, GitHub's Revert button may be available on the merged PR. If available, use it to create a revert PR, have it reviewed, and merge it. For other merge methods or when the button is unavailable, ask the integration lead to prepare a suitable git revert commit on a new branch and open a PR.

Do not use git reset --hard and force-push to rewrite shared main. Reverting a PR reverses its changes, but if later PRs depend on them, additional fixes may be required. Confirm CI and deployment after the revert.
13. Example: two developers working simultaneously
Assume Tom is building a new API helper and Amy is building the UI. These are illustrative file names, not existing files in this initially empty repository.

Tom:
git switch main
git pull --ff-only origin main
git switch -c feature/10-api-tom
mkdir -p js
# Create and edit js/api.js
git add js/api.js
git commit -m "feat(api): add API helper"
git push -u origin feature/10-api-tom
Amy:
git switch main
git pull --ff-only origin main
git switch -c feature/11-ui-amy
mkdir -p js
# Create and edit js/view.js
git add js/view.js
git commit -m "feat(ui): add view renderer"
git push -u origin feature/11-ui-amy
Both open PRs targeting main. A reviewer approves Tom's PR; the integration lead merges it. Amy then merges the latest origin/main into her own branch, resolves any shared-file conflicts, tests, and requests review. After approval, the integration lead merges Amy's PR. Neither developer needs to push to main.
14. Quick reference
Goal	Command or action
Get latest main	git switch main && git pull --ff-only origin main
Create a branch	git switch -c feature/issue-description
See changes	git status && git diff
Commit	git add <file> && git commit -m "feat: description"
First push	git push -u origin <branch>
Later push	git push
Sync feature branch	git fetch origin && git merge origin/main
Request review	GitHub → Pull requests → New pull request
Merge	Integration lead merges an approved PR on GitHub
Tag release	git tag -a v1-integration -m "Milestone" && git push origin v1-integration


Team checklist
- [ ] I started from the latest main.
- [ ] I worked on my own branch.
- [ ] My commits are small and clearly named.
- [ ] I tested my changes.
- [ ] My PR explains the change and links its issue.
- [ ] Someone else reviewed the PR.
- [ ] The integration lead merged it after approval.
Remember: main stays stable; all development happens in feature branches; reviews happen before merging.
