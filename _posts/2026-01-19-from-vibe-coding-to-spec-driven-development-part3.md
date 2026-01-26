---
layout: post
title: "From Vibe Coding to Spec-Driven Development: Part 3 - Best practices and troubleshooting"
date: 2026-01-19
categories: [AI, Development, DevOps]
tags: [ai-assisted-development, spec-kit, github, copilot, vibe-coding, specification-driven-development, series]
author: hidde
description: "Part 3 of our series on mastering AI-assisted development. Learn advanced specification techniques, debugging strategies, iteration patterns, and real-world troubleshooting for production-ready AI-generated code."
image: /images/spec-kit/image-03.png
series: "Mastering Spec-Driven Development"
featured: false
toc: true
---

> 🎯 **This is Part 3 of a 5-part series on mastering AI-assisted development.**
>
> We've covered the why and the how. Now let's tackle the messy reality: debugging, iteration, and real-world troubleshooting.

## Series overview

| Part | Topic | Status |
|:-----|:------|:------:|
| [Part 1](/from-vibe-coding-to-spec-driven-development) | The problem and the solution | ✓ |
| [Part 2](/from-vibe-coding-to-spec-driven-development-part2) | Deep dive into the Spec-Kit workflow | ✓ |
| **Part 3** | **Best practices and troubleshooting** | 📍 |
| Part 4 | Team collaboration and advanced patterns | Jan 26 |
| Part 5 | Case studies and lessons learned | Feb 2 |

---

## The reality of spec-driven development

Last week, I was helping a team at a European airline build a proof-of-concept for managing contractors. Everything was going smoothly until the AI started generating code that referenced a workflow library that simply didn't exist. Once we'd untangled the hallucination, it got me thinking: **we need a field guide for when things go wrong.**

In Part 2, we walked through a clean, linear workflow: constitution → spec → plan → tasks → implementation. In reality, development is rarely that smooth.

You'll encounter:

| Challenge | What happens |
|:----------|:-------------|
| **AI hallucinations** | The AI invents APIs that don't exist |
| **Spec ambiguities** | Unclear requirements lead to wrong implementations |
| **Performance issues** | Code works but doesn't scale |
| **Integration failures** | Components don't play well together |
| **Requirement changes** | What you built isn't what users need |

This post is about handling those messy situations. **Think of it as your field guide for when things go wrong.**

---

## Section 1: Advanced specification techniques

### Writing specs that survive reality

The quality of your specification directly determines the quality of your output. Here's how to write specs that work in practice.

> 💡 **Quick win**
>
> Start with Technique #1 (Given-When-Then). It alone will significantly improve your AI output quality.

#### Technique #1: Use Given-When-Then scenarios

Instead of vague descriptions, use structured scenarios:

**DON'T** (Vague):
```markdown
Users should be able to filter tasks
```

**DO** (Structured):
```markdown
**Scenario: Filter tasks by assignee**

**Given** I'm viewing the team dashboard
**And** there are 50 tasks assigned to various team members
**When** I select "John Smith" from the assignee filter
**Then** I see only tasks assigned to John Smith (expected: 12 tasks)
**And** the count updates to show "Showing 12 of 50 tasks"
**And** the filter persists when I navigate away and return
```

**Why this works**: The AI has concrete inputs, actions, and expected outputs. No room for interpretation.

#### Technique #2: Specify error messages exactly

Don't say "show an error." Say exactly what the error should say.

**DON'T** (Vague):
```markdown
If the user enters an invalid email, show an error
```

**DO** (Specific):
```markdown
**Validation: Email format**

**Invalid inputs:**
- Empty string → "Email is required"
- "notanemail" → "Please enter a valid email address"
- "test@" → "Please enter a valid email address"
- "test@domain" → "Please enter a valid email address"

**Valid inputs:**
- "user@example.com" → Validation passes
- "user+tag@example.co.uk" → Validation passes

**Display:**
- Error appears below input field
- Input border turns red (#dc3545)
- Error icon displayed to the left of message
- Error clears immediately when user starts typing
```

#### Technique #3: Mock the UI with ASCII art

Don't assume the AI knows what you want visually.

