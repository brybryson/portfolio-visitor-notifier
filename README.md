# Portfolio Visitor Notification

A lightweight n8n workflow that sends a Telegram notification whenever someone
visits your portfolio website. The notification includes the visitor's
approximate location, ISP, IP address, and visit timestamp.

---

## Overview

When a visitor loads your portfolio, the website sends a small `POST` request to
an n8n webhook. The workflow retrieves geolocation information using the
visitor's IP address and sends a formatted notification to a Telegram channel.

```text
Portfolio Website
        │
        ▼
Webhook
        │
        ▼
IP Geolocation Lookup
        │
        ▼
Telegram Notification
```

---

## Workflow

| Step | Node             | Description                                                                                                                  |
| ---- | ---------------- | ---------------------------------------------------------------------------------------------------------------------------- |
| 1    | **Webhook**      | Receives a `POST` request when a page is loaded.                                                                             |
| 2    | **HTTP Request** | Looks up the visitor's IP using the `x-forwarded-for` header and retrieves location and network information from `ipwho.is`. |
| 3    | **Telegram**     | Sends a formatted HTML notification to a Telegram channel.                                                                   |

---

## Notification Format

The Telegram message includes:

- Visit timestamp
- City
- Region
- Country
- ISP
- Public IP address

All fields automatically fall back to `N/A` if the lookup service cannot provide
a value.

Example:

```text
New Portfolio Visitor
July 22, 2026 · 3:05 AM

Location
• City: Gavle
• Region: Gavleborg County
• Country: Sweden

Network
• ISP: Telia Company AB
• IP: xxx.xxx.xxx.xxx
```

The timestamp is formatted using the `Asia/Manila` timezone by default.

---

## Requirements

Before importing the workflow, ensure you have:

- An n8n instance
- A Telegram bot
- A Telegram channel
- The bot added as a channel administrator

Create your bot using **@BotFather**.

---

## Setup

### 1. Create a Telegram Credential

In n8n, create a **Telegram API** credential using your bot token.

> **Important**
>
> Never commit your bot token to source control. Store it only in n8n's
> credential manager.

---

### 2. Configure the Telegram Channel

Update the **Send a text message** node with your channel's `chatId`.

---

### 3. Publish the Workflow

Once activated, n8n exposes a webhook endpoint similar to:

```text
POST https://<your-n8n-host>/webhook/<your-webhook-id>
```

---

## Website Integration

Send a request whenever your portfolio loads.

```html
<script>
    fetch("https://<your-n8n-host>/webhook/<your-webhook-id>", {
        method: "POST",
        headers: {
            "Content-Type": "application/json",
        },
        body: JSON.stringify({
            page: window.location.pathname,
        }),
    });
</script>
```

The workflow reads the visitor's IP address from the `x-forwarded-for` request
header, which is typically added automatically by your hosting provider or
reverse proxy. You do not need to include the IP address in the request body.

---

## Customization

### Message Layout

Modify the **Send a text message** node to customize the Telegram message. The
workflow uses Telegram's HTML formatting, including tags such as:

- `<b>`
- `<i>`
- `<code>`

### Timezone

Update the timestamp expression if you want to use a different timezone instead
of `Asia/Manila`.

### Geolocation Provider

The workflow uses the free `ipwho.is` API. You can replace it with another
provider by updating the **HTTP Request** node.

---

## Notes

- IP geolocation is approximate and reflects the visitor's network location
  rather than their exact address.
- The free `ipwho.is` service has rate limits. High-traffic sites may require a
  paid provider or an alternative API.
- The Telegram bot must be an administrator of the target channel before it can
  send messages.

---

## License

MIT License
