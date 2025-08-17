# Why 90% of Projects Don’t Need More Than SQLite

Let’s be real. Not every project needs a heavyweight database system like PostgreSQL or MySQL. In fact, **90% of the time**, you’re better off keeping it simple.

And for me? I use **SQLite** in most of my personal projects. Here’s why.

---

## What is SQLite?

SQLite is a **serverless**, **self-contained**, and **lightweight** SQL database engine.

- No need to set up a database server.
- No user accounts or network configs.
- Just a single `.db` file.

You include it in your app, and it just works. That’s it.

---

## Why Most Projects Don’t Need More

Before you spin up Docker, install PostgreSQL, and configure backups, ask yourself:

- Is this project high traffic?
- Will thousands of users hit it at once?
- Do I need complex transactions across microservices?

If the answer is _no_, SQLite is probably all you need.

### Examples of perfect fits for SQLite:

- Personal blogs and portfolios
- Admin dashboards
- Internal company tools
- Desktop apps
- Mobile apps
- API prototypes
- IoT devices

In short—**anything that’s not a massive, multi-user system**.

---

## Why I Use SQLite in My Projects

Here’s the deal—I build a lot of tools for myself.

- Invoice generators
- Micro SaaS prototypes
- Internal dashboards
- One-off client apps

Using SQLite keeps things fast and simple.

- No need for a local database server
- Easy backups—just copy the file
- Migrations are quick
- Less configuration
- And most importantly… **zero friction** when shipping

When I’m working solo or on a small app, I care more about speed than complexity.

---

## But What About Scaling?

Let’s clear this up:

> **SQLite is not a toy.**

It can handle **reads at thousands per second** and supports **multiple processes**.

Sure, it's not ideal for **heavy concurrent writes**. But for 90% of projects, you're not going to run into that problem.

And when you do? It’s a **good problem**—because it means you’ve outgrown your MVP. At that point, switching to Postgres is easy with a bit of planning.

---

## When You Shouldn’t Use SQLite

Let’s be fair. SQLite is amazing, but not for everything.

Avoid it if:

- You need **concurrent writes** from many users
- You have **real-time dashboards** with massive traffic
- You're building a **multi-tenant SaaS** at scale
- You require **advanced features** only PostgreSQL offers

But if you’re building a **fast, clean MVP** or just solving a problem for yourself, **skip the complexity**.

---

## Final Thoughts

Don’t overengineer.

SQLite gives you a fully functional SQL database with **zero setup**, **high performance**, and **minimal overhead**.

If you’re building for yourself, a client, or your team—**chances are SQLite is more than enough**.

I use it in my personal projects all the time, and it hasn’t let me down.

Give it a try. You might never look back.

---

**🚀 Let’s build something amazing! If you have a project in mind or need help with your next design system, feel free to reach out.**  
📧 Email: [safi.abdulkader@gmail.com](mailto:safi.abdulkader@gmail.com) | 💻 LinkedIn: [@abdulkader-safi](https://www.linkedin.com/in/abdulkader-safi/) | 📱 Instagram: [@abdulkader.safi](https://www.instagram.com/abdulkader.safi/) | 🏢 [DSRPT](https://www.dsrpt.com.au/kw/contact)

_Drop me a line, I’m always happy to collaborate!_ 🚀
