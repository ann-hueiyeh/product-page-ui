# Agent: Update README CDN Links with Latest CSS

## Purpose

Automates the process of updating jsDelivr CDN links in README.md whenever a new CSS file is generated in the `docs/` directory.

## Context

- **Repository**: ann-hueiyeh/product-page-ui
- **Target File**: `README.md`
- **CSS Location**: `docs/`
- **CSS Pattern**: `global-[hash].css` (e.g., `global-CSq0trNl.css`)
- **Update Locations**: 2 places in the jsDelivr section (example URL and HTML code block)

## Workflow Steps

### 1. Find Latest CSS File

Identify the newest CSS file in the `docs/` directory by modification time.

**Command:**

```bash
ls -lt docs/*.css | head -1 | awk '{print $NF}'
```

**Expected Output:**

```
docs/global-[hash].css
```

**Extract Filename Only:**

```bash
basename $(ls -lt docs/*.css | head -1 | awk '{print $NF}')
```

### 2. Delete Old CSS Files

Remove all CSS files except the latest one to keep the `docs/` directory clean.

**Command:**

```bash
# Find the latest CSS file
LATEST_CSS=$(ls -lt docs/*.css | head -1 | awk '{print $NF}')

# Delete all other CSS files
ls docs/*.css | grep -v "$LATEST_CSS" | xargs rm -f
```

**Safety Check:**

```bash
# Count remaining CSS files (should be 1)
ls docs/*.css | wc -l
```

### 3. Update README.md CDN Links

Update the CSS filename in two locations within the jsDelivr section:

#### Location 1: Example URL (line ~12)

```markdown
https://cdn.jsdelivr.net/gh/ann-hueiyeh/product-page-ui/docs/[filename].css
```

#### Location 2: HTML Link Tag (line ~18)

```html
<link rel="stylesheet" crossorigin href="https://cdn.jsdelivr.net/gh/ann-hueiyeh/product-page-ui/docs/[filename].css" />
```

**Automated Replacement:**

```bash
# Get current and new filenames
OLD_CSS=$(grep -o 'global-[a-zA-Z0-9]*.css' README.md | head -1)
NEW_CSS=$(basename $(ls -lt docs/*.css | head -1 | awk '{print $NF}'))

# Replace in README.md (macOS compatible)
sed -i '' "s/$OLD_CSS/$NEW_CSS/g" README.md
```

### 4. Verify Updates

Confirm both CDN URLs now reference the latest CSS file.

**Command:**

```bash
grep "global-.*\.css" README.md
```

**Expected Output:**
Both occurrences should show the same latest filename.

### 5. Git Commit and Push

Stage, commit, and push the changes to the remote repository.

**Commands:**

```bash
# Stage README.md and docs/ directory (includes deleted files)
git add README.md docs/

# Commit with descriptive message
git commit -m "Update CDN links to latest CSS ($NEW_CSS) and remove old CSS files"

# Push to remote
git push origin $(git branch --show-current)
```

## Complete Automation Script

```bash
#!/bin/bash
set -e

echo "🔍 Finding latest CSS file in docs/..."
LATEST_CSS_PATH=$(ls -lt docs/*.css | head -1 | awk '{print $NF}')
NEW_CSS=$(basename "$LATEST_CSS_PATH")
echo "Latest CSS: $NEW_CSS"

echo "📝 Checking current CSS in README.md..."
OLD_CSS=$(grep -o 'global-[a-zA-Z0-9]*.css' README.md | head -1 || echo "")

echo "🧹 Checking for old CSS files to delete..."
OLD_CSS_FILES=$(ls docs/*.css | grep -v "$LATEST_CSS_PATH" || echo "")

if [ -n "$OLD_CSS_FILES" ]; then
  echo "Found old CSS files to delete:"
  echo "$OLD_CSS_FILES"
  echo "$OLD_CSS_FILES" | xargs rm -f
  echo "✓ Old CSS files deleted"
else
  echo "✓ No old CSS files to delete"
fi

echo "📊 Remaining CSS files in docs/:"
ls docs/*.css

if [ "$OLD_CSS" = "$NEW_CSS" ] && [ -z "$OLD_CSS_FILES" ]; then
  echo "✅ README.md already uses the latest CSS file and no old files to clean up. No updates needed."
  exit 0
fi

if [ "$OLD_CSS" != "$NEW_CSS" ] && [ -n "$OLD_CSS" ]; then
  echo "🔄 Updating README.md..."
  sed -i '' "s/$OLD_CSS/$NEW_CSS/g" README.md
  echo "✓ README.md updated"
else5
  echo "✓ README.md already up to date"
fi

echo "✓ Verifying updates..."
grep "global-.*\.css" README.md

echo "📦 Committing changes..."
git add README.md docs/
git commit -m "Update CDN links to latest CSS ($NEW_CSS) and remove old CSS files"

echo "🚀 Pushing to remote..."
git push origin $(git branch --show-current)

echo "Old CSS files deleted from `docs/` directory
- [ ] Only one CSS file remains in `docs/`
- [ ] ✅ Update complete! CDN links now reference $NEW_CSS and old CSS files removed"
```

## Usage

### Manual Execution

Follow steps 1-4 sequentially when a new CSS file is added to `docs/`.

### Automated Execution

1. Save the automation script as `update-css-links.sh`
2. Make it executable: `chmod +x update-css-links.sh`
3. Run: `./update-css-links.sh`

### Integration Points

- **Post-build hook**: Run after CSS compilation
- **CI/CD pipeline**: Add as a deployment step
- **Git pre-push hook**: Ensure links are current before pushing

## Verification Checklist

- [ ] Latest CSS file identified correctly
- [ ] Both CDN URLs in README.md updated (lines ~12 and ~18)
- [ ] Both URLs reference the same filename
- [ ] Changes committed with descriptive message
- [ ] Changes pushed to remote repository
- [ ] GitHub Pages serves the correct CSS file

## Notes

- **Modification Time**: Uses file modification time to determine "latest," not hash values (hashes aren't chronological)
- **No Backup Needed**: Previous filenames preserved in git history
- **GitHub Pages Sync**: Changes to `docs/` may take a few minutes to reflect on https://ann-hueiyeh.github.io/product-page-ui/
- **jsDelivr CDN Caching**: CDN may cache the old file; use version tags in URL if immediate updates required (e.g., `@main` or `@v1.0.0`)

## Troubleshooting

### Problem: Multiple CSS files with same modification time

**Solution**: Use explicit filename or add secondary sort criteria (e.g., alphabetical)

### Problem: Git push fails (no remote configured)

**Solution**: Verify remote exists with `git remote -v`, add if needed

### Problem: jsDelivr serves old CSS

**Solution**:

1. Use versioned URL: `@main` or tag release
2. Purge CDN cache: https://www.jsdelivr.com/tools/purge
3. Wait up to 24 hours for natural cache expiration

## Last Updated

2026-03-03
