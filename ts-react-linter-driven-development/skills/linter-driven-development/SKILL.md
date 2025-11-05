---
name: linter-driven-development
description: META ORCHESTRATOR for complete implementation workflow - design, test, lint, refactor, review, commit. Use for any code change that should result in a commit (features, bug fixes, refactors). Ensures clean code with tests, linting passes, and design validation.
---

# Linter-Driven Development Workflow (TypeScript + React)

META ORCHESTRATOR for implementation workflow: design → test → lint → refactor → review → commit.
Use for any commit: features, bug fixes, refactors.

## Prerequisites

**IMPORTANT**: Before using this skill, the project MUST have linter configurations:

### Required Configurations

1. **TypeScript** (`tsconfig.json`)
   - Type checking configured for your project

2. **ESLint** (`eslint.config.mjs` or `.eslintrc.js`)
   - Must include `eslint-plugin-sonarjs` for complexity metrics
   - Recommended: TypeScript ESLint, React plugins
   - Complexity thresholds: cognitive, cyclomatic, expression

3. **Prettier** (`.prettierrc.json` or `prettier.config.js` or `.prettierrc`)
   - Consistent formatting rules defined
   - Integration with ESLint recommended

4. **Stylelint** (`stylelint.config.js`) - if using CSS/SCSS
   - CSS/SCSS linting rules configured

### Required npm Scripts

Project must have scripts for running quality checks. **Script names vary by project** - detect them from `package.json`.

Common patterns to look for:
- **Testing**: `test`, `test:unit`, `vitest`, `jest`
- **Type checking**: `typecheck`, `type-check`, `tsc`, `check-types`
- **Linting (check)**: `lint`, `lint:check`, `eslint`, `lintcheck`
- **Linting (fix)**: `lint:fix`, `eslint:fix`, `lint --fix`
- **Formatting (check)**: `format`, `format:check`, `prettier:check`, `formatcheck`
- **Formatting (fix)**: `format:fix`, `prettier:write`, `prettier --write`
- **Styling (check)**: `stylelint`, `style:check`, `stylecheck`
- **Styling (fix)**: `stylelint:fix`, `style:fix`
- **Combined check**: `check`, `checkall`, `validate`, `verify`
- **Combined fix**: `fix`, `fixall`, `format:all`

**Detection strategy**: Read `package.json` scripts and identify which commands serve each purpose.

### Step 0: Detect Project Setup

**FIRST STEP - Do this before starting any workflow**:

1. **Detect Package Manager**
   - Check for lock files in project root:
     - `yarn.lock` → Use `yarn` commands
     - `package-lock.json` or `npm-shrinkwrap.json` → Use `npm` commands
     - `pnpm-lock.yaml` → Use `pnpm` commands
     - No lock file → Ask user which package manager to use

2. **Detect Available Scripts**
   - Read `package.json` scripts section
   - Identify which scripts exist for each purpose:
     - Type checking (e.g., `typecheck`, `type-check`, `tsc`)
     - Linting check (e.g., `lint`, `eslint`, `lint:check`)
     - Linting fix (e.g., `lint:fix`, `eslint:fix`)
     - Formatting check (e.g., `format:check`, `prettier:check`)
     - Formatting fix (e.g., `format:fix`, `prettier:write`)
     - Testing (e.g., `test`, `vitest`, `jest`)
     - Combined checks (e.g., `check`, `checkall`, `validate`)
     - Combined fixes (e.g., `fix`, `fixall`)

3. **Build Command Map**
   - Store detected commands for use throughout workflow
   - Example: `{ typecheck: 'typecheck', lint: 'lint', lintFix: 'lint:fix', test: 'test' }`

**Remember**: Use detected package manager + detected script names consistently throughout ALL workflow phases.

### Verification

Before starting, verify setup by running detected commands:
- [ ] Type checking works (using detected typecheck script)
- [ ] Linting works (using detected lint script)
- [ ] Formatting works (using detected format script)
- [ ] Tests work (using detected test script)
- [ ] SonarJS plugin installed and configured

**If scripts are missing**:
1. Check if functionality exists but with different script name
2. Look for combined commands (e.g., `check` that runs multiple tools)
3. If truly missing, ask user:
   - "What command should I run to check/fix linting?"
   - "Where is this documented?" (suggest adding to README.md or CLAUDE.md)
