# 🚀 FCA-xnil6x (xnil-ypb-fca)

[![npm version](https://img.shields.io/npm/v/xnil-ypb-fca.svg)](https://www.npmjs.com/package/xnil-ypb-fca)
[![npm downloads](https://img.shields.io/npm/dm/xnil-ypb-fca.svg)](https://www.npmjs.com/package/xnil-ypb-fca)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Node.js Version](https://img.shields.io/node/v/xnil-ypb-fca.svg)](https://nodejs.org)

💁 **xnil-ypb-fca** is a bug-fixed and production-ready fork of `xnil-ypb-fca`, an advanced Facebook Chat API (FCA) client for **reliable**, **real-time**, and **modular** interaction with Facebook Messenger.

## 🔧 What's Fixed in This Fork?

This fork addresses **critical production bugs** found in the original `xnil-ypb-fca`:

✅ **Memory Leak Fixed** - Old MQTT clients and event listeners are now properly cleaned up  
✅ **Hot Reconnect Loop Resolved** - No more infinite reconnection loops consuming CPU  
✅ **Proper Resource Management** - All timers, connections, and listeners are properly released  
✅ **Maximum Retry Limits** - Bot stops gracefully after reasonable attempts (50 reconnects, 10 getSeqID failures)  
✅ **Unhandled Promise Rejections Fixed** - Safe promise handling prevents Node.js crashes  
✅ **Production-Ready Stability** - Proper notifications when bot stops due to connection issues  

**Before:** Bot would run forever, consuming memory and CPU indefinitely  
**After:** Bot manages resources efficiently and stops gracefully when limits are reached

---

## 📚 Documentation

- **[Theme Features](THEME_FEATURES.md)** - Comprehensive guide to theme management
- **[Changelog](CHANGELOG.md)** - Version history and updates
- **[Examples](examples/)** - Code examples and usage patterns

### Support & Issues

If you encounter issues or want to contribute, please open an issue on GitHub.

---

## ✨ Features

* 🔐 **Precise Login Mechanism**
  Dynamically scrapes Facebook's login form and submits tokens for secure authentication.

* 💬 **Real-time Messaging**
  Send and receive messages (text, attachments, stickers, replies).

* 📝 **Message Editing**
  Edit your bot's messages in-place.

* ✍️ **Typing Indicators**
  Detect and send typing status.

* ✅ **Message Status Handling**
  Mark messages as delivered, read, or seen.

* 📂 **Thread Management**
  * Retrieve thread details
  * Load thread message history
  * Get lists with filtering
  * Pin/unpin messages

* 👤 **User Info Retrieval**
  Access name, ID, profile picture, and mutual context.

* 🖼️ **Sticker API**
  Search stickers, list packs, fetch store data, AI-generated stickers.

* 💬 **Post Interaction**
  Comment and reply to public Facebook posts.

* ➕ **Follow/Unfollow Users**
  Automate social interactions.

* 🌐 **Proxy Support**
  Full support for custom proxies.

* 🧱 **Modular Architecture**
  Organized into pluggable models for maintainability.

* 🛡️ **Robust Error Handling**
  Retry logic, consistent logging, and graceful failovers.

* 🔄 **Production-Ready Reconnection**
  Smart exponential backoff with maximum retry limits.

---

## ⚙️ Installation

> **Requirements:** Node.js v18.0.0 or higher

```bash
npm install xnil-ypb-fca
```

Or use the latest version:

```bash
npm install xnil-ypb-fca@latest
```

---

## 🔒 Security Warning

**IMPORTANT:** `appstate.json` contains your Facebook session credentials and should be treated as sensitive information.

- ⚠️ **Never commit `appstate.json` to version control**
- ⚠️ **Never share your `appstate.json` file publicly**
- ⚠️ **Keep it in `.gitignore`** (already configured in this project)
- ⚠️ **Use environment-specific credentials** for production deployments

Your `appstate.json` gives full access to your Facebook account. Treat it like a password!

---

## 🚀 Getting Started

### 1. Generate `appstate.json`

This file contains your Facebook session cookies. Follow these steps:

1. **Install a cookie export extension** for your browser:
   - Chrome/Edge: "C3C FbState" or "CookieEditor"
   - Firefox: "Cookie-Editor"

2. **Log in to Facebook** in your browser

3. **Export cookies** using the extension and save them in this format:

```json
[
  {
    "key": "c_user",
    "value": "your-user-id"
  },
  {
    "key": "xs",
    "value": "your-xs-token"
  },
  {
    "key": "datr",
    "value": "your-datr-token"
  }
]
```

Save this as `appstate.json` in your project directory.

### 2. Basic Usage Example

```javascript
const { login } = require("xnil-ypb-fca");

login({ appState: require("./appstate.json") }, (err, api) => {
  if (err) {
    console.error("Login failed:", err);
    return;
  }

  console.log("✅ Logged in successfully!");

  // Listen for messages
  api.listenMqtt((err, message) => {
    if (err) {
      console.error("Listen error:", err);
      return;
    }

    if (message.type === "message") {
      console.log(`Message from ${message.senderID}: ${message.body}`);

      // Reply to the message
      api.sendMessage(
        `You said: ${message.body}`,
        message.threadID,
        (err) => {
          if (err) console.error("Send error:", err);
        }
      );
    }
  });
});
```

### 3. Advanced Usage with Options

```javascript
const { login } = require("xnil-ypb-fca");

const loginOptions = {
  selfListen: false,          // Don't listen to own messages
  listenEvents: true,         // Listen to events (joins, leaves, etc.)
  autoReconnect: true,        // Auto-reconnect on disconnect
  autoMarkRead: true,         // Auto-mark messages as read
  online: true,               // Show as online
  emitReady: true,           // Emit ready event when connected
  userAgent: "Mozilla/5.0 ..." // Custom user agent
};

login({ appState: require("./appstate.json") }, loginOptions, (err, api) => {
  if (err) {
    console.error("Login failed:", err);
    return;
  }

  api.listenMqtt((err, event) => {
    if (err) {
      // Handle errors - bot will auto-reconnect
      if (err.type === "stop_listen") {
        console.error("Bot stopped due to:", err.error);
      }
      return;
    }

    // Handle different event types
    switch (event.type) {
      case "message":
        console.log("New message:", event);
        break;
      case "message_reaction":
        console.log("New reaction:", event);
        break;
      case "ready":
        console.log("Bot is ready!");
        break;
      default:
        console.log("Event:", event);
    }
  });
});
```

---

## 📦 API Methods

### Messaging
- `sendMessage(message, threadID, callback)` - Send a message
- `sendTypingIndicator(threadID, callback)` - Send typing indicator
- `markAsRead(threadID, callback)` - Mark thread as read
- `markAsDelivered(messageID, threadID, callback)` - Mark message as delivered
- `editMessage(text, messageID, callback)` - Edit a message

### Thread Management
- `getThreadInfo(threadID, callback)` - Get thread information
- `getThreadHistory(threadID, amount, timestamp, callback)` - Get message history
- `getThreadList(limit, timestamp, tags, callback)` - Get thread list
- `changeThreadColor(color, threadID, callback)` - Change thread color
- `changeGroupImage(image, threadID, callback)` - Change group image

### User Information
- `getUserInfo(ids, callback)` - Get user information
- `getUserID(name, callback)` - Get user ID from name
- `getFriendsList(callback)` - Get friends list

### Other
- `logout(callback)` - Logout and cleanup
- `setOptions(options)` - Update API options

---

## 🎨 AI Theme Generation

xnil-ypb-fca includes advanced AI theme generation capabilities. See [THEME_FEATURES.md](THEME_FEATURES.md) for details.

```javascript
// Create a custom AI-generated theme
api.createAITheme({
  threadID: "thread_id_here",
  themePrompt: "ocean sunset with purple gradients",
  callback: (err, result) => {
    if (err) console.error(err);
    else console.log("Theme created:", result);
  }
});
```

---

## 🔧 Configuration Options

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `selfListen` | boolean | false | Listen to your own messages |
| `listenEvents` | boolean | true | Listen to events (joins, leaves, etc.) |
| `autoReconnect` | boolean | true | Automatically reconnect on disconnect |
| `autoMarkRead` | boolean | true | Automatically mark messages as read |
| `autoMarkDelivery` | boolean | false | Automatically mark messages as delivered |
| `online` | boolean | true | Show as online |
| `emitReady` | boolean | false | Emit ready event when connected |
| `userAgent` | string | (default) | Custom user agent string |
| `proxy` | string | null | Proxy server URL |

---

## 🐛 Troubleshooting

### Bot Not Receiving Messages
- Verify your `appstate.json` is valid and up-to-date
- Check that you're logged in to Facebook in a browser
- Ensure your account isn't checkpoint restricted

### Login Errors
- Re-export your cookies using the browser extension
- Clear Facebook cookies and login again
- Try using a different browser

### Connection Issues
- Bot will auto-reconnect up to 50 times with exponential backoff
- If you see "Max reconnect attempts exceeded", check your network connection
- If you see "Max consecutive getSeqID failures", your session may be invalid

### Memory/CPU Issues
- ✅ **FIXED in this fork!** The original `xnil-ypb-fca` had memory leaks
- This version properly cleans up resources and stops after reasonable retry attempts

---

## 📄 License

MIT License - see [LICENSE](LICENSE) file for details.

---

## 🙏 Credits

- Original `xnil-ypb-fca` by **YPB Team**
- Bug fixes and production improvements in this fork
- Inspired by `ws3-fca` and the Facebook Chat API community

---

## 🔗 Links

- **npm Package:** [https://www.npmjs.com/package/xnil-ypb-fca](https://www.npmjs.com/package/xnil-ypb-fca)
- **Original Package:** [https://www.npmjs.com/package/xnil-ypb-fca](https://www.npmjs.com/package/xnil-ypb-fca)

---

## 📝 Changelog

See [CHANGELOG.md](CHANGELOG.md) for version history and updates.

---

**Made with ❤️ for the Facebook Chat API community**
