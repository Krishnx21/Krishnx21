<div align="center">

<img src="https://capsule-render.vercel.app/api?type=waving&color=0:0F172A,100:1E293B&height=220&section=header&text=Krishna%20Kumar&fontSize=44&fontColor=E2E8F0&fontAlignY=35&desc=Backend%20Engineer%20%C2%B7%20Infrastructure%20%C2%B7%20Open%20Source&descAlignY=52&descSize=16&descColor=94A3B8" width="100%"/>

<img src="https://readme-typing-svg.demolab.com?font=JetBrains+Mono&weight=500&size=16&pause=1400&color=94A3B8&center=true&vCenter=true&width=600&lines=systems+that+fail+gracefully%2C+not+silently.;queues+over+blocking+calls.;reads+the+stack+trace+before+the+docs." alt="typing" />

<br/>

<a href="https://www.linkedin.com/in/krishna-kumar-89544b295"><img src="https://img.shields.io/badge/LinkedIn-0A66C2?style=for-the-badge&logo=linkedin&logoColor=white"/></a>
<a href="mailto:krishnakumarsharma8077@gmail.com"><img src="https://img.shields.io/badge/Email-D14836?style=for-the-badge&logo=gmail&logoColor=white"/></a>
<a href="https://github.com/krishnx21"><img src="https://img.shields.io/badge/GitHub-181717?style=for-the-badge&logo=github&logoColor=white"/></a>

<br/><br/>

<img src="https://komarev.com/ghpvc/?username=krishnx21&label=Profile%20Views&color=1E293B&style=flat-square" alt="profile views" />

</div>

<br/>

```bash
krishna@backend:~$ whoami
```

```text
> Third-year CS student. Backend-leaning, infra-curious, AI-assisted where it counts.
> Currently building : CertiVault — document verification platform
> Currently learning  : Kubernetes, Terraform, distributed systems
> Looking for         : Backend / Platform / Infrastructure internship
> Long-term target    : Software Engineer
```

<br/>

---

<br/>

## About Me

I'm a third-year Computer Science student who spends more time in the backend than the frontend — the auth layer, the queue, the cache, the part of a system that either holds up under load or quietly doesn't.

Frontend bugs are visible immediately: something looks wrong, someone notices. Backend bugs are patient. A race condition, a queue that silently drops a job, a token that never expires — these can sit for weeks before they cost someone data or trust. That asymmetry is what pulled me toward backend and infrastructure work in the first place, and it's the lens I build every project through: **what happens when this fails, and does the system know it failed?**

I care about systems that are:

| | |
|---|---|
| 🛡️ **Reliable** | keep working when a dependency doesn't |
| 🔐 **Secure** | access is scoped, not just authenticated |
| 👁️ **Observable** | failure is visible before a user reports it |
| 🧩 **Maintainable** | the next person (often me, in six months) can reason about it |
| 📈 **Scalable** | the design doesn't assume load stays where it is today |

<br/>

---

<br/>

## Engineering Philosophy

> **Reliability is a design decision, not a bugfix.**
> A system that works in the demo and falls over under concurrent writes isn't reliable, it's untested. I design for the failure case first — what happens if the worker dies mid-job, what happens if the same request arrives twice — and treat the happy path as the easy part.

> **Queues over blocking calls, whenever the work can wait.**
> If a request doesn't need to be answered synchronously, making the caller wait for it is a cost with no benefit. Offloading it to a queue keeps the API responsive under load, so a slow downstream dependency doesn't become the whole system's problem.

> **Caching is a trade-off, not a default.**
> Every cache is a promise to keep two copies of the truth in sync. I reach for it when the read cost is real and the staleness window is acceptable — not as a reflex for "making things faster."

> **Software should fail loud, not silent.**
> A try/catch that swallows an error is a bug wearing a disguise. If something fails, I want a log line, a retry policy, or a dead-letter queue — not a silent no-op that looks like success from the outside.

> **Observability is what turns "it's slow" into an actionable ticket.**
> Without logs, metrics, or traces, debugging production is guesswork with extra steps. I'd rather spend time instrumenting a system than debugging it blind.

<br/>

---

<br/>

## 🚀 Flagship Project — CertiVault

**Smart document management and verification platform** — authentication, role-based access, hash-based integrity checks on every uploaded file, expiring share links, and background processing for anything that doesn't need to block the request.

**Problem.** Document-sharing tools tend to get one of two things wrong: they either trust the client too much (anyone with a link has permanent access) or they do too much synchronous work on upload (hashing, notification, thumbnailing), so the API becomes only as fast as its slowest side-effect.

**Solution.** Separate the request path from the work path. The API's job is to accept the upload, persist it, and enqueue everything else. A pool of background workers does the actual verification and notification work, so a slow hash computation or a flaky email provider never makes a user wait on their upload request.

