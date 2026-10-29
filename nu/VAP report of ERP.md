# VAP Report of ERP

**Assessment type:** Preliminary, limited-scope external web security assessment

**Target:** https://nucleus.niituniversity.in/
**Assessment date:** 29 October 2026
**Environment:** Kali Linux
**Tools:** curl, OpenSSL, Docker, OWASP ZAP
**Assessment type:** External, unauthenticated, passive baseline assessment

## 1. Executive Summary

An initial security assessment was performed against the specified
university website using HTTP/TLS checks and an OWASP ZAP baseline scan.

The root URL returned HTTP 200. TLS certificate verification succeeded,
and several security-related HTTP response headers were observed.

ZAP discovered 70 URLs and reported 53 PASS checks, 15 warning types,
and zero FAIL alerts. These scanner results do not establish that the
website is free of vulnerabilities.

The assessment identified configuration observations involving Content
Security Policy (CSP), wildcard Cross-Origin Resource Sharing (CORS),
third-party resource integrity, and potential CSRF protection gaps.
Exploitability has not been established for these observations.

## 2. Scope and Limitations

**In-scope hostname:** nucleus.niituniversity.in

Testing performed:
- DNS resolution
- HTTPS response and response-header inspection
- TLS certificate inspection
- OWASP ZAP baseline crawling and passive analysis

No active exploitation, credential attacks, destructive testing, or
state-changing form submissions were performed.

The results do not constitute a comprehensive penetration test.
Authenticated functionality, authorization controls, server-side
business logic, and other untested components remain outside the
verified results of this assessment.

Authorization was reported as confirmed by the assessor. The written
approval reference and detailed exclusions should be retained with
the assessment records.

## 3. Technical Environment and Results

### DNS
Resolved address: 20.219.206.11

### HTTP
Requested URL: https://nucleus.niituniversity.in/
Observed status: HTTP 200
curl TLS verification result: 0

### TLS certificate
Subject: CN=*.niituniversity.in
Issuer: Go Daddy Secure Certificate Authority - G2
Valid from: 30 January 2026
Valid until: 3 March 2027
Subject Alternative Names:
- *.niituniversity.in
- niituniversity.in

The wildcard certificate covers the assessed hostname. The recorded
certificate dates were valid on the assessment date.

### Security headers observed
- Strict-Transport-Security: max-age=31536000; includeSubDomains
- X-Content-Type-Options: nosniff
- Permissions-Policy was present.
- Content-Security-Policy was present.
- Cache-control and Pragma directives were present.

## 4. Preliminary Findings

### WEB-001 — CSP Hardening Opportunity

**Status:** Observed configuration weakness; exploitability unverified  
**Severity:** Provisional; reassessment required

Evidence:
The observed Content-Security-Policy includes unsafe-inline and
unsafe-eval in script-src, and unsafe-inline in style-src.
The policy also permits numerous external resource origins.

Impact:
These directives reduce the protection CSP can provide against
certain script-injection attacks. No XSS exploit was demonstrated.

Recommendation:
Review application dependencies and remove unsafe-inline and
unsafe-eval where feasible. Prefer nonce- or hash-based script
policies, restrict permitted origins, use HTTPS for external resources,
and explicitly define appropriate directives such as base-uri and
form-action. Test the revised policy for application compatibility.

### WEB-002 — Wildcard CORS Policy Observed

**Status:** Observed; security impact unverified  
**Severity:** Informational pending endpoint review

Evidence:
A GET request to the root URL returned:
Access-Control-Allow-Origin: *
Access-Control-Allow-Methods: POST, OPTIONS
Access-Control-Allow-Headers: Content-Type

No Access-Control-Allow-Credentials header was observed in that
response.

Impact:
The wildcard permits cross-origin JavaScript to read eligible
non-credentialed responses. The tested root response was public.
No unauthorized access to private information was demonstrated.

Recommendation:
Review the intended CORS policy for each API and sensitive endpoint.
Keep wildcard access only where public cross-origin access is intended.
Restrict private APIs to specifically approved origins as appropriate.

