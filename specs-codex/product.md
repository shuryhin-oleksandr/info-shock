# Product specification

Status: working draft. Confirmed requirements and open decisions are recorded separately below.

## Purpose

Help users apply knowledge they have previously saved when it becomes relevant to their current work, even if they have forgotten the original source.

The product serves several users. Each user has a private personal library. Team libraries and knowledge sharing are outside the first version's scope.

## Source inputs

The first version should accept:

- Article and web page URLs.
- Pasted text.
- PDF documents.
- YouTube video URLs.

When a source cannot be read, report the issue to the user. The first version should not attempt OCR for scanned PDFs or transcription for videos without available transcripts. The user can submit readable text instead.

If a user submits the same source again, reuse the existing entry and refresh its extracted content rather than creating a duplicate. Preserve user edits during refresh; refreshed extraction must not overwrite those edits. How sources are recognized as duplicates and how refreshed content is reconciled with edits remain undecided.

## Core user experience

1. A user submits a source, such as an article about AI.
2. The product analyzes the source and saves useful knowledge from it.
3. Later, the user works with their AI on a task without explicitly referring to the saved source.
4. When the current topic matches relevant knowledge in the library, that knowledge informs suggestions about better ways to perform the task.
5. Suggestions can also describe how to configure the user's AI to perform tasks more effectively.

Users should not need to remember a source or explicitly search for it for this experience to work. The integration and the mechanism for triggering suggestions are still undecided.

Successfully processed knowledge should become available immediately, without a user review or approval step. Approval of an applied change is separate from ingestion of knowledge.

## Knowledge extracted from sources

The initial idea identifies three outputs:

- Notes: main ideas extracted from the source.
- Actions: things the user could do based on the source.
- AI configuration applications: changes to the user's AI setup or workflow that could improve how it performs tasks.

Users must be able to edit extracted notes, actions, and AI configuration applications. The exact structure of these outputs remains undecided.

User edits take precedence over the original extraction when generating future suggestions. Preserve the original source for reference while it remains in the library.

## AI integration

Suggestions should appear inside the user's existing AI conversation. The product should be provider agnostic and target web chat, desktop AI applications, and CLI coding agents. Requested examples include Claude Code, Codex, and Gemini across applicable interfaces; CLI support is included alongside web and desktop support.

Broad compatibility across these categories is the product goal. Required capabilities, connection mechanisms, and the client-by-client rollout remain to be verified. Support for every client is not yet a validated first-version commitment.

Clients that cannot consult the product automatically may still be supported through user-triggered consultation. Automatic consultation is the preferred experience; user-triggered consultation is an accepted fallback. Clearly communicate which behavior a connected client supports.

The intended starting behavior is for the user's AI to consult the product for every user message. Consultation frequency must be configurable so users can limit the amount of library information added to their AI's context. The precise timing within message handling is not yet specified.

Consultation settings should include:

- Every user message: the intended starting behavior.
- Every N user messages: the user chooses a positive integer interval.
- At task start or substantial topic changes.
- Only when explicitly requested.

The settings above were proposed during discussion, and the user added every N user messages. The complete set still needs final confirmation. For the interval setting, when counting starts or resets remains undecided.

Consultation before proposing persistent AI configuration changes was also discussed. How that trigger interacts with the selected frequency remains to be defined. "Before processing" is not a separate confirmed trigger.

Consultation does not imply that a suggestion must appear: suggestions still require a strong match, and permanent declines must be respected. The product receiving work context and evaluating relevance is also an option, but it is not yet a first-version requirement.

Each consultation should send the latest user message plus a short summary of the current task. The summary provides context for follow-up messages that do not explain the task on their own. The full conversation is not a required consultation input. How the summary is generated and its size limit remain undecided.

If library consultation is slow or unavailable, the user's AI should continue the task without library advice and briefly indicate that the check could not complete. The timeout and how to avoid repetitive failure notices remain undecided.

The initial idea proposes MCP as a way for users' AI tools to access the library. The required experience is for relevant knowledge to influence suggestions during the user's ongoing conversation. MCP alone is not being treated as a confirmed mechanism for triggering this behavior.

## Onboarding and connection setup

The product is a hosted service. Users sign in and connect their AI without deploying or managing a server themselves.

The first version should provide guided automatic setup for connecting a user's existing AI to the product and establishing the configured consultation behavior. The experience should be as simple as adding a plugin in Claude. Complex installations or a separately managed local helper are not the intended experience. A manual guide requiring users to edit configuration or add consultation instructions themselves is also not the intended onboarding experience.

The product should remain provider agnostic. Available setup mechanisms and permissions across web, desktop, and CLI clients need investigation before automatic setup can be specified technically. Automatic setup is a product requirement, not yet a verified capability of any particular client.