**Engineering decisions:**

- **Why BullMQ** — a job queue with retries, backoff, and visibility into failed jobs, backed by something already in the stack (Redis), rather than standing up a separate broker for a project this size.
- **Why Redis** — doing two jobs here: backing the BullMQ queue, and caching share-link validation so a hot link doesn't hit MongoDB on every access.
- **Why background workers** — verification (hashing, integrity checks) and notification (email) are both variable-latency and both tolerant of a few seconds of delay. Neither belongs on the request path.
- **Why hashing** — every uploaded file gets a SHA-256 checksum computed and stored at upload time. Verification later means recomputing the hash and comparing, not trusting a stored "verified" flag that could be stale or tampered with.
- **Maintaining reliability** — jobs are idempotent where possible, failed jobs go to a dead-letter queue instead of disappearing, and share links carry their own expiry rather than relying on a cleanup cron to catch them in time.

**Lessons learned.** The hardest part wasn't any single feature — it was the coordination problems that only show up outside the happy path: two requests hitting the same document at once, a share link that outlives its intended use, a worker that dies mid-job and needs its work picked back up rather than lost.

`Node.js` `Express` `MongoDB` `Redis` `BullMQ` `Docker` `JWT` `Google OAuth` `Cloudinary`

<br/>

---

<br/>

## Other Projects

<table>
<tr>
<td width="50%" valign="top">

### 📤 Cloud File Sharing

**Problem.** Sharing a file via a link is easy; sharing it *safely* is the hard part — a link that works for anyone forever isn't sharing, it's a leak with a delay.

**Solution.** JWT-scoped share links with defined access windows, backed by Cloudinary for storage so the app isn't managing raw file I/O itself.

**Challenge.** Getting the scope right: a token needs to prove *what* it grants access to and *until when*, without requiring a database round-trip on every access check.

**Lesson.** Authorization is much easier to get wrong quietly than authentication is — a missing expiry check doesn't throw an error, it just works forever.

`Node.js` `Express` `JWT` `Cloudinary`

</td>
<td width="50%" valign="top">

### 🧠 AI Resume Analyzer

**Problem.** Resume feedback tools are either too generic (keyword matching) or require a human reviewer that doesn't scale.

**Solution.** Score a resume against a specific job description using the Claude API, returning structured feedback the UI can render as sections, not a wall of text.

**Challenge.** LLM output isn't naturally structured — getting consistent, parseable JSON back on every call (not just most) meant being explicit about output format and handling the cases where it wasn't.

**Lesson.** Prompting an LLM for a UI-facing feature is an API contract problem as much as a prompting problem — the schema has to hold even when the input resume is messy.

`React` `Express` `MySQL` `Claude API`

</td>
</tr>
<tr>
<td width="50%" valign="top">

### 🌤️ Weather Dashboard

**Problem / Solution.** Live forecasts with location search — the most straightforward project on the list.

**Challenge.** Honestly, none technically. This was the first project where I cared about interface quality as much as function, which was its own useful exercise.

**Lesson.** Not every project needs a hard backend problem to be worth building — this one taught me frontend polish instead.

`JavaScript` `CSS` `Weather API`

</td>
<td width="50%" valign="top">

### 🔭 Coming Next

Next up in the roadmap after CertiVault ships: a self-hosted **secret scanning and rotation intelligence** platform — deliberately built on a different stack, with the scan engine (regex + entropy detection) as the core engineering problem.

More on this soon.

</td>
</tr>
</table>

<br/>

---

<br/>

## Open Source Leadership

| Program | Role | What I actually do |
|---|---|---|
| **ECSoC 2026** | Project Admin | Own the project end to end — set scope, define the roadmap, and review every pull request before it merges |
| **SSOC 2026** | Mentor | Get first-time contributors to a merged PR — mostly explaining *why* a change should look a certain way, not just approving or rejecting it |
| **GSSoC 2026** | Contributor | Active across issues and PRs on other maintainers' projects |

Running a project rather than just contributing to one changes what you're responsible for. Reviewing a PR isn't just "does this work" — it's judging whether a contributor's approach fits the project's direction, and being able to explain that clearly enough that a first-time contributor learns something instead of just getting a rejection.

<br/>

---

<br/>

## Tech Stack

<div align="center">

**Languages**
<br/>
<img src="https://img.shields.io/badge/JavaScript-F7DF1E?style=flat-square&logo=javascript&logoColor=black"/>
<img src="https://img.shields.io/badge/TypeScript-3178C6?style=flat-square&logo=typescript&logoColor=white"/>
<img src="https://img.shields.io/badge/Python-3776AB?style=flat-square&logo=python&logoColor=white"/>
<img src="https://img.shields.io/badge/Java-007396?style=flat-square&logo=openjdk&logoColor=white"/>
<img src="https://img.shields.io/badge/C-A8B9CC?style=flat-square&logo=c&logoColor=black"/>

