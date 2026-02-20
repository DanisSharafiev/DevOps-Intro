# Lab 1 Submission - Introduction to DevOps & Git Workflow

## Task 1 — SSH Commit Signature Verification

### Benefits of Signing Commits
Signing commits provides cryptographic proof of the author's identity. It prevents malicious actors from impersonating developers and injecting malicious code under a trusted name. In a DevOps workflow, this is a cornerstone of supply chain security, ensuring that every change in the repository can be traced back to a verified entity.

### "Why is commit signing important in DevOps workflows?"
In automated DevOps pipelines, verifying the authenticity of code is critical. If an attacker can push code that looks like it came from a lead developer, they could bypass manual scrutiny or trust-based automated checks. Signed commits effectively mitigate this risk by linking every commit to a private key held only by the authorized developer. It builds a chain of trust from the developer's machine to the production deployment.

### Evidence of Setup
**Git Configuration:**
```
user.signingkey=C:/Users/harne/.ssh/id_ed25519.pub
commit.gpgsign=true
gpg.format=ssh
```

**Verification Badge:**
![Verified commit screenshot](screenshots/verified_commit.png)
