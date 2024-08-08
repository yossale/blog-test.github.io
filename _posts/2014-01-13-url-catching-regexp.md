---
title: Url catching regexp

published: true
---

Not exactly a brilliant piece of engineering, but this is a useful dirty hack.  If you want to clean urls from a string, you can match it using this regexp:

For `http://` kind of urls:

```java
text.replaceAll(/(https|http):\\/\\/[a-zA-Z0-9\-\._~:\\/\?#\[\]@!\\u0024&'\(\)\*\+,;=]+/, "")
```

For the `www.` kind:

```java
text.replaceAll(/www\.[a-zA-Z0-9\-\._~:\\/\?#\[\]@!\\u0024&'\(\)\*\+,;=]+/, "")
```

The main point here is that the above characters are the only one allowed in urls, so every string that matches these is a url. It doesn't work for stuff like `io.com`.

