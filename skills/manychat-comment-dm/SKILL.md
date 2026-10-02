---
name: manychat-comment-dm
description: >
  Use when someone wants an Instagram "comment a keyword, get the link in DMs"
  automation set up in ManyChat — e.g. "set up ManyChat for my reel", "comment
  to DM", "auto-DM the link when people comment", "follow to get the link",
  "keyword automation". Builds (or clones from a share link) a flow with a
  public comment reply, an opening DM, a follow check with recheck, a link
  button, and an optional 1-hour bump. Drives the user's own Chrome through
  Claude in Chrome. No ManyChat API key needed.
---

# ManyChat comment-to-DM (follow-gated)

Builds this flow in the user's own ManyChat account:

```
Someone comments on the reel
  ├─ Public reply under the comment: "Sent! Check your DMs 📩"
  └─ DM 1: "Hey {First Name}, you wanted to get the link right?"   [Yes, send it]
                                                                        │
                     ┌──────────── Condition: follows your account? ────┘
                     │ yes                                  │ no
                     ▼                                      ▼
   DM 2: "Here you go 👇"  [Get the link 👇 → URL]    DM: "Almost there! Follow @handle
                     │                                     first, then tap below 👇"
             Smart Delay 1 hour (optional)                  [I followed ✅] → back to Condition
                     ▼
   Bump: "Hey {First Name}, in case you missed it,
          here's the link again 👇"  [Get the link 👇 → URL]
```

Why DM 1 has a button: Instagram only lets a business send one private reply
to a commenter. Tapping "Yes, send it" opens the 24-hour messaging window, which
is what allows the follow check, the link and the 1-hour bump to go out.

No API is involved. ManyChat's public API cannot create flows, triggers or
delays, so everything is done in the ManyChat web app through the browser.

## Step 0 — Collect inputs

Ask in one message. Offer the defaults so the user can just say "defaults".

| Input | Default |
|---|---|
| Instagram handle (for the follow prompt) | required |
| Link to send | required |
| Trigger type | ask: keyword on one reel / any comment on one reel / keyword on any post |
| Keyword(s) (if keyword trigger) | required for keyword triggers, e.g. `LINK` |
| Which reel (if one reel) | URL of the reel, "most recent", or "not posted yet" (use Next Post or Reel) |
| Opening DM | `Hey {First Name}, you wanted to get the link right?` |
| Opening button | `Yes, send it` |
| Link message + button | `Here you go 👇` / `Get the link 👇` |
| Public replies (ManyChat rotates them) | `Sent! Check your DMs 📩` · `Just sent it to your DMs 🙌` · `Check your inbox 👀` |
| Not-following message | `Almost there! Follow @{handle} first, then tap below 👇` / button `I followed ✅` |
| 1-hour bump | on, text `Hey {First Name}, in case you missed it, here's the link again 👇` |
| Template share link | optional. If they have one, use Path A |
| Flow name | `Comment DM - {keyword or reel topic}` |

If the reel isn't posted yet, use the trigger's **Next Post or Reel** option. ManyChat
binds the automation to whatever goes up next on Instagram, so the user can build first
and post straight after. Warn them to post nothing else on Instagram in between,
otherwise the trigger attaches to the wrong post.

If the user gives you the video instead of the keyword, take the keyword from the
on-screen text or the frames. Don't trust a transcript alone: Whisper hears "claude"
as "cloth".

## Step 1 — Preflight (tell the user, don't do it for them)

1. Instagram is a Professional (Business or Creator) account connected to ManyChat.
2. In the Instagram app: Settings → Messages and story replies → Message controls →
   Connected tools → **Allow access to messages** is ON. Without this, ManyChat can't DM anyone.
3. The user is logged into ManyChat in Chrome. Never type their password or 2FA code;
   if you land on a login page, stop and ask them to log in.

## Step 2 — Connect to Chrome

Load the Claude in Chrome tools in one call:

```
ToolSearch "select:mcp__claude-in-chrome__tabs_context_mcp,mcp__claude-in-chrome__navigate,mcp__claude-in-chrome__computer,mcp__claude-in-chrome__find,mcp__claude-in-chrome__read_page,mcp__claude-in-chrome__javascript_tool,mcp__claude-in-chrome__tabs_create_mcp,mcp__claude-in-chrome__list_connected_browsers,mcp__claude-in-chrome__select_browser"
```

If more than one browser is connected, `list_connected_browsers` and ask which one
is the user's. Open `https://app.manychat.com/`. After login ManyChat redirects to
`https://app.manychat.com/fb<ACCOUNT_ID>/...`. Read the account ID from the URL and
use it for every later URL. If the user has several ManyChat accounts, confirm which
one before building anything.

## Path A — Clone from a share link (fastest)

1. Navigate to the share link. ManyChat shows a template preview with an install/clone button.
2. Click it, pick the user's account, confirm. ManyChat opens the cloned flow.
3. Continue at Step 4 (customise). All nodes already exist; only edit text, URLs, handle and trigger.

