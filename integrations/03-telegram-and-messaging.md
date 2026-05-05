# Telegram, Messaging Apps & Trigger Comparison

> Author: Shrinivas Ramaprasad | May 2026

---

## Telegram — Complete Node Guide

**Credential setup:** `credentials/02-credentials-guide.md` → Telegram section

### Telegram Node — Resource: Message, Operation: Send Message

| Field | Options | Sample Value | Required | What It Does | If Wrong |
|---|---|---|---|---|---|
| Credential | Dropdown | `Telegram - IT Support Bot` | Yes | Auth all API calls | Node fails |
| Resource | Chat, Callback, File, Message | `Message` | Yes | What to work with | Wrong ops shown |
| Operation | Send Message, Send Photo, Send Document, Send Audio, Edit, Delete, Pin, Send and Wait | `Send Message` | Yes | Action to perform | Wrong fields |
| Chat ID | Number or expression | `{{ $json.chatId }}` | Yes | Where to send | `chat_id is empty` |
| Text | Any text or HTML | `Hello {{ $json.name }}!` | Yes | Message content | Empty message error |
| Parse Mode | Markdown, HTML, None | `HTML` | No | Text formatting | Plain text if none |
| Disable Notification | Toggle | OFF | No | Silent message | Notification sent by default |
| Disable Link Preview | Toggle | OFF | No | Hide URL previews | Previews shown by default |
| Reply to Message ID | Number | `{{ $json.messageId }}` | No | Thread reply | No thread if empty |
| Reply Markup | Inline Keyboard, Reply Keyboard | Inline Keyboard | No | Add buttons | No buttons if empty |

Parse Mode HTML sample:

```
<b>New IT Ticket</b>

<b>From:</b> {{ $json.name }}
<b>Email:</b> {{ $json.email }}
<b>Issue:</b> {{ $json.issue }}
<b>Priority:</b> {{ $json.priority }}
<b>Ticket ID:</b> #{{ $json.ticketId }}

<a href="{{ $json.ticketUrl }}">View in system</a>
```

### Operation: Send and Wait for Response

| Field | Options | Sample | What It Does |
|---|---|---|---|
| Chat ID | Number | `{{ $json.chatId }}` | Who to send to |
| Message | Text | `Approve this request?` | Message shown |
| Response Type | Approval, Free Text, Custom Form | `Approval` | How user responds |
| Approval Options | Add buttons | Approve / Reject | Button labels |
| Free Text → Button Label | Text | `Submit Response` | Button text |
| Options → Limit Wait Time | Toggle + duration | ON, 2 Days | Auto-timeout |
| Options → Append Attribution | Toggle | OFF | Hides "Sent with n8n" |

- **Approval:** Adds Approve/Reject inline buttons. Workflow resumes when tapped.
- **Free Text:** n8n generates a form URL. User opens it and types.
- **Custom Form:** You define fields. n8n hosts the form.

### Operation: Send Photo

| Field | Sample | Notes |
|---|---|---|
| Chat ID | `{{ $json.chatId }}` | Destination |
| Binary Property / URL | `data` or `https://...` | Photo source |
| Caption | `Report {{ $now.toFormat('dd MMM') }}` | Text below photo |

### Telegram Trigger Node

| Field | Options | What It Does |
|---|---|---|
| Updates | Message, Edited Message, Callback Query, Inline Query, Channel Post | Which events fire |
| Additional Fields → Download Images | Toggle | Auto-download photos |

Trigger output:

```json
{
  "message": {
    "message_id": 123,
    "from": { "id": 456789, "first_name": "Shrinivas", "username": "shrinivas_r" },
    "chat": { "id": 456789, "type": "private" },
    "text": "/ticket My laptop won't boot",
    "date": 1714456800
  }
}
```

Access data:

```
{{ $json.message.chat.id }}      → Chat ID (use in Send Message)
{{ $json.message.from.first_name }}
{{ $json.message.text }}
{{ $json.callback_query.data }}  → Button data from inline keyboard
```

**⚠️ Common Telegram mistakes:**

| Mistake | Effect | Fix |
|---|---|---|
| Group Chat ID without negative sign | Wrong target | Group IDs are negative numbers |
| Bot not admin in channel | Cannot post | Channel Settings → Admins → add bot |
| Sending to user who blocked bot | 403 error | Handle with On Error: Continue |
| HTML tags not closed | Message fails | Always close: `<b>text</b>` |
| Bot not added to group | No messages | Add bot as member |

---

## Messaging App Comparison

| Platform | n8n Node | Setup | Stability | Free? | Best For |
|---|---|---|---|---|---|
| **Telegram** | ✅ Built-in | 5 min | ⭐⭐⭐⭐⭐ | ✅ Yes | Personal, small team, bots |
| **Slack** | ✅ Built-in | 20 min | ⭐⭐⭐⭐ | Freemium | Corporate teams |
| **Discord** | ✅ Built-in | 15 min | ⭐⭐⭐⭐ | ✅ Yes | Dev communities |
| **WhatsApp Business** | Via HTTP | Complex | ⭐⭐⭐ | ❌ Paid | Customer comms |
| **Microsoft Teams** | ✅ Built-in | Medium | ⭐⭐⭐⭐ | MS license | Enterprise |
| **Email (SMTP)** | ✅ Built-in | Easy | ⭐⭐⭐⭐⭐ | ✅ Yes | Formal, universal |

### Recommendation

**🥇 Telegram** — Best for most self-hosted n8n automations:
- Free forever
- Works without ngrok (polling mode)
- `Send and Wait for Response` built-in (approvals)
- 5-minute setup via BotFather
- Works on all devices

**🥈 Slack** — Best if team already uses Slack for work.

**⚠️ WhatsApp** — Only if your audience specifically requires it. Complex Meta verification, per-message costs.

---

## Discord Node

Quick webhook method (no bot token needed):

```
1. Discord server → channel → Edit Channel → Integrations → Webhooks
2. New Webhook → copy URL
3. n8n: HTTP Request node
   Method: POST
   URL: (your Discord webhook URL)
   Body:
   {
     "content": "{{ $json.message }}",
     "username": "n8n Bot"
   }
```

Rich embed:

```json
{
  "embeds": [{
    "title": "{{ $json.title }}",
    "description": "{{ $json.description }}",
    "color": 3066993
  }]
}
```
