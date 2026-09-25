# Odoo Web Security Checklist

## Route Boundary

- Is the route `public`, `user`, or `none` intentionally?
- Is the request type (`http` or `json`) appropriate?
- Are methods and route parameters constrained?
- Are authentication failures and missing records indistinguishable where enumeration matters?
- Are redirects restricted to safe local destinations?

## Record Boundary

- Does the current user have access to the model and operation?
- Is ownership, publication, company, or token validation explicit?
- Does the code browse with the current environment before any elevation?
- If `sudo()` is required, is the elevated recordset limited to the validated record and fields?
- Can another user's record be guessed, enumerated, or mutated?

## Input Boundary

- Are IDs, domains, sort fields, filters, and pagination bounded and validated?
- Are uploaded files limited by size, type, name, and storage policy?
- Are external URLs checked against an allowlist or safe scheme policy?
- Is user content parsed with a safe parser rather than `eval()`?

## Output Boundary

- Are text values rendered with escaped output such as `t-out`?
- Is any intentional HTML sanitized and narrowly sourced?
- Are JSON responses limited to fields the caller is allowed to see?
- Are secrets, internal IDs, debug details, and cross-company data excluded?
- Are translations and error messages safe for the output context?

## Browser Boundary

- Are state-changing actions protected against CSRF where applicable?
- Are cookies, tokens, and sensitive values kept out of URLs and logs?
- Are DOM insertion, `innerHTML`, and unsafe client-side URL uses reviewed?
- Are security controls tested with anonymous, portal, internal, and cross-company users?
