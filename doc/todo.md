# banhaxmer, a moderation tool for your open community

TODO: possible protection measures.

## Attack types

Moderators need elevated privileges for dealing with the following distinct use cases.

### Troublemakers

* deliberately "trolling" on purpose
* to annoy others
* posting text or attachments that the user is sure will outrage members and mods
* flooding

### Rule violations

* violations against laws, the code of conduct or terms of service of the community

### Atmosphere toxicity

These should ideally be spelled out in the ToS/CoC, but most are borderline:

* "you know it when you see it"
* genuine users disagreeing or arguing too much
* one leaching too much
* one hogging all conversations, forcing their viewpoint on others all the time
* one who is not being friendly in response to friendly comments
* one who gets personal too often in response to objective comments

## Threat level

Depending on the minute by minute state of the network, subsets of security measures may be disabled.

### Threat level bumping

Triggered by:

* manual override
* moderator redaction, kick, ban
* seniors voting on an abusive message (it may get reviewed by moderators or evaluated according to the reputation level of the sender)
* automated moderator actions
* may revert after a cool down period based on recent history of level changes
* possible influence threat level in correlated rooms as well

### Happy place level

* basic rate limits are in place
* duplicates of previously reported text, links and attachments detected
* user banlists enforced

### Occasional pranksters level

* most protections enforced except: CAPTCHA, email, country, initial message forwarding through the bot
* new joiners: voiced, lax statistical spam filtering, lax word filters

### Under attack level

* every possible protection enforced

## Workflows

### Under attack workflow

* user joins main room
* user clicks link in room description, types in username mxid, optionally email address, selects country, solves CAPTCHA, page detects account duplication and country mismatch
* bot always blocks all incoming chat invitations
* on successful solution, bot invites user for private chat
* if account duplicated suspected (due to typing in the name of someone else), presents a unique CAPTCHA URL (e.g., signed)
* on successful solution, bot creates a poll in the main room whether the user should be allowed
* a senior member reacts with plus sign reactji
* bot starts forwarding the messages typed in the private chat by the user to the main room and relays all responses back (subject to read receipts and for up to 24 hours)
* user now has 24 hours to prove that she is not a troll by chatting
* a senior member approves user by reacting with a brain reactji on the original poll after she is confident enough that the user is not a troll and can be voiced without forwarding
* the bot reminds the user and participants before the deadline
* if not approved until the deadline, forwarding stops for the user and needs to solve another CAPTCHA to retry
* bot voices user in main room
* bot stops relaying in private chat and unvoices user in private chat
* bot then progressively promotes user according to reputation and polls

### Explicit grants

* A user with a low reputation may (should) ask permission to post an item that would require a higher power level
* It would only apply for a single item posted next and within a certain amount of time (1 day?)
* It could be done by slash commands or by keyword matching
* A senior member may provide a grant proactively for a given user similarly, by asking the user to post the screenshot or log snippet in question

### Implicit grants

* the user posts an item that would require a higher power level
* the bot copies the content to a private moderation room, redacts it from the main room (or holds back forwarding) and posts an explanation and/or an explicit grant request in the main room
* Only a single outstanding event per user per kind must be allowed within the moderation queue - triggering further such messages should demote the user
* If a moderator sees the content as inappropriate in the moderation room, the user could be demoted, the content moved to the moderation trail archive room and the grant request revoked
* If a moderator approves the content in the moderation room or if a senior member approves the grant in the main room, the bot forwards the content to the main room and removes it from the moderation room
* If a non-moderator approved the grant and the item proves to be indecent, they can remove the reactji to revoke the grant (or react with the opposite reactji): then the bot moves the content to a moderation trail archive and demotes the newbie until moderator review

### Option to retain first incident

