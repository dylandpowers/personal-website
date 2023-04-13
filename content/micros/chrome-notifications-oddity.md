---
title: A Chrome Notifications API Oddity
tags: [Chrome, JavaScript, API]
date: 2023-04-12
---

I'm developing a Chrome extension which is basically the same thing as [Key Promoter X](https://plugins.jetbrains.com/plugin/9792-key-promoter-x) in IntelliJ, which reminds you about common keyboard shortcuts you could be using instead of using the mouse like a Plebeian.

I was using the `chrome.notifications.create` API to create a notification with a passed-in `notificationId`:

```javascript
chrome.notifications.create(notificationId, {
  type: "basic",
  title: "Chrome Key Promoter",
  message,
  iconUrl: "../icon.jpeg",
  buttons: [{ title: "Silence" }],
});
```

However, I could not get the notification to show up no matter what I did. Reloading the extension didn't help. The [documentation](https://developer.chrome.com/docs/extensions/reference/notifications/#method-create) clearly states that, if you pass a `notificationId`:

> If it matches an existing notification, this method first clears that notification before proceeding with the create operation.

However, this is not the case. I believe the API documentation is simply out of date, because when I manually cleared the notification beforehand using `chrome.notifications.clear(notificationId)`, the notification starting appearing as expected.

TL;DR if you run into this same issue, just manually clear the notification before creating it.
