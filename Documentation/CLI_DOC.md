# CLI Documentation

Complete reference for `smart-env` CLI commands.

## Installation

```bash
npm install smart-env-validator
```

Or use directly with `npx`:

```bash
npx smart-env <command>
```

## Commands Overview

| Command | Description |
|---------|-------------|
| `init` | Initialize configuration (interactive) |
| `validate` | Validate current environment |
| `check` | Compare schema with .env.example |
| `fix` | Interactively fix missing variables |
| `docs` | Generate documentation table |

---

## `smart-env init`

Create a new environment configuration file with interactive setup.

### Usage

```bash
npx smart-env init
```

### Interactive Prompts

1. **Language Selection**
   - TypeScript (creates `env.config.ts`)
   - JavaScript (creates `env.config.js`)

2. **Git Integration**
   - Option to auto-add `.env` files to `.gitignore`

### Output Files

- `env.config.ts` or `env.config.js` - Configuration with example schema
- `.env.example` - Template with example values

### Example

```bash
$ npx smart-env init
✔ Which language are you using? › TypeScript
✔ Add .env files to .gitignore? › Yes
✓ Created env.config.ts
✓ Created .env.example
✓ Updated .gitignore

Next steps:
  1. Copy .env.example to .env.local
  2. Fill in your environment variables
  3. Run: npx smart-env validate
```

### Notes

- Will not overwrite existing `env.config.*` files
- Creates `.gitignore` if it doesn't exist
- TypeScript configuration uses ES6 modules
- JavaScript configuration uses CommonJS

---

## `smart-env validate`

Validate your current environment against the schema.

### Usage

```bash
npx smart-env validate
```

### Behavior

1. Loads `env.config.ts` or `env.config.js` from current directory
2. Reads environment files based on configuration
3. Validates all variables against schema
4. Displays results

### Success Output

```bash
✓ All environment variables are valid
Loaded from: .env.local (6 vars)
```

### Error Output

```bash
❌ Environment validation failed:

  DATABASE_URL
    ✗ Required
    📄 Not found in any .env file
    💡 Add DATABASE_URL to your .env file

  PORT
    ✗ Expected number, received "abc"
    📄 Found in: .env.local (line 3)
    💡 Fix: PORT=3000

Run: npx smart-env fix (to auto-fix missing variables)
```

### Exit Codes

- `0` - Validation successful
- `1` - Validation failed or config not found

### Notes

- Reads from environment files specified in `envFiles` config
- Supports both `.ts` and `.js` config files (no ts-node needed)
- Shows which file each variable was loaded from

---

## `smart-env check`

Compare your schema with `.env.example` to find discrepancies.

### Usage

```bash
npx smart-env check
```

### What It Checks

1. **Missing Variables** - In `.env.example` but not in schema
2. **Extra Variables** - In schema but not in `.env.example`
3. **Consistency** - Ensures `.env.example` is up to date

### Success Output

```bash
✓ .env.example matches schema
```

### Error Output

```bash
❌ .env.example validation failed:

Missing from .env.example:
  - API_KEY
  - API_SECRET

Extra in .env.example (not in schema):
  - OLD_VARIABLE

💡 Update .env.example to match your schema
```

### Exit Codes

- `0` - Check passed
- `1` - Discrepancies found or files not found

### Use Cases

- Pre-commit hooks to ensure `.env.example` is updated
- CI/CD validation
- Onboarding new developers (they know what vars to set)

---

## `smart-env fix`

Interactively fix missing or invalid environment variables.

### Usage

```bash
npx smart-env fix
```

### Workflow

1. Validates current environment
2. Identifies missing/invalid variables
3. Prompts for each variable value
4. Asks which file to update
5. Appends new variables to selected file

### Interactive Prompts

1. **File Selection**
   ```
   ? Which file would you like to update?
   › .env.local (recommended)
     .env
     .env.development
     .env.production
   ```

2. **Value Input**
   ```
   ? Enter value for DATABASE_URL: postgresql://localhost:5432/mydb
   ? Enter value for API_KEY: **********************
   ```

### Example Session

```bash
$ npx smart-env fix

❌ Found missing or invalid environment variables:

  DATABASE_URL
    ✗ Required
    📄 Not found in any .env file

  API_KEY
    ✗ Required
    📄 Not found in any .env file

✔ Which file would you like to update? › .env.local
✔ Enter value for DATABASE_URL: postgresql://localhost:5432/mydb
✔ Enter value for API_KEY: my_secret_api_key_12345

✓ Updated .env.local with 2 variable(s)

Run: npx smart-env validate
```

