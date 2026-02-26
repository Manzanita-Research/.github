---
name: update-org-readme
description: >
  Update the GitHub org profile README with a fresh table of all public repos.
  Use this skill whenever the user wants to refresh, update, or sync the org's
  profile/README.md with current repo data — including after creating new repos,
  renaming repos, or updating repo descriptions. Triggers on phrases like
  "update the org readme", "refresh the profile", "sync repos to readme",
  "add new repos to the profile", or just "update profile README".
---

# Update Org Profile README

Refreshes the project table in a GitHub org's `profile/README.md` with live data from the GitHub API.

## How it works

1. **Detect the org.** Run `gh repo view --json owner -q '.owner.login'` from the current repo to get the org name. If that fails or the user specifies an org, use that instead.

2. **Fetch all public repos.** Use the GitHub CLI:
   ```bash
   gh repo list <ORG> --visibility public --limit 100 --json name,description,url
   ```

3. **Build the table rows.** For each repo:
   - Skip any repo named `.github` — that's the org meta repo, not a project.
   - Check if the description starts with an emoji. If it does, split it into the emoji column and the rest as the description. If not, leave the emoji column blank and use the full description.
   - Link the project name to its GitHub URL.
   - Sort rows alphabetically by repo name (case-insensitive).

4. **Replace the table in the README.** Find the existing markdown table in `profile/README.md` (the block between `| | Project |` and the next blank line or section header) and replace it with the new table. Preserve everything else in the file — the intro, beliefs, website link, sign-off, all of it.

   The table format:
   ```markdown
   | | Project | What it does |
   |---|---|---|
   | 🌳 | [grove](https://github.com/Org/grove) | 3D spatial audio mixer visualization |
   | | [some-repo](https://github.com/Org/some-repo) | A repo with no emoji in its description |
   ```

5. **Show the user the diff.** After editing, run `git diff profile/README.md` so they can see exactly what changed. Don't commit unless asked.

## Parsing the emoji from the description

GitHub repo descriptions are plain text. When we added emojis earlier, they went at the very start of the description string, like `"🌳 3D spatial audio mixer..."`. To split:

- Check if the first character (which may be a multi-byte emoji sequence) is followed by a space and then text.
- A simple heuristic: if the first space-separated token is 1-2 characters long and contains no ASCII letters, treat it as the emoji.
- If the description is empty or null, both columns are blank.

## Edge cases

- **New repo with no description**: empty description cell, blank emoji cell.
- **Archived or forked repos**: include them — they're still public. The user can manually exclude if they want.
- **The README doesn't have a table yet**: create one in the "What we're working on" section, or ask the user where to put it.
