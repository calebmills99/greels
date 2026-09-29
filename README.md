# greels

Real stories. Real voices.

This repository is a planning space for a YouTube channel and a future n8n
publishing workflow. It does not yet contain a runnable workflow, video assets,
or an application. Start with a manually approved upload before automating
production or publication.

## Prepare the channel

- [ ] Define the audience, format, publishing cadence, and who can approve a
  story, script, voiceover, thumbnail, and final video.
- [ ] Create the YouTube channel, configure its name, handle, description,
  branding, and contact details, and complete any account verification needed
  for the features you plan to use.
- [ ] Set channel defaults and decide per-video audience designation, visibility,
  captioning, and whether to schedule publication. Review YouTube's
  [channel setup](https://support.google.com/youtube/answer/1646861) and
  [upload guidance](https://support.google.com/youtube/answer/57407).
- [ ] Obtain permission to use each story and voice, document the rights to
  footage, music, images, and thumbnails, and avoid publishing private details
  without consent. Review YouTube's
  [copyright](https://support.google.com/youtube/answer/2797466) and
  [synthetic-content disclosure](https://support.google.com/youtube/answer/14328491)
  guidance where applicable.
- [ ] Decide where source files, rendered videos, captions, and thumbnails
  will live. Limit access to unpublished stories and retain a backup.

## Prepare n8n access

1. Choose n8n Cloud or a self-hosted instance with persistent storage, HTTPS,
   access controls, backups, and an encryption key retained securely. Keep
   credentials in n8n's credential store, **not** in this repository or workflow
   exports.
2. In a Google Cloud project, enable the YouTube Data API v3, configure the
   OAuth consent screen, and create an OAuth client. Add the redirect URL shown
   by your n8n credential configuration to the client's authorized redirect
   URIs. Connect the Google account with access to the intended channel, using
   only the scopes required for upload and any other selected operations.
   Test the connection and confirm the intended channel before publishing.
3. Check the project's OAuth publishing/testing status and YouTube API
   [quota and upload restrictions](https://developers.google.com/youtube/v3/docs/videos/insert)
   before relying on unattended uploads. An unverified API project may have
   restrictions on video visibility; do not assume successful upload means
   public publication.
4. Decide how n8n will access approved files (for example, private cloud
   storage) and where it will record review state and upload results (for
   example, a restricted spreadsheet or database). Do not place access tokens,
   personal story submissions, or unpublished media in Git.

See the [n8n YouTube integration](https://docs.n8n.io/integrations/builtin/app-nodes/n8n-nodes-base.youtube/)
and [Google OAuth setup](https://docs.n8n.io/integrations/builtin/credentials/google/oauth-single-service/)
for current node and credential options. Confirm that the chosen node supports
the upload and metadata operations you need; use the API only where necessary.

## First workflow to build

Use one record per video with at least: `content_id` (stable unique ID),
`title`, `description`, `video_location`, `thumbnail_location` (optional),
`audience`, `visibility`, `approval_status`, `publish_at` (optional),
`youtube_video_id` (empty until uploaded), and `last_error`. Keep the consent
and licensing evidence in restricted storage linked to the record, not in a
public workflow export.

1. Trigger on a manually approved record, then fetch the approved video and
   metadata. Reject missing assets, incomplete metadata, or missing rights and
   audience decisions before attempting an upload.
2. Require an explicit final human approval of the rendered video and metadata.
   Recheck approval immediately before uploading.
3. If `youtube_video_id` is already set, stop rather than uploading a duplicate.
   Process one `content_id` at a time and preserve the returned video ID before
   retrying any failed downstream step.
4. Upload with private visibility first. Record the returned ID and verify the
   video, metadata, and any thumbnail/captions before a human changes visibility
   or enables scheduling. Treat publication as a separate, deliberate step.
5. Log failures to the record and notify the reviewer without exposing tokens
   or personal submissions. Retries should check upload state first: a timeout
   can occur after YouTube accepted the video.

Start with a single test video you own. Run the workflow manually, check the
video in YouTube Studio, confirm the correct channel and visibility, then test
missing-file and retry cases before adding scheduling or unattended triggers.
Use [YouTube Data API documentation](https://developers.google.com/youtube/v3/)
to confirm current API behavior and quotas when implementing the workflow.
