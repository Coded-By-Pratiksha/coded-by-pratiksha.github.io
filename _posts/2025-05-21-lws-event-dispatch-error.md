---
layout: post
title: "Fixing the 'dispatchEvent' TypeError in Lightning Web Components"
date: 2025-05-21
cover-img: /assets/img/path.jpg
thumbnail-img: /assets/img/thumb.png
share-img: /assets/img/path.jpg
categories: [Salesforce, LWC, Debugging]
tags: [LWC, Salesforce, Error Handling, Lightning Web Security]
author: Pratiksha
---

While working on a Lightning Web Component (LWC) recently, I encountered an error that was quite confusing at first:

<span style="color:red">
Error fetching base URL: TypeError: Failed to execute 'dispatchEvent' on 'EventTarget': parameter 1 is not of type 'Event'.
</span>

This issue appeared unexpectedly and blocked some basic event-driven functionality within my component. In this post, I’ll explain the cause of the error and how I resolved it using **Lightning Web Security (LWS)** settings.

---

## 🔍 Root Cause

The error points to a misuse of the `dispatchEvent` function — specifically, the parameter passed is **not an instance of `Event`**. In standard JavaScript and the DOM API, `dispatchEvent()` expects a valid `Event` object. For example:

const myEvent = new CustomEvent('myEventName');
element.dispatchEvent(myEvent);
If you mistakenly pass something like a plain object or a string, the browser will throw a TypeError — exactly what happened in this case.

🛡️ The Role of Lightning Web Security (LWS)
Salesforce uses Lightning Web Security (LWS) to sandbox components and enforce better security practices. When LWS is enabled, it enforces stricter type checking than Locker Service used to, and that’s when I saw this error.

LWS does not allow dispatching non-Event objects, so it throws an error even if it seemed to work before under Locker.

✅ The Fix

The issue was ultimately resolved by enabling the "Enable Lightning Web Security" setting in the org, which aligns behavior with modern DOM APIs and enforces secure coding practices.

To enable LWS:

Go to **Session Settings** in Setup

Search for "Lightning Web Security" in Salesforce Setup.

Enable the toggle.

![lws](https://coded-by-pratiksha.github.io/assets/img/setup_lwsec_enable.avif)

Clear browser cache (just to be safe).

Refresh your components.

Additionally, make sure you're constructing and dispatching events properly in your JavaScript code:

const customEvent = new CustomEvent('someevent', {
  detail: { data: 'value' }
});
this.dispatchEvent(customEvent);

💡 Key Takeaways


Always ensure that you're passing an actual Event or CustomEvent object to dispatchEvent().

Enabling Lightning Web Security (LWS) can help catch improper event handling early.

LWS aligns your code with standard web practices and ensures more secure component isolation.

Have you faced similar LWC or LWS quirks? Drop a comment or let me know — this blog will continue to share hands-on Salesforce experiences and PD2 preparation notes.

Stay tuned! ✨


---

Let me know when you're ready for the next post!
