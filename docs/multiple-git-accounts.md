# Multiple git accounts (per-repo identity + SSH key)

Use a personal and a work account, even when both are on `github.com`, by giving each account its own SSH Host alias.

## 1. One SSH key per account

    ssh-keygen -t ed25519 -C "personal@example.com" -f ~/.ssh/id_ed25519_personal
    ssh-keygen -t ed25519 -C "work@example.com"     -f ~/.ssh/id_ed25519_work

Add each `.pub` to its own account (Settings > SSH and GPG keys). If the org enforces SSO, authorize the key for it.

## 2. `~/.ssh/config`: one Host alias per account

    Host github-personal
        HostName github.com
        User git
        IdentityFile ~/.ssh/id_ed25519_personal
        IdentitiesOnly yes

    Host github-work
        HostName github.com
        User git
        IdentityFile ~/.ssh/id_ed25519_work
        IdentitiesOnly yes

Test:

    ssh -T git@github-personal    # Hi <personal-user>!
    ssh -T git@github-work        # Hi <work-user>!

## 3. Use the alias in the remote URL

    git clone git@github-work:my-org/my-repo.git
    # existing clone
    git remote set-url origin git@github-work:my-org/my-repo.git

## 4. Set the commit identity per repo

    git config user.name  "Your Name"
    git config user.email "work@example.com"

Verify: `git config --show-origin user.email`

## 5. git-tools tokens

`git is-merged` / `git branch-cleanup` read a token named after the host in the `origin` URL:

    export GIT_TOOLS_TOKEN_GITHUB_PERSONAL="ghp_..."   # git@github-personal:...
    export GIT_TOOLS_TOKEN_GITHUB_WORK="ghp_..."       # git@github-work:...

## Troubleshooting

  * **`git pull`/`ssh` hangs on github.com:** some VPNs/MTUs drop the large post-quantum key exchange packets.
    Add `KexAlgorithms curve25519-sha256` to the Host blocks. Confirm with `ssh -v` (hangs after
    `expecting SSH2_MSG_KEX_ECDH_REPLY`).
  * HTTPS remotes ignore SSH keys; use SSH URLs.
  * Existing commits keep their old author; only new commits use the new identity.
