# Security policy

## Report a vulnerability

Email [support@formbase.so](mailto:support@formbase.so) with the subject "Security". Do not open a public issue.

Include:

- what you found and where (the tool, the URL or the step in the sign-in flow)
- how to reproduce it
- what an attacker could do with it

Never include a real token (`fb_...` or `fbo_...`) or a recipient's answers. If a token leaked, delete it under **OAuth and API Keys** in your workspace sidebar first.

## Scope

This repository holds documentation and example configs only. The policy covers the hosted server at `https://api.formbase.so/api/mcp` and its OAuth sign-in.