```markdown
## Task Card Layout

┌─────────────────────────────────────────┐
│ [✓] Task title goes here        [Edit]  │
│                                          │
│ Brief description preview shown here,    │
│ truncated at 100 characters...          │
│                                          │
│ 👤 Assigned to: John Smith              │
│ 📅 Due: Jan 25, 2026                    │
│ 🏷️  Status: In Progress                 │
└─────────────────────────────────────────┘

**Interactions:**
- Checkbox: Click to toggle complete/incomplete
- Card body: Click to open detail view
- Edit button: Click to enter edit mode
- Hover: Box shadow elevation (0.5rem)
```

#### Technique #4: Define the negative space

Explicitly state what you're NOT building.

```markdown
## Out of Scope (Explicitly NOT Implementing)

### Comments on tasks
**Why**: Adds complexity without MVP validation
**Future consideration**: Phase 2 after user feedback

### File attachments
**Why**: Requires file storage infrastructure
**Alternative**: Use task description to link to external files

### Recurring tasks
**Why**: Complex scheduling logic
**Alternative**: Manual task duplication for now

### Subtasks/nested tasks
**Why**: Violates "Simplicity First" principle
**Alternative**: Use task dependencies in future phase
```

**Why this matters**: Prevents the AI from "helpfully" adding features you didn't ask for.

---

## Section 2: Debugging strategies

### When the AI goes off track

AI hallucinations are real. Here's how to diagnose and fix them.

> 📋 **TL;DR: The 5-step debug checklist**
>
> 1. Validate dependencies exist
> 2. Check for hallucinated APIs
> 3. Reset context if issues repeat
> 4. Provide minimal reproductions
> 5. Audit dependencies after updates

### Strategy #1: The validation loop

After implementation, run this three-step validation:

```bash
# Step 1: Verify dependencies exist
npm list  # Check all packages are installed
dotnet list package  # For .NET projects

# Step 2: Check for type errors
npm run typecheck  # TypeScript
dotnet build  # C#

# Step 3: Run tests
npm test  # Run test suite
dotnet test  # Run unit tests
```

**Common issues caught:**
- AI referenced packages that don't exist
- API endpoints that were never implemented
- Type mismatches between frontend and backend

### Strategy #2: The hallucination detector

AI models sometimes invent plausible-sounding but non-existent APIs.

**Example hallucination:**
```javascript
// AI-generated code
import { usePrismaQuery } from '@prisma/react';

// This hook doesn't exist!
const { data } = usePrismaQuery('users', { where: { email } });
```

**How to catch it:**

1. Check official documentation
2. Search GitHub for the import
3. Test in isolation

**Fix:**
```javascript
// Correct approach
import { prisma } from '@/lib/prisma';

// Manual query in server component
const data = await prisma.users.findUnique({ where: { email } });
```

### Strategy #3: The context refresh

When the AI keeps making the same mistake, it's context-poisoned.

**Symptom**: You fix a bug, the AI reintroduces it in the next iteration.

**Solution**: Reset context with explicit correction:

```
/implement fix and memorize: The TaskService uses async/await, NOT promises with .then()

INCORRECT PATTERN (never use):
```typescript
taskService.createTask(data).then(result => { ... })
```

CORRECT PATTERN (always use):
```typescript
const result = await taskService.createTask(data);
```

Update all existing code that uses .then() to use async/await.
Update the plan to reflect this as a coding standard.
```

The AI will:
1. Fix all instances
2. Update the plan to document the pattern
3. Remember the pattern for future tasks

### Strategy #4: The minimal reproduction

When something breaks, isolate it.

**INEFFECTIVE bug report**:
```
The app crashes when I try to create a task
```

**EFFECTIVE bug report**:
```
/implement fix bug: Creating a task with empty title crashes the application

**Steps to reproduce:**
1. Log in as user test@example.com
2. Navigate to /dashboard
3. Click "New Task" button
4. Leave title field empty
5. Enter description: "Test description"
6. Click "Create Task"

**Expected**: Validation error "Title is required"
**Actual**: Server crashes with 500 error

**Error log:**
```
TypeError: Cannot read property 'trim' of undefined
  at TaskService.createTask (services/TaskService.cs:47)
  at TaskController.Create (controllers/TaskController.cs:23)
```

**Hypothesis**: Title validation happens after attempting to trim, should happen before.
```

### Strategy #5: The dependency audit

When things mysteriously break after an update, check dependencies.

```bash
# Check for breaking changes
npm outdated  # Shows available updates
npm audit  # Security vulnerabilities

# Lock down working versions
npm ci  # Install exact versions from lockfile
```

