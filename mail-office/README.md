# GBSBFORYOU AI Mail Office

Mobile-first unified inbox prototype for the existing GBSBFORYOU Outreach OS.

## Status
**Prototype only.** No live mailbox is connected. All visible emails are demo data. Draft preview is local UI only and does not save or send email.

## Planned providers
Gmail / Google Workspace, Microsoft Outlook / Microsoft 365, Yahoo Mail, custom-domain IMAP, and other providers only when supported by a verified secure adapter.

## Human approval rule
Every send, delete, archive, label/folder mutation, and other account-changing action must show the exact account and action and require Ajay's explicit approval. No auto-send or bulk-send.

## Files
- `mail-office/index.html`: mobile-first interface prototype
- `mail-office/PROVIDER_SETUP.md`: provider auth plan, security rules and acceptance tests

## Next steps
Configure official OAuth apps and HTTPS redirect URIs, provision a dedicated backend and protected database, then connect read-only Gmail/Outlook before adding draft creation. Keep this separate from the AI Guru Supabase project until the mail system has its own approved data boundary.
