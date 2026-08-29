# Buzz 功能 Surface 地图

本表用于从用户功能快速定位代码，不表示每个 feature 都只存在于单一目录。

| Surface | Desktop | Mobile | Relay/Core | CLI/Agent |
|---|---|---|---|---|
| Community/onboarding | `desktop/src/features/communities/`、`onboarding/` | `features/invites/`、`settings/` | `buzz-relay/src/tenant.rs`、`buzz-db/src/store/community.rs`、invite APIs | channel/community相关命令 |
| Stream/messages | `features/messages/`、`chat/`、`channels/` | `features/channels/` | `handlers/ingest.rs`、`req.rs`、`buzz-db/src/store/event.rs`、`thread.rs` | `buzz messages` |
| Forum | `features/forum/` | `features/forum/` | forum kinds、ingest、thread store | `buzz messages --kind`、vote |
| DMs | message/channel UI | channels/DM presentation | `buzz-db/src/store/dm.rs`、DM command handlers | `buzz dms` |
| Home/activity | `features/home/`、`notifications/`、`reminders/` | `features/home/`、`activity/` | `buzz-db/src/store/feed.rs`、`reminder.rs`、reminder kinds | `buzz feed` |
| Agents | `features/agents/`、`agent-memory/` | channel agent activity | `buzz-acp`、observer frames、managed-agent kinds | `buzz agents`、`buzz mem`、`buzz pack` |
| Workflows | `features/workflows/` | 暂无同等完整编辑器 | `buzz-workflow`、`workflow_sink.rs`、`buzz-db/src/store/workflow.rs` | `buzz workflows` |
| Search | `features/search/` | `features/search/` | `buzz-search`、REQ NIP-50 | `buzz messages search` |
| Profiles/presence | `features/profile/`、`presence/`、`user-status/` | `features/profile/` | profile/status kinds、Redis presence | `buzz users` |
| Projects/repos | `features/projects/` | 当前非主要 surface | `api/git/`、`buzz-db/src/store/git_repo.rs`、NIP-34/MP kinds | `buzz repos/projects/patches/issues/pr` |
| Web repo browser | 不适用 | 不适用 | relay SPA fallback + git smart HTTP | `web/src/features/repos/` |
| Media/GIF | message composer、`features/gifs/` | channel attachments | `buzz-media`、media/GIF APIs | `buzz media/upload` |
| Huddle/voice | `features/huddle/` | channel huddle actions | `buzz-relay/src/audio/`、`buzz-voice` | 暂无核心日常 CLI surface |
| Moderation | `features/moderation/` | 非主要 surface | moderation handlers/store/audit | `buzz moderation` |
| Pairing | Desktop onboarding/native | `features/pairing/` | `buzz-pair-relay`、pairing core | `buzz-pairing-cli` |
| Mesh compute | `features/mesh-compute/` | 暂无 | `buzz-relay-mesh`、`mesh_boot.rs` | provider/ops tools |
| Custom emoji | `features/custom-emoji/` | emoji picker | emoji kinds + ingest validation | `buzz emoji` |
| Pulse/social notes | `features/pulse/` | `features/pulse/` | standard Nostr note/list kinds | `buzz social`、`buzz notes` |

## Desktop 当前 feature inventory

`agent-memory`、`agents`、`channels`、`channel-templates`、`chat`、`communities`、`community-members`、`custom-emoji`、`forum`、`gifs`、`home`、`huddle`、`identity-archive`、`local-archive`、`mesh-compute`、`messages`、`moderation`、`notifications`、`onboarding`、`presence`、`profile`、`projects`、`pulse`、`reminders`、`search`、`settings`、`sidebar`、`terminal`、`user-status`、`workflows`。

## Mobile 当前 feature inventory

`activity`、`channels`、`forum`、`home`、`invites`、`pairing`、`profile`、`pulse`、`search`、`settings`。

## Web 当前 feature inventory

`invite`、`repos`。

目录证据来自当前 checkout 的 `desktop/src/features/`、`mobile/lib/features/` 和 `web/src/features/`。
