
https://www.youtube.com/watch?v=79GIGeDA8_k&list=PLSUu-5i14fZ0
https://www.youtube.com/watch?v=7iHl71nt49o&list=PLdNCznBZ77NriBGbbHTdM34cvx7dLKi-_


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

# Caching Explained: The Story of the Librarian and Her Desk

*A complete guide for system design interviews, told as one story. A librarian, a basement archive, and a desk that makes everything faster.*

---

## The Opening Scene: The $5 Billion Speed Problem

- A **tenth of a second** once cost Amazon **billions of dollars**.
- Every **100 ms** a page got slower, they lost about **1% of sales**.
- So Google, Netflix, Amazon and others went to war over speed, and almost all of them won with **the same trick**.
- Not a faster server. Not a bigger database.
- The trick: **remembering what you've already looked up.** That's **caching**.

> **Core idea in one line:** Caching is not about making computers faster. It's about **avoiding unnecessary work**.

---

## Chapter 1: The Library with Endless Trips

### The story
You walk into one of the biggest libraries in the world: thousands of shelves, millions of books.

1. You ask the librarian: **"Who invented Java?"**
2. She goes down to a **massive basement archive**, searches hundreds of books, finds the answer, walks back, and says: **"James Gosling."** Slow, but correct.
3. A minute later, **another person asks the exact same question.**
4. She walks **all the way down again**, searches the **same shelves**, finds the **same book**, walks back with the **same answer**.
5. Another visitor, same question, same trip. Again and again.

**Is this efficient? No.** The answer hasn't changed, yet she repeats the same work every time.

### The fix
You say: *"Why don't you just keep that book on your desk?"*

- She walks down **once**, brings the book back, and **keeps a copy on her desk**.
- Next visitor asks the same question → she **opens the book on her desk** and answers **immediately**.

**Same answer, much less work.**

---

## Chapter 2: What Is a Cache?

That **desk** is the **cache**.

| Library story | Real application |
|---|---|
| **Basement archive** | **Database** (where all original data lives) |
| **Librarian** | **Application server** |
| **Desk** | **Cache** (smaller, faster, keeps frequently used data close) |

### Key Points
- The cache **does not replace the database**.
- The **database is still the source of truth**.
- The cache is just a **shortcut**.
- The app first checks: *"Do I already have a copy nearby?"*
  - **Yes** → saves time.
  - **No** → goes to the database.

---

## Chapter 3: A Customer Opens a Product Page (Hit vs Miss)

### Request #1: Cache Miss
1. A user opens a **product page**. The request reaches your backend.
2. The app asks: **"Do I already have this in my cache?"**
3. The cache is **empty**, so the app queries the **database**.
4. The database finds the product and returns it.
5. The app **stores a copy in the cache**, then sends the response.

→ This is a **cache miss**. (Librarian walks to the basement, then puts a copy on her desk.)

### Request #2: Cache Hit
1. Another customer opens the **same product page** seconds later.
2. The app asks again: *"Do I already have this?"*
3. **Yes**, it's in the cache. **No database query.**
4. The app returns the cached copy **instantly**.

→ This is a **cache hit**. (Librarian opens the book on her desk.)

### Side-by-side

| | Cache Miss | Cache Hit |
|---|---|---|
| Journey | Request → App → Cache (empty) → **Database** → store copy → response | Request → App → Cache (found) → response |
| Speed | Slow | **Instant** |
| DB involved? | Yes | **No** |

---

## Chapter 4: Is the Cache Actually Helping? (Hit Rate)

Out of **100 requests**, **95 are served from cache** and only **5 visit the database**. That's fantastic.

> **Cache hit rate** = % of requests served directly from the cache.
> **95 hits out of 100 = 95% hit rate.**

### Why it matters
- As your app grows from 1 user to **millions**, a good cache means most requests **never touch the database**.
- Instead of **1,000,000 queries**, the DB might handle only **~50,000**.

---

## Chapter 5: The Desk Is Small (Cache Eviction)

### Why not cache everything?
- **Memory is expensive.** A database may store **terabytes**; keeping that all in memory isn't practical.
- A cache is meant to be **small, fast, and selective**.

### The story
- The librarian's desk is **much smaller than the basement**; she can hold only **a handful of books**.
- A visitor asks a **different question**. She fetches another book, but **the desk is full**.
- New problem: not *"where do I find the answer?"* but **"which book do I throw away?"**

