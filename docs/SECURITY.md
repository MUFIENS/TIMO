# Security — Timo

## Security objectives

Protect local activity history, prevent unintended data disclosure, reduce attack surface in the desktop shell, and ensure discovered executables are treated as data rather than trusted code.

## Threat model

Assets:
- Usage history and application metadata.
- User display name and preferences.
- SQLite database and backups.
- API keys used only during optional development-time mascot generation.
- Update signing private key.

Threats:
- Malicious or malformed executable metadata.
- Path traversal or unsafe file handling.
- Renderer compromise attempting privileged commands.
- SQL injection through app names/categories/search.
- Secrets committed to source control.
- Diagnostic logs leaking usernames/full paths.
- Tampered update or installer.
- Corrupted database or interrupted writes.

## Required controls

### Tauri boundary
- Use least-privilege capabilities.
- Expose only required commands.
- Validate command input in Rust.
- Do not enable arbitrary shell execution or broad filesystem access from the renderer.
- Keep CSP restrictive and review remote content policy.

### Rust and process metadata (`ADR-006`)
- Never execute a discovered EXE or invoke `LoadLibrary` / shell icon handlers for identification or icon extraction; read PE `.rsrc` / version bytes strictly as read-only data.
- Do not build shell command strings from file paths or metadata.
- Normalize and validate paths defensively using documented Windows APIs.
- Never request UAC elevation when `OpenProcess` returns `ERROR_ACCESS_DENIED`; prioritize documented Win32 fallback APIs (`CreateToolhelp32Snapshot` / `WTSEnumerateProcessesW`) and do not rely on undocumented `NtQuerySystemInformation` as a primary dependency (`ADR-006`).
- Treat file descriptions, publisher strings, manifests, and package names as untrusted text.
- Handle access-denied and malformed metadata safely.

### Database
- Use parameterized SQL queries.
- Enable foreign key checks.
- Apply migrations transactionally.
- Restrict database file permissions to the current user where supported.
- Avoid placing the database in a shared/public directory.

### Secrets
- Never commit API keys, signing keys, or provider tokens.
- Use environment variables/local secret tooling for development credentials.
- Do not log authorization headers or complete environment dumps.
- Keep updater signing private keys outside the repository and CI logs.

### Logs and diagnostics
- Redact usernames, full paths, window titles, and tokens.
- Use structured logs and bounded rotation.
- Diagnostic export is user-initiated and reviewed for privacy.

### Updates and distribution
- Verify updater signatures.
- Publish checksums and release provenance where practical.
- Do not enable automatic updates until a signing key and secure release process are ready.
- Test installer/uninstaller and rollback behavior.

## Security testing

- Fuzz/validate metadata parsers where practical.
- Test malformed names, Unicode paths, long paths, deleted files, and access denied.
- Test crafted category/search values against SQL.
- Inspect Tauri capability configuration.
- Scan dependencies and licenses.
- Review network activity in release builds.

## Incident response

If unexpected data transmission, unauthorized file access, or signing-key exposure is found: disable the affected feature/release, preserve evidence without collecting extra user activity, rotate exposed secrets, document impact, and release a fix only after verification.
