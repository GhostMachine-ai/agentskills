```markdown
# agentskills Development Patterns

> Auto-generated skill from repository analysis

## Overview
The `agentskills` repository is a TypeScript codebase focused on modular agent skill development. It emphasizes clean code organization, conventional commit practices, and maintainable testing patterns. This skill teaches you how to structure TypeScript projects without a framework, follow consistent coding conventions, and manage code changes effectively.

## Coding Conventions

### File Naming
- Use **kebab-case** for all file names.
  - Example: `agent-skill.ts`, `user-profile.test.ts`

### Import Style
- Use **relative imports** for referencing other modules.
  - Example:
    ```typescript
    import { Skill } from './skill';
    ```

### Export Style
- Use **named exports** for all modules.
  - Example:
    ```typescript
    // skill.ts
    export function performSkill() { ... }
    ```

### Commit Messages
- Follow **Conventional Commits** with the `feat` prefix for new features.
  - Example:
    ```
    feat: add user authentication skill
    ```

## Workflows

### Feature Development
**Trigger:** When adding a new skill or feature  
**Command:** `/feature-development`

1. Create a new TypeScript file using kebab-case.
2. Implement the feature using named exports.
3. Import dependencies using relative paths.
4. Write corresponding test files with the `.test.ts` suffix.
5. Commit your changes using a conventional commit message (e.g., `feat: add new skill`).

### Testing
**Trigger:** When verifying code functionality  
**Command:** `/run-tests`

1. Identify or create test files matching the `*.test.*` pattern.
2. Run the test suite using your preferred test runner (framework not specified).
3. Review test results and fix any issues.

### Code Review & Commit
**Trigger:** Before pushing changes  
**Command:** `/prepare-commit`

1. Ensure all files follow kebab-case naming.
2. Check that all imports are relative and exports are named.
3. Verify commit messages use the `feat` prefix and are under 50 characters.
4. Push your branch for review.

## Testing Patterns

- Test files use the `*.test.*` naming convention (e.g., `agent-skill.test.ts`).
- The specific testing framework is not detected; use standard TypeScript testing tools (e.g., Jest, Mocha).
- Place tests alongside or near the modules they test.

**Example:**
```typescript
// agent-skill.test.ts
import { performSkill } from './agent-skill';

describe('performSkill', () => {
  it('should perform the skill correctly', () => {
    expect(performSkill()).toBe(true);
  });
});
```

## Commands
| Command             | Purpose                                      |
|---------------------|----------------------------------------------|
| /feature-development| Start a new feature or skill implementation  |
| /run-tests          | Run all tests in the codebase                |
| /prepare-commit     | Prepare code and commit for review           |
```
