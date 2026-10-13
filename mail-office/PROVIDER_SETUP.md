# GBSBFORYOU AI Mail Office — Secure Provider Setup

## Current status
This branch adds a mobile-first UI prototype only. It does not connect to any mailbox and its example messages are synthetic demo records. Do not treat the demo UI as a live inbox.

## Providers and authorization plan
- Gmail / Google Workspace: use Google OAuth 2.0 and Gmail API. Start with the least-privilege scope needed for listing/reading. Add compose/draft permission separately. Add send permission only when approval-gated send flow has been tested. Google may classify requested scopes as sensitive/restricted and require OAuth consent-screen configuration, verification, or other compliance steps depending on distribution and data use.
- Outlook / Microsoft 365: use Microsoft identity platform OAuth 2.0 and Microsoft Graph delegated permissions. Start with Mail.Read; add Mail.ReadWrite only for draft/folder/category functions; add Mail.Send only for the approved send action. Request only the permissions needed.
- Yahoo Mail: use Yahoo's supported OAuth/API path if available to this app/account; otherwise use provider-supported IMAP with TLS and OAuth where supported. Do not assume a password-based IMAP flow is available.
- Custom domain / other IMAP: configure provider hostname, port, TLS and OAuth/app-password policy per provider. Prefer OAuth; secrets must be stored only server-side and encrypted. Avoid unsupported providers rather than bypassing authentication.

## Required secure flow
1. User chooses a provider and completes its official sign-in/consent page.
2. OAuth callback is served only on the registered HTTPS redirect URI.
3. Authorization code is exchanged server-side; access/refresh tokens are never exposed to frontend JavaScript, Excel, logs, or public GitHub.
4. Store provider connection metadata and encrypted tokens in a server-side secret store or protected database; apply row-level security so each user can access only their own mailbox records.
5. Sync headers/metadata first. Fetch message bodies and attachments on demand; limit retention and allow disconnect/delete.
6. Draft creation stores a draft with provider + account identity clearly displayed.
7. Sending or destructive changes require an explicit per-action confirmation screen showing From account, recipient, subject, attachment names, and exact action. No bulk send, auto-send, delete or archive by default.
8. Log user ID, provider, action, timestamp, message ID and approval event; never log tokens or full sensitive message bodies.
9. Add CSRF/state/PKCE, strict redirect URI allowlist, rate limits, MIME/HTML sanitization, attachment size/type checks, malware scanning where available, SSRF protection for links, and audit logging.

## Environment configuration (names only; values must be set privately)
- PUBLIC_APP_URL
- GOOGLE_CLIENT_ID
- GOOGLE_CLIENT_SECRET (server-only)
- GOOGLE_OAUTH_REDIRECT_URI
- MICROSOFT_CLIENT_ID
- MICROSOFT_CLIENT_SECRET (server-only)
- MICROSOFT_TENANT (common or tenant ID, selected deliberately)
- YAHOO_CLIENT_ID / YAHOO_CLIENT_SECRET if supported for the chosen flow
- SUPABASE_URL
- SUPABASE_PUBLISHABLE_KEY (frontend only, with RLS)
- SUPABASE_SECRET_KEY (server-only; never client-side)
- TOKEN_ENCRYPTION_KEY (server-only secret manager)
Never commit .env files or real credentials.

## Recommended implementation phases
1. Prototype UI and provider registry.
2. Secure auth and read-only Gmail/Outlook connection.
3. Search, metadata sync, on-demand body and attachment download.
4. Draft creation; verify draft appears in provider's own Sent/Drafts area.
5. Approval-gated send, initially disabled behind a feature flag.
6. Yahoo/IMAP adapters one provider at a time, based on verified API/OAuth support.
7. Outreach OS integration: contacts, campaign/letter IDs, sent/received logs, follow-up reminders, and traceable source records.

## Acceptance tests before real use
- One test mailbox per provider; verify sign-in and account identity.
- Confirm user A cannot read user B's messages (RLS).
- Verify attachment download uses authorized message ID and safe file handling.
- Confirm no email is sent before explicit approval and confirmation screen.
- Confirm reply always uses the selected source account, not a default account.
- Revoke provider consent and confirm token deletion/disconnection.
- Confirm no token, message body, or attachment is exposed in logs or public pages.
