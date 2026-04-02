# Local Patch Notes

This branch carries a small security-hardening patch stack intended to stay easy to rebase onto upstream.

## Patches

1. Validate branch names before shelling out in `validate_branch_request`.
2. Prevent workspace file explorer and download flows from following symlinks outside the workspace root.
3. Restrict direct `/ws`, `/pairing`, and `/mcp/agents` access to local connections by default.
4. Allow opting back into remote direct access with `PASEO_ALLOW_REMOTE_DIRECT_CONNECTIONS=1`.

## Rebase Workflow

1. Keep each security fix as a small, isolated commit.
2. Rebase this branch onto upstream instead of merging it.
3. Re-run `npm run build --workspace=@getpaseo/highlight` and `npm run typecheck` after rebases.
4. Drop local commits if upstream lands equivalent fixes.