4. If no script exists, run tool directly (e.g., `eslint .`) or skip that phase

**Command detection examples**:
```bash
# If package.json has:
"scripts": {
  "lint": "eslint .",
  "lint:fix": "eslint . --fix",
  "check": "tsc && eslint .",
  "test": "vitest"
}

# Detected commands:
typecheck: (not found, will run 'tsc' directly)
lint: 'lint'
lintFix: 'lint:fix'
test: 'test'
combined: 'check'
```

---

## When to Use
- Implementing any code change that should result in a commit
- Need automatic workflow management with quality gates
- Want to ensure: clean code + tests + linting + design validation + accessibility

## Workflow Phases

**IMPORTANT**: Start every workflow by detecting the package manager (Step 0 in Prerequisites).

### Phase 1: Design (if needed)
- If new components/types/major changes needed → invoke @component-designing skill
- Output: Component design plan with types, hooks, and structure

### Phase 2: Implementation
- Follow @testing skill principles (React Testing Library with Jest/Vitest)
- Write tests + implementation in parallel (not necessarily test-first)
- Follow project's Prettier/ESLint formatting rules
- Use project's test runner (Jest, Vitest, or other)
- Aim for 100% coverage on new leaf components/hooks (pure logic with no external dependencies)
  - Leaf types: Pure logic (can compose other leaf types), no API/DB/file system access
  - Orchestrating types: Coordinate leaf types and external systems, need integration tests
- Test from user perspective (public API only)

### Phase 3: Linter Loop

**Use detected package manager and script names from Step 0** for all commands below.

Run quality checks in this order (using detected script names):
1. **Type Check**: Run detected typecheck script (e.g., `npm run typecheck` or `tsc` directly)
2. **Lint Check**: Run detected lint check script (e.g., `npm run lint`, `npm run lint:check`)
3. **Format Check**: Run detected format check script (e.g., `npm run format:check`, `npm run prettier:check`)
4. **Style Check**: Run detected style check script (e.g., `npm run stylelint`, `npm run style:check`) - if CSS/SCSS in project

**Handling missing scripts**:
- If type check script not found → Run `tsc --noEmit` directly (TypeScript is required)
- If lint check script not found → Run `eslint .` directly (ESLint is required)
- If format check script not found → Run `prettier --check .` directly (Prettier is required)
- If style check script not found and CSS/SCSS exists → Run `stylelint "**/*.{css,scss}"` or skip if Stylelint not installed

If any failures detected:
- Run auto-fixes using detected fix scripts:
  - **Lint fix**: Run detected lint fix script (e.g., `npm run lint:fix`) or `eslint . --fix`
  - **Format fix**: Run detected format fix script (e.g., `npm run format:fix`) or `prettier --write .`
  - **Style fix**: Run detected style fix script (e.g., `npm run stylelint:fix`) or `stylelint "**/*.{css,scss}" --fix`
- Re-run quality checks
- If still failing (complexity, design issues):
  - Interpret failures (cognitive complexity, cyclomatic complexity, etc.)
  - Invoke @refactoring skill to fix (use storifying, extract functions/hooks, early returns)
  - Re-run checks
- Repeat until all checks pass clean

**Alternative**: If project has combined commands (detected in Step 0):
- Check: Use detected combined check script (e.g., `npm run check`, `npm run checkall`, `npm run validate`)
- Fix: Use detected combined fix script (e.g., `npm run fix`, `npm run fixall`)

**Example workflow with detected commands**:
```bash
# Step 0 detected: { packageManager: 'npm', typecheck: 'typecheck', lint: 'lint', lintFix: 'lint:fix', format: 'format:check', formatFix: 'format:fix', test: 'test' }

# Run checks
npm run typecheck
npm run lint
npm run format:check

# If failures, run fixes
npm run lint:fix
npm run format:fix

# Re-run checks
npm run typecheck
npm run lint
npm run format:check
```

### Phase 4: Pre-Commit Design Review (ADVISORY)
- Invoke @pre-commit-review skill
- Review validates design principles (not code correctness)
- Includes accessibility checks (ARIA, semantic HTML, keyboard nav)
- Categorized findings: Design Debt / Readability Debt / Polish Opportunities
- If issues found in broader file context, flag for potential refactor
- **User decides**: commit as-is, apply fixes, or expand scope

