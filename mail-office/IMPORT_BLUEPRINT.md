# AI Mail Office — Email Import & Intelligence Blueprint

**Product goal:** Users import copies of their own email into a separate workspace. The product manages imported data; Gmail/Outlook are sources, not the product's primary inbox UI.

## 1. MVP scope

### Supported import formats
- **MBOX:** first-class import format for Gmail/Google Takeout email exports.
- **EML:** individual message import; batch EML import can follow.
- **ZIP containing MBOX/EML:** extract and validate with size and file-count limits before import.
- **PST/OST:** out of MVP scope; add only through a tested parser or documented conversion workflow.
- CSV/JSON contact and metadata imports may be added separately; they are not email-message substitutes.

Gmail Takeout exports can include message bodies, headers, attachments, and Gmail labels. Encrypted messages may remain encrypted after export. Source: Google Gmail Help, “Export your data from Gmail.”

## 2. Import flow

1. User signs into the AI Mail Office workspace.
2. User selects a local export file and confirms they are authorized to process it.
3. Validate file type, file size, archive expansion ratio, entry count, MIME structure, and malware scan status.
4. Parse messages into a staging area; show a preview with message count, date range, senders, attachments, and parse warnings.
5. User confirms the import.
6. Deduplicate using provider message ID when available; otherwise use a conservative fingerprint of normalized headers, date, and content. Never discard a message solely because its subject matches.
7. Store the original message and attachments separately from extracted/searchable fields.
8. Create search index and thread links; retain source filename and import batch ID.
9. Run AI extraction only after the user enables it; show citations back to the source message.
10. Show import report: imported, duplicates, skipped, failed, encrypted/unreadable, and attachment totals.

## 3. Initial data model

- **workspaces:** owner, name, retention policy, created_at
- **mail_accounts:** workspace_id, provider/source label, address if known, import-only vs connected
- **import_batches:** workspace_id, filename, checksum, started_at, status, totals, errors
- **messages:** workspace_id, account_id, provider_message_id, message_id_header, thread_key, from/to/cc, subject, sent_at, received_at, body_text, body_html, labels, attachment_count, source_batch_id, content_hash
- **attachments:** workspace_id, message_id, safe_filename, MIME type, size, object_storage_key, scan_status
- **message_facts:** workspace_id, message_id, fact_type, fact_value, confidence, source_span
- **action_items:** workspace_id, message_id, action_type, summary, due_at, status, user_confirmed
- **reply_drafts:** workspace_id, source_message_id, recipient, subject, body, status, created_by_ai, approved_at
- **audit_events:** workspace_id, actor_id, action, target_id, timestamp, metadata_redacted

Every user-owned record must be scoped by workspace_id and protected with database row-level security. Never expose provider tokens, full message bodies, or private attachments in public repositories, client logs, analytics, or error reports.

## 4. Intelligence layer

Classify messages into configurable categories:
- Reply required
- Deadline / appointment / follow-up
- Important reference or document
- Financial / invoice / payment
- Government / legal / official correspondence
- Newsletter / promotion
- Low priority / bulk
- Potential phishing / suspicious

Extract facts with provenance:
- People and organizations
- Dates and deadlines (including timezone where available)
- Amounts and currencies
- Requests, commitments, decisions, and missing documents
- Suggested next action and suggested response

Every generated fact or summary must link to its source message and quote a short supporting excerpt. Confidence must be shown; uncertain items go to “Review needed,” not silently treated as facts. The AI must not invent missing details.

## 5. One-click action centre

Each message card should offer:
- Open original imported message
- Show related messages in the same conversation
- View extracted facts and source excerpt
- Draft a reply
- Copy/export draft
- Create reminder or action item
- Download a scanned/approved attachment
- Mark reviewed / archive within this product

Sending a live email, deleting from a provider, or changing provider labels is **not part of import-only MVP**. When live sending is added, show the sending account, recipient, subject, body, and attachments and require explicit approval every time.

## 6. Security and privacy

- HTTPS in production; authentication before any import or search.
- Workspace-level tenant isolation and database RLS.
- Private object storage for attachments; short-lived signed links only.
- File size, count, decompression, and parser time limits; safe filename normalization.
- Treat imported email text as untrusted data, not as instructions to the AI system.
- Do not execute HTML, scripts, remote images, links, or attachment content automatically.
- Encrypt data in transit and at rest; define deletion/export and backup/restore procedures.
- Avoid uploading sensitive real mail to a demo, public repository, or third-party AI provider without explicit, informed opt-in.
- Support user deletion of an import batch and its derived facts/index entries, subject to documented backup retention.

## 7. MVP acceptance tests

1. Import a small test MBOX and report the correct parsed message count.
2. Import the same file twice; duplicates are detected without deleting legitimate similar messages.
3. Search by sender, subject, date range, phrase, and attachment presence.
4. Open a message and find its original headers/body and source batch.
5. Extracted facts display source links and supporting excerpts.
6. Malformed, oversized, encrypted, or suspicious files fail safely with a clear report.
7. A second workspace cannot query or download the first workspace's messages or attachments.
8. No email is sent and no external mailbox is changed by importing or analysing a file.
9. All UI counts distinguish imported data from live provider data.
10. The import UI clearly states that the prototype does not yet provide secure production storage or AI processing until the backend is deployed.

## 8. Delivery stages

- **Stage A:** Import UX and parser tests using synthetic/test fixtures only.
- **Stage B:** Secure backend, authentication, database RLS, private object storage, import job queue.
- **Stage C:** Search, threading, deduplication, import report.
- **Stage D:** AI facts, priorities, reply drafts with source citations.
- **Stage E:** Optional provider connections and approval-gated sending.

No live user mailbox data should be committed to GitHub. No paid API or infrastructure should be enabled without explicit user approval and a cost review.
