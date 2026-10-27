# Discord publishing

The LeviMod Discord Bot reads the configured forum while its host PC is online. Pins are optional. The published catalog and mirrored downloads remain online when that PC is off.

## Authorization and IDs

Server permissions control who can create posts in the configured forum. The bot accepts release messages only from the person who created that post, comparing their actual Discord user ID with the post's owner ID. Replies from other developers, bots and webhooks cannot publish. No additional developer approval, user list or role check is needed. `Author` is display credit, not authentication. Old approval settings in `config.json` are ignored.

The checkpoint `discord/state.json` binds a mod ID to its forum post and owner. A copied ID in another post is rejected, including higher versions. Existing catalog IDs require an explicit post binding before adoption. Reserve unregistered IDs with `existing_mod_threads` before first publication when needed. This binding never allows someone other than the post's creator to publish. A removed or disabled mod is not automatically recreated.

## First release

```text
[LeviMod]
ID: camera
Name: Camera
Author: Your Name
Version: 1.0.0
Minecraft: 1.26.52.X
Summary: Smooth camera movement.
Description: Full explanation, features and usage instructions.
Download: attachment
```

The trigger must occupy its own first line. Attach one `.so`, `.levipack` or supported `.zip` to that message. `ID`, `Name`, `Version`, `Minecraft`, `Download` and Summary or Description are required. Author is optional and initially defaults to the publisher's display name.

## Updates

Send a new message in the same post with the same ID and a higher version:

```text
[LeviMod]
ID: camera
Version: 1.1.0
Changelog: Fixed camera jitter and improved smoothing.
Download: attachment
```

Name, Author, Summary, Description, Minecraft, Tags, Homepage and Icon are inherited when omitted. A supplied field replaces its old value. Changelog is optional, applies only to that version and is never inherited. `What's New` is an alias for Changelog. Blank required text values are invalid; omit them on updates. Use `Tags: none`, `Icon: none`, or `Homepage: none` to clear an optional value.

For a download page, use `Download: https://example.com/camera` and optionally `Download Type: browser`. For advertising pages, use `Download Type: ad`. For a direct file, use `Download Type: direct`; add `Filename: Camera.levipack` if the URL has no file extension. Direct file extensions are `.so`, `.levipack` and `.zip`.

The bot fetches actual Discord attachments with refreshed signed URLs and mirrors them to GitHub Releases before changing the catalog. A copied Discord CDN URL from another message is rejected. Keep offline messages and their files until collected.

## Launcher fields

The existing catalog schema remains version 2. New fields are additive:

- `mods[].summary`: a short card preview; derived from Description for older registrations.
- `mods[].description`: the full explanation.
- `mods[].releases[].changelog`: release-specific notes.
- `mods[].releases[].source_url`: an optional release or Discord-post link.
- `mods[].legacy_names`: prior published names retained for migration of older launcher installations. New installations use a recorded catalog ID, so changing a display name does not change the installed identity.

URL registrations may include `metadata_urls`, newest first, containing up to `max_releases` immutable metadata snapshots. `metadata_url` identifies the current mod details. The builder reads this history and combines it with previously published releases without duplicating versions, so rapid releases do not depend on deployment timing. Historical metadata failures are logged while the current valid release remains publishable.

GitHub manifests use `info.summary`, `info.description` and top-level `changelog`. If a GitHub manifest omits Changelog, its GitHub release notes are used. Static registrations may include Summary and a Changelog on each release. Existing descriptions and download behavior remain compatible with older launchers.

## Offline recovery

Startup and periodic scans discover active and accessible archived posts. Saved per-post positions fetch new messages with pagination, while failed formatted messages remain tracked separately. Details from waiting releases are merged in version order and the newest valid version is published. The original saved cutoff remains fixed, so historical messages before it are excluded. Local-state loss recovers the same cutoff and bindings from the checkpoint.