This is **cache eviction**: when the cache is full, it **must remove something** before storing new data.

### How does it decide? Three common strategies

| Strategy | Meaning | Desk analogy |
|---|---|---|
| **LRU** (Least Recently Used) | Removes the item **not accessed for the longest time** | Remove the book **nobody has touched in 2 hours** (the most common choice) |
| **LFU** (Least Frequently Used) | Removes the item **accessed the fewest times** | Remove the book **opened only once** |
| **FIFO** (First In, First Out) | Removes the **oldest** item | Remove the book **that was placed on the desk first** |

> **No universal winner.** They all solve the same problem: *a cache is small, so something eventually has to go.*

---

## Chapter 6: The Price Changed, but the Desk Didn't (Stale Data)

Our cache looks almost perfect: faster requests, less DB load, smart about what it keeps. But there's one big problem.

**What happens when the data changes?**

### The story
1. A customer opens the product page → cache hit → instant. 
2. An **admin updates the price from $50 → $40**. The **database is now correct**.
3. But the **cache still has the old $50**. Nobody told it.
4. The next customer opens the page → cache hit → sees **$50** instead of **$40**.

> **The app is fast but wrong.**
> Users can forgive a **slow** app. They won't forgive **incorrect** data.

This is the **stale data** problem.

(Library version: the book on the desk is an **old edition**, but the librarian keeps reading from it.)

---

## Chapter 7: Fix #1: Put a Timer on It (TTL)

### Idea
Don't keep cached data **forever**. Every item gets a **timer** (say, **5 minutes**). The moment it's cached, the countdown starts. When it hits zero, the cache **removes it**, whether anyone asked or not.

> **TTL = Time To Live.**

### The catch
- Suppose **TTL = 10 minutes** and the price changes just **1 minute** after caching.
- For the next **9 minutes**, every customer still sees the **old price**.

### When is that okay?
| Fine | Not fine |
|---|---|
| **News website** (slightly old content is harmless) | **Banking app** |
| **Blog** | **Ticket booking** (one seat left: two people think they own it) |
| | **Inventory in a flash sale**, **airline tickets**, **stock prices** |

Sometimes **waiting for the timer isn't an option.**

---

## Chapter 8: Fix #2: Tell the Cache Directly (Cache Invalidation)

### Idea
Instead of waiting for the timer, the **moment data changes**, the app tells the cache: *"This data has changed. Remove the old copy."*

### Walkthrough
1. Admin updates the price.
2. The app writes the **new value to the database**.
3. Immediately after, it sends one more instruction: **"Delete the cached product."**
4. The cache **removes the old copy instantly**.
5. Next customer arrives → cache is empty → **cache miss**.
6. App fetches the **fresh $40** from the DB and **stores a new copy**.

### Notice
- The cache was **not updated directly**. It was **cleared**, and the **next request rebuilt it** automatically.
- This pattern is called **Cache-Aside**, one of the **most common patterns** in real apps.

### TTL vs Invalidation

| | TTL | Invalidation |
|---|---|---|
| Simplicity | **Simpler** | More work |
| Freshness | Can serve old data until timer ends | **More accurate** (cleared right when data changes) |
| Best for | Blogs, news, rarely-changing data | Tickets, bank balances, stock prices, flash-sale inventory |

> The right choice depends on **how fresh your data needs to be**.

---

## Chapter 9: Real Apps Have Many Desks (Where Caches Live)

Real applications don't have just **one** cache. Follow a single request when you open **Amazon's homepage**:

1. **Browser cache** (on your own computer)
   - Checks for a valid copy of images, logos, style sheets.
   - If found → **request ends right there**, nothing travels across the internet.
2. **CDN** (Content Delivery Network)
   - Servers **all over the world**; your request goes to the **closest one**.
   - If the nearby server has a copy → response sent, **main app never contacted**.
3. **Redis** (application-level cache)
   - An **in-memory data store** built for speed. Keeps data in **memory instead of disk**, so it's **incredibly fast**.
4. **Database**
   - Only if **Redis also misses** does the app finally query the DB.

### Why this matters
- **1st request** travels all the way to the **database**, leaving copies behind (browser, CDN, Redis).
- **2nd request** might stop at **Redis**.
- **3rd** might stop at the **CDN**.
- **4th** might **never leave the browser**.

