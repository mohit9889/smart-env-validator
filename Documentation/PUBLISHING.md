# Publishing to npm with GitHub Actions

This guide explains how to automatically publish your package to npm when you create a GitHub release.

## Prerequisites

1. **npm Account** - Create one at https://www.npmjs.com/signup if you don't have one
2. **GitHub Repository** - Your code should be in a GitHub repository
3. **Package Name** - Ensure your package name is available on npm

## Setup Steps

### 1. Get Your npm Token

1. Log in to npm.com
2. Click your profile picture → **Access Tokens**
3. Click **Generate New Token** → **Classic Token**
4. Select **Automation** type (for CI/CD)
5. Copy the token (starts with `npm_...`)

### 2. Add Token to GitHub Secrets

1. Go to your GitHub repository
2. Navigate to **Settings** → **Secrets and variables** → **Actions**
3. Click **New repository secret**
4. Name: `NPM_TOKEN`
5. Value: Paste your npm token
6. Click **Add secret**

### 3. Verify Workflow File

The workflow file has been created at `.github/workflows/publish.yml`. It will:
- ✅ Run when you create a GitHub release
- ✅ Install dependencies
- ✅ Run tests
- ✅ Build the package
- ✅ Publish to npm

### 4. Update package.json (If Needed)

Ensure your `package.json` has:

```json
{
  "name": "smart-env-validator",
  "version": "0.1.0",
  "repository": {
    "type": "git",
    "url": "https://github.com/YOUR_USERNAME/smart-env-validator.git"
  },
  "author": "Your Name <your.email@example.com>",
  "license": "MIT"
}
```

## How to Publish

### Method 1: Create a GitHub Release (Recommended)

1. **Update Version**
   ```bash
   npm version patch  # or minor or major
   git push --follow-tags
   ```

2. **Create GitHub Release**
   - Go to your GitHub repository
   - Click **Releases** → **Draft a new release**
   - Click **Choose a tag** → Create new tag (e.g., `v0.1.0`)
   - Fill in release title and description
   - Click **Publish release**

3. **Automatic Publishing**
   - GitHub Actions will automatically run
   - Check the **Actions** tab to see progress
   - Package will be published to npm

### Method 2: Manual Trigger

1. Go to **Actions** tab in your repository
2. Select **Publish to npm** workflow
3. Click **Run workflow**
4. Select branch and click **Run workflow**

## Version Management

### Semantic Versioning

Use npm's built-in version commands:

```bash
# Patch release (0.1.0 → 0.1.1) - Bug fixes
npm version patch

# Minor release (0.1.0 → 0.2.0) - New features
npm version minor

# Major release (0.1.0 → 1.0.0) - Breaking changes
npm version major
```

These commands will:
- Update `package.json` version
- Create a git commit
- Create a git tag

Then push with tags:
```bash
git push --follow-tags
```

## First-Time Publishing

For your first publish, you can either:

**Option A: Use GitHub Actions**
1. Set up the token (steps above)
2. Create a release as described
3. Let the workflow publish

**Option B: Publish Manually First**
```bash
npm login
npm publish --access public
```

After the first publish, use GitHub Actions for subsequent releases.

## Verification

After publishing:

1. **Check npm**
   - Visit https://www.npmjs.com/package/smart-env-validator
   - Verify the version is updated

2. **Test Installation**
   ```bash
   # In a new directory
   mkdir test-install
   cd test-install
   npm init -y
   npm install smart-env-validator
   npx smart-env --version
   ```

## Troubleshooting

### "You do not have permission to publish"

- Verify the npm token is correct
- Make sure the token type is "Automation"
- Check if the package name is already taken

### "version already exists"

- You published this version already
- Bump the version: `npm version patch`
- Push the new tag and create a new release

### Workflow Fails on Tests

- Make sure all tests pass locally: `npm test`
- Fix failing tests before publishing

### Package Missing Files

Check your `package.json` `files` field:
```json
{
  "files": [
    "dist",
    "README.md",
    "LICENSE"
  ]
}
```

## Security Best Practices

✅ **Never commit npm tokens** to your repository  
✅ **Use Automation tokens** for CI/CD  
✅ **Enable 2FA** on your npm account  
✅ **Review the Actions logs** after each publish  
✅ **Use `--access public`** for public packages

## Additional Workflows

You can also add these workflows:

### Auto-Test on Pull Requests

Create `.github/workflows/test.yml`:

```yaml
name: Test

on: [pull_request, push]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '18'
      - run: npm ci
      - run: npm test
      - run: npm run build
```

### Auto-Update Dependencies

Create `.github/workflows/update-deps.yml`:

```yaml
name: Update Dependencies

on:
  schedule:
    - cron: '0 0 * * 1'  # Every Monday
  workflow_dispatch:

jobs:
  update:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
      - run: npm update
      - run: npm test
      # Create PR with updated dependencies
```

## Next Steps

1. ✅ Set up npm token in GitHub secrets
2. ✅ Update `package.json` with your details
3. ✅ Ensure all tests pass: `npm test`
4. ✅ Create your first release
5. ✅ Watch the Actions tab for automatic publishing

Your package will now be automatically published to npm whenever you create a GitHub release! 🚀
