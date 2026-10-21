
https://www.youtube.com/watch?v=79GIGeDA8_k&list=PLSUu-5i14fZ0

# 10,000 Users Hit One Server and It Dies: The Story of Three Boxes

*One app grows from a single server into a full system. At each stage a "box" is added in front of it, and the three boxes look identical on a diagram. Let's see why they are not.*

---

## The Opening Scene

**9 PM.** Your app hits the front page. **10,000 people land on one server. It dies.**

- Your senior adds **one box** in front of it → everything works again.
- A week later, a hacker attacks → **a box** stops him.
- A month later, you split the app into many small services → **a box** fixes that too.

**The strange part:** those were **three different boxes**, yet on a diagram they look the same: same spot, same arrows. Mixing them up is how interviews go wrong.

> **Analogy for the whole story:** Think of your app as a **busy restaurant** that grows into a **big shopping mall**. At each stage, a different person stands at the entrance, and each one answers a *different question*.

---

## Chapter 1: The server is exposed (Reverse Proxy)

### The problem
The address users connect to is the **address of your real server**. Anyone on the internet can find it, **including the attacker**.

### The restaurant version
Imagine customers walking straight into your **kitchen** because the kitchen door opens onto the street. Anyone can walk in, even someone with bad intentions.

### The fix
Stop letting people talk to the real server. Put **one machine in front of it** and give the world **that machine's address** instead. This front machine is the **reverse proxy**.

> **Analogy:** a **receptionist / front door**. Customers talk only to the receptionist. The receptionist walks the order to the kitchen and brings the food back. Customers **never even know the kitchen exists**.

### What happens now
- **Normal user:** talks to the proxy → proxy forwards to Server A → answer comes back the same way. User never learns Server A exists.
- **Attacker:** tries to reach Server A directly → **Blocked.**
  - Server A has **no public address**.
  - The network **only accepts traffic that comes through the proxy**.

