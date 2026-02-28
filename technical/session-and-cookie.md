Most people are familiar with cookies. For example, after logging into a website for a while, you will be asked to log in again. Or if you have tried web scraping, you may notice some websites can block your scraper. These things are related to cookies.

If you understand how the server handles cookies and sessions, you can explain these situations, and even find ways to bypass some website restrictions. Let me explain step by step.


<!-- hide -->

## Introduction to Session and Cookie

The reason we have cookies is because HTTP is a stateless protocol. In other words, the server cannot remember you. Each time you refresh a page, you would have to log in again with your username and password. This is obviously not acceptable, so cookies work like tags. The server "tags" you, and every time you send a new request to the server, it can recognize you through the cookie.

In short: **A cookie can be seen as a "variable" like `name=value`, stored in the browser. A session can be seen as a JSON object, stored on the server (usually in a cache database like Redis).**

Note that I said "a cookie can be seen as a variable," but the server can set multiple cookies at once. So sometimes we say a cookie is "a group of key-value pairs," and that's correct too.

Cookies can be set on the server side by using the HTTP SetCookie field. Here is a simple Servlet example:

```java
public void doGet(HttpServletRequest request, HttpServletResponse response) {
    // Set two cookies
    Cookie cookie1 = new Cookie("name1", "value1");
    response.addCookie(cookie1);

    Cookie cookie2 = new Cookie("name2", "value2");
    response.addCookie(cookie2);

    // Write string to web page
    response.getWriter().println("html content");
}
```

When you visit the corresponding website, you can use the developer tools in your browser to check the details of the HTTP communication. You will see that the server sends two `SetCookie` commands:

![](../pictures/session/1.png)

After this, every request from the browser contains these two cookies in the `Cookie` field:

![](../pictures/session/2.png)

**So, the purpose of cookies is quite simple: the server gives each client (browser) a tag to help identify them.** Of course, HTTP can set more cookie parameters, like expiration time, or only allowing a cookie in a specific path.

However, cookies have a size limit, and they are stored in the HTTP header. So every time a request is sent, cookies are transferred, which uses bandwidth. More importantly, some information is not good to keep on the client side, such as login status.

**Session works together with cookies to solve these problems.** The most common example is logging in.

After you log in successfully, the server creates a session to store your login information, for example:

```json
{
  "user_id": 12345,
  "username": "Zhang San",
  "role": "admin",
  "login_time": "2025-11-01 10:00:00"
}
```

Then, the server generates a random session ID (like `abcd1234`) and sends it to your browser with a cookie: `sessionID=abcd1234`.

After that, when you visit any page, your browser always sends this cookie. The server uses the ID to find the right session and knows your user ID and permissions, so you don’t need to log in every time.

Session data is usually stored in the server's memory database (like Redis), which is fast. These sessions are not saved forever—they have an expiration time. For example, if you do not use the website for three days, your session will be deleted automatically. That’s why some websites ask you to log in again after you haven’t used them for a while.

By the way, once you understand cookies, you'll notice that some websites allow you to use a service a limited number of times if you don’t log in. How do they do this?

They just set a session ID in your browser's cookie. If you delete your cookies, you can bypass the use limit. Of course, don’t overdo this, as websites need to make money too.

These are the basics of cookies and sessions. Cookie is a part of the HTTP protocol and not very complex. Session is something that can be customized. Next, let’s look at how to implement session management in code.


## How Session Works

The idea behind sessions is simple, but the implementation has some tricks. Usually, it needs three parts working together: `Manager`, `Provider`, and `Session`.

![](../pictures/session/4.jpg)

Let's use the example where a logged-in user visits their profile page to understand the process:

1. The user (already logged in) visits the `/profile` page. The Handler function gets the request and reads the sessionID (like `abcd1234`) from the cookie in the HTTP header, then gives this ID to the `Manager`.

2. The `Manager` handles session management and stores global settings (like session timeout, cookie name, etc). It passes the sessionID to the internal `Provider` to look up the session.

3. The `Provider` is like a container. It stores sessions for all users in a hash table (for example: Zhang San's session, Li Si's session, and so on). It finds the `Session` object for this sessionID and returns it.

4. The `Session` object holds the user's login info (such as user_id, username, role, etc). The Handler reads this data and creates a personalized page for the user (for example: "Welcome back, Zhang San!") and returns it to the client.

You may wonder: Why make it so complicated? Why not just use a hash table in the Handler to map `sessionID → login info`?

Below is the reason why we split it into three layers.

### Why Do We Need the Session Layer?

Although a session is just a key-value store (user_id, username, etc.), we can't simply use a hash table. There are two reasons:

**1. Need to store extra data:** Besides login info, we also have to keep sessionID, last access time, expiry time, etc, to help clear expired sessions (like the LRU algorithm).

**2. Support different storage methods:** If you just use an in-memory hash table, all users will be logged out when the server restarts, and it takes too much memory with lots of users. In real use, sessions can be kept in Redis, MySQL, or elsewhere.

The `Session` interface gives a standard way to use sessions so that the storage details are hidden:

```java
interface Session {
    // Set a key-value pair
    void set(String key, Object value);

    // Get the value of a key
    Object get(String key);

    // Delete a key
    void delete(String key);
}
```

### Why Do We Need the Provider Layer?

The `Provider` manages all online users' sessions. In simple cases, a hash table (sessionID → Session) is enough, but real apps are more complex:

**Need to clean up expired sessions automatically:** If a website has 10,000 users online, you can't manually delete expired sessions. You can use an LRU algorithm to remove them, which needs a hash-linked list data structure.

::: tip Tip

For more details about the LRU algorithm, see [LRU Algorithm Explained](https://labuladong.online/en/algo/data-structure/lru-cache/).

:::

The `Provider` hides these details and only shows simple add, delete, find, and update methods:

```java
interface Provider {
    // Create and return a session
    Session sessionCreate(String sid);

    // Delete a session
    void sessionDestroy(String sid);

    // Find a session
    Session sessionRead(String sid);

    // Update a session
    void sessionUpdate(String sid);

    // Remove expired sessions using an LRU-like method
    void sessionGC(long maxLifeTime);
}
```

### Why Do We Need the Manager Layer?

The `Manager` handles global settings, such as:
- Session timeout (30 minutes? 2 hours?)
- Cookie name (is it `sessionID` or `sid`?)
- Where to store sessions (memory? Redis? MySQL?)

The actual add, delete, find, and update work is done by `Provider` and `Session`. The `Manager` only manages configuration. This makes it easy to switch between implementations, like using memory in development but Redis in production.

The main idea of this three-layer design is decoupling: The Session layer handles single user data; the Provider layer manages all users; the Manager handles global settings. Each part can be changed separately without affecting others.
