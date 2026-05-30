# Doc Curator — Advanced Reference

## Configuration Deep Dive

### Full curator-config.yaml

```yaml
# Global aggression setting (default: high for /docs, medium for /.docs)
aggression: high

# Time threshold for marking features "old" (default: 6 months)
age-threshold-months: 6

# Folder-specific aggression levels
folder-config:
  /docs:
    aggression: high
    age-threshold-months: 6
  /.docs:
    aggression: high  # Temp docs: remove aggressively
    auto-archive-days: 30  # Auto-move to archive after N days

# What signals trigger staleness detection?
staleness-signals:
  git-history:
    enabled: true
    # Flag docs where `updated` < last code commit in linked files
  broken-links:
    enabled: true
    # Check bidirectional references to PRDs, ADRs, stories
  doc-comments:
    enabled: true
    # Compare code doc comments with formal docs
    languages: [js, ts, py, go, rust]
  speculative:
    enabled: true
    # Flag TODO, FIXME, TBD in docs
  redundancy:
    enabled: true
    # Flag when code comments fully replace formal docs
  age-based:
    enabled: true
    # Features > threshold should be condensed

# Frontmatter schema
frontmatter:
  required:
    - title
    - created
    - updated
    - status
  optional:
    - relevance
    - linked-to
    - owner
    - version

# Status values and their meanings
status-values:
  current: "Active, regularly updated, reflects code intent"
  draft: "Work in progress, not yet finalized"
  deprecated: "No longer relevant, will be removed"
  archived: "Historical reference only, do not update"
  deprecated-by: "Superseded by another doc"

# Relevance values
relevance-values:
  high: "Critical context for understanding system"
  medium: "Useful but not essential"
  low: "Nice to have, consider archiving"
  none: "Candidate for removal"

# Link format conventions
link-conventions:
  prd: "PRD#<number>"
  adr: "ADR#<number>"
  story: "STORY#<number>"
  issue: "ISSUE#<number>"

# Aggression mode behaviors
aggression-behaviors:
  low:
    mark-deprecated: false
    suggest-removal: true
    condense-old-features: false
    remove-speculative: false
  medium:
    mark-deprecated: true
    suggest-removal: true
    condense-old-features: true
    remove-speculative: false
  high:
    mark-deprecated: true
    suggest-removal: true
    condense-old-features: true
    remove-speculative: true
```

## Staleness Detection Rules

### Git History Matching

The skill links docs to code by:

1. **Explicit links** — `linked-to` frontmatter
   ```yaml
   linked-to: [src/cache.ts, src/cache.test.ts]
   ```

2. **Implicit detection** — Filename/folder matching
   - Doc: `docs/features/authentication.md`
   - Matches: `src/auth/`, `src/authentication.ts`

3. **Content analysis** — Keywords in doc body
   - Scans for function/class names, module paths
   - Compares with git history of mentioned files

**Staleness triggered when:**
- `updated` date < last commit to linked file + X days (default: 7 days)
- Multiple commits since last doc update
- Code was significantly refactored but doc unchanged

### Doc Comment Extraction

For supported languages (JS/TS, Python, Go, Rust):

```javascript
/**
 * Caches API responses in memory
 * @param {string} key - Cache key
 * @param {number} ttl - Time to live in seconds
 * 
 * Note: This implementation is in-memory only.
 * For distributed caching, see distributed-cache.md
 */
function cache(key, ttl) { ... }
```

The skill:
1. Extracts `/**...*/` comments and docstrings
2. Indexes by function/class name
3. Compares with formal docs
4. Flags if formal doc duplicates the explanation
5. Flags if comment and doc contradict

**Redundancy triggered when:**
- Formal doc explains function behavior already in doc comment
- Code comment provides all necessary detail
- Low-level implementation details in formal doc

### Speculative Content Detection

Patterns that trigger flags:
- `TODO:` / `FIXME:` / `TBD:`
- `WIP` (work in progress)
- `Experimental` / `Alpha` / `Beta`
- `Placeholder`
- `Under review`
- Date checks: "as of [old date]"

**High aggression:** Remove speculative content entirely  
**Medium aggression:** Flag for manual review  
**Low aggression:** Note but leave

### Broken Link Detection

The skill scans frontmatter `linked-to` fields and doc body for:

```markdown
linked-to: [PRD#5, ADR#3, STORY#42]
```

For each link:
1. Check if PRD#5 exists in `docs/prd/`
2. Check if ADR#3 exists in `docs/adr/`
3. Scan code for closed issues matching number

**Bidirectional check:**
- If a doc links to an ADR, the ADR should link back
- If an ADR links to removed code, flag as orphaned

## Planning Mode: Example Output

### .docs/.plan.md

