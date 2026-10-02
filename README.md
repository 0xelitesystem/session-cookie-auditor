# session-cookie-auditor

Browser-based auditor for `Set-Cookie` headers. Paste one or more headers, get a hardening grade and a list of missing or weak attributes per cookie.

**Live demo:** https://0xelitesystem.github.io/session-cookie-auditor/

Single HTML file. No build step, no dependencies, no network calls. Forensic-style output: monospaced finding blocks with redacted spans for cookie values.

## Use

1. Paste one or more raw `Set-Cookie` headers into the box, one per line, with or without the `Set-Cookie:` prefix. Or click one of the sample buttons.
2. Click Audit.
3. Read the grade and the findings for each cookie, then fix the missing or weak attributes on your server.

## Why this exists

Cookie attributes like `Secure`, `HttpOnly` and `SameSite` are easy to miss and nothing complains when they are absent. This auditor grades the header text you paste, without sending it anywhere. It is one HTML file with no tracking and no network calls. MIT licensed.

## What it checks

For every parsed `Set-Cookie` header:

- **`Secure`**, required if cookie carries authentication; flags cookies sent over plaintext.
- **`HttpOnly`**, required for session and authentication cookies; flags cookies accessible to `document.cookie`.
- **`SameSite`**, flags missing attribute (treated as `Lax` by modern browsers, but explicit is safer), `SameSite=None` without `Secure` (rejected by Chrome), and `SameSite=None` on cookies that don't need cross-site context.
- **`Domain`**, flags overly broad scopes (parent domain when child would suffice).
- **`Path`**, flags `Path=/` on cookies that should be scoped to a sub-path.
- **`Expires` / `Max-Age`**, flags missing values (session cookies are fine, but auth tokens with no rotation policy are a concern), unreasonably long lifetimes (>1 year), and conflicting values.
- **`__Host-` and `__Secure-` prefixes**, checks that cookies using these prefixes meet the prefix requirements; suggests prefixes for cookies that look auth-related.
- **Name patterns**, heuristic match for names like `session`, `sid`, `token`, `auth`, `jwt`, `csrf` to apply stricter rules.

## Hardening grade

A composite letter grade A, F per cookie, derived from missing critical attributes:

- **A**, `Secure`, `HttpOnly`, `SameSite=Strict` or `Lax`, scoped `Path` and `Domain`, `__Host-` or `__Secure-` prefix where applicable
- **B**, one minor attribute missing or weak
- **C**, `SameSite` missing or attribute combination weakens isolation
- **D**, `HttpOnly` missing on auth cookie, or `Secure` missing
- **F**, multiple critical attributes missing, or `SameSite=None` without `Secure`

## Severity scale

| Tag | Meaning |
|-----|---------|
| critical | Auth cookie exposed to JS, or sent over plaintext |
| high | CSRF defense missing or weakened |
| medium | Scope too broad, lifetime too long |
| low | Hygiene (missing prefix, vague name) |
| info | Observation without strong recommendation |

## What this is NOT

Not a runtime tester. It does not make HTTP requests, follow redirects, or evaluate how the cookie behaves under specific browser versions. It parses the literal header text you paste. For dynamic testing, use the browser devtools, Burp Suite, or `OWASP ZAP`.

## Privacy

Cookie values are masked in the output (only attribute structure is shown). Nothing leaves the browser. No analytics, no storage.

One exception: if you click the theme toggle, your light or dark choice is saved in your browser's localStorage under the key `theme`. Nothing you type is stored.

## Run locally

```bash
git clone https://github.com/0xelitesystem/session-cookie-auditor
cd session-cookie-auditor
```

Open `index.html` in any browser. Or serve the folder with `python -m http.server 8000` and visit http://localhost:8000/.

## Build

No build step. The whole tool is one `index.html` file with its CSS and JavaScript inline.

## Samples

Three samples are wired to header buttons: a hardened auth cookie, a permissive third-party tracking cookie, and a cookie with conflicting attributes.

## Related repositories

Part of a 10-repo security audit set.

Browser-based audit tools:
- [iam-policy-analyzer](https://github.com/0xelitesystem/iam-policy-analyzer)
- [terraform-security-linter](https://github.com/0xelitesystem/terraform-security-linter)
- [kubernetes-manifest-security-scanner](https://github.com/0xelitesystem/kubernetes-manifest-security-scanner)
- [regex-redos-checker](https://github.com/0xelitesystem/regex-redos-checker)

Reference collections:
- [incident-response-runbooks](https://github.com/0xelitesystem/incident-response-runbooks)
- [ai-llm-security-audit](https://github.com/0xelitesystem/ai-llm-security-audit)
- [api-security-audit-checklist](https://github.com/0xelitesystem/api-security-audit-checklist)
- [secrets-leak-response-runbook](https://github.com/0xelitesystem/secrets-leak-response-runbook)
- [threat-modeling-worksheets](https://github.com/0xelitesystem/threat-modeling-worksheets)

## More

Part of a catalog of single-file browser tools and plain-language references, all MIT licensed and dependency-free: [0xelitesystem.github.io](https://0xelitesystem.github.io/). Built by [elitesystem.ai](https://elitesystem.ai).

## License

MIT. See [LICENSE](LICENSE).
