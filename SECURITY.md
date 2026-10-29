# Security policy

## Reporting a vulnerability

**Please do not report security vulnerabilities through public GitHub issues, discussions or pull requests.**

Report them privately by either:

- emailing **[SECURITY CONTACT EMAIL, e.g. security@your-domain]**, or
- using GitHub's [private vulnerability reporting](https://docs.github.com/en/code-security/security-advisories/guidance-on-reporting-and-writing-information-about-vulnerabilities/privately-reporting-a-security-vulnerability) on this repository ("Security" tab → "Report a vulnerability").

Please include:

- the affected file, component or deployment configuration;
- steps to reproduce, or a proof of concept;
- the impact you believe it has (for example, cross-tenant data access, credential exposure, privilege escalation);
- any suggested fix.

Do not access, modify or download data that does not belong to you, and do not test against production deployments. This platform processes personal health data.

## What to expect

| Stage | Target |
|---|---|
| Acknowledgement of your report | within 3 working days |
| Initial assessment and severity | within 10 working days |
| Fix or mitigation for critical and high severity issues | as soon as possible, typically within 30 days |
| Public disclosure | coordinated with you after a fix is available |

We will credit you in the advisory unless you prefer to remain anonymous.

## Scope

In scope: the code and configuration in this repository, in particular authentication, authorisation, tenant isolation (partitions and the `partition_id` check) and image storage.

Vulnerabilities in HAPI FHIR itself or other upstream libraries should be reported to those projects. Tell us as well if the issue affects how this repository uses them.

## Supported versions

Only the latest commit on `main` and the latest published image are supported.
