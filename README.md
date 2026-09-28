# manychat-comment-dm

A Claude Code skill that sets up an Instagram comment-to-DM automation in ManyChat:

1. Someone comments on your reel and gets a public reply ("Sent! Check your DMs 📩").
2. They get a DM: "Hey {name}, you wanted to get the link right?"
3. ManyChat checks whether they follow you. If not, it asks them to follow and rechecks when they tap "I followed ✅".
4. Followers get the link button.
5. An hour later, a bump resends the link (optional).

Claude builds it by driving your own Chrome (Claude in Chrome extension). No ManyChat API key.

## Install

```bash
git clone https://github.com/kingsleylow123/manychat-comment-dm /tmp/manychat-comment-dm && mkdir -p ~/.claude/skills && cp -r /tmp/manychat-comment-dm/skills/manychat-comment-dm ~/.claude/skills/
```

## Before you run it

- Instagram is a Professional account connected to ManyChat.
- Instagram app → Settings → Messages and story replies → Message controls → Connected tools → Allow access to messages: ON.
- Claude in Chrome is installed and you're logged into ManyChat in Chrome.

## Use

In Claude Code, say: `set up ManyChat comment to DM for my reel`. Claude asks for your handle, link, keyword and reel, then builds it and tells you how to test it from a second account.