### Phase 5: Commit Ready
- Type checking passes ✅
- ESLint passes ✅
- Prettier passes ✅
- Stylelint passes ✅
- Tests pass with target coverage ✅
- Design review complete (advisory) ✅
- Present summary + commit message suggestion

## Output Format

```
📋 COMMIT READINESS SUMMARY

✅ Type Check: Passed (0 errors)
✅ ESLint: Passed (0 issues)
✅ Prettier: Passed (all files formatted)
✅ Stylelint: Passed (0 style issues)
✅ Tests: 92% coverage (3 leaf hooks at 100%, 1 orchestrating component, 18 test cases)
⚠️  Design Review: 3 findings (see below)

🎯 COMMIT SCOPE
Modified:
- src/components/LoginForm.tsx (+65, -20 lines)
- src/hooks/useAuth.ts (+30, -5 lines)

Added:
- src/types/auth.ts (new: UserId, Email types)
- src/contexts/AuthContext.tsx (new context provider)

Tests:
- src/components/LoginForm.test.tsx (+95 lines)
- src/hooks/useAuth.test.ts (new)
- src/types/auth.test.ts (new)

⚠️  DESIGN REVIEW FINDINGS

🔴 DESIGN DEBT (Recommended to fix):
- src/components/LoginForm.tsx:45 - Primitive obsession detected
  Current: function validateEmail(email: string): boolean
  Better:  Use Zod schema or branded Email type with validation
  Why: Type safety, validation guarantee, prevents invalid emails
  Fix: Use @component-designing to create self-validating Email type

- src/hooks/useAuth.ts:78 - Prop drilling detected
  Auth state passed through 3+ component levels
  Why: Tight coupling, hard to maintain
  Fix: Extract AuthContext or use composition pattern

🟡 READABILITY DEBT (Consider fixing):
- src/components/LoginForm.tsx:120 - Mixed abstraction levels
  Component mixes validation logic with UI rendering
  Why: Harder to understand and test independently
  Fix: Use @refactoring to extract custom hooks (useValidation)

- src/components/LoginForm.tsx:88 - Cognitive complexity: 18 (max: 15)
  Nested conditionals for form validation
  Why: Hard to understand logic flow
  Fix: Use @refactoring to extract validation functions or use Zod

🟢 POLISH OPPORTUNITIES:
- src/types/auth.ts:12 - Missing JSDoc comments
  Public types should have documentation
- src/components/LoginForm.tsx:45 - Consider semantic HTML
  Use <form> with proper ARIA labels for better accessibility
- src/hooks/useAuth.ts:34 - Missing error boundaries
  Consider wrapping async operations with error handling

📝 BROADER CONTEXT:
While reviewing LoginForm.tsx, noticed similar validation patterns in
RegisterForm.tsx and ProfileForm.tsx (src/components/). Consider
extracting a shared validation hook or creating branded types for common
fields (Email, Username, Password) used across the application.

💡 SUGGESTED COMMIT MESSAGE
Add self-validating Email and UserId types to auth feature

- Introduce Email type with RFC 5322 validation using Zod
- Introduce UserId branded type for type safety
- Refactor LoginForm to use validated types
- Extract useAuth hook for auth state management
- Add AuthContext to eliminate prop drilling
- Achieve 92% test coverage with React Testing Library

Follows component composition principles and reduces primitive obsession.

────────────────────────────────────────

Would you like to:
1. Commit as-is (ignore design findings)
2. Fix design debt only (🔴), then commit
3. Fix design + readability debt (🔴 + 🟡), then commit
4. Fix all findings (🔴 🟡 🟢), then commit
5. Refactor broader scope (address validation patterns across features), then commit
```

## Complexity Thresholds (SonarJS)

These metrics trigger @refactoring when exceeded:
- **Cognitive Complexity**: max 15 (how hard to understand)
- **Cyclomatic Complexity**: max 10 (number of paths through code)
- **Expression Complexity**: max 5 (operators in single expression)
- **Function Length**: max 200 lines
- **File Length**: max 600 lines
- **Nesting Level**: max 4 (depth of nested control structures)
- **Max Union Size**: max 4 types (union types with too many options)

## Priority Levels

