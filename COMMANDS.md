# UglyTags Commands

The full command reference. See the [README](README.md) for an overview, TagScript,
setup, and deployment. Permissions are at the [end](#permissions).

## Running Tags

- `/t name [arguments] [attachment]`: runs a tag. `arguments` fills `{args}`;
  `attachment` is used by `{image}`, `{media}`, and similar functions. Discord does
  not allow `/tag` to take a tag name because it has subcommands, so showing a tag
  uses NotSoBot's `t` alias.
- `.t name [arguments]` and `.tag name [arguments]`: run a tag from chat and reply to
  the command message without pinging its author. Quote names containing spaces. The
  original `.t show`/`.tag show` aliases also work.
- `.t view name [arguments]`: previews a tag privately, which NotSoBot does not offer.
  A chat command cannot get a private reply, so the bot deletes the command (when it
  has Manage Messages) and posts a **Preview** button that only the author can press.
  Pressing it shows the preview privately and deletes the button; unpressed, the button
  is deleted after 5 minutes. A command with an attachment is deleted together with
  the button instead, because a deleted message's attachment stops loading. An unknown
  tag is answered as usual and the command is kept. Previews follow the rules of
  `/tag info`.

### Unknown tag suggestions

When a chat or slash command names a tag that does not exist, the reply suggests up to
three similarly spelled tag names from the server: "Unknown Tag. Did you mean
`hello`?" NotSoBot only says "Unknown Tag". Matching uses PostgreSQL's `pg_trgm`
extension, so very short or symbol-only names may get no suggestions.

When showing a tag (`.t`, `.t show`, or `/t`), the suggestion comes with a **Show**
button, or a menu when there are several. Picking one shows that tag with the arguments
and attachment already given; only the person who ran the command can pick. For `.t`
the suggestion reply becomes the tag's output; for `/t` the private suggestion is
removed and the output is posted as usual.

The output starts with "Showing `name`" so it is clear which tag was picked. This line,
here and in the sections below, is left off when it would push the output past
Discord's length limit.

### Numbered variations

Tags whose names differ only by a trailing number (`cat`, `cat2`, `cat10`; no number
counts as 1) are variations of each other. A shown tag with variations gets arrow
buttons named after the previous and next variation. They replace the output with that
variation, with the same arguments, starting with "Showing `name`".

Only the person who ran the command can use them, each use counts, and they are removed
after 5 minutes. Tags whose output has its own components get no arrows. NotSoBot has
no variations.

### Editing and undoing chat commands

- Editing a `.t` or `.tag` chat command within 15 minutes of sending it updates the
  bot's earlier reply instead of sending a new one. Editing it into something that is
  not a command, or deleting it, removes the reply, like NotSoBot. If the new reply
  cannot replace the old one (for example a Components V2 reply becoming a plain one),
  the old reply is deleted and a new one is sent. Re-running an edited command counts
  as another use of the tag. Replies are tracked in memory, so edits made after a
  restart are not matched to their old replies.
- `.undo [amount]`: deletes the bot's replies to your last `amount` commands in this
  channel, counting both chat commands and public slash command tag output (`/t`,
  `/tag random`, and tag reply). The amount is 1 to 5, default 1, and only commands
  from the last 15 minutes since the last restart count. When the bot has Manage
  Messages it also deletes your chat command messages and the `.undo` message itself,
  and a short notice saying how many commands and slash interactions were deleted
  disappears after one second; without it the notice replies to `.undo`. Editing an
  `.undo` message runs it again. Unlike NotSoBot it does not reach commands older than
  15 minutes.
- `/tag undo [amount]`: the slash version of `.undo`, answered privately. It deletes
  the bot's replies to your last commands in this channel, but cannot delete chat
  command messages without Manage Messages, the same as `.undo`.

## Tag Reply (Apps Shortcut)

Right-click a message (or long-press on mobile) and choose **Apps > tag reply**
(prefixed with a speech-bubble icon). A private form offers your 25 most recently used
tags in this server, newest first, with optional arguments and one attachment.

Submitting posts the tag publicly under "Showing `name`", followed by an author mention
on a smaller subtext line beneath the content and a compact reply card with the
original author's name, avatar, and linked message excerpt, like CopyCat's reply
shortcut. Messages are still sent as UglyTags, not as your profile. Only that author
can be pinged; bots are not pinged. The usual TagScript, usage counting, and trash can
controls still apply.

History uses the retained one-year use log, including tags run through aliases, groups,
and nested tags; previews do not enter history. Deleted tags are left out, and renamed
tags appear with their current names. Both you and the bot need permission to view the
channel and send messages there.

The reply card stays on every pagination page and component update; tags that leave no
room for it report an error and can still be run with `/t`.

## Creating and Editing Tags

- `/tag create name [content]`: creates a tag. Without `content`, a form opens with
  multi-line content and up to 10 file uploads. Creating or editing a tag always
  requires confirmation before saving, including after submitting a form. If the name
  is taken, nothing is overwritten: the bot asks whether to overwrite it (only offered
  to the tag's owner and server administrators) or create the tag under the first free
  suggested name. `name` suggests `name1`, and a name ending in a number counts up, so
  `name1` suggests `name2`.
- `/tag edit name [content]`: edits a tag, the same way. Owners can edit their own
  tags; server administrators can edit any tag.
- `/tag alias tag name`: creates or repoints an alias that runs another tag.
- `/tag rename tag name`: renames a tag. Owners can rename their own tags; server
  administrators can rename any tag.
- `/tag info name [arguments] [attachment]`: privately shows the tag's raw content,
  owner, ID, uses, dates, and stored files, with a **Preview** button that privately
  shows its output using the given arguments and attachment. `.t info name` posts the
  same information publicly; anyone can press its Preview button to see the output
  privately, without arguments. A preview does not count as a use and does not save
  persistent variables; a group previews one of its tags. The button is removed after
  5 minutes.

### Chat management commands

`.t create`, `.t edit`, `.t alias`, `.t rename`, `.t info`, `.t list`, `.t random`, and
`.t remove` provide the original bot's core chat management commands. `.t add`,
`.t delete`, `.t owner`, and `.t raw` remain aliases. Create, edit, and remove always
ask for confirmation. Create and edit store files attached to the command message. Use
the slash commands for forms, filters, imports, exports, groups, statistics, and
pruning.

- **Creating from a reply**: reply to a message with `.t add name` or `.t create name`
  (without content or command attachments) to create a tag from the replied message's
  text, attachments, and bot embed or component media links (for a forward, the
  original message's). Unlike NotSoBot, which keeps only the first link, the full text
  is copied, unless the message has attachments, in which case its text is treated as a
  caption and skipped. Attachments are stored with the tag; links stay links and are
  not downloaded from external hosts. Explicit content or command attachments take
  priority over the reply.
- **Taken names**: a taken name gets the same choices as `/tag create`: overwrite (when
  allowed), create it as the next free numbered name (from 2, such as `name2`), or
  enter a custom name, which is checked again. The create confirmation also has a
  Rename button.

## Finding and Random Tags

- `.t search name`: lists the server's tags whose names contain `name`
  (case-insensitive), matching `/tag list server name:`. Quote names containing spaces.
  Without a name it behaves like `.t list`. See `/tag list` for similar-name results.
- `/tag list server [content] [name] [user]` and `/tag list user [user] [content] [name]`:
  list tags. Filters match case-insensitive substrings.
  - With a name filter (here or in `.t search`), the results have an **Include similar
    names** button that adds tags with similarly spelled names after the substring
    matches, and a search that finds nothing shows the similar names instead.
  - Name search results also have a menu of the current page's tags: picking one shows
    it (without arguments), starting with "Showing `name`" unless the search was for
    that exact name.
  - From `.t search` the tag's output replaces the list and has a **Back to results**
    button (kept through its variation arrows) that runs the search again in its place,
    from the first page. From `/tag list` it is posted publicly and the private list
    stays, so it needs no Back button.
  - NotSoBot has none of these.
- `/tag random [arguments] [content] [name] [user] [attachment]`: runs a random matching
  tag. Its output starts with "Showing `name`" (NotSoBot says "Showing tag:").
  - The output of this and `.t random` has a **Re-roll** button that replaces it with
    another random matching tag, with the same arguments. After a re-roll, a button
    named after the previous tag goes back to it.
  - Like the variation arrows, these buttons are only for the person who ran the
    command, each use counts, they are removed after 5 minutes, and tags whose output
    has its own components get no buttons. Random output gets no variation arrows.
  - NotSoBot has neither button.

## Removing Tags

- `/tag remove tag name`: removes a tag after confirmation, even for its owner. For
  someone else's tag without the permissions below, it starts a public removal vote.
- `/tag remove all [content] [name] [user]`: removes matching tags after confirmation,
  with an option to export them first.

## Import and Export

- `/tag export [content] [name] [user]`: exports matching tags as JSON.
- `/tag import file`: imports a NotSoBot or UglyTags export. To move your tags to
  another server, run `/tag export` in the old server and `/tag import` in the new one.

The file format and limits are described in the README's
[Import and Export](README.md#import-and-export) section.

## Groups

`/tag group create name tag`, `/tag group add group tag`, and
`/tag group remove group tag` manage tag groups. A group is a tag whose name runs a
random tag from the group, so `/t cats` posts one of the group's tags.

- Groups share names with tags and work with `/tag info` (which lists the group's
  tags), `rename`, `alias`, `remove`, `random`, and `{tag:ID}`.
- The group's owner and server administrators can add or remove any of the server's
  tags, up to 100 per group.
- Groups cannot contain other groups or have content of their own, and a deleted tag
  leaves every group it was in.

## Statistics

- `/tag stats overview [period]`: tag and use totals, uses in the period, storage, the
  10 most used tags, the 5 members who ran the most tags, and an unused-tags summary
  (never used, used at most once, and not used in 30, 90, or 365+ days) to size up a
  prune.
- `/tag stats tag name [period]`: one tag's uses (all time and in the period), last
  use, creation, owner, storage, and top members.
- `/tag stats member [member] [period]`: a member's tag count, uses of their tags, tags
  they ran, storage, and the tags they ran most.
- `/tag stats ranking [sort] [period] [owner] [unused_days] [max_uses] [min_age_days]`:
  a paged ranking (default: most used) by most used, least used, recently used,
  longest unused, newest, or oldest. Each row shows the tag, owner, uses, last use,
  creation date, and size. The filters work exactly like `/tag prune`'s, so running it
  with the same values previews what a prune would match.
- `/tag stats storage`: the server's storage use (files, tag text, and stored
  variables), the largest tags, and the members using the most storage. Sizes use
  1 MB = 1,000,000 bytes.

Stats are visible to everyone as private replies. The period is the last 7 days, 30
days, year, or all time (the default).

Running an alias also counts as a use of its target, and running a group also counts
for the tag it picked, so tags used only through aliases or groups are not treated as
unused. Totals count each run once.

The bot logs each use (tag, member, and time) for period and per-member statistics and
deletes log entries after one year; all-time totals and last-used times are kept on the
tag. Usage from before this log existed, including imported NotSoBot counts, appears
only in all-time totals.

## Pruning

`/tag prune [unused_days] [max_uses] [least_used] [owner] [min_age_days]` removes
unused tags.

- `unused_days` matches tags not used in that many days (never-used tags count).
- `max_uses` matches tags with fewer total uses.
- `owner` limits it to one member's tags.
- `least_used` keeps only that many of the least-used matches.
- `min_age_days` (default 7) protects tags created in the last that many days; they
  are never pruned.

Give at least one of `unused_days`, `max_uses`, or `least_used`; combined criteria must
all match. Before anything is removed, it lists every matching tag in the same detailed
format as the ranking, across as many pages as needed, with "Remove these tags",
Cancel, and Export buttons. Removing deletes exactly the listed tags; nothing happens if
no button is pressed within five minutes. Aliases and groups can be pruned like any
tag, and pruning a tag also removes its aliases.

## Permissions

These follow NotSoBot, except that server administrators can also overwrite tags:

- Only the owner or a member with Administrator (the server owner always has it) can
  edit, rename, or overwrite a tag; this keeps the original owner. Other members are
  offered a different name instead.
- Members with Manage Server or Ban Members can remove anyone's tag after a
  confirmation.
- Other members who remove someone else's tag start a public vote instead (an
  UglyTags addition). The tag and its aliases are removed once 5 members, counting the
  one who started it, press the vote button. The vote expires after 10 minutes and
  does not survive a bot restart.
- `/tag remove all` without a user, or for another user, requires Administrator.
  Other members can only remove their own tags.
- `/tag export` without a user, or for another user, requires Manage Server. Other
  members can only export their own tags.
- `/tag import` imports every tag in the file for members with Manage Server. Other
  members can import too, but only tags they own in the file; the rest are skipped
  and counted.
- `/tag prune` requires Manage Server. `/tag stats` is open to everyone.

Removing a tag also removes its aliases, stored files, stored variables, and use log.
