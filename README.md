Best Practice Configuration for develop Branch
✅ Require pull requests before merging
✅ Require code owner reviews (1 approval from code owners)
✅ Require status checks to pass (enforces CI/CD)
✅ Require signed commits (security)
✅ Dismiss stale reviews when new commits are pushed (keeps reviews current)
✅ Do NOT allow bypassing these settings (even admins must follow rules)
Step 1: Create a CODEOWNERS File

First, create a .github/CODEOWNERS file in your repository to define who owns which files:

Code
# Everyone owns the CODEOWNERS file itself
/.github/CODEOWNERS @fazal-ul-rehman66

# Define your code owners below
# Example:
# /src/ @fazal-ul-rehman66
# /docs/ @fazal-ul-rehman66
Step 2: Set Up Branch Protection

Go to your repository → Settings → Branches

Click Add rule

Enter branch name pattern: develop

Enable:

✅ Require a pull request before merging
✅ Require approvals (set to 1)
✅ Dismiss stale pull request approvals when new commits are pushed
✅ Require review from Code Owners
✅ Require status checks to pass before merging
✅ Require signed commits
✅ Do not allow bypassing the above settings
Click Create

Why This Configuration is "Perfect":

Setting	Why It's Best
Code Owner Reviews	Only designated owners can approve changes to their code
Status Checks	Automated tests must pass before merging
Signed Commits	Verifies commit authenticity and identity
Dismiss Stale Reviews	Forces re-review when code changes after approval
No Bypass Options	Ensures consistency—even admins follow the rules

Notes:
- Use team handles (e.g., @org/team) where possible instead of individuals.
- Make ownership granular for large repos (per-package or per-directory).

---

## Step 2 — Set up branch protection (UI)
Repository → Settings → Branches → Add rule

- Branch name pattern: `develop`
- Enable:
  - Require a pull request before merging
  - Require approvals (set to 1)
  - Dismiss stale pull request approvals when new commits are pushed
  - Require review from Code Owners
  - Require status checks to pass before merging — choose the exact CI check names (see checklist below)
  - Require signed commits
  - Do not allow bypassing the above settings (disable "Allow bypassing required pull request rules" and "Allow force pushes")
  - Optionally: Restrict who can push (select specific teams/users)

Click Create.

Important: When enabling status checks, pick the actual check names reported by your CI. Branch protection shows a list of checks only after the checks have run at least once on the branch or PR.

---

## Required status check names — examples
(Replace with your actual CI job names)
- `ci/build`
- `ci/test`
- `ci/lint`
- `security/snyk` (or another SCA job)

Add the exact names used by your CI; otherwise branch protection cannot require them.

---

## Automation examples

### Using `gh` (GitHub CLI)
Create a branch-protection rule with `gh` (replace placeholders):

```bash
gh api \
  repos/:owner/:repo/branches/develop/protection \
  -X PUT -f required_status_checks='{"strict":true,"contexts":["ci/build","ci/test"]}' \
  -f enforce_admins=true \
  -f required_pull_request_reviews='{"dismiss_stale_reviews":true,"require_code_owner_reviews":true,"required_approving_review_count":1}' \
  -f restrictions='null' \
  -f allow_force_pushes=false
Using Terraform (github_branch_protection)

Example resource snippet:
resource "github_branch_protection" "develop" {
  repository_id = "owner/repo"
  pattern       = "develop"

  required_status_checks {
    strict   = true
    contexts = ["ci/build", "ci/test", "ci/lint"]
  }

  required_pull_request_reviews {
    dismiss_stale_reviews          = true
    require_code_owner_reviews     = true
    required_approving_review_count = 1
  }

  enforce_admins = true
  require_signed_commits = true
  allow_force_pushes = false
}

Signed commits — guidance

GitHub supports verified commits signed with GPG, S/MIME, or SSH commit signing.
Instruct contributors to enable signing:
Local: git config --global user.signingkey <key-id> and git commit -S -m "message" or git config --global commit.gpgSign true.
Reference: GitHub Docs — "About commit signature verification" (see References).
Note: If contributors cannot sign, provide guidance or consider using CI to record author verification (but prefer requiring signatures).

Additional recommendations

Require linear history if you want to avoid merge commits.
Use "Require conversation resolution" if your repository benefits from resolving threads before merge.
Protect release/main branches with stricter rules (e.g., more approvals, release role).
Maintain a short contributor guide that explains the PR workflow and how to sign commits.
Troubleshooting & rollback checklist

If a protected branch blocks merges unexpectedly:

Verify the CI checks ran and their names match protection contexts.
Confirm code-owner approvals exist and are from the listed owners.
Ensure commits are signed and appear as “Verified” on GitHub.
Check whether "Include administrators" / enforce_admins is toggled — admins might be blocked.
If immediate unblock required, a repository admin can temporarily modify protection (audit and revert ASAP).
For automation errors, check GitHub API logs or Terraform plan/apply output.
FAQ

Q: What if a CI job name changes? A: Update the branch protection required contexts to match the new job name. Consider stable job names for CI pipelines.

Q: Can admins bypass rules? A: Avoid allowing bypass; prefer enforce_admins so admins follow the same path. Use temporary changes only with audit.

References
https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/customizing-your-repository/about-code-owners
Changelog

2026-08-30: Expanded README with automation examples, troubleshooting, and references.

Next steps I can take for you
- Commit this README.md update to a new branch and open a PR in your repo.
- Add a sample Terraform config or GH Actions workflow to produce the CI checks named above.
- Create a CONTRIBUTING.md explaining how to sign commits and run CI locally.
