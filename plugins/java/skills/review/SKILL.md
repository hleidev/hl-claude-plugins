---
name: review
description: "Enterprise Java/Spring code review — design, architecture, layering, complexity, naming, and framework-specific rules, graded [MUST]/[WARN]/[HINT]. Reviews the changed code on the current branch against Alibaba P3C and Google Engineering Practices, and verifies every blocking finding before reporting it. Java/Spring only — use /code-review for other languages."
disable-model-invocation: true
allowed-tools: Bash(git diff:*) Bash(git log:*) Bash(git show:*) Bash(git status:*) Read Grep
argument-hint: "[dimension]"
---

Review the changed code on the current branch against enterprise engineering standards. ultrathink

$ARGUMENTS

If a dimension is specified above (e.g. `complexity`, `architecture`, `naming`), focus only on that dimension. Otherwise run a full review across all dimensions.

## Output language

Default output language is **Chinese**. If the user writes in — or explicitly asks for — another language, produce all output (findings, summary, and any commentary) in that language instead. The "write in Chinese" instruction below follows this setting.

## Scope

Commits:
!`git log main...HEAD --oneline --no-merges 2>/dev/null || git log HEAD -5 --oneline`

Changed files:
!`git diff main...HEAD --name-only 2>/dev/null || git diff HEAD --name-only`

Diff:
```!
git diff main...HEAD 2>/dev/null || git diff HEAD
```

Read each changed file in full before reviewing.

## Review Order (design before details)

1. **Design first** — Is the overall approach sound? Does the change belong here or in a shared layer/library?
2. **Architecture & layering** — Are layer boundaries respected?
3. **Complexity** — Is the code harder than it needs to be?
4. **Code quality** — Naming, magic values, dead code, duplication
5. **Java/Spring specifics** — Framework rule compliance

## Checklist

### Design & Architecture (Alibaba P3C · Google Eng Practices)
- Business logic lives in Service, not Controller or Repository/Mapper
- Controllers only handle request binding and response mapping
- Entities are never returned directly as API responses — use DTOs
- New utility logic belongs in an existing utility class or established library (Commons/Hutool/Guava), not inline

### Complexity (Google: "can't be understood quickly = flag it")
- Methods over 30 lines warrant scrutiny — does this need splitting?
- Nested conditionals beyond 3 levels — consider early returns or extraction
- No over-engineering for hypothetical future requirements
- No clever one-liners that sacrifice readability

### Code Quality
- No magic numbers — extract to named constants
- No hardcoded config values — use `@Value` or `application.yml`
- Dead code not left behind from this change
- Duplicate logic — check for extractable patterns
- Boolean method/field names prefixed with `is` / `has` / `can`

### Java / Spring (Alibaba P3C)
- `@Transactional` on Service layer only — never Controller or Mapper
- SLF4J for all logging — no `System.out.println`
- No sensitive data in logs (passwords, tokens, PII) — check log call arguments
- Log levels match severity: DEBUG diagnostics, INFO normal flow, WARN degraded state, ERROR failures
- Do not catch bare `Exception` or `RuntimeException` without a pre-check justification
- Lombok used where it reduces boilerplate (no manual getters/setters)
- No N+1 query patterns in MyBatis mapper calls

### Naming (Alibaba P3C Mandatory)
- Class names: UpperCamelCase nouns (`OrderService`, not `orderservice` or `OrderSvc`)
- Method names: lowerCamelCase verb phrases (`getUserById`, not `user` or `GetUser`)
- Constants: `UPPER_SNAKE_CASE` declared as `static final`
- Package names: all lowercase, no underscores
- No cryptic abbreviations — `userId` not `usrId`, `orderList` not `ol`

### Input Validation
- Controller params or DTO annotated with `@Valid` / `@Validated`
- DTO fields carry constraint annotations (`@NotNull`, `@NotBlank`, `@Size`, etc.)
- No hand-rolled null checks replacing framework validation

### Dependency Injection
- Constructor injection preferred — `@Autowired` field injection is discouraged (P3C)
- `@RequiredArgsConstructor` (Lombok) is acceptable as constructor injection shorthand
- No circular dependencies introduced

### Exception Handling
- Business exceptions extend a custom base exception, not bare `new RuntimeException("...")`
- No empty catch blocks — swallowed exceptions are a [MUST] violation
- `@RestControllerAdvice` / `GlobalExceptionHandler` handles unified error responses — no ad-hoc try/catch in Controller

### API Response & Pagination
- All endpoints return the project's unified response wrapper (e.g. `Result<T>`) — no naked POJOs
- List endpoints return `Page<T>` / `IPage<T>`, never unbounded `List<T>`
- Error responses include a meaningful code and message

### Comments
- Comments explain **why**, not what — remove comments that just restate the code
- Complex algorithms or non-obvious constraints should have a brief explanation

## Severity Classification

| Level | Meaning | Action |
|---|---|---|
| **[MUST]** | Architecture violation or logic defect — blocks commit | Fix before proceeding |
| **[WARN]** | Code quality or standards issue | Fix unless there's a reason not to |
| **[HINT]** | Optional improvement | Mention only, don't push |

Reference: Alibaba P3C classifies rules as Mandatory / Recommended / Reference — same intent.

## Verification (run before writing the report)

Every [MUST] is a claim about code you have not fully read. Verify each one
before it reaches the report.

For each [MUST] candidate, name the assumption it rests on, then confirm that
assumption with Read or Grep — outside the changed files:

| Finding | Assumption to confirm | How |
|---|---|---|
| `@Transactional` ineffective | Called from another bean, not self-invoked | Grep the method name for call sites |
| Business logic in Controller | Not a thin delegation to a Service | Read the Controller method in full |
| Entity returned as response | The type is an entity, not a same-named DTO | Read the class declaration |
| N+1 query | Mapper call is in a loop and has no batch variant | Read the loop and the mapper interface |
| Magic number | No named constant for that value exists elsewhere | Grep the literal across the module |
| Missing `Result<T>` wrapper | The project actually defines such a wrapper | Grep for the wrapper type repo-wide |

Rules:
- A finding must quote the source line that triggered it and name the rule it
  violates. If you cannot produce both, drop it — do not downgrade it to [HINT].
- A convention this project does not use is not a violation. If `Result<T>`
  appears nowhere in the repo, the rule does not apply here.
- If verification is inconclusive, report it under [HINT] prefixed with
  `待确认` — never under [MUST].
- [WARN] and [HINT] need only the quoted source line, no assumption check.

## Output Format

**Output — write in Chinese** (see "Output language" above):

```
## Code Review — <branch or commit range>

### [MUST] Findings
- `UserController.java:42` — Business logic (password validation) belongs in UserService, not Controller

### [WARN] Findings
- `OrderService.java:87` — Magic number `7` should be extracted to a named constant (e.g. DEFAULT_EXPIRE_DAYS)

### [HINT] Suggestions
- `UserMapper.java:15` — Hutool's `StrUtil.isBlank` replaces the manual null-check here

---
Summary: 1 MUST · 2 WARN · 1 HINT — 复核 5 条，丢弃 2 条
```

After reporting, ask: **"Fix all [MUST] and [WARN] issues? (y/n)"** — do not auto-fix without confirmation.

## Rules

- Do not manufacture findings to fill the report — "no [MUST] findings" is a
  valid result

## Out of Scope

- Security vulnerabilities → use `/security-review`
- Non-Java/Spring code → use `/code-review`
- Test coverage and UI/styling — not covered here
