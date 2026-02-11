# Accessibility Guides

This directory contains all accessibility checking guides used by Claude Code commands, the GitHub app, and other accessibility tools.

## 📄 Single Source of Truth: GUIDES_MANIFEST.json

**The guide list is maintained in ONE place:**

```
guides/GUIDES_MANIFEST.json
```

Both Claude Code commands and the GitHub app read from this manifest to determine which guides to load. This ensures perfect consistency across all tools.

## 📁 Directory Structure

```
guides/
├── GUIDES_MANIFEST.json          ← SINGLE SOURCE OF TRUTH
├── README.md                      ← You are here
│
├── COMPREHENSIVE_AUDIT_WORKFLOW.md
├── COMMON_ISSUES.md
├── GUIDE_WCAG_REFERENCE.md
│
├── wcag/                          ← WCAG validation guides
│   ├── README.md
│   ├── CONTRAST_CHECKING.md       ← Visual contrast
│   ├── ID_VALIDATION.md           ← ID generation
│   ├── LIVE_REGION_FREQUENCY.md   ← aria-live patterns
│   ├── EDGE_CASE_VALIDATION.md    ← Edge cases
│   └── ...
│
├── patterns/                      ← Pattern-specific guides
│   ├── AVOID_ROLE_IN_LABEL.md
│   ├── BUTTONS_ACTING_AS_TABS.md
│   └── ...
│
├── reference/                     ← Reference materials
│   └── ARIA_PATTERNS_REFERENCE.md
│
└── platforms/                     ← Platform-specific guides
    ├── GUIDE_WEB.md
    ├── GUIDE_ANDROID.md
    ├── GUIDE_IOS.md
    ├── GUIDE_REACT_NATIVE.md
    ├── GUIDE_FLUTTER.md
    ├── GUIDE_ANDROID_TV.md
    └── GUIDE_TVOS.md
```

## ✅ Adding a New Guide

**Simple 2-step process:**

### 1. Create the Guide File

```bash
# Choose the appropriate directory:
guides/wcag/YOUR_NEW_GUIDE.md            # For WCAG validation
guides/patterns/YOUR_PATTERN.md           # For patterns
guides/reference/YOUR_REFERENCE.md        # For reference materials
guides/platforms/GUIDE_YOUR_PLATFORM.md   # For platform-specific guides
```

### 2. Add to Manifest

Edit `guides/GUIDES_MANIFEST.json`:

```json
{
  "wcag": [
    "wcag/CONTRAST_CHECKING.md",
    "wcag/ID_VALIDATION.md",
    "wcag/YOUR_NEW_GUIDE.md"  // ← Add here
  ]
}
```

### 3. Done! 🎉

Both Claude commands and the GitHub app will automatically:
- ✅ Load your new guide
- ✅ Apply its checks
- ✅ Stay in sync

**No need to update:**
- ❌ `.claude/commands/audit.md` - Reads from manifest
- ❌ `github-app/src/guides-loader.ts` - Reads from manifest

## 🔄 Modifying an Existing Guide

1. Edit the guide file directly
2. Update the "Last Updated" date in the guide
3. Done! Changes apply everywhere automatically

## 🧪 Verification

### Test with Claude Code

```bash
/audit
```

Claude will load all guides from the manifest and perform the audit.

### Test with GitHub App

```bash
npm test -- guide-consistency.test.ts
```

Verifies the manifest is loaded correctly and all guides exist.

## 📖 Guide Categories

### Core Guides
Essential workflow and reference guides loaded for all audits.

### WCAG Validation Guides (`wcag/`)
Specific WCAG success criteria validation logic:
- **CONTRAST_CHECKING.md** - Visual contrast (1.4.3, 1.4.11, 1.4.1)
- **ID_VALIDATION.md** - ID generation and ARIA references (4.1.1, 4.1.2)
- **LIVE_REGION_FREQUENCY.md** - aria-live patterns (4.1.3)
- **EDGE_CASE_VALIDATION.md** - Defensive patterns and edge cases

### Pattern Guides (`patterns/`)
Common accessibility patterns and anti-patterns.

### Reference Materials (`reference/`)
Technical reference documentation (ARIA patterns, specifications, etc.).

### Platform Guides (`platforms/`)
Platform-specific implementation guidance (Web, iOS, Android, React Native, Flutter, TV platforms).

## 🔍 How It Works

```
┌─────────────────────────┐
│ GUIDES_MANIFEST.json    │  ← Single source of truth
└───────────┬─────────────┘
            │
            ├─→ Claude Code Commands
            │   └─ Reads manifest
            │   └─ Loads all guides
            │   └─ Performs audit
            │
            └─→ GitHub App
                └─ Reads manifest
                └─ Loads all guides
                └─ Performs PR review
```

## 📝 Guide Writing Guidelines

When creating a guide:

1. **Follow the template:**
   - Title and purpose
   - WCAG SCs covered
   - Overview
   - Detailed sections with examples
   - Platform-specific patterns
   - Testing strategies
   - Documentation templates
   - Last updated date

2. **Include examples:**
   - ❌ Bad code
   - ✓ Good code
   - Rationale for each

3. **Be specific:**
   - File paths and line numbers
   - Contrast ratios and calculations
   - Severity guidelines

4. **Cross-reference:**
   - Link to other guides
   - Link to WCAG documentation
   - Link to platform documentation

## 🔗 Related Files

- **`.claude/commands/audit.md`** - Claude audit command (reads manifest)
- **`GITHUB_APP_INTEGRATION.md`** - GitHub app integration guide
- **`guides/wcag/README.md`** - WCAG guides documentation

## ❓ FAQ

### Q: Do I need to update both Claude commands and GitHub app when adding a guide?

**A: No!** Just add it to `GUIDES_MANIFEST.json`. Both systems read from there.

### Q: What if I want to add a guide that only applies to one platform?

**A: Two options:**
1. Add to the `platforms` section if it's a platform guide
2. Add conditional logic in the guide itself (e.g., "For Web only...")

### Q: How do I know if my guide is being loaded?

**A: Test it:**
- Run `/audit` with Claude - it should apply your checks
- Run GitHub app tests - should load without errors
- Create a PR - GitHub app should flag issues from your guide

### Q: Can I temporarily disable a guide?

**A: Yes!** Comment it out in the manifest:
```json
{
  "wcag": [
    "wcag/CONTRAST_CHECKING.md",
    // "wcag/YOUR_GUIDE.md",  // Temporarily disabled
  ]
}
```

## 🚀 Quick Start

1. **View current guides:**
   ```bash
   cat guides/GUIDES_MANIFEST.json
   ```

2. **Add new guide:**
   ```bash
   # Create guide
   touch guides/wcag/MY_NEW_GUIDE.md

   # Edit manifest
   nano guides/GUIDES_MANIFEST.json

   # Test
   /audit
   ```

3. **Update existing guide:**
   ```bash
   # Edit guide directly
   nano guides/wcag/CONTRAST_CHECKING.md

   # No other changes needed!
   ```

---

**Maintained by:** accessibility-fixer project
**Last Updated:** 2026-02-10
**Status:** Production - Single Source of Truth