### Important detail
- The proxy is the **controlled front door**.
- The **network rules** are what *actually* block direct access. (A proxy alone doesn't protect you if the server is still publicly reachable.)

### Bonus: the front door can do useful work
Since **every request passes through this one door**, the proxy can:
- Serve a **cached copy** instantly (like the receptionist handing out a ready-made menu instead of asking the kitchen every time)
- Handle **TLS** (HTTPS encryption)
- **Route** requests
- **Filter bad traffic**

All of this happens **before anything touches your real server**.

**Notes to remember**
- Reverse proxy = **protective front door** for your backend.
- Question it answers: **"Where should I forward this request?"**
- Users see the proxy only; the real server stays hidden.

---

## Chapter 2: The app goes viral (Load Balancer)

### The problem
A front door doesn't help when **one server cannot carry the load**.

### The restaurant version
Your restaurant is packed. One chef can't cook for everyone. So you hire **two more identical chefs**. Now a new question appears: **when an order arrives, which chef should take it?**

### The fix
You run **three identical copies** of your app. That decision ("which copy takes this request?") gets its **own machine: the load balancer**. Its whole job is to answer that one question, **thousands of times every second**.

> **Analogy:** the **restaurant host** who assigns each new customer to a waiter so that nobody is overloaded.

### Simplest strategy: Round Robin
- Request 1 → Server 1
- Request 2 → Server 2
- Request 3 → Server 3
- Request 4 → back to Server 1 ...

The load stays **even**, and **no single machine drowns**.

### Why it's essential: health checks
Server 3 suddenly becomes **unhealthy** (the chef faints).
- The load balancer **health-checks its servers all the time**.
- It notices **within seconds** and **stops sending traffic there**.
- The request meant for Server 3 is **rerouted to a healthy copy**.
- **Users never notice anything happened.**

### Reverse proxy vs load balancer
| Reverse proxy | Load balancer |
|---|---|
| **Hides one backend** | **Picks between many identical copies** |

**Notes to remember**
- Load balancer = **spreads traffic + skips unhealthy servers**.
- Question it answers: **"Which healthy instance should receive this?"**
- Works with **identical copies** of the same app.

---

## Chapter 3: The app splits into microservices (API Gateway)

### The problem
The app has grown into **microservices**:
- **Login** lives in one service
- **Orders** in another
- **Payments** in a third

Without help, this becomes a **mess fast**:
- The client must **know the address of every service**.
- Every service must build **its own authentication, rate limiting, and logging**: **the same code copied everywhere**.
- And this is only **six** services. Real systems have far more.

### The mall version
Your restaurant grew into a **shopping mall** with different shops (food court, bank, pharmacy). Without a main entrance desk:
- Visitors must know where every shop is.
- Every shop hires its **own security guard** to check IDs and count visitors. Wasteful and inconsistent.

### The fix
Put **one smart door** in front of all of them: the **API gateway**.

> **Analogy:** the **mall's main information + security desk**. It checks your ID **once**, counts how many times you've asked, writes it in the log, and then points you to the right shop.

### What makes it different
The gateway **reads each request and routes it by its path**:
- `/login` → **auth service**
- `/orders` → **order service**
- `/payment` → **payment service**

And all the shared work is done **once, at the door**:
- **Authentication**
- **Rate limiting**
- **Logging**

The duplication is **gone**.

### Load balancer vs API gateway
| Behind a **load balancer** | Behind a **gateway** |
|---|---|
| Servers are usually **identical copies** | Every service is **different** |

**Notes to remember**
- API gateway = **one door for microservices** + shared policies.
- Question it answers: **"Which service should get this, and which API policies apply?"**
- A load balancer **cannot** solve this problem.

---

## The Trick: Ask What Question It Answers

Don't ask *"what does each box do?"*
**Ask: "What question does it answer about an incoming request?"**

| Box | Question it answers | Chooses between | Restaurant/mall analogy |
|---|---|---|---|
| **Reverse Proxy** | *Where should I forward this request?* | **Which server** (controlled entry to the backend) | Receptionist at the front door |
| **Load Balancer** | *Which healthy instance should receive it?* | **Which instance** (identical copies) | Host assigning customers to identical waiters |
| **API Gateway** | *Which service should receive it, and which API policies apply?* | **Which service** (different services) | Mall info/security desk pointing to different shops |

> **Which server. Which instance. Which service.**
> Three different questions, answered at the **very same spot**. That's the whole trick.

---

## Why They Look Identical on Diagrams

Because they genuinely **overlap**:
- Many **load balancers also act as reverse proxies**: same forwarding idea **+ load spreading**.
- **API gateways** usually sit in the same reverse-proxy spot, **+ routing and authentication**.

> The **reverse proxy is the base idea**. The other two are that same idea **plus extra responsibilities**.

So they're **not three separate things you pick one of. They are jobs.** One tool can do **more than one job at the same time**.

---

## Think in Roles, Not Products

### Analogy: the smartphone
A smartphone is a **camera, a map, and a music player in one device**. Our tools work the same way.

### Real example: NGINX
- Most people know it as a **reverse proxy**.
- The same NGINX can also be your **load balancer**.
- And it can **partly** be your **API gateway**.
- One tool, three jobs; it depends on **how you configure it**.

### Others
| Tool | Jobs it commonly does |
|---|---|
| **NGINX** | Reverse proxy, load balancer, partly API gateway |
| **HAProxy** | Reverse proxy + load balancing |
| **Traefik** | Proxy + gateway |
| **Kong** | Proxy + gateway |

> **The tool is just the thing you install. The job is what it actually does for you.**
> Always ask **what job a box is doing, not what it's called.**

---

## Where Each One Lives in a Real System

> ⚠️ There is **no single correct diagram**. What follows is **one common setup, not a law.**

**A typical request journey:**
1. Request comes from the **internet**, often through a **CDN**.
2. Hits a **load balancer** first → spreads traffic so **no entry point gets crushed**.
3. Reaches the **API gateway** → the one checkpoint that:
   - **checks who you are** (authentication)
   - **limits how often you can ask** (rate limiting)
   - **sends you to the right service** (routing)
4. Deeper inside, you may find **more proxies and balancers**:
   - One service might sit behind **its own reverse proxy**.
   - Another might have **its own load balancer** in front of a few copies.
   - Some services have **none at all**.

**Takeaway:** there's **no fixed order to memorize**. Each box shows up **wherever its job is needed**.

---

## Interview Time: Questions That Actually Get Asked

### Q1. Why not just put authentication inside every microservice?
- That's **exactly the duplication we removed**.
- **Six services = six chances to get auth wrong.**
- The gateway does it **once, at the door**: **centralized, consistent, one place to fix**.
- ⚠️ **Don't forget:** the gateway does **not** remove security from the services. **Each service still checks its own permissions.**

### Q2. Can a load balancer just replace an API gateway?
- **Not always.**
- A load balancer picks between **identical copies**.
- A gateway routes between **different services** and handles **auth and rate limiting**.
- **Different questions.** A load balancer simply **cannot answer the gateway's question**.

### Q3. Can an API gateway replace a load balancer?
- **Sometimes, one product can do both jobs.**
- But the **responsibilities stay different**:
  - Gateway → *which service gets it, which policies apply*
  - Load balancer → *spread traffic across instances*
- **Different responsibility. Different question. Possibly the same product.**

> **Sounding senior vs being senior:** don't recite definitions. **Reason from what each box decides.**

---

## The Whole System, One Last Time

| Stage | What went wrong | Box added | Question it answers |
|---|---|---|---|
| 1 | Real server exposed to the world / attackers | **Reverse Proxy** | Where should I forward this request? |
| 2 | One server can't handle the traffic | **Load Balancer** | Which healthy instance should receive it? |
| 3 | Many different services, duplicated auth/rate-limit/logging | **API Gateway** | Which service should get it, and which policies apply? |

**The entire memory trick:** *Which server? Which instance? Which service?*

---

## 1-Minute Revision Sheet

- **Reverse proxy** = receptionist at the front door. **Hides** the backend. Can **cache, handle TLS, route, filter**. Real protection also needs **network rules** (backend has no public address).
- **Load balancer** = host assigning customers to identical waiters. **Spreads load** (e.g., **round robin**) and **health-checks** to skip bad servers.
- **API gateway** = mall info/security desk. **Routes by path** to **different services** and does **auth, rate limiting, logging once**.
- **Overlap is normal:** load balancers often act as reverse proxies; gateways sit in the reverse-proxy spot **plus** extras.
- **They are jobs, not products.** One tool (like **NGINX**) can do several jobs. Think of a **smartphone**.
- **No single correct diagram.** Each box appears **where its job is needed**.
- **Gateway doesn't replace service-level security.** Services still check their own permissions.
- **Interview tip:** answer by **the question each box answers**, not by memorized definitions.

---

*Notes based on the video on reverse proxy, load balancer, and API gateway.*

# Authentication, Told as Rahul's Story

*One user, one app that keeps growing, and every authentication concept showing up exactly when Rahul needs it.*

**The puzzle that starts it all:** Rahul types his password once. After that it is never sent again. So how does the server still know it's him on every single request?

The answer isn't "cookies" or "sessions" or "JWTs". Those are **pieces of one bigger system**. Let's watch the system get built.

---

## Chapter 1: Rahul opens a photo-sharing app

Rahul opens a photo-sharing app, types his **email and password**, and hits login.

The server checks the credentials. They're valid. Now the server **knows who Rahul is**.

> **Authentication = "Who are you?"**

Rahul's gallery loads and shows his pictures. Fine. But now imagine Rahul gets curious and **edits the URL** to peek at another user's private photos.

Should the app allow it? Of course not. Rahul is logged in, but that doesn't mean he can see *everyone's* data.

> **Authorization = "What are you allowed to do?"**

**Notes to remember**
- Authentication → establishes **identity**.
- Authorization → decides **access**.
- Being logged in ≠ being allowed to see everything.

**But here's the catch:** the server verified Rahul at login. His *next* request arrives. How does the server recognize him? HTTP has no memory of the previous request. We need some way to connect Rahul's separate requests together.

---

## Chapter 2: The coffee shop receipt (Sessions & Cookies)

Think of Rahul ordering a coffee. He pays, and the barista hands him a **receipt with an order number**. Behind the counter, the staff keep a list: *order #42 belongs to this order*. When #42 is called, Rahul shows his receipt and collects his coffee.

The web app does exactly the same thing for Rahul:

1. Rahul logs in successfully.
2. The server creates a **session**. Think of it as a new entry in the server's notebook: *"this session belongs to Rahul (user ID 101)"*.
3. The server generates a **random session ID**, like the order number.
4. It sends that ID to the browser inside a **cookie** (a small piece of data the browser stores), like the receipt.
5. On every next request, the browser **automatically attaches the cookie**.
6. The server looks up the session ID in its notebook → *"Ah, that's Rahul."*

> **The cookie carries the ticket. The session is the record the ticket points to.**

That is **session-based authentication**.

**Notes to remember**
- Cookie = the receipt (lives in the browser).
- Session = the staff's order list (lives on the server).
- Rahul's password isn't sent again; only the session ID is.

---

## Chapter 3: The app gets popular and breaks Rahul's login

Rahul's photo app becomes a hit. The team adds a **second server** to handle the traffic.

Rahul logs in through **Server 1**. His next request lands on **Server 2**. Server 2 opens *its* notebook... and there's no entry for Rahul. His session exists, but Server 2 has no idea about it. Rahul gets kicked back to the login page. Annoying!

It's like ordering coffee at one counter and trying to collect it from a different counter that never saw your order.

Two ways to fix it:

1. **Shared notebook:** put session data in a shared store that every server can read.
2. **Self-proving ticket:** give Rahul a token the server can verify **without any lookup**.

Option 2 brings us to JWTs.

---

## Chapter 4: Rahul gets a stamped ticket (JWT)

This time, after Rahul logs in:

1. The server checks his email and password.
2. It creates a **signed token** called a **JWT** (JSON Web Token) and sends it back.
3. The browser stores it.

Now Rahul opens a protected page. The browser attaches the token in a header:

```
Authorization: Bearer <the token>
```

The server then:
- **Verifies the signature.** If anyone changed even one character in the token, verification fails.
- **Checks the claims** (like *audience* and *expiration*).
- Only then does it trust the token and return the protected data.

Notice: **no notebook lookup**. Server 1, Server 2, Server 10... any of them can verify the token on its own. Problem from Chapter 3 solved.

### What's inside Rahul's token?
A JWT has 3 parts:
- **Header**: which algorithm was used to sign it.
- **Payload**: the claims (who, until when, etc.).
- **Signature**: proof that nobody tampered with it.

### Two things Rahul's team must not forget
1. **A signed JWT is NOT encrypted.** Anyone can decode the payload and read it. So **never put secrets inside**.
2. **Logout isn't instant.** With a session, the server just deletes the notebook entry and Rahul is out immediately. A JWT stays valid **until it expires**, unless you build a revocation mechanism.

> A JWT is **not automatically better** than a session. It's a different tool with different trade-offs.

**Notes to remember**
- JWT = self-contained, signed, verifiable by any server.
- Signed ≠ encrypted.
- Trade-off: easy to scale, harder to revoke.

---

## Chapter 5: What if someone steals Rahul's ticket? (Bearer tokens)

Imagine Rahul's token is a valuable ticket. **Whoever holds it can use it.** The system can't tell whether the person presenting it is the real Rahul. That's the idea of a **bearer token**.

So if an attacker steals Rahul's token, they can **pretend to be Rahul** until the token expires or gets rejected.

How do we limit the damage? That's the next chapter.

---

## Chapter 6: Two tokens with two different jobs (Access & Refresh tokens)

Rahul's team decides to use **two tokens**:

| | Access token | Refresh token |
|---|---|---|
| Job | Sent with **every request** | Gets a **new access token** when the old one expires |
| Lifetime | **Short** | Longer |
| Power | Only the permissions it needs | Provides continued access, so **guard it carefully** |

Here's how Rahul's app lives day to day:

1. Rahul's app sends an **authorization grant** to the **authorization server**.
2. The server signs him in and returns **two tokens**: access + refresh.
3. The app calls the **resource server** (where the data lives) with the **access token** → gets the photos.
4. After a while, the access token **expires**. The next call still carries the old token, so the resource server replies: *"invalid token"*.
5. The app quietly sends the **refresh token** to the authorization server and asks for a new access token.
6. The server returns a **fresh access token** and **rotates the refresh token** (the old one stops working).

Rahul notices nothing. The app just keeps working.

> **Key idea:** short-lived access tokens limit how long a *stolen* one is usable. Protecting and revoking refresh tokens controls the *long-term* access.

⚠️ This does **not** make token theft harmless. How much damage is possible depends on the design.

**Notes to remember**
- Access token = short-lived, used on every request.
- Refresh token = used only to get new access tokens; protect it; rotate it.
- Two servers involved: **authorization server** (issues tokens) and **resource server** (holds data).

---

## Chapter 7: Rahul gets a job and uses 10 apps (SSO)

Rahul now works at a company. He uses **Gmail** for email, **Drive** for files, **Docs** for documents. Without shared sign-in, he'd have to log in to **each one separately**. Inconvenient for him, and harder for the company to manage.

**Single Sign-On (SSO)** fixes this: sign in **once** through a trusted **identity provider** (Google here), then reach every connected app. Think of the identity provider as a **central sign-in desk** that all apps trust.

**Morning, first app (Gmail):**
1. Rahul opens Gmail. No session yet.
2. Gmail **redirects** him to Google (the identity provider).
3. Google shows a login page. Rahul enters his credentials.
4. Google **creates a session** and **issues a token**.
5. The browser forwards the token to Gmail.
6. Gmail validates it and shows his inbox.

**A little later, second app (Drive):**
1. Rahul opens Drive. Drive also redirects him to Google.
2. This time Google **already has a session**, so it **skips the login page completely**.
3. It issues a **fresh token straight away**.
4. Drive validates it and lets him in.

One sign-in, every Google app. That's SSO.

> **SSO is a sign-in *experience*, not a token format.** Protocols like **OpenID Connect** and **SAML** can power it.

**Notes to remember**
- Identity provider = the one trusted sign-in desk.
- Apps don't handle passwords themselves; they trust the identity provider.

---

## Chapter 8: Rahul wants to print his photos (OAuth 2.0)

Rahul finds **PhotoPrint**, an app that prints photos. But his photos live in **Google Photos**. PhotoPrint needs permission to read them.

The lazy answer: give PhotoPrint his Google password. But that hands over his **whole account**, way more than needed.

Think of a **valet key**. It lets the valet **park your car**, but not open the trunk or everything else you own. **OAuth** applies the same idea: Rahul authorizes an app to access **specific things** without sharing his password. What it may touch is limited by **scope**.

**The full story, step by step:**
1. In PhotoPrint, Rahul clicks **Connect**.
2. PhotoPrint sends an **authorization request** to Google.
3. Google redirects Rahul to **its own login page**.
4. Rahul signs in with his Google password. **PhotoPrint never sees it.**
5. Google asks for **consent** ("PhotoPrint wants to read your photos"). Rahul approves.
6. Google gives PhotoPrint an **authorization code**.
7. PhotoPrint sends that code to Google's **token endpoint** and **exchanges it for an access token**.
8. PhotoPrint calls the **Google Photos API** with that access token.
9. Google Photos checks the token with Google, **confirms the scope**, and returns the photos.

PhotoPrint gets exactly what it needs, **the photos and nothing else**.

> OAuth is an **authorization framework for delegated access**.

**Remember our very first question?** Authentication asks *who you are*. OAuth answers *what access has been granted*.

> ⚠️ **An access token does NOT prove identity.**

**Notes to remember**
- OAuth = valet key (limited, delegated access).
- Password stays with Google; PhotoPrint only gets a token.
- Scope = the boundary of what the app may touch.

---

## Chapter 9: "Sign in with Google" (OpenID Connect)

Now Rahul visits a brand-new website and clicks **"Sign in with Google"**. This time the site isn't asking for access to his photos. It wants to know **who Rahul is**.

Since OAuth can't prove identity, we use **OpenID Connect (OIDC)**: an **identity layer on top of OAuth**.

What happens? **The exact same OAuth flow from Chapter 8:**
- The site redirects Rahul to Google.
- He authenticates.
- The app exchanges the code at the token step.

But here's the **one new thing**: at that token step, the app **also receives a signed ID token**. It's usually a **JWT** with claims about the user. The app validates it and **learns who Rahul is**.

| | ID token | Access token |
|---|---|---|
| Job | Proves **who the user is** | Grants **access to resources** |

> **Do not confuse the two.**
>
> **OIDC = OAuth flow + one identity token.**

### "Wait, why didn't PhotoPrint get an ID token?"
Because PhotoPrint **never needed to know who Rahul was**. It only wanted his photos. If it *also* wanted his identity, it would simply **ask for an ID token too**. The flow stays the same; the app just requests one more thing.

**Notes to remember**
- OAuth → access. OIDC → identity.
- ID token = proof of who you are. Access token = permission to use something.

---

## Epilogue: Looking back at Rahul's journey

| Chapter | Rahul's situation | Concept that solved it |
|---|---|---|
| 1 | Logs into the photo app; tries another user's photos | **Authentication** vs **Authorization** |
| 2 | Server must recognize him on the next request | **Sessions + Cookies** (coffee receipt) |
| 3 | A second server doesn't know his session | Shared store or **a self-verifying token** |
| 4 | Any server must verify him without lookup | **JWT** (signed, not encrypted) |
| 5 | Stolen token = stolen identity | **Bearer token** risk |
| 6 | Limit the damage of stolen tokens | **Access + Refresh tokens** |
| 7 | Ten apps, ten logins | **SSO** via an identity provider |
| 8 | PhotoPrint needs his photos, not his password | **OAuth 2.0** (valet key) |
| 9 | "Sign in with Google" must prove identity | **OpenID Connect** (ID token) |

**The big picture:**
- **Authentication** verifies who the user is.
- **Authorization** decides what they can access.
- **Sessions** keep login state, usually via a session ID in a cookie.
- **JWTs** are a standard format for signed claims a server can verify.
- **Access tokens** reach protected resources; **refresh tokens** renew them.
- **SSO** is one shared sign-in.
- **OAuth** is delegated access.
- **OpenID Connect** adds identity on top.

> These are **not seven competing choices**. They're **pieces that work together**. Once you know the role each piece plays, you can **design authentication with intention** instead of just picking popular tools.

---

## 30-Second Revision Sheet

- **Server forgets you between requests** → give the browser something to carry (cookie with session ID, or a token).
- **Cookie** = receipt. **Session** = the record on the server.
- **More servers?** Share sessions, or use **JWT**.
- **JWT** = signed, readable by anyone, so **no secrets inside**. Hard to revoke before expiry.
- **Bearer token** = whoever holds it, wins. Keep access tokens **short-lived**.
- **Refresh token** = renews access tokens. Guard it and **rotate** it.
- **SSO** = log in once at the identity provider, enter many apps.
- **OAuth** = valet key. Gives **access**, not identity.
- **OIDC** = OAuth + **ID token** → gives **identity**.

---

*Notes based on the "Visual Coders" video on authentication concepts.*