```markdown
# Doc Cleanup Plan — 2026-05-30

Generated with aggression: high

## Summary
- 23 issues detected
- 8 docs flagged for staleness
- 5 redundancy issues
- 3 broken links
- 12 condensing suggestions

## Issues by Type

### Staleness: /docs/architecture/caching.md
**Severity:** High
**Reason:** Last updated 2024-08-15; code refactored 2026-04-20
**Evidence:** 
  - Last doc change: 2024-08-15
  - Related code: src/cache.ts (13 commits since)
  - Latest commit: 2026-04-20 "refactor: move cache to Redis"
**Suggested action:** Review and update or mark as deprecated
**Status option:** deprecated

---

### Redundancy: /docs/features/logger.md
**Severity:** Medium
**Reason:** Code comments fully explain this; formal doc duplicates
**Evidence:**
  - src/logger.ts has comprehensive JSDoc
  - Explanation matches /docs/features/logger.md word-for-word
  - No additional architectural context in formal doc
**Suggested action:** Remove formal doc; enhance code comments
**Status option:** archived

---

### Broken Link: /docs/prd/payment-v2.md
**Severity:** High
**Reason:** References closed PRD
**Evidence:**
  - Doc links to: PRD#342
  - PRD#342: closed 2026-03-15 (SOLVED)
  - Current implementation: PRD#518 (active)
**Suggested action:** Update links or mark as archived

---

### Age + Condensing: /docs/features/xml-support.md
**Severity:** Low
**Reason:** Feature is 18 months old; should be condensed
**Evidence:**
  - Created: 2024-11-20
  - Age: 18 months
  - Status: current
  - Code: stable, no recent changes
**Suggested action:** Condense (current: 2.5KB → suggested: 0.8KB)
**Condensing tips:**
  - Remove historical context (why we built it)
  - Keep: current intent, integration points
  - Move: deep implementation details to code comments

---

### Speculative: /docs/prd/future-auth.md
**Severity:** Medium
**Reason:** Contains WIP / experimental content; should be in /.docs
**Evidence:**
  - Keywords found: "WIP", "Experimental", "Under review"
  - Not linked to active story
  - No code yet
**Suggested action:** Move to /.docs or remove

---

## Guidelines for Other Agents

See `.docs/.guidelines.md` for:
- How to write docs that stay current
- Frontmatter conventions
- When to document vs. when code comments suffice
- Link format standards

## How to Apply This Plan

1. **Review** this plan
2. **Approve/modify** sections as needed
3. **Run:** `doc-curator --apply`
4. **Commit:** `git add docs/ .docs/` && `git commit -m "docs: curator maintenance"`

---

## Approved Changes (Modify Above)

- [x] Deprecate /docs/architecture/caching.md
- [ ] Remove /docs/features/logger.md
- [x] Update PRD links in /docs/prd/payment-v2.md
- [ ] Condense /docs/features/xml-support.md
- [ ] Move /docs/prd/future-auth.md to /.docs
```

## CI/CD Integration

### GitHub Actions Example

```yaml
name: Doc Curator

on:
  pull_request:
    paths:
      - 'docs/**'
      - '.docs/**'

jobs:
  audit:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
        with:
          fetch-depth: 0  # Full history for git-history signal
      
      - name: Run doc-curator
        run: |
          doc-curator --repo . --dry-run > /tmp/plan.md
      
      - name: Comment PR
        uses: actions/github-script@v6
        with:
          script: |
            const fs = require('fs');
            const plan = fs.readFileSync('/tmp/plan.md', 'utf8');
            github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: plan
            });
```

### Pre-commit Hook

```bash
#!/bin/bash
# .git/hooks/pre-commit

doc-curator --dry-run > /tmp/curator-check.txt
if [ $? -ne 0 ]; then
  echo "❌ Documentation issues detected:"
  cat /tmp/curator-check.txt
  exit 1
fi
echo "✓ Documentation audit passed"
```

## Bulk Operations

### Bulk Status Update

```bash
# Mark all docs older than 12 months as "archived"
doc-curator --folder docs \
  --filter "age > 12 months" \
  --status archived
```

### Bulk Link Update

```bash
# Update all links from old PRD to new one
doc-curator --find-replace "PRD#342" "PRD#518" \
  --folder docs
```

### Bulk Condensing

```bash
# Auto-condense features older than 9 months
doc-curator --aggression high \
  --filter "age > 9 months" \
  --auto-condense
```

## Custom Staleness Rules

Add to `curator-config.yaml`:

```yaml
custom-rules:
  - name: "no-outdated-version-numbers"
    pattern: 'version: [0-9]'
    check: "Does doc mention current major version?"
    severity: medium
    
  - name: "deprecated-api-references"
    pattern: 'api-v1'
    check: "API v1 was sunset in 2025; should reference v2"
    severity: high
    
  - name: "outdated-tech-stack"
    pattern: 'We use (jQuery|Backbone|CoffeeScript)'
    check: "Verify tech stack is still in use"
    severity: high
```

## Language-Specific Doc Comment Formats

### JavaScript/TypeScript

```javascript
/**
 * Short description
 * @param {type} name - Description
 * @returns {type} Description
 */
```

### Python

```python
"""
Short description

Args:
    name: Description

Returns:
    Description
"""
```

### Go

```go
// Package cache provides in-memory caching.
// Short description for functions.
```

### Rust

```rust
/// Short description
/// 
/// Longer description
```

The skill extracts and compares these across formats.

## Troubleshooting

**Problem:** "Doc comment detected but no formal doc exists"  
**Solution:** Either create formal doc or enhance code comment. Decide based on context: Does this need architectural explanation, or is code comment enough?

**Problem:** Broken link to PR that was closed/merged  
**Solution:** Update link to appropriate version tag or ADR if decision was captured.

**Problem:** "Relevance: low" but I disagree  
**Solution:** Set `relevance: high` manually in frontmatter; skill won't override explicit values.

**Problem:** Age-based staleness too aggressive  
**Solution:** Increase `age-threshold-months` in config or set `status: archived` explicitly (won't be flagged for updates).
