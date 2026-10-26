# CLAUDE.md

System Design Gym: a visual, LeetCode-style app for practising system design.
The spec lives in `docs/SPEC.md`.

## Workflow (required for every change)

1. **Open an issue first.** Every change starts with a GitHub issue that
   describes the context, scope and done criteria.
2. **Create a branch from `main`** named
   `<type>/<issue-number>/<short-description>`:
   - `type` is one of `feat`, `fix`, `refactor`, `chore`
   - `short-description` is kebab-case, a few words
   - e.g. `feat/12/cache-component`, `fix/18/lb-health-check`
3. **Commit with [Conventional Commits](https://www.conventionalcommits.org/)**:
   `<type>(<optional scope>): <summary>` in the imperative, lower case, no
   trailing period. Types: `feat`, `fix`, `refactor`, `chore`, `docs`, `test`,
   `perf`, `build`, `ci`, `style`. Add `Refs #<issue>` in the commit body.
4. **Open a pull request** into `main` that links the issue (`Closes #<issue>`).
   PR title follows the same Conventional Commits format.

Never commit directly to `main`.

## Attribution

No tool branding anywhere: no `Co-Authored-By` or session trailers in
commits, no "Generated with …" lines in PRs, issues or comments. Commits are
authored as the repository owner.

Write issues, PR descriptions and comments in the first person, as the
repository owner ("I chose…", "I want…"). Never refer to the owner in the
third person ("Arsène asked…").