If the link is broken, or the clone is missing the condition or delay nodes, switch to Path B.

## Path B — Build from scratch

Work in the Flow Builder (visual canvas). Labels move around between ManyChat
versions, so find controls by their visible text with `find` and don't rely on
fixed coordinates.

0. **Check for keyword clashes.** In Automations, click "Load more" and every "Show All"
   row, then list the automations using the same keyword. Ones bound to a specific post are
   fine, since each only fires on its own post. Only a LIVE automation on **any post** with
   the same keyword clashes. Ask the user before pausing it, and never delete it.
1. **New flow.** Go to `https://app.manychat.com/fb<ID>/cms` → New Automation → Start from scratch.
   Rename it to the flow name.
2. **Trigger.** Add Trigger → Instagram → **User comments on your Post or Reel**.
   - Post: specific post (pick the reel), **Next Post or Reel** (reel not posted yet), or
     any post, based on the trigger type.
   - Keywords: "contains" + the keyword(s), or "any word" for any-comment.
   - Turn ON **reply to comments** publicly and add the public reply variations.
   - Save.
3. **DM 1.** The trigger connects to an Instagram Send Message node. Text = opening DM.
   Insert First Name through the variable picker (`{}` icon); don't type braces by hand.
   Add button = opening button text. Use a regular message button whose next step is the
   Condition, not a website button. ManyChat counts the tap as a reply, which opens the
   24-hour window. If this DM shows no "Next Step" option and a website button only offers
   actions, that's the private-reply limit: the opener button is the only way forward.
4. **Condition.** Drag the button's connector → Condition. Rule: Instagram follower status,
   shown as "Is follower" or "Follows your account". Search the condition list for "follow".
   If no follow condition exists in this account, stop and tell the user. Don't fake it.
5. **Yes branch.** Send Message: link message text + button → Open website → the link URL.
6. **No branch.** Send Message: not-following text + button "I followed ✅". Connect that
   button's output back into the same Condition node, so it rechecks every time they tap.
7. **Bump (if on).** From the Yes-branch message, add Smart Delay → 1 hour → Send Message
   with the bump text + the same URL button.
8. **Publish.** Click Set Live / Publish. The header should read LIVE / Active.

## Step 4 — Customise (Path A) or double-check (Path B)

Open every node and confirm against the inputs:
- trigger post and keywords, and that public reply is on with the variations
- DM 1 text uses the real First Name variable (shows as a chip, not literal `{First Name}`)
- the condition is the follow check, and the No branch loops back to it
- every URL button points to the user's link. Cloned flows keep the template owner's URL until changed.
- the Smart Delay is 1 hour, not 1 day
- no leftover empty "New Button" buttons
- the flow is LIVE
- if you used Next Post or Reel: after the reel goes up, the automations list shows the
  trigger bound to a specific post with its thumbnail. Check the thumbnail is the right reel.

## Step 5 — Test

ManyChat ignores comments from the account itself. Ask the user to comment the keyword
from a second Instagram account, once while not following and once after following. Expected:
public reply appears → DM 1 → tap → follow prompt (not following) or link (following) →
bump after 1 hour. Report what actually happened, step by step.

## Driving ManyChat reliably

- Prefer `find` and small `read_page` calls with `ref_id`. A full `read_page` of the flow builder is huge.
- Use real clicks (`computer` left_click on a ref or coordinate). JS `.click()` is ignored by some
  ManyChat controls, toggles especially.
- To replace text in a field: click it, select all (cmd+a / ctrl+a), Delete, then `type`. Check the
  field value afterwards, because some ManyChat inputs append instead of replacing.
- If the flow canvas sits on a loading spinner forever, run this once through `javascript_tool`
  on the page and reload the builder view:
  ```js
  const of = window.fetch;
  window.fetch = async (...a) => { const u = typeof a[0]==='string'?a[0]:(a[0]?.url||'');
    if (u.includes('/settings/data')||u.includes('/customEvents/list'))
      return new Response(JSON.stringify({status:'success',data:[]}),{status:200,headers:{'Content-Type':'application/json'}});
    return of.apply(this,a); };
  ```
- If the canvas still won't load (common when the Chrome window is in the background),
  build in ManyChat's **Basic Builder** instead. It edits the same automation: messages,
  buttons, condition, Smart Delay and loop-back all work there. Afterwards, check the
  Flow Builder view shows every link.
- If the save button still says Update after editing, click it before moving on.
- If the session expires, stop and ask the user to log in. Never enter credentials.

## For the template owner: making a share link

Build the flow once with Path B in your own account, using placeholder text
(`https://example.com`, `@yourhandle`) and no reel selected. Then in the flow's menu use
Share / Clone link, copy it, and give it to members as the Path A link. Check that a
clone of it includes the condition, the loop-back and the Smart Delay before handing it out.
