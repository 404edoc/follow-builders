# follow-builders on Hermes Agent + Obsidian (macOS)

## Verified local paths
- Skill install dir: `~/.hermes/skills/follow-builders`
- Runtime config: `~/.follow-builders/config.json`
- Seen-state file: `~/.follow-builders/seen-content.json`
- Obsidian vault used in this session: `/Users/agent/Library/CloudStorage/OneDrive-Personal/Obsidian/AlecObsidian`

## Observed issues and durable fixes

### 1. Obsidian CLI approach was fragile
The repository's original `deliver.js` attempted to use the Obsidian desktop binary for `daily create` and `append`. For local automation on this machine, the durable fix was to bypass the app binary and write directly to the vault markdown file.

### 2. Podcast dedup state could become invalid
`prepare-digest.js` originally deduped podcast items only on `videoId`. In practice, the feed item available to the skill did not always expose a stable `videoId`, which produced this bad local state shape:

```json
{
  "videos": {
    "undefined": 1778296322534
  }
}
```

The durable fix was to dedup by fallback keys in order:
1. `videoId`
2. `url`
3. `title`

and skip the item if none are present.

### 3. Chinese digest quality needed explicit constraints
The user rejected mixed-language summaries and weak "今日重点". A good final Obsidian digest for this setup should:
- keep summaries fully in Chinese
- synthesize 2–4 real conclusions in `今日重点`
- use clickable Markdown links instead of naked URLs
- remove test content before writing the cleaned final note

## Verified real-data run
After resetting `~/.follow-builders/seen-content.json` to empty maps, `node prepare-digest.js` returned real JSON with:
- `podcastEpisodes: 1`
- `xBuilders: 17`
- `totalTweets: 34`
- `blogPosts: 0`
- `feedGeneratedAt: 2026-05-08T07:58:39.860Z`

## Cron created in this session
A Hermes cron job was created for the verified local workflow:
- name: `follow-builders-obsidian-daily`
- job_id: `3ada980ab20c`
- schedule: `0 8 * * *`
- timezone effect observed: next run at `2026-05-10T08:00:00+08:00`

If a future session revisits this workflow, inspect the current cron config before creating another one to avoid duplicates.
