# Fixing Commit Author/Committer Email (Git)

This guide explains the simplest way to rewrite past commits so they use the correct author/committer name and email.

## Full Steps (Mailmap Method)

1. **Create a backup branch**
   ```bash
   git branch backup/before-rewrite
   ```

2. **Install or update git-filter-repo**
   ```bash
   pip install -U git-filter-repo
   ```

3. **Create `.mailmap` to map WRONG → RIGHT**
   Minimal form:
   ```bash
   cat > .mailmap <<'EOF'
   RIGHT_NAME <RIGHT_EMAIL> <WRONG_EMAIL>
   EOF
   ```
   Use one of these forms per line as needed:
   ```
   RIGHT_NAME <RIGHT_EMAIL> <WRONG_EMAIL>
   RIGHT_NAME <RIGHT_EMAIL> OLD_NAME <WRONG_EMAIL>
   RIGHT_NAME <RIGHT_EMAIL> OLD_NAME
   ```

4. **Rewrite history using the mailmap**
   ```bash
   git filter-repo --force --mailmap .mailmap
   ```

5. **Re-add the remote (if removed) and push**
   ```bash
   git remote add origin <REPO_URL>    # add if origin is missing
   git fetch origin                    # refresh leases
   git push --force-with-lease origin <BRANCH>
   # If you also rewrote tags:
   git push --force-with-lease origin --tags
   ```

### Replace These Placeholders

- `RIGHT_NAME` → your canonical name (e.g., Jane Doe)
- `RIGHT_EMAIL` → your canonical email (e.g., jane@example.com)
- `WRONG_EMAIL` → the incorrect email to replace
- `<REPO_URL>` → your repository URL (HTTPS or SSH)
- `<BRANCH>` → the branch you rewrote (often the default branch, e.g., `main`)

## Verify

**Local check**
```bash
git log -1 --format='author:%an <%ae>  committer:%cn <%ce>'
```

**Spot-check**
```bash
git log --format='%h %an <%ae> | %cn <%ce>' | head -20
```

**Hosting platform**
- Ensure `RIGHT_EMAIL` is verified on your account.
- Ensure rewritten commits are on (or merged into) the default branch.

## Prevent Future Mistakes

```bash
git config --global user.name  "RIGHT_NAME"
git config --global user.email "RIGHT_EMAIL"
git config --global user.useconfigonly true
```

Clear any environment overrides in this shell:
```bash
unset GIT_AUTHOR_NAME GIT_AUTHOR_EMAIL GIT_COMMITTER_NAME GIT_COMMITTER_EMAIL
```
