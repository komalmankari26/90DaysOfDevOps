# Day 26 – GitHub CLI (gh): Manage GitHub from Your Terminal

## Objective

Learn how to use the GitHub CLI (`gh`) to perform GitHub operations directly from the terminal without opening a web browser. This improves productivity, automation, and DevOps workflows.

---

# What is GitHub CLI (gh)?

GitHub CLI (`gh`) is the official command-line tool developed by GitHub. It allows developers and DevOps engineers to interact with GitHub repositories, issues, pull requests, workflows, releases, and more directly from the terminal.

Official Website:
https://cli.github.com/

---

# Advantages of GitHub CLI

- No need to switch between browser and terminal
- Faster GitHub management
- Easy automation using shell scripts
- Perfect for CI/CD pipelines
- Supports GitHub Enterprise
- JSON output available for scripting

---

# Task 1 – Install & Authenticate

## Installation

Windows

```bash
winget install --id GitHub.cli
```

Verify installation

```bash
gh --version
```

Example Output

```
gh version 2.xx.x
```

---

## Login to GitHub

```bash
gh auth login
```

Authentication Steps

- Choose GitHub.com
- Choose HTTPS or SSH
- Login using browser
- Authorize GitHub CLI

---

## Verify Logged-in User

```bash
gh auth status
```

Example

```
Logged in to github.com as komalmankari26
Git operations protocol: SSH
```

---

## Authentication Methods Supported

GitHub CLI supports:

1. Browser-based authentication (Recommended)
2. Personal Access Token (PAT)
3. SSH authentication
4. GitHub Enterprise authentication

---

# Task 2 – Working with Repositories

## Create Repository

```bash
gh repo create demo-gh-cli \
--public \
--clone \
--add-readme
```

Creates

- Public repository
- README.md
- Clones automatically

---

## Clone Repository

```bash
gh repo clone owner/repository
```

Example

```bash
gh repo clone komalmankari26/devops-git-practice
```

---

## View Repository Details

```bash
gh repo view
```

Specific repository

```bash
gh repo view owner/repository
```

---

## List Your Repositories

```bash
gh repo list
```

Limit results

```bash
gh repo list --limit 10
```

---

## Open Repository in Browser

```bash
gh repo view --web
```

---

## Delete Repository

```bash
gh repo delete demo-gh-cli
```

GitHub CLI asks for confirmation before deleting.

---

# Task 3 – Issues

## Create Issue

```bash
gh issue create
```

Example

```bash
gh issue create \
--title "Bug in Login" \
--body "Login button not working" \
--label bug
```

---

## List Issues

```bash
gh issue list
```

Only open issues

```bash
gh issue list --state open
```

---

## View Issue

```bash
gh issue view 1
```

---

## Close Issue

```bash
gh issue close 1
```

---

## Observation

### How can `gh issue` be used in automation?

- Automatically create issues when CI fails
- Create issues from monitoring tools
- Generate bug reports automatically
- Close issues after successful deployment
- Integrate with shell scripts and GitHub Actions

---

# Task 4 – Pull Requests

## Create Branch

```bash
git checkout -b feature-readme
```

---

## Make Changes

```bash
git add .
git commit -m "Updated README"
git push -u origin feature-readme
```

---

## Create Pull Request

```bash
gh pr create --fill
```

or

```bash
gh pr create \
--title "Update README" \
--body "Improved project documentation"
```

---

## List PRs

```bash
gh pr list
```

---

## View PR Details

```bash
gh pr view
```

---

## Merge PR

```bash
gh pr merge
```

---

## Merge Methods Supported

GitHub CLI supports

### Merge Commit

```bash
gh pr merge --merge
```

Keeps complete commit history.

---

### Squash Merge

```bash
gh pr merge --squash
```

Combines all commits into one.

---

### Rebase Merge

```bash
gh pr merge --rebase
```

Maintains linear history without merge commits.

---

## Reviewing Someone Else's PR

Useful commands

```bash
gh pr checkout 10
```

Check out someone else's PR locally.

```bash
gh pr review 10 --approve
```

Approve PR.

```bash
gh pr review 10 --comment
```

Leave comments.

```bash
gh pr review 10 --request-changes
```

Request changes before merging.

---

# Task 5 – GitHub Actions

## List Workflow Runs

```bash
gh run list
```

---

## View Workflow Status

```bash
gh run view
```

Specific run

```bash
gh run view RUN_ID
```

---

## Observation

### How are `gh run` and `gh workflow` useful?

- Trigger workflows
- Monitor CI/CD pipeline
- Download workflow logs
- Debug failed builds
- Restart failed workflows
- Automate deployments
- Integrate with shell scripts

---

# Task 6 – Useful GitHub CLI Commands

## GitHub API

```bash
gh api user
```

Makes raw GitHub API requests.

---

## GitHub Gist

Create gist

```bash
gh gist create notes.txt
```

List gists

```bash
gh gist list
```

---

## Releases

Create release

```bash
gh release create v1.0
```

List releases

```bash
gh release list
```

---

## Aliases

Create shortcut

```bash
gh alias set co "pr checkout"
```

Use

```bash
gh co 15
```

---

## Search GitHub Repositories

```bash
gh search repos devops
```

Example

```bash
gh search repos kubernetes
```

---

# Useful Flags

```bash
--help
--repo owner/repo
--json
--web
--limit
--state
```

---

# Important Commands Summary

| Command | Description |
|----------|-------------|
| gh auth login | Login to GitHub |
| gh auth status | Check authentication |
| gh repo create | Create repository |
| gh repo clone | Clone repository |
| gh repo list | List repositories |
| gh repo view | View repository |
| gh repo delete | Delete repository |
| gh issue create | Create issue |
| gh issue list | List issues |
| gh issue close | Close issue |
| gh pr create | Create pull request |
| gh pr list | List pull requests |
| gh pr view | View PR |
| gh pr merge | Merge PR |
| gh run list | List workflow runs |
| gh run view | View workflow |
| gh api | GitHub API |
| gh gist | Manage Gists |
| gh release | Manage Releases |
| gh alias | Create shortcuts |
| gh search repos | Search repositories |

---

# Key Learnings

- GitHub CLI allows complete GitHub management from the terminal.
- Browser switching is minimized, improving productivity.
- It supports repository, issue, pull request, workflow, release, and API management.
- JSON output makes it ideal for automation and scripting.
- GitHub CLI is an essential tool for DevOps engineers working with CI/CD pipelines.

---

# Conclusion

Today I learned how to use GitHub CLI (`gh`) to manage repositories, issues, pull requests, GitHub Actions, releases, and API requests directly from the terminal. Using GitHub CLI improves productivity, simplifies automation, and is an essential skill for modern DevOps engineers.
