---
marp: true
theme: gaia
size: 4K
class: default
paginate: true
footer: '![](images/twitter_24.png) [asm0di0](https://twitter.com/asm0di0)'
backgroundImage: "linear-gradient(to bottom, #000 0%, #1a2028 50%, #293845 100%)"
color: white
title: "Projector: Your Remote IDE"
---
<!--
_class: lead
_paginate: false
_footer: ""
-->

<style>
.hljs-variable { color: lightblue }
.hljs-string { color: lightgreen }
.hljs-params { color: lightpink }
</style>

# <!-- fit --> Projector: Your Remote IDE

Pasha Finkelshteyn, JetBrains

---

# <!-- fit --> Did you ever complain?

- You need to set up a VPN to access data?
- You need to use RDP to access your instance of IDEA?
- Or even VNC?
- *Long* indexing?
- Huge project compiles forever?

![bg right:40%](https://source.unsplash.com/y6HpQzW87Vc)

---

# Did you ever want?

- Edit code from tablet/smartphone?
- Help your colleague to fix some issues?
- Show demo which accesses data from your cloud?
- Work from a weak laptop?

![bg right:40%](https://source.unsplash.com/UP7JSnodG2M)

---
<!--
_class: lead
-->
![bg fit](images/icon_128.svg)

---

# Issues with existing solutions. VPN

1. OpenVPN does not scale
    1. No central identity management
    1. No failover OOTB
1. Proprietary solutions are… proprietary
1. And expensive sometimes
1. IKE is not completely cross-platform

---

# Issues with existing solutions. VNC

1. **Extremely** slow
2. Complex infrastructure (VPN?)

---

# Issues with existing solutions. RDP

RDP is awesome

1. Single-user
1. Requires special client
2. Still needs complex infrastructure to provide access

---

![bg brightness:50%](https://source.unsplash.com/jyoSxjUE22g)

# Stop this

---

# Let's dig!

![bg brightness:70%](https://source.unsplash.com/bV5AnCQgeUY)

---

<style scoped>
p > img {
    display:block;
    margin:auto;
}
</style>

## Typical Swing interface
![height:540](images/swing_interface.png)

---

# And in reality?

- Lines
- Simple shapes
- Strings
- Fonts

---

# Swing architecture

![bg](images/swing_tree.png)

---

# Definitions

**Toolkit** — abstraction over the whole graphics
**Heavyweight components** — system-controlled components
**Lightweight components** — user-controlled components
**Graphics2D** — drawing controller

---

# Types of drawing surfaces

- Visible
- Invisible
    Part of the editor, which you don't see, is separate and is drawn on an invisible surface.

Each window has its surface(s)

---

# Heavyweight components

There is a corresponding `Peer` class for each heavyweight component.

```java
interface ComponentPeer {
    void paint(Graphics g);
    void setBounds(int x, int y, int width, int height, int op);
    void handleEvent(AWTEvent e);
    void setBackground(Color c);
    void setFont(Font f);
    // etc.
```

Responds for delegation to Graphics and setting several params

---
<!-- _class: lead -->
# Heavyweight components

We have implementations of peers only for several heavyweight components.

---

# Graphics2D

*Anything* may contain Graphics2D

Responds for drawing:

```java
abstract class Graphics2D {
    public abstract void drawImage(BufferedImage img, 
                                   BufferedImageOp op,
                                   int x, int y);
    public abstract void drawString(String str, int x, int y);
    public abstract void fill(Shape s);
    // etc.
```

---
<!-- _class: lead -->
![bg right:40%](https://source.unsplash.com/49uySSA678U)

# How the hell will it work in a browser?

---
# How the hell will it work in a browser?

The initial version just sent the whole screen to the client.

It had a whopping performance of ≈5 fps :hushed:

![bg right:40%](images/9000.png)

---

# How the hell will it work in a browser?

Welcome [DrawEventQueue](https://github.com/JetBrains/projector-server/blob/master/projector-awt/src/main/kotlin/org/jetbrains/projector/awt/service/DrawEventQueue.kt)

Queue that will transform any action into an event, based on

```java
ConcurrentLinkedQueue<List<ServerWindowEvent>>()
```

---
<style scoped>
p > img {
    display:block;
    margin:auto;
}
</style>

# `DrawEventQueue`

Each component has its queue (for now)
![width:1150](images/command.png)

---

# Images

There are at least five types of images in AWT :scream:

`VolatileImage`:
- is used to display off-screen surfaces
- is drawn by primitives 
- is being sent as commands.

---

# Images

`BufferedImage`:
- is being sent as bitmap
- we have sophisticated reflection-based heuristics to determine if it is changed
- may be optimized in future

---

# Images

`MultiresolutionImage`:
- used for drawing icons
- supported only if it has  the single resolution

---

# Images

**Other types** of images are not supported right now.

---

# So we have commands. Now what?

```java
while(!stopped){
    sendAllCommands() // sends both images and command
    Thread.sleep(10)
}
```

Every 10ms (roughly), aggregate all commands and send them to the client over WebSocket.

---

# Format

Default format: highly optimized JSON
Supported OOTB formats: Protobuf, JSON
May support anything if we'll implement it on both sides.

---
<style scoped>
p > img {
    display:block;
    margin:auto;
}
</style>

# Format

![width:1150](images/json.png)

Everything is determined in runtime — types, fields, etc.
For example, `"e"` at the very beginning is type!

---

# Backend

* Ktor?
* Tomcat?
* Undertow?
* Netty?
* Jetty?

---

![bg](https://source.unsplash.com/vBxbZokRL10)

---

# GPL is not always a good thing

Popular products are not compatible with it, and we HAVE TO publish source code under GPL license.

So…

## [TooTallNate/Java-WebSocket](https://github.com/TooTallNate/Java-WebSocket)

And a bunch of crutches to add HTTP support there.

---

# Clients

A client should implement ≈30 API methods of drawing + request resources

The excellent part of Kotlin: we can share some logic between client and server (and we do).

`kotlinx.serialization` _may_ do things more straightforward (but deserialization is sooo slow).

A non-read-only client should be able to send mouse and keyboard events, and so on.

---

# In browser

![bg](images/in_browser.png)

---

# In projector-launcher

![bg](images/in_client.png)

---

# Desktop client

- Electron-based
- (Almost) all browser hotkeys are disabled
- May me expanded to full screen

---
<!-- _footer: "" -->
# Installation: GNU/Linux

```bash
pipx install projector-installer
```

![height:400](images/projector_run.png)

---

# Configuration

![bg fit](images/config.png)

---

# Installation: Docker

[JetBrains/projector-docker](JetBrains/projector-docker)

```
docker pull \
    registry.jetbrains.team/p/prj/containers/projector-idea-u
docker run --rm -p 8887:8887 -it \
    registry.jetbrains.team/p/prj/containers/projector-idea-u

```

---

# Useful scripts

- Build a container with IDE of your choice
`build-container.sh [containerName [ideDownloadUrl]]`
- Run Stateless container
`run-container.sh [containerName]`
- Run container with mounted directory to preserve state
`run-container-mounted.sh [containerName]`

---

<style scoped>
p > img {
    display:block;
    margin:auto;
}
</style>

# Installation: IDE

![height:520 drop-shadow](images/plugin.png)

---
<!-- _footer: '' -->
# How does it work?

![width:1100](images/insert_java.png)

Adds one more call to every drawing method

---

# How does it compare to CWM?

- It's not a tool for collaborative development (for now)
- Allows displaying any IDE component
- Very comfortable to help/comment on somebody's issues

---

# Place for improvement

- Visual glitches
- Support for (more) AWT components
- Improve HiDPI support
- Speculative typing (?)
- iOS (Android?) native clients (Flutter?)
- Queue optimization
- **You** name it

---

# What did we learn?

- UI operations can be buffered
- There are not so many REAL UI operations
- Cross-platform drawing is achievable
- Parts of code may be reused between server and client

---

<!--
_class: lead
_footer: ""
-->

[projector.jetbrains.com](https://lp.jetbrains.com/projector/)

# Thank you

![](images/twitter_24.png) asm0di0
![](images/telegram.png) asm0dey

🎙️ [newpodcast2.live](https://newpodcast2.live)