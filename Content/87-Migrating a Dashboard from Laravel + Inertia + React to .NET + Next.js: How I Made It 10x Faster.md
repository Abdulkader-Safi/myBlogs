# Migrating a Dashboard from Laravel + Inertia + React to .NET + Next.js: How I Made It 10x Faster

When I first built an in-store offline analytics dashboard for a client, I reached for the stack I knew best: Laravel + Inertia + React. It worked well. The dashboard pulled data from sensors and activity logs inside the store and gave the client’s team visibility into what was happening day to day.

But as the data grew, so did the performance problems.

We were reading around 60,000 records per dashboard load. Even with caching, pages were slow—sometimes 10 seconds to load without cache and about 2 seconds with cache. For a tool meant to give quick insights, that delay was unacceptable.

That’s when I decided to do something bold: rebuild the entire project in .NET with Next.js.

The result?
The same dashboard now loads almost instantly. No caching tricks required.

Here’s the story.

---

## Why I Started with Laravel + Inertia + React

Like most developers, I go with the tools I’m comfortable with—especially for client projects where speed to delivery matters.

- Laravel gave me backend speed and easy database handling.
- Inertia.js bridged Laravel and React without needing a standalone API.
- React handled the UI in a modern way.

It was the right choice at the beginning. The client got their dashboard quickly, and everything worked.

But once the dataset hit tens of thousands of records, the cracks showed. Performance dropped, and no matter how much I optimized queries or caching, I couldn’t get the instant response times the client wanted.

---

## Why I Migrated to .NET + Next.js

I didn’t plan to switch stacks at first. My goal was just to optimize Laravel further. But I hit a wall.

That’s when I decided to rebuild with .NET for the backend and Next.js for the frontend.

- .NET is well-known for handling large datasets and heavy API operations efficiently.
- Next.js with SSR (Server-Side Rendering) gives blazing-fast page loads and comes with TypeScript support for better maintainability.

In short:
Laravel was great for prototyping. But for performance + maintainability, .NET + Next.js was the smarter long-term move.

---

### The Migration Process

This wasn’t a gradual switch. I went for a full rebuild.

- Database schema: I kept the same. No need to complicate the migration.
- Frontend: Rebuilt fully in Next.js with SSR.
- State management: I used URL parameters, which worked smoothly with the new stack.
- Challenges:
- It was my first real .NET project, so the learning curve was steep.
- Understanding how to best structure state and APIs took trial and error.

Interestingly, I didn’t rely on many extra libraries. The core power of .NET + Next.js handled most of the work.

---

### Performance Before vs After

This is where things get exciting.

Before (Laravel + Inertia + React):

- ~10 seconds load time without cache.
- ~2 seconds load time with cache.

After (with .NET + Next.js):

- Almost instant load, even for 60K records.
- Up to 10x faster than the old stack.
- Bonus: lower server resource usage = less hosting cost.

For a dashboard meant to be used daily by a team, this speed difference was game-changing.

---

## Developer Experience: Laravel vs .NET + Next.js

As a developer, here’s how the two stacks compared for me:

- Laravel + Inertia + React: Faster to start with. Easy if you know PHP. But scaling up performance? Tough.
- .NET + Next.js: Harder at the beginning, especially as it was my first .NET project. But once I got comfortable, the combination was much more maintainable.
- TypeScript in Next.js: Huge win. Type safety gave me confidence while building and maintaining the frontend.

So while Laravel was the “comfortable” choice, .NET + Next.js was the “powerful” choice.

---

## Client Reaction

The best part? The client’s reaction.

When I demoed the new dashboard, they were shocked.

- Pages loaded instantly.
- Their team could navigate without waiting on data.
- They were saving server costs.

For them, it wasn’t just a “technical upgrade.” It was a better experience every single day.

---

### Key Takeaways

If you’re considering what stack to use for a data-heavy dashboard, here’s my advice:

- Laravel + Inertia + React is excellent for quick builds and smaller datasets.
- But if you’re working with tens of thousands of records and performance is critical, .NET + Next.js might be worth the leap.
- Don’t underestimate TypeScript for maintainability.
- Sometimes, a full rebuild is easier than endlessly patching optimizations.

---

## Final Thoughts

For me, this migration wasn’t just about technology. It was about learning when to step out of my comfort zone for the client’s benefit.

Moving from Laravel + Inertia + React to .NET + Next.js turned a sluggish tool into a lightning-fast dashboard. The client was happy. I became more versatile as a developer. And now, I know which stack I’d pick next time performance is a top priority.

---

### 🤝 Need a Custom RSVP System or Dashboard?

I help businesses build tools that _actually work_ — even on tight deadlines.

Whether you're planning an event, need internal tools, or want a custom dashboard for your team — I can help.

### Reach out

📧 Email: [safi.abdulkader@gmail.com](mailto:safi.abdulkader@gmail.com) | 💻 LinkedIn: [@abdulkader-safi](https://www.linkedin.com/in/abdulkader-safi/) | 📱 Instagram: [@abdulkader.safi](https://www.instagram.com/abdulkader.safi/) | 🏢 [DSRPT](https://www.dsrpt.com.au/kw/contact)

_Drop me a line, I’m always happy to collaborate!_ 🚀