**Real example**: Express 5.0 (released late 2024) changed middleware signatures. Code using Express 4.x patterns breaks silently.

**Solution**: Pin major versions in constitution:

```markdown
## Technical Constraints

### Dependency Versions (Pinned for Stability)
- express: ^4.18.0 (NOT 5.x until migration planned)
- react: ^18.2.0
- typescript: ^5.0.0

**Rationale**: Avoid breaking changes mid-development
```

---

## Section 3: Iteration patterns

### Evolving specs without chaos

Requirements change. Your spec needs to evolve. Here's how to do it systematically.

> 📐 **Rule of thumb**
>
> **Add** → Do it freely (safe)
> **Modify** → Document everything (risky)
> **Remove** → Phase it out gradually (dangerous)

### Pattern #1: Additive changes (safe)

**Scenario**: Add a new feature without modifying existing behavior.

**Example**: Add task comments to existing task system.

**Process:**

1. Update spec with new section:

```markdown
## Feature: Task Comments (Added 2026-01-19)

**As a team member**, I want to add comments to tasks to discuss details with my team.

### User Stories
[... detailed scenarios ...]

### Acceptance Criteria
[... specific criteria ...]
```

2. Run `/speckit.plan` → Generates updated plan
3. Run `/speckit.tasks` → Adds new tasks (existing tasks unchanged)
4. Run `/speckit.implement` → Implements only new tasks

**Result**: Clean addition, no regression risk.

### Pattern #2: Modificative changes (risky)

**Scenario**: Change existing behavior.

**Example**: Change task status from boolean (complete/incomplete) to enum (open/in-progress/completed).

**Process:**

1. Document the change explicitly:

```markdown
## Breaking Change: Task Status Model (Changed 2026-01-19)

**Previous behavior:**
- Task.complete: boolean (true/false)

**New behavior:**
- Task.status: enum (open | in_progress | completed)

**Migration plan:**
1. Add new status column
2. Migrate data: false → open, true → completed
3. Update API to accept status enum
4. Update UI to show three states
5. Remove old complete column

**Affected components:**
- Database schema (tasks table)
- TaskService.createTask, updateTask
- TaskCard component
- TaskDetailModal component
- API endpoints: PUT /api/tasks/:id
```

2. Run `/speckit.analyze` to find all affected artifacts

3. Update each artifact explicitly:

```
/implement migration: Task.complete (boolean) → Task.status (enum)

Follow the migration plan in spec.md exactly:
1. Add status column to database
2. Run data migration script
3. Update TypeScript types
4. Update API validation
5. Update UI components
6. Remove complete column

Test each step before proceeding to next.
```

**Result**: Controlled migration, clear rollback path.

### Pattern #3: Subtractive changes (dangerous)

**Scenario**: Remove a feature.

