---
marp: true
theme: gaia
size: 4K
class: default
paginate: true
footer: ![](images/twitter_24.png) asm0di0
backgroundImage: "linear-gradient(to bottom, #000 0%, #1a2028 50%, #293845 100%)"
color: white
title: "Projector: Your Remote IDE"
---
<!--
_class: lead
_paginate: false
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
...huge projector logo...

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

## Usual Swing interface
![bg height:75%](images/swing_interface.png)

---