### 🔴 High Priority: Type Safety Issues
Rules that can cause runtime errors:
- `@typescript-eslint/no-unsafe-member-access` (416 violations)
- `@typescript-eslint/no-unsafe-assignment` (267 violations)
- `@typescript-eslint/no-explicit-any` (194 violations)
- `@typescript-eslint/no-unsafe-argument` (101 violations)
- `@typescript-eslint/no-unsafe-call` (72 violations)
- `@typescript-eslint/no-unsafe-return` (50 violations)

**Fix Strategy**: Use proper types, type guards, Zod schemas, or branded types

### 🟡 Medium Priority: Code Quality & Maintainability
Rules that affect readability and maintenance:
- `no-magic-numbers` (243 violations) - Extract to named constants
- `react/forbid-dom-props` (94 violations) - No inline styles, use CSS modules
- `sonarjs/cyclomatic-complexity` (34 violations) - Reduce branches, early returns
- `sonarjs/prefer-read-only-props` (25 violations) - Props should be immutable
- `react-hooks/exhaustive-deps` (41 violations) - Fix dependencies or simplify

**Fix Strategy**: Apply @refactoring patterns (storifying, early returns, extract functions)

### 🟢 Low Priority: Style & Convention
Rules that improve consistency:
- `no-console` (28 violations) - Use proper logging
- Import/export conventions
- Styling conventions (camelCase, keyframes naming)

**Fix Strategy**: Auto-fix or manual cleanup

## Workflow Control

**Sequential Phases**: Each phase depends on previous phase completion
- Design must complete before implementation
- Implementation must complete before linting
- Linting must pass before review
- Review must complete before commit

**Iterative Linting**: Phase 3 loops until clean
**Advisory Review**: Phase 4 never blocks, always asks user

## Integration with Other Skills

This orchestrator **invokes** other skills automatically:
- @component-designing (Phase 1, if needed)
- @testing (Phase 2, principles applied)
- @refactoring (Phase 3, when linter fails on complexity)
- @pre-commit-review (Phase 4, always)

After committing, consider:
- If feature complete → invoke @documentation skill
- If more work needed → run this workflow again for next commit

## Common Linter Failures and Resolutions

### TypeScript Errors (detected typecheck script)
- Type mismatches → Fix types or add proper type guards
- Missing types → Add explicit types or interfaces
- Cannot fix automatically → Manual intervention required

### ESLint Failures (detected lint check script)
**Auto-fixable**:
- Import sorting (simple-import-sort)
- Unused imports (unused-imports)
- Formatting issues covered by Prettier
- Simple style violations

**Requires refactoring** (invoke @refactoring):
- Cognitive/cyclomatic complexity
- Max lines per function
- Expression complexity
- Nested control flow
- React hooks violations
- Component design issues

### Prettier Failures (detected format check script)
- Always auto-fixable with detected format fix script or `prettier --write .`
- No manual intervention needed

### Stylelint Failures (detected style check script)
- Most auto-fixable with detected style fix script or `stylelint "**/*.{css,scss}" --fix`
- Class naming violations may require manual fixes

## Best Practices

1. **Run checks frequently** during development
2. **Fix one complexity issue at a time** (don't batch refactoring)
3. **Trust the advisory review** (design debt causes future pain)
4. **Test after each refactoring** (ensure behavior unchanged)
5. **Commit frequently** (small, focused commits)

## Implementation Phases

### Phase 1: Type Safety Foundation (🔴 High Priority)
- [ ] Fix all `@typescript-eslint/no-unsafe-*` errors
- [ ] Replace `any` types with proper types
- [ ] Add type definitions for external APIs
- [ ] Use type guards for runtime checks
- [ ] Implement Zod schemas for validation

### Phase 2: Code Quality (🟡 Medium Priority)
- [ ] Fix complexity issues (cyclomatic, cognitive)
- [ ] Remove inline styles, use CSS modules
- [ ] Make props readonly
- [ ] Extract magic numbers to constants
- [ ] Apply refactoring patterns (IIFE → lookup, empty blocks → early returns)

### Phase 3: Polish (🟢 Low Priority)
- [ ] Clean up console statements
- [ ] Fix remaining hooks dependencies
- [ ] Address styling conventions
- [ ] Improve comment quality
- [ ] Use EMPTY_STRING constant

## Additional Resources

- **Common Linter Failures** section above for resolution strategies
- **@refactoring skill** - For complexity issues (cognitive, cyclomatic, expression)
- **@component-designing skill** - For type safety and architecture issues
- **@pre-commit-review skill** - For design validation (runs automatically in Phase 4)
