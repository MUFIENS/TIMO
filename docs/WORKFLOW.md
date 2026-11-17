# Development Workflow — Timo

## Guiding approach

Documentation-driven, skill-driven, test-first for the native tracking core, and milestone-based. Build the full product direction but implement in verified stages.

## Workflow for every task

1. **Orient:** inspect repository, branch, current code, and relevant docs.
2. **Skills:** follow `SKILL.md`; discover/read/apply relevant installed Antigravity skills.
3. **Clarify scope:** identify requirement IDs and acceptance criteria.
4. **Research:** use Context7/official docs for current APIs and dependency versions.
5. **Plan:** identify files, risks, tests, and any ADR required.
6. **Implement:** smallest coherent change; preserve existing conventions.
7. **Verify:** run targeted tests, broader tests/build, and manual checks as appropriate.
8. **Review:** security, privacy, accessibility, performance, and regression check.
9. **Document:** update specs, TODO, skill utilization log, and ADR if decisions changed.
10. **Report:** give factual summary with exact tests and limitations.

## Definition of Ready

- The task maps to a requirement or documented product decision.
- Relevant feature specification has been read.
- Required skill discovery has been performed.
- Acceptance criteria are testable.
- Dependencies and permissions are understood.

## Definition of Done

- Implementation matches requirement and is typed/documented.
- Tests pass and build succeeds.
- No unreviewed security/privacy regression.
- Accessibility/reduced motion considered for UI work.
- Performance impact checked for tracking/rendering work.
- Documentation and TODO are updated.
- Skill-use report reflects actual use.

## Branching and commits

Use small feature branches and descriptive commits. Do not rewrite history or discard user changes without explicit approval. Keep generated mascot assets and provider credentials out of source control unless the license and asset size justify storing them.

## Quality gates by area

- Tracking: native integration tests before dashboard dependence.
- Recognition: labeled corpus and user-correction tests.
- UI: visual audit at multiple window sizes and Windows scaling percentages.
- Mascot: actual GLB + clips + runtime proof, not only concept renders.
- Release: installer, uninstall, startup, data migration, privacy, and performance checks.
