# EU Compliance Rule Catalog (skeleton)

Rulesets tracked for GlobalReel:

| ID prefix | Source | Scope |
|-----------|--------|-------|
| AVMSD-6A  | Audiovisual Media Services Directive, Art. 6a | Child protection, harmful content |
| DSA-14    | Digital Services Act, Art. 14 | Notice-and-action, transparency |
| CDSM-17   | Copyright DSM Directive, Art. 17 | Platform copyright liability |
| GDPR-*    | GDPR | Personal data, consent, minors |
| UCPD-*    | Unfair Commercial Practices Directive | Misleading ads, native content |

Each rule is encoded as `{ id, severity, pattern, markets, remediation }`. Full schema lives in `src/schema.ts` once implemented.