## Suggestions and approval

The product should suggest improvements inside the existing AI conversation. Each suggestion offers three actions:

- Approve: authorize application of the proposed change.
- Decline: reject this occurrence without permanently suppressing the suggestion. It may appear again in a new conversation or when the task context changes substantially. Neither event removes the strong-match requirement.
- Decline forever: reject the suggestion and permanently suppress the underlying recommendation for that user across all sources, including other articles that recommend the same thing. This does not suppress unrelated recommendations from those sources. How to recognize equivalent recommendations across different wording remains undecided.

"Suggest mode" refers to proposing changes for user approval, not a separate mode that users enable.

Approval should support both changes to the current task and persistent changes to the user's AI configuration, including instructions or skills. The product should propose the change before the user approves its application.

Before approval, the user must see the exact proposed change. Configuration edits should include a before/after diff alongside the explanation and source citation. How other types of task changes are previewed remains undecided.

The product supplies the exact proposed change. After user approval, the user's existing AI applies it using its own tools and permissions. How approval is presented across supported clients and which configuration locations and formats are supported remain undecided.

Suggestions should require a strong match between the user's current work and saved knowledge. Each suggestion must explain why it applies and link to the original source. What constitutes a strong match and how to evaluate suggestion quality remain undecided.

Each consultation should return at most three suggestions. Return fewer, including none, when there are not enough strong matches. Whether the limit is user configurable remains undecided.

When relevant saved sources provide conflicting advice, present all conflicting alternatives and identify a recommended option. Explain why that option is recommended for the current task and cite the sources behind the alternatives. The user retains the choice of which change to approve.

## Library management

Users should be able to manage their library through both a web UI and their existing AI conversation. Both interfaces should support adding sources, browsing and editing extracted knowledge, deleting sources, and managing permanently declined suggestions, including undoing a permanent decline.

The interfaces should operate on the same personal library and suggestion preferences. How each AI client presents these operations remains undecided.

Deleting a source removes it and its extracted knowledge from the active library and stops them from informing future suggestions. Changes the user previously approved and applied remain in place; source deletion does not revert them.

If a source contributed to a suggestion with a recorded result, soft delete the source to preserve historical statistics. Approve, decline, and decline forever each count as a result. An unanswered suggestion does not create a result record, though it still counts toward suggestions shown.

If a source has never contributed to a suggestion with a recorded result, delete it normally rather than retaining a soft-deleted record. This includes sources with only unanswered suggestions. The source data retained by soft deletion remains to be defined.

## Suggestion history and reversal

Users should have a history showing proposed suggestions, their sources, and whether each suggestion was approved, declined, or permanently declined.

The first version does not provide its own undo mechanism for applied changes. Users can ask their existing AI to reverse a change using that AI's tools and permissions.

Record approval separately from application status. The user's existing AI should report whether an approved change was applied successfully or failed, and failures should appear in history. Approval alone must not be recorded as successful application. How outcome reporting works across supported clients remains undecided.

Preserve suggestion history and statistics for soft-deleted sources. Such sources remain excluded from future suggestions.

Expose the following statistics as totals and broken down by source:

- Suggestions shown.
- Approved suggestions.
- Declined suggestions.
- Permanently declined suggestions.
- Successfully applied changes.
- Failed applications.

These counters describe different events: an approved suggestion may later have a successful or failed application. They should not be treated as mutually exclusive categories.

## Example scenario

A user saves an article about AI and subsequently forgets it. During a later task involving a related topic, their AI suggests a better approach informed by the saved article. A suggestion could involve changing the task approach or configuring the user's AI differently.

## Open decisions

- How can each target client deliver simple plugin-style setup without complex installation?
- Which web, desktop, and CLI clients can support connection to the library, automatic or user-triggered consultation, suggestion decisions, and application of approved changes? How should limitations beyond consultation frequency be handled?
- How should the short task summary be generated and limited, and should direct context delivery beyond consultations be included in the first version?
- Confirm the proposed consultation settings, define interval counting, and decide whether the three-suggestion limit is user configurable.
- How should duplicate sources be identified and refreshed content reconciled with user edits?
- How should citations identify uploaded PDFs and pasted text that have no original URL?
- How are proposed changes presented and applied across supported clients?
- What makes a suggestion useful enough to present?
- How should permanent suppression recognize equivalent suggestions?
- What timeout should consultation use, and how should repeated failures be communicated?
- How should supported AI clients report application success or failure?
- Which data should a soft-deleted source retain?

## Scope status

This draft captures the product direction and agreed first-version behavior. Acceptance scenarios are recorded in acceptance-criteria.md. Remaining scope details, implementation architecture, and delivery milestones have not yet been agreed.