* Optional, probably during the `happy place` threat level
* If a user does something above its power level, instead of waiting for approval, it would appear instantly
* The bot would immediately unvoice the user
* The last posted content would remain visible in the main room
* If senior members voted to remove it, it would be moved to the moderation trail archive room and the bot would mark the user as waiting to be banned
* If senior members or moderators voted to keep it, the power level of the user would be restored and the mxid of the voters and the content URI stored in the moderation trail (if it wasn't a moderator)
* If a senior member voted to keep it but a moderator later determines that the content was indecent, it would both remove the content and demote both the newbie user and possibly the voting senior member(s)

## Privileges per power level

If certain items can not be constrained by power levels within matrix itself, the bot just redacts every such event individually and demotes after repeated attempts.

* unvoiced: default, show 1 CAPTCHA, or 2 if risky (e.g., if from a home server lacking email address and CAPTCHA)
* unvoiced with numeric emoji allowed: to allow for client-side moderation implementations
* rate limit messages, especially if no senior is talking between the messages of a user
* message may include textual Unicode (only US-ASCII in English rooms)
* no reactions
* statistical spam filtering level strict
* forward messages through bot
* no forwarding, voiced
* 1 character emoji reactions from the child-safe subset
* reply to or mention somebody who replied to or mentioned the user
* statistical spam filtering level lax
* standard English spell checker not enforced
* any 1 character emoji reaction
* setting a display name
* message may include non-textual Unicode (emoji)
* message formatting other than preformatted text and code
* preformatted text and code snippet
* links to room and safe sites
* other links
* potentially nasty words
* reply
* mentions to 1 recently active user
* a few mentions
* setting a profile picture
* upload media
* upload file attachments
* voting to voice newbies
* voting to unvoice newbies
* elevated limits for message length
* elevated limits for vertical message size (more lines of text)
* elevated limits for event rate limiting
* free text reactions
* inviting others
* more mentions

### Matrix power levels

* -2..39 power level range managed automatically without moderator action
* -3..49 power level range a moderator can set through a bot
* -3 silenced by moderator
* -2 silenced due to senior voting, pending moderator review
* -1 silenced automatically due to an event, pending moderator review
* 0 default power level (fully voiced except when under attack)
* 1 allow emoji (for in-band CAPTCHA purposes, redact non-numeric)
* 2 forward messages through bot
* 3 can send a (safe) message to a room even in under attack level
* ...
* 40 manually promoted user not a candidate for moderator
* 41 manually promoted user who declined a moderator position
* 42..49 manually promoted user candidate for moderator
* 50 decentralized bot requiring only message redaction
* 51 decentralized bot requiring kick
* 52 decentralized bot requiring ban
* 53 centralized bot
* 54..60 human moderators: promoted incrementally and manually
* 54 most recent human moderator: also invited to the private moderator room
* 57 human moderator who may command a bot to change the power level of those within the designated range
* 60 most senior human moderator
* 70 bot requiring changing power levels of others
* >=71 other general moderator/admin levels of power
* 90 human moderator who can change Server ACL through a bot
* 91 bot requiring Server ACL

## Power level changes

### Elevated initial power level

A new member may start at an intermediary power level around where a profile picture can be set:

* those invited
* member of an invite-only space on the allowlist
* high past reputation within common rooms according to HS federation stats or a traveler bot

### Automatic promotion

* ask in a poll before first forwarding so that a potent human will be around
* on the first few levels, based on the number of senior interactions
* limit the number of users that can be newly voiced within a time interval
* wait at least a few minutes on lower levels (an hour on higher levels) before each member proposal so that a few senior read receipts or message events can arrive

### Automatic demotion

With low reputation:

* circuit breaker & voting
* mute on first incident
* depending on action, request another CAPTCHA before a moderator review

### Moderator promotion

The most senior members can be manually promoted to moderator candidate power levels based on statistics of past messages:

* Is the user right in the given message or is she then falsified?
* Is it strictly on topic?
* Is it friendly? Does it incite further friendly and informative comments or does it result in a chilled atmosphere?
* Is it motivated by altruism (e.g., to help someone solve a problem or answer a question) or egoism (leaching, asking questions all the time)?

It would be desirable to automate this in the long run.
Not all of these can be detected trivially, but most can actually be analyzed statistically via NLP.
We may first consider a crude approximation by aggregating reactions given by senior members: reactji and a few strongly correlating keywords in direct replies and mentions.

## Reputation

### User reputation

* time since first voiced message
* number of days when read receipt was updated
* number of days when sent a new message
* number of days when sent a reacji on a recent message at most 1 day old or within the last 20 visible messages in a room
* total number of replies and mentions received from seniors
* total number of sent messages
* total number of commons rooms present
* special roles (kick, ban, redaction) given in foreign rooms in common where a reputable user is an admin
* number of device sessions and their change or renaming over time
* have set a profile image
* have set a display name
* bot detection based on read receipt and online status updates: no downtime, periodicity

### User reputation from threads

We could periodically analyze the chat for threads:

* https://gitlab.com/bkil/freedom-fighters/-/blob/master/en/server/linear-chat-forum-threading.md

Within each thread:

* determine who the user replied to (and then later received the reply) and who replied to the user
* the more seniors a user interacts with, the faster its reputation increases
* only count non-redacted and non-negative interactions
* it is stronger if somebody reacts to the user than the other way around, unless it is part of a chain

### User event risk

* a long time had elapsed since last message
* any recently redacted message
* was demoted recently
* any redacted message ever
* was demoted ever

### Home server reputation

* software updated recently
* email address mandatory
* CAPTCHA mandatory (ideally at the date of registration)
* DDoS protection on its homepage
* number of server lists where it is advertised is low
* historical abuse rate is low (indicative of presence on a server list)
* recent abuse rate is low (indicative of scripts)
* server has been part of the federation long enough
* not dynamic DNS or a home ISP
* IP ASN in a data center
* mature age of last IP change
* server ACL not in ban list rooms (only as a signal)

### Soft blocking of home server

* against advanced username or display name spamming
* if in under attack mode
* if an abnormally high amount of joins received overall or from this home server
* apply a much more strict username spam filtering
* kick all new joiners coming from here and redact join events
* save to moderation log
* consider reinviting some of them after the troll waves are over

## CAPTCHA

For an overview about the various solutions to filter out robots from the website that serves us the CAPTCHA:

* https://gitlab.com/bkil/freedom-fighters/-/blob/master/hu/server/passive-abuse-prevention.md
* https://gitlab.com/bkil/freedom-fighters/-/blob/master/hu/server/active-abuse-prevention.md

For our purposes, the difficulty and type of the CAPTCHA could be influenced automatically by the threat level.
Easier and self-hosted CAPTCHA could be server on lower threat levels.
More thorough analysis could be ran at higher levels.
