---
name: openbase-messaging-contacts
description: >-
  Use this skill before an agent reads, summarizes, or sends personal messages
  through the WhatsApp or iMessage skills, or when the user asks about contact
  categories, who agents may message, or the Contacts review queue.
version: 0.1.0
---

# Messaging contact categories

Openbase Coder keeps a category for every WhatsApp and iMessage contact the user has exchanged messages with in the last year. The user assigns categories on **Configure → Contacts** in the console or desktop app; the same person is one entry even when they appear on both services. Each category carries an agent-access policy:

- `ask`: the normal flow. Reading or sending still goes through the skill's own approval gate, and the approval request shows the category.
- `block`: every skill approval request that names the contact is declined automatically. Do not retry, rephrase, or route around it.

An uncategorized contact behaves like `ask`, but approval requests are labelled "Uncategorized" so the user notices.

## Before acting on a contact

Check the category first, using the exact identifier the messaging skill uses (a WhatsApp JID or an iMessage handle or chat id):

```sh
openbase-coder contacts lookup whatsapp "15551234567@s.whatsapp.net" --json
openbase-coder contacts lookup imessage "handle:+15551234567" --json
```

- If the policy is `block`, tell the user the contact is blocked for agents and stop. Only the user can change that on the Contacts page.
- If the contact is uncategorized or unknown to the index, you may proceed through the skill's approval gate, and you may suggest the user categorize them.
- If the user asks who is in a category, or how many contacts are still uncategorized, use the list commands below instead of the skills' raw catalogs.

```sh
openbase-coder contacts categories --json
openbase-coder contacts list --pending --json
openbase-coder contacts set "phone:15551234567" --category family
```

`contacts set` records the user's decision; only run it when the user explicitly states the category for that person. Person keys come from `contacts list` (for example `phone:...`, `email:...`, or `<source>:<id>` for groups).

## Privacy

Output from these commands contains names and identifiers. Mention only what the user's request needs, never paste listings into reports, commits, logs, or test fixtures, and never send a message on the strength of a category alone: sending always needs the skill's own approval.
