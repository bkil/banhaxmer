# Development plans

## Matrix API

### Tier 1

* authorize with token
* read (sync) all new messages
* filter and process incoming messages piecewise by room (and sender)
* invite user to new private chat
* send out message in a public room
* add emoji reaction to a message
* send out message in a private, non-E2EE chat room
* get power level of user within room
* change power level of a user within room
* redact a specified message (or reaction)
* redact all messages of user

### Tier 2

* change minimal required power level for sending messages and reactions within room
* ban and kick user from room
* load manual user bans within room
* get all members of room
* load Server ACL from room
* load user and server bans from a room containing a mjolnir ban list
* apply Server ACL to room (probably only in case of network attack related rate limits)

### Tier 3

* get read receipts
* send read receipt update
* send out message in a private E2EE chat room
* change own settings of bot user: block incoming chat invitations, change display name, avatar, notifications
* get name of verification devices of a given user

## Implementation

### Message analysis

* locally, on top of Matrix API
* watch event rate per user, per HS and per room and unvoice to throttle
* listen to certain events and act as per rules: user mentions, media, prohibited words, spam, changes to displayed name, avatar
* listen to reactions, mentions and commands of seniors and moderators
* aggregate reactions to a given message
* aggregate senior replies to a given message (user)
* aggregate senior reactions to a given message (user)
* aggregate senior mentions to a given user

### Polling

* hourly scheduled cron execution
* manual invocation through a webpage URL
* optional continuous long polling push by daemon via Server-Sent Events EventSource API
* forwarding Matrix notification email to pipe it as input
* webhooks: Matrix notification pusher, ActivityPub S2S API, XMPP S2S API, GitHub, GitLab, gitea, BitBucket, SourceForge, LaunchPad, Pagure, gogs

### Caching

* Synapse is very slow
* The traffic volume in case of an attack can be potentially high
* Cache as much as possible locally with generous expiration
* Revalidate in case of conflict
* Ideally in a flat file database for easier setup and less imposed limits
* Seamlessly recreate on first start
* May want to submit requests in batches
* Handle out of order messages

## Deployment

### Centralized

* VPS
* free PaaS provider
* Matrix homeserver plugin

### Decentralized

The backend could be installed and operated redundantly and simultaneously at each moderator:

* command line or via a background service
* OpenWrt router
* either via its own REST API or through Matrix client-server API
* a dedicated webpage and background worker that should be kept open
* a browser WebExtension, UserScript or bookmarklet
* Matrix Element widget

https://gitlab.com/bkil/freedom-fighters/-/blob/master/en/server/backend-optional-web-apps.md

## Components

### Automatic update

* Crucial so that every instance must operate in the same way
* If a centralized bot is also in operation, it should have a higher power level and kick out desynced decentralized bots
* If a decentralized bot is kicked out by a centralized bot, it should update itself, purge its caches, and wait based on exponential backoff before joining again
* A decentralized bot may send a notice to the centralized bot (and others listening) if some other decentralized bot seems to be out of sync from their perspective

### Message ingestion

* via the Matrix client-server API
* optional custom API for commands, voting and to invoke the poll webhook
* optional custom API to ingest new messages

### CAPTCHA webpage frontend

* a static HTML page linked in from the room topic
* should include the room alias or ID as an anchor argument
* explain workflow in a few words, link to detailed room rules
* ask for user mxid
* optionally ask for email address
* ask for country
* display CAPTCHA
* JavaScript

### CAPTCHA webpage backend

* optionally confirm email address
* verify CAPTCHA response
* verify country
* deduplicate based on network fingerprint
* verify that the user is a member of the main room with the voicable but unvoiced power level
* initiate a private chat with the user, explain next steps briefly
* send a poll-like message to the main room to ask whether this user would be welcome by anyone present
* if the user is a moderator within the room, provide commands for inviting to private chat and altering protection levels

### Panic button

* any member could visit the CAPTCHA URL
* maybe could be selected via a button on the CAPTCHA page that only polling is requested, not voicing
* it should enforce much less user background checks
* it might still require a CAPTCHA, unless whitelisted based on fingerprint or cookie
* if the tool is installed with a relaxed cron polling interval (1 hour)
* on the backend, verify that multiple requests of this kind should not run in parallel or in rapid succession

### Newbie relay

* Forward messages and reactions between private room and main room, possibly back and forth

### HS reputation tracking

### User reputation tracking

* power level
* behavior history
* HS reputation
* user risk

### Threat level tracking

### Message threat

Determine which protection rules should apply to a message

* user reputation & risk
* threat level

### Message rule engine

* verify protection rules applicable to message
* provide feedback for user demotion, threat level, HS reputation

### Reactive user promotion engine

Watch votes of seniors and moderators in the workflows for:

* first voicing
* abuse reports
* retain first incident
* implicit grants
* explicit grants

### Proactive user promotion engine

* propose polls for user promotion based on user reputation

### Moderation trail archive

* certain actions must be recorded in a private room until later investigation for a limited time

### Approval queue

Moderators must review messages here before they appear in the main room.
Messages flagged to fail automated safety checks by:

* implicit grants
* newbie relay
