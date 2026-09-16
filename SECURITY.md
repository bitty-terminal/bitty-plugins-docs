# Security Policy

This document describes how security reports for the `bitty-plugins-docs`
repository are handled. Normative product security requirements live in the
canonical `bitty-docs` security corpus (`docs/security/`) and take precedence
over anything stated here.

## Supported Versions

| Version    | Supported                                         |
| ---------- | ------------------------------------------------- |
| Unreleased | No — no version of this project has been released |

There are currently no supported releases. Do not rely on this repository for
production use; all documented behavior is pre-implementation design material.

## Reporting a Vulnerability

Report security vulnerabilities privately by opening a
[GitHub Security Advisory](https://github.com/bitty-terminal/bitty-plugins-docs/security/advisories/new).

Do not report security vulnerabilities through public GitHub issues, pull
requests, or discussion channels.

When reporting, please include as much of the following as possible:

- A description of the vulnerability and its potential impact.
- Steps to reproduce, or a proof of concept.
- Affected specification sections, contracts, or generated outputs, and (once
  they exist) plugin, SDK version, and manifest.
- Any known mitigations or workarounds.

## Disclosure Policy

Reports are handled through coordinated disclosure:

1. The report is acknowledged and triaged privately.
2. A fix is developed and validated out of public view.
3. Once releases exist, a release containing the fix is published.
4. A public advisory is published afterward, crediting the reporter unless
   anonymity is requested.

## Response Expectations

The targets below take effect once this repository accepts them:

- Acknowledge a new advisory within 5 business days.
- Provide a status update at least every 14 calendar days while a report is
  open.
- Publish the advisory after a fixed version is available, or after 90 days if
  no fix is feasible, whichever comes first.

## Scope Note

This repository is a documentation corpus: it ships no executable product and
grants no runtime authority. In-scope findings include a specification that
would let a plugin escape the capability model, contradict a P0 trust boundary,
or define an install path that executes package code. Security requirements for
future implementations live in the canonical `bitty-docs` security corpus,
which takes precedence when statements conflict.