### WEB-003 — Subresource Integrity Review

**Status:** Hardening opportunity  
**Severity:** Provisional; impact unverified

Evidence:
ZAP identified external resource references without integrity
attributes, including a Bootstrap stylesheet hosted on
maxcdn.bootstrapcdn.com and a script hosted on apis.google.com.

Impact:
For suitable static third-party resources, missing Subresource
Integrity (SRI) removes a browser mechanism for verifying that
downloaded content matches an expected cryptographic hash.

Recommendation:
Use supported, pinned resource versions and SRI where appropriate.
Review dynamic third-party scripts separately when stable integrity
hashes are not practical. Remove unnecessary third-party dependencies.

### WEB-004 — Potential Missing CSRF Protections

**Status:** Unverified scanner finding  
**Severity:** Not assigned pending validation

Evidence:
ZAP did not identify a known anti-CSRF token in several HTML forms,
including forms on the root page and ForgotPassword.aspx.

Impact:
A vulnerability is not established merely by the absence of a
recognizable token. The relevant operations, server-side validation,
and alternative protections require review.

Recommendation:
Using an explicitly authorized test account and approved test
environment, verify CSRF protections for relevant state-changing
operations. Use appropriate server-validated CSRF tokens and
defense-in-depth controls where needed.

No form submissions or state-changing requests were tested in
this assessment.

### WEB-005 — Potential XSS Hotspots

**Status:** Unconfirmed; not a demonstrated vulnerability  
**Severity:** Informational pending validation

Evidence:
ZAP reported user-controllable HTML attribute hotspots involving
fields such as __PREVIOUSPAGE and __VIEWSTATEGENERATOR.

Impact:
The scanner identified locations requiring review. No executable
cross-site scripting payload or JavaScript execution was demonstrated.

Recommendation:
Review server-side input handling and context-appropriate output
encoding. Perform any additional validation only within the approved
testing scope.

### WEB-006 — Debug Message Alert

**Status:** False-positive candidate

Evidence:
ZAP reported possible debug error messages in CSS resources.
The extracted evidence included the phrase "Under Construction".

Impact:
This text alone does not establish exposure of internal error details.

Recommendation:
Review the full responses. Exclude this as a confirmed finding unless
actual diagnostic or sensitive internal information is identified.

## 5. Other Scanner Observations

ZAP also reported:
- CSP directives requiring further review
- Missing Cross-Origin-Embedder-Policy and
  Cross-Origin-Opener-Policy headers
- SameSite=None on a cookie
- Third-party JavaScript inclusion
- Suspicious source comments
- Potentially dangerous JavaScript functions
- Informational authentication and session-management alerts

These observations have not been established as independently
exploitable vulnerabilities. Their importance depends on the affected
resource, application design, and supporting evidence.

## 6. Risk Summary

Confirmed exploitable vulnerabilities: None demonstrated during
this limited assessment.

Configuration observations requiring review:
- CSP permits directives that weaken its protection.
- A wildcard CORS header was returned for the public root resource.
- External resources were identified without SRI attributes.

Unverified candidates:
- Missing anti-CSRF tokens
- XSS hotspots
- Possible debug-message disclosure

No overall security certification or assurance that the entire
university environment is secure is implied by these results.

## 7. Recommended Next Steps

1. Review the exact CSP directives and update the policy where feasible.
2. Establish intended CORS behavior across relevant endpoints.
3. Review external resource integrity and dependencies.
4. Validate CSRF controls through explicitly approved testing.
5. Review XSS hotspots and suspected debug disclosures.
6. Document remediation evidence and retest approved findings.

## 8. Evidence Files

- evidence/assessment-time.txt
- evidence/dns.txt
- evidence/root-summary.txt
- evidence/root-headers-sanitized.txt
- evidence/tls-certificate.txt
- reports/zap-baseline.html
- reports/zap-baseline.md
- reports/zap-baseline.json

**Report status:** Preliminary; findings require review and, where
indicated, further authorized validation.
