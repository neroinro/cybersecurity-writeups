# HTB: NeoVault

> **Platform:** Hack The Box  
> **Difficulty:** Very Easy  
> **Category:** Web  
> **Status:** Completed

## 1. Challenge Overview

NeoVault is a banking application that allows users to perform fund transfers and download transaction statements.

The objective was to investigate the application's API, identify a potential vulnerability, and retrieve the challenge flag.

## 2. API Enumeration

While exploring the application, I identified the following endpoint:

```http
GET /api/v2/auth/inquire?username=neo_system
```

The response revealed the account's identifier:

```json
{
  "_id": "<redacted-user-id>",
  "username": "neo_system"
}
```


I also tested an object-style input intended to check for NoSQL injection. The application returned `user not found`, so this test did not establish a NoSQL injection vulnerability.

## 3. Discovering the Legacy Endpoint

The transaction statement feature used:

```http
POST /api/v2/transactions/download-transactions
```
<img width="473" height="253" alt="image" src="https://github.com/user-attachments/assets/9497f07e-d903-47a8-a66b-cedeb051f1e7" />

I changed the API version from `v2` to `v1`:

```http
POST /api/v1/transactions/download-transactions
```

The legacy endpoint indicated that an `id` parameter was required.

## 4. Retrieving a Transaction Statement

I supplied the identifier obtained from the inquiry endpoint.

The application returned a PDF statement associated with `neo_system`.

The statement contained a transaction involving an account named `user_with_flag`, with an amount of `1337.00`. This provided a lead for further investigation.

<img width="581" height="308" alt="image" src="https://github.com/user-attachments/assets/0afaf441-b2ac-464d-8ed9-4a33b67d374a" />

## 5. Flag Retrieval

Following the `user_with_flag` lead, I accessed the relevant account through the application's functionality and successfully retrieved the challenge flag.

**Flag:** `HTB{REDACTED}`

<img width="482" height="287" alt="image" src="https://github.com/user-attachments/assets/41286b0d-6650-4950-b0f0-560e73cad5ef" />

## 6. Vulnerability Analysis

The legacy endpoint accepted a client-supplied account identifier and returned a transaction statement.

This behavior suggests a possible **Broken Object Level Authorization (BOLA/IDOR)** vulnerability if the server failed to verify that the authenticated user was authorized to access the requested account.

The identifier itself is not the vulnerability. The security issue is unauthorized access to an object belonging to another user.

## 7. Impact

If proper authorization checks are missing, an attacker may be able to:

- Access other users' transaction statements.
- Expose sensitive financial information.
- Use disclosed transaction details to investigate other accounts.

## 8. Remediation

- Enforce server-side authorization for every requested account.
- Verify account ownership or access permissions for each statement request.
- Apply consistent authorization controls to both legacy and current API versions.
- Add automated tests for unauthorized cross-account access.

## 9. Key Takeaways

- Inspect API requests and responses during application exploration.
- Investigate legacy API versions.
- Follow useful clues found in application data.
- Verify authorization boundaries before classifying behavior as IDOR.
- Document the evidence and exact steps required to reproduce a finding.

## 10. Conclusion

The investigation of NeoVault's legacy transaction-download endpoint led to a statement containing a useful account clue, which helped me retrieve the challenge flag.

This challenge reinforced the importance of API enumeration, object-level authorization, and systematic evidence collection.
