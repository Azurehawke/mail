# PLAN

Working plan for Steve's fork of Nextcloud Mail (`Azurehawke/mail`), branch `steve/stable5.13-custom`.

Status: draft. The vision section is a placeholder until Steve fills it in.

## Vision

Gmail-style conversation threading. Every reply from any party sits in one thread, not in separate messages.

Threads should follow the message headers (`In-Reply-To`, `References`) first, the way Gmail does. Subject matching is only a fallback. A forwarded message or a reply with a changed subject should not break the thread.

Open questions for Steve:
- Which threads currently break? An example from a real mailbox would help.
- Should a forward (`Fwd:`) join the original thread, or start a new one as Gmail does?
- Should subject matching be kept as a fallback, and how strict should it be?
- Is this for personal use only, or will others use the fork?

Current state (checked in the code):
- Threading already exists. `lib/IMAP/Threading/ThreadBuilder.php` groups messages using `References` first, then subject.
- The subject fallback only strips `Re:`, so `Fwd:` and localized prefixes such as `AW:` and `SV:` are not handled.
- Threads are stored with `thread_root_id` in the database during sync.

### Inbox tabs

Gmail-style tabs across the top of the Inbox: Primary, Social, Updates, Promotions, plus tabs Steve creates himself.

Decisions:
- Tabs are a view over the Inbox. Messages stay in the Inbox and are not moved into folders, so other clients and IMAP are unaffected.
- Scope is the Inbox only. Other folders are unchanged.
- Automatic sorting starts with header rules (`List-Unsubscribe`, `List-Id`, `Precedence: bulk`, `Auto-Submitted`, `noreply` senders). A learned model is a later option.
- Moving a message to a tab sets a rule for its sender, so later mail from that sender goes to the same tab.
- A thread appears in the tab of its first message. New messages in the thread stay with that tab.
- Folders remain Steve's way of sorting important mail. Tabs don't replace them.

Precedence:
- Folder rules (Sieve filters) run first and win. A message moved out of the Inbox by a folder rule never reaches the tabs.
- Tab rules apply only to messages that stay in the Inbox.

Status:
- Custom tabs are on hold until the rest of the tab work is done.

Open items (for when custom tabs resume):
- How user-created tabs are defined and whether they can have their own rules.
- Whether tab unread counts are shown.

### Interface (Gmail-like, not a copy)

Scope: desktop browser only, all light and dark modes. Steve reads mail on Thunderbird on his phone, so there is no mobile layout work.

Decisions:
- Left side menu collapses to a narrow icon rail, as in the reference screenshot. Chat is excluded.
- The menu has Inbox, Starred, Sent and so on, then folders and labels. Labels are the existing user tags.
- Thread list sits in a translucent panel over the background image, so the background shows through the rows.
- Tabs across the top of the list (Primary, Promotions, Social, Updates), as described above.
- Colours, background and fonts come from the global Nextcloud theme. No separate theme for now.

Notes:
- Thunderbird connects over IMAP, so it sees folders but not tags. Labels and tabs are web-only. Folder rules still reach Thunderbird.

Open items:
- Confirm that Nextcloud's navigation component supports an icon-only collapsed state, or whether it needs custom CSS.
- Confirm where the theme background image is applied, so the list panel can be translucent over it.

## Environment

- Nextcloud 35.0.2, installed via snap.
- Mail base: upstream `stable5.13` at 5.13.1 (commit `972c9c65e`).
- App location: `/var/snap/nextcloud/current/nextcloud/extra-apps/mail`.
- Admin commands: `sudo nextcloud.occ ...`.
- Build happens off the server (Node 24, npm 11, PHP 8.2+, Composer).

## Phases

### Phase 0: Baseline

- [ ] Confirm the unmodified `stable5.13` build runs on the snap server.
- [ ] Back up the server with `sudo nextcloud.export` and keep the store copy of Mail.
- [ ] Record the test accounts and mailboxes used for checks.

Exit: the upstream build works on the server and can be restored from the backup.

### Phase 1: Define the changes

- [ ] Write each planned change as a one-line goal under Vision.
- [ ] Order the changes by risk. Keep UI-only work separate from backend or database work.
- [ ] Decide which changes need a database migration.

Exit: a list of changes Steve has approved, each small enough for its own commit.

### Phase 2: Implement

- [ ] One concern per commit, following `AGENTS.md`.
- [ ] Add the AGPL-3.0-or-later SPDX header to any new file.
- [ ] Run `composer cs:check`, `composer psalm`, `composer test:unit`, and `npm run lint` for every change.
- [ ] Push each change to `steve/stable5.13-custom`.

Exit: all approved changes are on the branch and pass the checks.

### Phase 3: Test and release

- [ ] Build with `composer install --no-dev -o` and `npm ci && npm run build`.
- [ ] Package the result as `mail.tar.gz` with the folder named `mail`.
- [ ] Install on the test server using the swap steps in the setup notes. Keep the store copy for rollback.
- [ ] Check: login, sync, send, reply, attachments, and any changed feature.

Exit: Steve has tested the build on the server and approved it.

### Phase 4: Maintenance

- [ ] Skip the store update for Mail. The store offers 5.13.2 and will replace the fork if clicked.
- [ ] Merge upstream `stable5.13` fixes into the branch as they're released.
- [ ] Rebuild and retest after each merge.
- [ ] Decide when to move to `main` (6.x), since that branch includes a database upgrade.

Exit: the fork stays current with upstream security and bug fixes.

## Open decisions

- Target Nextcloud version after 35.x upgrades.
- Whether to stay on `stable5.13` or move to `main`.
- How to handle upstream fixes that conflict with the fork's changes.