<br/><br/>

**Backend**
<br/>
<img src="https://img.shields.io/badge/Node.js-339933?style=flat-square&logo=node.js&logoColor=white"/>
<img src="https://img.shields.io/badge/Express-000000?style=flat-square&logo=express&logoColor=white"/>
<img src="https://img.shields.io/badge/REST_APIs-005571?style=flat-square&logo=fastapi&logoColor=white"/>
<img src="https://img.shields.io/badge/JWT-000000?style=flat-square&logo=jsonwebtokens&logoColor=white"/>
<img src="https://img.shields.io/badge/OAuth_2.0-000000?style=flat-square&logo=auth0&logoColor=white"/>
<img src="https://img.shields.io/badge/Redis-DC382D?style=flat-square&logo=redis&logoColor=white"/>
<img src="https://img.shields.io/badge/BullMQ-DC382D?style=flat-square&logo=redis&logoColor=white"/>

<br/><br/>

**Data**
<br/>
<img src="https://img.shields.io/badge/MongoDB-47A248?style=flat-square&logo=mongodb&logoColor=white"/>
<img src="https://img.shields.io/badge/Mongoose-880000?style=flat-square&logo=mongoose&logoColor=white"/>
<img src="https://img.shields.io/badge/MySQL-4479A1?style=flat-square&logo=mysql&logoColor=white"/>

<br/><br/>

**Infrastructure & Delivery**
<br/>
<img src="https://img.shields.io/badge/Docker-2496ED?style=flat-square&logo=docker&logoColor=white"/>
<img src="https://img.shields.io/badge/GitHub_Actions-2088FF?style=flat-square&logo=githubactions&logoColor=white"/>
<img src="https://img.shields.io/badge/AWS-232F3E?style=flat-square&logo=amazonaws&logoColor=white"/>
<img src="https://img.shields.io/badge/Render-46E3B7?style=flat-square&logo=render&logoColor=white"/>
<img src="https://img.shields.io/badge/Vercel-000000?style=flat-square&logo=vercel&logoColor=white"/>
<img src="https://img.shields.io/badge/Cloudinary-3448C5?style=flat-square&logo=cloudinary&logoColor=white"/>

<br/><br/>

**Testing & Tooling**
<br/>
<img src="https://img.shields.io/badge/Jest-C21325?style=flat-square&logo=jest&logoColor=white"/>
<img src="https://img.shields.io/badge/Supertest-1F1F1F?style=flat-square"/>
<img src="https://img.shields.io/badge/Git-F05032?style=flat-square&logo=git&logoColor=white"/>
<img src="https://img.shields.io/badge/Postman-FF6C37?style=flat-square&logo=postman&logoColor=white"/>
<img src="https://img.shields.io/badge/ESLint-4B32C3?style=flat-square&logo=eslint&logoColor=white"/>
<img src="https://img.shields.io/badge/Prettier-F7B93E?style=flat-square&logo=prettier&logoColor=black"/>

</div>

<sub>Self-assessed, updated as I go — not a certification tracker.</sub>

<br/>

---

<br/>

## GitHub Metrics

<div align="center">

<img height="165" src="https://github-readme-stats.vercel.app/api?username=krishnx21&show_icons=true&theme=tokyonight&hide_border=true&count_private=true" />
<img height="165" src="https://github-readme-stats.vercel.app/api/top-langs/?username=krishnx21&layout=compact&theme=tokyonight&hide_border=true" />

<br/>

<img height="180" src="https://github-readme-streak-stats-eight.vercel.app/?user=krishnx21&theme=tokyonight&hide_border=true" />

</div>

<br/>

---

<br/>

## Contact

<div align="center">

Open to backend and infrastructure internships.

<br/>

<a href="mailto:krishnakumarsharma8077@gmail.com"><img src="https://img.shields.io/badge/Email-D14836?style=for-the-badge&logo=gmail&logoColor=white"/></a>
<a href="https://www.linkedin.com/in/krishna-kumar-89544b295"><img src="https://img.shields.io/badge/LinkedIn-0A66C2?style=for-the-badge&logo=linkedin&logoColor=white"/></a>
<a href="https://github.com/krishnx21"><img src="https://img.shields.io/badge/GitHub-181717?style=for-the-badge&logo=github&logoColor=white"/></a>

<br/><br/>

<img src="https://capsule-render.vercel.app/api?type=waving&color=0:0F172A,100:1E293B&height=100&section=footer" width="100%"/>

</div>