### Notes

- Validates input as you type (basic checks)
- Appends to existing file (doesn't overwrite)
- Adds comment marker for traceability
- Skips variables if you press Enter without input

---

## `smart-env docs`

Generate documentation table for environment variables.

### Usage

```bash
npx smart-env docs
```

### Output

Generates a Markdown table with:
- Variable name
- Type (string, number, boolean, enum)
- Required/Optional
- Default value
- Description (from config)

### Example Output

```markdown
## Environment Variables

| Variable | Type | Required | Default | Description |
|----------|------|----------|---------|-------------|
| `NODE_ENV` | enum | No | `development` | - |
| `PORT` | number | No | `3000` | Server port number |
| `DATABASE_URL` | string | Yes | - | PostgreSQL connection string |
| `API_KEY` | string | Yes | - | API authentication key |
```

### README Integration

If you have a `README.md` with markers, it can auto-update:

1. Add markers to your README.md:
   ```markdown
   <!-- smart-env-start -->
   <!-- smart-env-end -->
   ```

2. Run the command:
   ```bash
   $ npx smart-env docs
   ✔ Update README.md with this table? › Yes
   ✓ Updated README.md
   ```

### Use Cases

- Keep documentation in sync with code
- Generate API documentation
- Provide clear onboarding guide
- Include in project wikis

### Notes

- Reads descriptions from `config.descriptions`
- Detects types from Zod schema
- Handles nested schemas (defaults, optionals)

---

## Global Options

These options work with all commands:

### `--help`

Show help for a specific command:

```bash
npx smart-env --help
npx smart-env validate --help
```

### `--version`

Show package version:

```bash
npx smart-env --version
```

---

## Configuration File Reference

All commands read from `env.config.ts` or `env.config.js`:

```typescript
import { defineConfig, z } from 'smart-env-validator';

export default defineConfig({
  // Required: Zod schema
  schema: {
    PORT: z.coerce.number().default(3000),
    DATABASE_URL: z.string().url(),
  },
  
  // Optional: Environment files to load
  envFiles: ['.env.local', '.env'],
  
  // Optional: Required in production
  requiredInProduction: ['DATABASE_URL'],
  
  // Optional: Fail on unknown variables
  strict: true,
  
  // Optional: Expand ${VAR} references
  expand: true,
  
  // Optional: Descriptions for docs
  descriptions: {
    PORT: 'Server port number',
    DATABASE_URL: 'PostgreSQL connection string',
  },
});
```

---

## Exit Codes

All commands use standard exit codes:

- `0` - Success
- `1` - Error (validation failed, config not found, etc.)

Use in scripts:

```bash
npx smart-env validate && npm start
```

---

## Common Workflows

### New Project Setup

```bash
npx smart-env init
cp .env.example .env.local
# Edit .env.local
npx smart-env validate
```

### Pre-Deployment Check

```bash
npx smart-env validate
npx smart-env check
```

### Fix Environment Issues

```bash
npx smart-env validate
# If errors occur:
npx smart-env fix
```

### Update Documentation

```bash
npx smart-env docs
# Add to README manually or use auto-update
```

### CI/CD Pipeline

```bash
#!/bin/bash
npx smart-env check || exit 1
npx smart-env validate || exit 1
npm run build
```

---

## Troubleshooting

### "No env.config.ts or env.config.js found"

Run `npx smart-env init` to create the configuration file.

### "Cannot use import statement outside a module"

This is fixed in V1.1! The package now uses `jiti` to load TypeScript configs natively.

If you still see this:
1. Make sure you're using the latest version
2. Your config should be in the project root
3. Check that the file is named exactly `env.config.ts` or `env.config.js`

### Validation Passes But App Fails

Make sure you're using the validated `env` object:

```typescript
// ❌ Don't use process.env directly
console.log(process.env.DATABASE_URL);

// ✅ Use the validated env object
import { env } from 'smart-env-validator';
console.log(env.DATABASE_URL);
```

### Interactive Prompts Not Working

The `init` and `fix` commands require a TTY. If running in CI/CD or non-interactive environments, pre-create your config files.

---

## Examples

See the [example](../example/) directory for a complete working project with all features demonstrated.
