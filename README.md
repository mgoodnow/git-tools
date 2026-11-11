# Git Tools

These are additional git command-line tools.

## How to add tools

This describes how to add tools for local usage.

 1. Checkout the repo

        git clone https://github.com/mgoodnow/git-tools.git

 2. Add the following to the path

        echo 'export PATH="$PATH:<your-path-here>/git-tools/commands"' >> ~/.zshenv

 3. Add the following environment variables for use by git tools.

        # One token per remote host, named GIT_TOOLS_TOKEN_<HOST>
        export GIT_TOOLS_TOKEN_GITHUB_COM="ghp_mytoken"
        export GIT_TOOLS_TOKEN_GITHUB_WORK="ghp_othertoken"

        # Optional
        export GIT_TOOLS_ORG="my-org"

  * `<HOST>` is the host exactly as written in the repo's `origin` URL, uppercased with non-alphanumerics
    as `_`. For `git@github.com:org/repo.git` it is `GITHUB_COM`; for an ssh alias such as
    `git@github-work:org/repo.git` it is `GITHUB_WORK`. Use an alias to give a second account on the same
    host its own token (see [docs/multiple-git-accounts.md](docs/multiple-git-accounts.md)).
  * The real hostname for the API is resolved from the alias via `ssh -G`. `github.com` uses
    `https://api.github.com`; other hosts use `https://<host>/api/v3`.
  * Get the token from the matching github account: [https://github.com/settings/tokens](https://github.com/settings/tokens)  
     If the org enforces SSO, authorize the token for it.
  * Set `GIT_TOOLS_ORG` to limit git's api search (this only works if all your work is out of a single github org).   
     _By setting this can speed up the API query a wee-bit._
  * Install PyGithub

        python3 -m pip install PyGithub

## Tool: git is-merged

This tool determines if there are any PRs associated with a branch and the status of them.

**Usage**

```
# Current branch
git is-merged

# Specific branch
git is-merged <branch>
```

**Output**

```
git is-merged
[CLOSED] https://github.com/my-org/project-name/pull/1 (merged at: 2022-11-16T17:48:55Z)

git is-merged another-branch
[NONE] There are no pull requests associated with: 2259c126530eec3e5b55ad77c2e314db0027ec23
```

*This is a tool used by `git branch-cleanup`*

Source: tool inspired by [https://github.com/larsks](https://github.com/larsks/github-tools/blob/master/git-is-merged)

## Tool: git branch-cleanup

This tool _safely_ deletes branches which where merged via a PR.

Because of the nature of `squash-and-merged`, cannot rely on git's ability to "safely" delete a branch.  
This tool provides that ability by determining if there is a CLOSED PR and if that PR was merged!  
If "yes" then we can **safely** delete _this_ branch.

**Usage**

```
git branch-cleanup -h
Usage: git branch-cleanup [-n --noop] [-v --verbose] [-h --help]
       -n --noop      Run in noop mode; will not cleanup any branches (default: false)
       -v --verbose   Print verbose output about branch PR details
       -h --help      Print this help
```

By default, will run cleanup - but if passed `--noop` will print out what it would do.

**Output**

```
git branch-cleanup -v

Keeping:  mgoodnow-test1 - [OPEN] https://github.com/my-org/my-project/pull/1 (merged at: Not merged)
Deleting: mgoodnow-test2 - [CLOSED] https://github.com/my-org/my-project/pull/2 (merged at: 2022-03-24T13:47:26Z)
Deleted branch mgoodnow-test2 (was 36f1b5e)
Keeping:  mgoodnow-test3 - [NONE] There are no pull requests associated with: 97bf18740251e6c497fde4bbbf8c1615880e9385
Ignoring: main
```

## Known Issues

1. For `branch-cleanup` it can only do 30 branches at a time, due to github's rate-limiting.

  * However, `search` rates get reset after 30 seconds so ...run again!
  * See [https://docs.github.com/en/enterprise-server@3.4/rest/rate-limit](https://docs.github.com/en/enterprise-server@3.4/rest/rate-limit)

        curl \
          -H "Accept: application/vnd.github+json" \
          -H "Authorization: Bearer ${GIT_TOOLS_TOKEN}" \
          https://git.corp.example.com/api/v3/rate_limit | jq '[.resources.search]'

        {
          "resources": {
            "search": {
              "limit": 30,          <=== limit
              "used": 0,
              "remaining": 30,
              "reset": 1668636278   <=== 30 secs from now
            },
            ...
