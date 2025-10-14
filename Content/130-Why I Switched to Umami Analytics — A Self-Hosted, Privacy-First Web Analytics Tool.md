# Why I Switched to Umami Analytics — A Self-Hosted, Privacy-First Web Analytics Tool

Keywords you can optimize for: Umami Analytics, self-hosted analytics, privacy-friendly analytics, Umami dashboard, Google Analytics alternative, open source web analytics

---

Introduction

If you run a website or blog and care about data privacy, lightweight performance, and full ownership of your analytics, you’ve probably heard of alternatives to Google Analytics. One standout in this field is Umami Analytics (<https://umami.is/>). Over the past two days, I’ve been using Umami on my own site (self-hosted), and I want to share my full review, pros/cons, setup tips, and a peek into the dashboard.

Let’s dive in.

---

## What Is Umami?

Umami is an open-source, privacy-focused web analytics platform that you can self-host. ￼

### Key highlights

- No cookies, no personal data stored. Umami does not collect or store personally identifiable information by default. ￼
- Lightweight script. The tracking script is minimal (on the order of a couple of kilobytes), meaning it adds very little overhead to page load times. ￼
- Self-hosting / full control. Since the tool is open source and designed for ease of deployment, you retain full control over your data and infrastructure. ￼
- GDPR / privacy compliance out of the box. Because it does not rely on third-party tracking, cookies, or invasive user profiling, it helps with privacy compliance. ￼
- Multiple sites, custom events, basic segmentation. It supports multiple websites (tracked in one dashboard), custom events (button clicks, form submits, etc.), and filtering/segmentation. ￼

In short: Umami is a streamlined, privacy-first analytics alternative that gives you what you need without the burden of what you don’t.

---

## Why Self-Host?

You might wonder: why not just use Umami’s cloud offering or some other hosted analytics? Here are the top reasons:

1. **Data ownership & privacy**

   With self-hosting, your analytics data never passes through (or resides on) third-party servers beyond your control.

2. **Cost control**

   After the infrastructure cost (server, database), you incur no per-event or per-site fees. Some users report saving hundreds of dollars annually compared to managed analytics services. ￼

3. **Performance & minimal overhead**

   The tracking script is very small and non-blocking; it has negligible impact on site performance. ￼

4. **Flexibility & customization**

   When self-hosted, you can tweak, extend, or integrate with other systems (e.g. APIs, internal dashboards).

5. **Avoid vendor lock-in**

   You’re not tied to a “cloud plan” or changing pricing. You control upgrades, migrations, and scaling.

Of course, self-hosting does come with responsibilities: security, backups, updates, uptime. But for many technically inclined users (or small teams), the trade-off is worth it.

---

## How to Set Up Umami (Self-Hosted)

Below is a high-level overview. (Many detailed guides exist; I link some at the end.)

### Prerequisites

- **A server** (VPS, cloud, or on-prem) capable of running Docker & Docker Compose (or Node.js + database setup)
- **A PostgreSQL** (or MySQL) database instance
- **A domain or subdomain** (for dashboard and tracking endpoint)
- **(Recommended)** reverse proxy / TLS setup (e.g. via Caddy, Nginx or Traefik) ￼

### Docker / Docker Compose Deployment

Many users deploy Umami via Docker Compose, which simplifies service orchestration (app + database). ￼

#### A simplified docker-compose.yml might look like

```docker-compose.yml
version: "3.8"
services:
  umami:
    image: ghcr.io/umami-software/umami:postgresql-latest
    environment:
      DATABASE_URL: postgresql://umami:password@db:5432/umami
      APP_SECRET: your_random_secret
      # optional: TRACKER_SCRIPT_NAME or COLLECT_API_ENDPOINT to reduce ad-blocker blocking
    depends_on:
      - db
    restart: unless-stopped

  db:
    image: postgres:15-alpine
    environment:
      POSTGRES_DB: umami
      POSTGRES_USER: umami
      POSTGRES_PASSWORD: password
    volumes:
      - umami-db-data:/var/lib/postgresql/data
    restart: unless-stopped

volumes:
  umami-db-data:
```

#### Then you run

```bash
docker-compose up -d
```

You’ll also want to configure a reverse proxy (Traefik, Nginx) to route domain/subdomain traffic into the Umami container, enable TLS (Let’s Encrypt), and possibly rename the tracking endpoints to evade ad blockers. ￼

### Detailed deployment tutorials

- Self-host Umami Analytics With Docker Compose (Paul’s Blog) ￼
- Self-Hosted Site Analytics with Umami, Docker, and Traefik (AaronJBecker) ￼
- How to Self-Host Umami Analytics on Linux / Docker ￼

### Configuration & Tracking

Once the instance is running:

1. Log into the Umami dashboard (default user: admin, default password: umami; change immediately)
2. In Settings → Websites, add your website (domain)
3. Copy the tracking script
4. Place the snippet in the `<head>` (or via your templating/component) of your site’s HTML pages
5. Optionally, set up custom events or filters
6. (Optional) Set up backups of your database, monitor logs, and automate updates

Also consider setting environment variables like TRACKER_SCRIPT_NAME or COLLECT_API_ENDPOINT to rename the tracking endpoint, helping to reduce blocking by ad blockers. ￼

---

## My Experience (After 2 Days of Use)

Having run Umami self-hosted on my own infrastructure for the past two days, here’s what I’ve observed so far:

### Setup & Ease of Use

The setup was fairly smooth (Docker + reverse proxy). No major surprises. It took me just a little tweaking to get TLS, domain, and tracking working end to end.

### Dashboard & Analytics

Here’s a screenshot from my dashboard (insert your screenshot here):

[Insert screenshot of your Umami analytics dashboard]

---

## Some early impressions

- The real-time view and unique visitors / page views stats are clean and intuitive.
- I was able to configure a custom event (button click) easily.
- The segmentation by device, browser, and referrer is solid and gives enough insight for a site of my size.
- The UI is minimal, fast, and easy to navigate — no clutter.

### Performance & Impact

- The tracking script is extremely light; I couldn’t detect any noticeable slowdown from its inclusion.
- Because no cookies or heavy tracking is involved, I don’t need a cookie consent banner strictly for analytics (depending on your jurisdiction).

### Challenges & Things to Watch

- Since it’s only day two, my dataset is small. Larger traffic volumes or sustained use might surface edge cases.
- I’ll need to ensure database backups and monitor resource usage (CPU, memory).
- Occasionally, ad blockers or browser privacy settings may block the tracking script or data endpoint; hence using a less obvious script name or custom endpoint helps.

Overall, I’m quite happy with the results so far. It gives the right balance of insight without overcomplication or privacy trade-offs.

---

## Pros, Cons & When to Use

### Pros

- **Privacy-first**: No cookies, no PII by default
- **Lightweight**: Minimal script overhead
- **Full control**: Self-hosted means you own your data
- **Multiple sites & events**: Manage several domains, custom events
- **Simple UI & quick insights**: No steep learning curve

### Cons / Trade-offs

- **Responsibility**: You must handle updates, security, backups
- **Scaling**: May require tuning or more robust infrastructure at high traffic
- **Ad-blockers / privacy software**: Some traffic may still be suppressed, so tracking might not be 100%
- **Fewer “advanced” features**: Compared to enterprise analytics suites, you won’t get massive funnels, heatmaps, predictive models, etc.

### Best Use Cases

- Blogs, personal sites, small to moderate traffic websites
- Projects where privacy, control, and minimal overhead matter more than advanced analytics bells & whistles
- Developers or teams comfortable with server management

For high-traffic enterprise sites needing deep funnels, big data modeling, or integrations with marketing stacks, Umami might need to be complemented with other tools or scaled carefully.

---

## Conclusion & Next Steps

In just two days of use, Umami Analytics has impressed me as an elegant, privacy-respecting, lightweight web analytics solution. Self-hosting gives me full control over my data, and the dashboard already yields useful insights without overcomplication.

If you’re interested in giving it a try:

1. Spin up a small server instance (or use an existing one)
2. Deploy via Docker (or manually)
3. Configure domain + TLS + reverse proxy
4. Add the tracking script
5. Monitor, backup, and iterate

If you like, I can help you write a tutorial tailored to your server setup (Ubuntu, Nginx, etc.), or help embed your screenshot and optimize this blog post further. Do you want me to generate the final version ready for your blog (with your screenshot inserted)?

---

### 🤝 Need a Custom RSVP System or Dashboard?

I help businesses build tools that _actually work_ , even on tight deadlines.

Whether you're planning an event, need internal tools, or want a custom dashboard for your team , I can help.

### Reach out

📧 Email: [safi.abdulkader@gmail.com](mailto:safi.abdulkader@gmail.com) | 💻 LinkedIn: [@abdulkader-safi](https://www.linkedin.com/in/abdulkader-safi/) | 📱 Instagram: [@abdulkader.safi](https://www.instagram.com/abdulkader.safi/) | 🏢 [DSRPT](https://www.dsrpt.com.au/kw/contact)

_Drop me a line, I’m always happy to collaborate!_ 🚀