> Every layer moves the data **one step closer to the user**. That's what makes apps feel **instant**, even with millions of users.

```
User → Browser Cache → CDN → Redis → Database
        (closest)                    (slowest, source of truth)
```

---

## Chapter 10: The Interview Question

> **Interviewer:** *"A product page is cached in Redis. An admin updates the price, but customers still see the old price for a few seconds. Why?"*

### Reasoning step by step
1. The **database was updated**, so the DB is correct.
2. **Redis still holds the old copy.**
3. The app **reads from the cache** instead of the database → customers see the old price.

> **Interviewer:** *"How would you fix it?"*

### There isn't one answer. Explain the trade-off:
- **Blog / news site (data rarely changes)** → a **short TTL** is perfectly fine.
- **Airline tickets, bank balances, stock prices, flash-sale inventory** → stale data causes **serious problems** → **invalidate the cache the moment the DB updates**.

> **What interviewers listen for:** not memorized definitions, but the **reasoning behind the design**.
> **TTL is simpler. Invalidation is more accurate.** Your job is to pick what fits.

---

## Chapter 11: The Answer to the Opening Question

**"Why was the first request slow, but the second almost instant?"**

- The **first request did all the work**: it travelled through **every layer** until it reached the **database**, and **left copies behind** (browser, CDN, Redis).
- The **second request** stops at the **first place that already has the answer**: sometimes the browser, sometimes the CDN, sometimes Redis.
- Only if **every cache misses** does the database get involved again.

---

## The One Question Behind Everything

> **"Do I already have the answer?"**
> - **Yes** → use it.
> - **No** → fetch it, **store a copy**, and make the next request faster.

Everything else (**hits, misses, hit rate, LRU, LFU, FIFO, TTL, invalidation, browser cache, CDN, Redis**) is just a **technique built around that one question**.

---

## Quick Glossary (Story Edition)

| Term | In the library story | In real life |
|---|---|---|
| **Database** | Basement archive | Source of truth, slow, huge |
| **Cache** | Desk | Small, fast, selective memory |
| **Cache miss** | Not on desk → walk to basement | Not in cache → query DB |
| **Cache hit** | Book already on desk | Found in cache → instant |
| **Hit rate** | % of questions answered from the desk | e.g., 95 / 100 = 95% |
| **Eviction** | Desk is full → remove a book | Cache full → remove an item |
| **LRU** | Remove book untouched the longest | Least Recently Used |
| **LFU** | Remove book opened the fewest times | Least Frequently Used |
| **FIFO** | Remove the book placed first | First In, First Out |
| **Stale data** | Desk has an old edition | Cache has outdated value |
| **TTL** | Sticky note: "throw away after 5 min" | Time To Live |
| **Invalidation** | New edition arrives → throw the old book out immediately | Delete cache entry when data changes |
| **Cache-Aside** | Check desk → if missing, fetch from basement and place on desk | App checks cache first, loads from DB on miss, stores a copy |
| **Browser cache** | Notes in your own pocket | Local copy on user's device |
| **CDN** | Branch library near your home | Nearby server with a copy |
| **Redis** | Fast desk in the main library | In-memory data store |

---

## 1-Minute Revision Sheet

- **Caching = remembering what you've already looked up** → avoids repeated work.
- **Cache ≠ database.** DB is the **source of truth**; cache is a **shortcut**.
- **Miss** = not found → query DB → **store copy**. **Hit** = found → instant.
- **Hit rate** = hits ÷ total requests (95/100 = 95%). Higher = less DB load.
- **Memory is expensive** → cache is small → needs **eviction** (**LRU**, **LFU**, **FIFO**).
- **Stale data**: DB updated but cache still old. *Fast but wrong is worse than slow.*
- **TTL**: auto-expire after a time. **Simple**, but data may be stale until it expires.
- **Invalidation**: delete cache entry when data changes. **Accurate**, more work.
- **Cache-Aside**: update DB → delete cache → next request misses and **rebuilds** the cache.
- **Layers**: **Browser → CDN → Redis → Database**. Each layer brings data closer to the user.
- **Interview tip:** explain the **trade-off** (TTL vs invalidation) based on **how fresh the data must be**.

---

*Notes based on the video "Caching Explained: The Complete Guide for System Design Interviews".*

---

*Notes based on the "Visual Coders" video on authentication concepts.*