**Example**: Remove task due dates (users don't use them).

**Process:**

1. Document removal and rationale:

```markdown
## Feature Removal: Task Due Dates (Removed 2026-01-19)

**Rationale**: Usage analytics show only 3% of tasks have due dates.
Feature adds UI complexity without user value.

**Affected components:**
- Database: tasks.due_date column (keep for rollback, but don't display)
- API: Remove dueDate from POST/PUT payloads
- UI: Remove date picker from TaskDetailModal
- UI: Remove date display from TaskCard

**Rollback plan**: Set UI flag to re-enable if users complain.

**Migration:**
1. Hide UI controls (don't delete database column yet)
2. Monitor for 2 weeks
3. If no complaints, remove database column
4. If complaints, re-enable UI controls
```

2. Implement in phases:

```
Phase 1: Hide UI
/implement: Remove due date UI controls but keep database column

Phase 2: Monitor (2 weeks wait)

Phase 3: Remove infrastructure
/implement: Remove due_date column and API support
```

**Result**: Safe removal with rollback option.

---

## Section 4: Performance optimization

### Keeping the AI focused and fast

Large projects can overwhelm the AI's context window. Here's how to maintain performance.

### Optimization #1: Scope reduction

Don't send everything to the AI at once.

**SLOW**:
```
/implement entire application from spec
```

**FAST**:
```
/implement phase 3: User authentication only
```

**Why**: Smaller context = faster responses = fewer errors.

### Optimization #2: Reference by location

Instead of pasting entire files, reference them:

**BLOATED context**:
```
Here's my entire spec.md (5000 lines) ...
```

**EFFICIENT reference**:
```
/implement task 4.3 from tasks.md: Create TaskCard component

Refer to spec.md section "Task Card Layout" for requirements.
Refer to plan.md section "Component Architecture" for structure.
```

### Optimization #3: Checkpoint saves

Save progress frequently to avoid re-generation.

```bash
# After each phase
git add .
git commit -m "Phase 3 complete: Authentication working"
git tag phase-3-complete

# If AI goes off track, reset to checkpoint
git reset --hard phase-3-complete
```

### Optimization #4: Parallel work streams

Split independent features across multiple AI sessions.

**Example**: Task management and user authentication are independent.

**Session 1** (AI Agent A):
```
/implement phase 3: User authentication
```

**Session 2** (AI Agent B):
```
/implement phase 4: Task CRUD operations
(stub out authentication checks for now)
```

**Integration step** (merge):
```
/implement integration: Connect task CRUD to authentication
```

**Result**: 2x faster implementation, no conflicts.

---

## Section 5: Real-world edge cases

### Lessons from production deployments

These are real issues encountered in production spec-driven projects.

> ⚠️ **Warning**
>
> These aren't hypothetical. Each of these cost real teams real time. Learn from their mistakes.

### Edge Case #1: Concurrent updates conflict

**Scenario**: Two users edit the same task simultaneously. Last write should win with conflict notification.

**Spec requirement**:
```markdown
**Scenario: Concurrent task editing**

**Given** User A and User B both open Task #42 for editing
**When** User A changes title to "Updated Title A" and saves
**And** User B changes title to "Updated Title B" and saves
**Then** User B sees notification: "This task was updated by User A. Your changes may conflict. Reload to see latest version?"
**And** Latest saved version (User B's) is stored
**And** User B can choose to reload or force save
```

**Implementation challenge**: AI initially used naive timestamp comparison. Under load, race conditions caused data loss.

**Solution**: Implement optimistic concurrency with version tokens:

```csharp
public class Task
{
    public int Id { get; set; }
    public string Title { get; set; }
    
    [Timestamp]  // EF Core concurrency token
    public byte[] RowVersion { get; set; }
}

// In TaskService
public async Task<Task> UpdateTaskAsync(Task task)
{
    try
    {
        _context.Tasks.Update(task);
        await _context.SaveChangesAsync();
        return task;
    }
    catch (DbUpdateConcurrencyException ex)
    {
        // Conflict detected
        var entry = ex.Entries.Single();
        var databaseValues = await entry.GetDatabaseValuesAsync();
        var databaseTask = (Task)databaseValues.ToObject();
        
        throw new ConcurrencyException(
            $"Task was modified by {databaseTask.LastModifiedBy}. Reload to see latest version.",
            databaseTask
        );
    }
}
```

**Lesson**: Specify concurrency handling explicitly in spec. Don't assume AI will get it right.

### Edge Case #2: N+1 query problem

**Scenario**: Loading 100 tasks takes 5 seconds. Performance requirement: < 500ms.

**Root cause**: AI generated naive queries:

```csharp
// AI-generated (slow)
public async Task<List<Task>> GetTeamTasksAsync(int teamId)
{
    var tasks = await _context.Tasks
        .Where(t => t.TeamId == teamId)
        .ToListAsync();
    
    // Loops for each task - N+1 queries!
    foreach (var task in tasks)
    {
        task.Creator = await _context.Users.FindAsync(task.CreatorId);
        task.Assignee = await _context.Users.FindAsync(task.AssigneeId);
    }
    
    return tasks;
}
```

**Problem**: 100 tasks = 1 query + 100 queries for creators + 100 queries for assignees = 201 queries.

**Solution**: Use eager loading:

```csharp
// Optimized
public async Task<List<Task>> GetTeamTasksAsync(int teamId)
{
    return await _context.Tasks
        .Include(t => t.Creator)
        .Include(t => t.Assignee)
        .Where(t => t.TeamId == teamId)
        .ToListAsync();
}
```

**Result**: 100 tasks = 1 query. Performance improved from 5s to 120ms.

**Lesson**: Add performance requirements to constitution:

```markdown
## Quality Standards

### Database Query Performance
- All list queries MUST use eager loading (.Include())
- Queries returning > 10 rows MUST include execution plan review
- N+1 queries are PROHIBITED - PR will be rejected
```

### Edge Case #3: Memory leak in real-time connections

**Scenario**: Application memory grows unbounded. After 2 hours, server crashes.

**Root cause**: SignalR connections not cleaned up:

```csharp
// AI-generated (leaky)
public class TaskHub : Hub
{
    private static readonly List<string> _connections = new List<string>();
    
    public override async Task OnConnectedAsync()
    {
        _connections.Add(Context.ConnectionId);  // Added but never removed!
        await base.OnConnectedAsync();
    }
}
```

**Problem**: `_connections` list grows forever. Disconnected clients never removed.

**Solution**: Use proper cleanup:

```csharp
// Fixed
public class TaskHub : Hub
{
    // Use ConcurrentDictionary for thread-safe operations
    private static readonly ConcurrentDictionary<string, string> _connections 
        = new ConcurrentDictionary<string, string>();
    
    public override async Task OnConnectedAsync()
    {
        _connections.TryAdd(Context.ConnectionId, Context.UserIdentifier);
        await base.OnConnectedAsync();
    }
    
    public override async Task OnDisconnectedAsync(Exception exception)
    {
        _connections.TryRemove(Context.ConnectionId, out _);  // Cleanup!
        await base.OnDisconnectedAsync(exception);
    }
}
```

**Lesson**: Specify cleanup explicitly in spec:

```markdown
## Real-Time Requirements

### WebSocket Connection Management
- Connections MUST be tracked in thread-safe collection
- Disconnections MUST remove connection from tracking
- Server MUST handle abrupt disconnections (network failures)
- Connection cleanup MUST complete within 30 seconds
```

### Edge Case #4: XSS vulnerability in task descriptions

**Scenario**: User enters `<script>alert('XSS')</script>` as task description. Alert executes when other users view the task.

**Root cause**: AI didn't sanitize user input:

```cshtml
@* AI-generated (vulnerable) *@
<div class="task-description">
    @((MarkupString)Task.Description)  @* Renders raw HTML! *@
</div>
```

**Problem**: User-supplied HTML executed in other users' browsers.

**Solution**: Sanitize and escape:

```cshtml
@* Fixed - Razor automatically escapes *@
<div class="task-description">
    @Task.Description  @* Escaped by Blazor automatically *@
</div>
```

Or if Markdown support is needed:

```csharp
using Markdig;
using Ganss.Xss;

public string SanitizeMarkdown(string markdown)
{
    // Convert Markdown to HTML
    var html = Markdown.ToHtml(markdown);
    
    // Sanitize HTML (HtmlSanitizer uses whitelist approach by default)
    // Safe tags like p, strong, em, ul, ol, li are allowed by default
    // Dangerous tags like script, iframe are NOT in the default allowlist
    var sanitizer = new HtmlSanitizer();
    
    // Optionally restrict to only specific tags if needed:
    // sanitizer.AllowedTags.Clear();
    // sanitizer.AllowedTags.Add("p");
    // sanitizer.AllowedTags.Add("strong");
    // etc.
    
    return sanitizer.Sanitize(html);
}
```

**Lesson**: Add security requirements to constitution:

```markdown
## Security Standards

### Input Validation and Sanitization
- ALL user input MUST be sanitized before display
- HTML rendering MUST use whitelist approach (never blacklist)
- Markdown MUST be sanitized before rendering
- SQL queries MUST use parameterized queries (no string concatenation)
- OWASP Top 10 MUST be addressed for all features
```

---

## Section 6: Common gotchas and fixes

### Quick reference guide

#### Gotcha #1: AI forgets the constitution mid-implementation

**Symptom**: Early tasks follow standards, later tasks deviate.

**Fix**:
```
/implement reminder: Review constitution.md before each task
Verify compliance before proceeding.
```

#### Gotcha #2: Generated tests don't run

**Symptom**: Tests pass when AI runs them, fail when you run them.

**Cause**: AI assumes mocked data, you have real database.

**Fix**: Add test data setup to tasks:

```markdown
- [ ] 5.1 Write unit test for TaskService.createTask
  - **Setup**: Seed test database with team and user
  - **Test**: Call createTask with valid data
  - **Assert**: Task created with correct properties
  - **Teardown**: Clean up test data
```

#### Gotcha #3: Environment variables missing in production

**Symptom**: Works locally, crashes in production with "undefined environment variable."

**Cause**: AI hard-coded localhost values.

**Fix**: Add environment configuration to constitution:

```markdown
## Deployment Requirements

### Environment Variables (REQUIRED in production)
- DATABASE_URL: PostgreSQL connection string
- JWT_SECRET: Random 32-character string
- SENDGRID_API_KEY: Email service API key
- NODE_ENV: "production"

**Validation**: Application MUST fail to start if required env vars are missing.
Show error: "Missing required environment variable: DATABASE_URL"
```

#### Gotcha #4: API versioning not considered

**Symptom**: Breaking API changes force mobile app update.

**Cause**: Spec didn't mention API versioning.

**Fix**: Add to constitution:

```markdown
## API Design Principles

### Versioning Strategy
- All API endpoints MUST include version: /api/v1/tasks
- Breaking changes require new version: /api/v2/tasks
- Previous version MUST be supported for 6 months minimum
- Deprecation warnings in headers: `X-API-Deprecated: v1 will be removed on 2026-07-01`
```

#### Gotcha #5: AI uses deprecated patterns

**Symptom**: Code works but uses old syntax (e.g., `var` instead of `let/const`, callback-style instead of async/await).

**Cause**: AI trained on older codebases.

**Fix**: Add coding style requirements to constitution:

```markdown
## Code Style Requirements

### Modern Patterns (REQUIRED)
- Use async/await, NOT .then() callbacks
- Use const by default, let when reassignment needed, NEVER var
- Use arrow functions for callbacks
- Use template literals, NOT string concatenation
```

#### Gotcha #6: AI generates over-engineered solutions

**Symptom**: Simple feature implemented with 5 classes, 3 interfaces, and a factory pattern.

**Cause**: AI saw enterprise patterns in training data.

**Fix**: Add simplicity constraints to constitution:

```markdown
## Simplicity Constraints

### YAGNI Principle
- No interfaces with single implementations
- No factory patterns unless 3+ concrete types exist
- No dependency injection for classes with no dependencies
- Maximum 3 levels of class inheritance

Before adding abstraction, AI MUST justify with: "This abstraction is needed because..."
```

---

## Section 7: Measuring success

### How do you know if spec-driven development is working?

Track these five key metrics:

| Metric | What to Measure | 🟢 Good | 🟡 Warning |
|:-------|:----------------|:--------|:-----------|
| **Velocity** | Tasks completed per day | 5-10/day | < 3/day |
| **Regression** | Bugs reintroduced after fix | < 5% | > 20% |
| **Stability** | Spec changes per week | < 2 major | > 5 major |
| **Coverage** | Test coverage % | > 80% | < 50% |
| **Recovery** | Bug report → fix deployed | < 30 min | > 4 hours |

---

#### Fixing slow velocity

- Break tasks smaller (2-4 hours each)
- Simplify constitution (too many constraints?)
- Provide more context in spec

#### Fixing high regression

- Use `/implement fix and memorize` instead of just `/implement fix`
- Update plan.md with lessons learned
- Add regression tests to tasks

#### Fixing high spec churn

- Spend more time on spec before implementation
- Validate spec with stakeholders early
- Build MVP, then iterate based on feedback

#### Fixing low coverage

- Add test requirements to tasks explicitly
- Require test task for each feature task
- Run coverage reports in CI/CD

#### Fixing slow recovery

- Improve bug report templates (see Strategy #4 above)
- Add health monitoring endpoints
- Implement feature flags for quick rollback

---

## Section 8: Tools and automation

### Helpers that make spec-driven development smoother

### Tool #1: Specification validator

Create a script to validate spec completeness:

```bash
#!/bin/bash
# validate-spec.sh

echo "Validating specification..."

# Check for required sections
required_sections=("User Stories" "Acceptance Criteria" "Data Requirements" "Edge Cases")

for section in "${required_sections[@]}"; do
    if ! grep -q "## $section" .speckit/spec.md; then
        echo "[FAIL] Missing required section: $section"
        exit 1
    fi
done

# Check for TODO markers
if grep -q "TODO" .speckit/spec.md; then
    echo "[FAIL] Specification contains TODO markers"
    exit 1
fi

# Check for vague language
vague_terms=("might" "maybe" "probably" "hopefully" "basically")
for term in "${vague_terms[@]}"; do
    if grep -i -q "$term" .speckit/spec.md; then
        echo "[WARN] Spec contains vague term '$term'"
    fi
done

echo "[PASS] Specification validation passed"
```

Usage:
```bash
./validate-spec.sh
```

### Tool #2: Constitution compliance checker

Verify code follows constitution standards:

```bash
#!/bin/bash
# check-compliance.sh

echo "Checking constitution compliance..."

# Check nullable reference types enabled
if ! grep -q "<Nullable>enable</Nullable>" *.csproj; then
    echo "[FAIL] Nullable reference types not enabled (constitution requirement)"
    exit 1
fi

# Check for raw SQL (prohibited)
if grep -r "ExecuteSqlRaw\|FromSqlRaw" ./Services ./Controllers; then
    echo "[FAIL] Raw SQL detected (constitution prohibits)"
    exit 1
fi

# Check for console.log in production code (debugging leftovers)
if grep -r "Console.WriteLine" ./Services ./Controllers | grep -v "// DEBUG"; then
    echo "[WARN] Console.WriteLine found in production code"
fi

echo "[PASS] Constitution compliance check passed"
```

### Tool #3: Performance testing automation

Add to CI/CD pipeline:

```yaml
# .github/workflows/performance.yml
name: Performance Tests

on: [push, pull_request]

jobs:
  performance:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Build application
        run: dotnet build
      
      - name: Start application
        run: dotnet run &
        env:
          ASPNETCORE_ENVIRONMENT: Development
      
      - name: Wait for startup
        run: sleep 10
      
      - name: Run performance tests
        run: |
          # Test API response times
          response_time=$(curl -w "%{time_total}" -s -o /dev/null http://localhost:5000/api/health)
          
          if (( $(echo "$response_time > 0.5" | bc -l) )); then
            echo "[FAIL] Health check took ${response_time}s (requirement: < 500ms)"
            exit 1
          fi
          
          echo "[PASS] Health check took ${response_time}s"
```

### Tool #4: Spec change impact analyzer

When you change the spec, see what's affected:

```python
#!/usr/bin/env python3
# analyze-impact.py

import re
import sys

def analyze_spec_changes(old_spec, new_spec):
    """Compare two spec versions and identify affected components"""
    
    changes = {
        'added_sections': [],
        'modified_sections': [],
        'removed_sections': [],
        'affected_tasks': [],
        'affected_files': []
    }
    
    # Simple diff logic (extend as needed)
    old_sections = re.findall(r'^## (.+)$', old_spec, re.MULTILINE)
    new_sections = re.findall(r'^## (.+)$', new_spec, re.MULTILINE)
    
    changes['added_sections'] = [s for s in new_sections if s not in old_sections]
    changes['removed_sections'] = [s for s in old_sections if s not in new_sections]
    
    print("\n=== Spec Change Impact Analysis ===\n")
    
    if changes['added_sections']:
        print("[+] Added sections:")
        for section in changes['added_sections']:
            print(f"    - {section}")
    
    if changes['removed_sections']:
        print("\n[-] Removed sections:")
        for section in changes['removed_sections']:
            print(f"    - {section}")
    
    if not changes['added_sections'] and not changes['removed_sections']:
        print("[OK] No structural changes detected")
    else:
        print("\n[!] Actions required:")
        print("    1. Run /speckit.plan to update technical plan")
        print("    2. Run /speckit.tasks to regenerate task breakdown")
        print("    3. Review affected files before re-implementation")

if __name__ == '__main__':
    if len(sys.argv) < 3:
        print("Usage: ./analyze-impact.py old-spec.md new-spec.md")
        sys.exit(1)
    
    with open(sys.argv[1]) as f:
        old_spec = f.read()
    
    with open(sys.argv[2]) as f:
        new_spec = f.read()
    
    analyze_spec_changes(old_spec, new_spec)
```

Usage:
```bash
cp .speckit/spec.md .speckit/spec-old.md
# Make changes to spec.md
./analyze-impact.py .speckit/spec-old.md .speckit/spec.md
```

---

## Section 9: When to abandon spec-driven development

### It's not always the right tool

As Addy Osmani notes in [Beyond Vibe Coding](https://beyond.addy.ie/), the key is knowing when structure helps and when it hinders. Spec-driven development has overhead. Know when to use simpler approaches.

| ❌ Don't use spec-driven for | ✅ Do use spec-driven for |
|:------------------------------|:--------------------------|
| **One-off scripts**: vibe code is fine | **Production applications**: users depend on it |
| **Proof-of-concept demos**: iterate fast | **Team projects**: shared understanding needed |
| **Extremely simple apps**: 3 features max | **Regulated industries**: audit trails required |
| **Rapidly changing requirements**: daily pivots | **Long-lived systems**: years of maintenance |
| **Learning new tech**: experimentation first | **Complex domains**: intricate business logic |

---

## Recap: Best practices checklist

Use this before starting any spec-driven project:

| Phase | Checklist Item | |
|:------|:---------------|:--:|
| **📜 Constitution** | Specific technology constraints listed | ☐ |
| | Performance targets are measurable numbers | ☐ |
| | Security requirements reference OWASP Top 10 | ☐ |
| | Explicit non-goals prevent feature creep | ☐ |
| | Amendment process defined | ☐ |
| **📝 Specification** | User stories use Given-When-Then format | ☐ |
| | Error messages specified exactly | ☐ |
| | UI mockups provided (ASCII art is fine) | ☐ |
| | Edge cases documented with expected behavior | ☐ |
| | Success metrics are measurable | ☐ |
| **🗺️ Planning** | Technology choices align with constitution | ☐ |
| | Data model includes indexes for performance | ☐ |
| | API design is RESTful and consistent | ☐ |
| | Security patterns specified | ☐ |
| | Open questions documented | ☐ |
| **📋 Tasks** | Tasks are 2-4 hours each | ☐ |
| | Dependencies are explicit | ☐ |
| | Acceptance criteria per task | ☐ |
| | Parallel opportunities identified | ☐ |
| | Test tasks included for each feature | ☐ |
| **🚀 Implementation** | Test after each phase | ☐ |
| | Rich context for bug reports | ☐ |
| | Use `/implement fix and memorize` for recurring issues | ☐ |
| | Checkpoint progress with git tags | ☐ |
| | Run automated compliance checks | ☐ |

---

## What's next

In **Part 4** (next week), we'll explore:

- **Team workflows**: How multiple developers use Spec-Kit together
- **Review processes**: Code review checklist for spec-driven projects
- **CI/CD integration**: Automating validation and deployment
- **Advanced patterns**: Microservices, event-driven architectures
- **Scaling strategies**: When your application outgrows the initial architecture

---

## Key takeaways from Part 3

| # | Takeaway | Remember |
|:-:|:---------|:---------|
| 1 | **Specifications need precision** | Given-When-Then, exact error messages, UI mocks |
| 2 | **Debug systematically** | Validation loops, hallucination detection, context refresh |
| 3 | **Iterate carefully** | Add = safe, Modify = risky, Remove = dangerous |
| 4 | **Performance matters** | Watch for N+1 queries, memory leaks, concurrency |
| 5 | **Measure success** | Track velocity, regression rate, time to recovery |

---

**Next week in Part 4**, we'll tackle team collaboration and how Spec-Kit scales beyond solo developers.

---

## Resources

| Resource | Description |
|:---------|:------------|
| [**Spec-Kit GitHub**](https://github.com/github/spec-kit) | Official toolkit repository |
| [**Beyond Vibe Coding**](https://beyond.addy.ie/) | Addy Osmani's guide to AI-assisted development |
| [**OWASP Top 10**](https://owasp.org/www-project-top-ten/) | Security requirements reference |
| [**Exploring Gen AI**](https://martinfowler.com/articles/exploring-gen-ai.html) | Martin Fowler's AI development series |
| [**Given-When-Then**](https://martinfowler.com/bliki/GivenWhenThen.html) | BDD specification pattern |

---

## Series navigation

- **Previous**: [Part 2 - The Spec-Kit workflow](/from-vibe-coding-to-spec-driven-development-part2)
- **📍 You are here: Part 3 - Best practices and troubleshooting**
- **Next**: Part 4 - Team collaboration and advanced patterns (Coming January 26, 2026)
- Part 5 - Case studies and lessons learned (Coming February 2, 2026)

---

> 💬 **Found a gotcha not covered here?**
>
> Connect with me on [LinkedIn](https://linkedin.com/in/hiddedesmet) to share your experience.
>
> **Want to get notified when Part 4 drops?** Follow me for updates!
