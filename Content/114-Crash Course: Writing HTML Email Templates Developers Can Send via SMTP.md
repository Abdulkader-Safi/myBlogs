# Crash Course: Writing HTML Email Templates Developers Can Send via SMTP

Email is one of the most effective communication channels, but writing HTML email templates is tricky. Unlike the web, where CSS grids and flexbox reign supreme, email design is still stuck in the land of <table> layouts, inline CSS, and weird client quirks.

This post is a crash course for developers who want to build and send HTML emails directly from code (via SMTP) in .NET, Node.js, or PHP, with lots of ready-to-use layouts.

---

## Why Email Templates Are Different

- 📧 Tables over divs – Email clients don’t fully support modern CSS layouts.
- 🎨 Inline CSS – Stylesheets are often stripped out.
- 🌐 Absolute URLs only – For images, backgrounds, and links.
- 📱 Responsive hacks – Use fluid tables and inline media queries (with limited support).
- 🔄 Plain text fallback – Always include a text-only version.

---

## Base Email Template (Boilerplate)

Every email starts with a wrapper:

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="UTF-8" />
    <title>Email Template</title>
  </head>
  <body style="margin:0; padding:0; background:#f6f6f6;">
    <table
      role="presentation"
      width="100%"
      cellspacing="0"
      cellpadding="0"
      border="0"
      bgcolor="#f6f6f6"
    >
      <tr>
        <td align="center" style="padding:20px;">
          <!-- Container -->
          <table
            width="600"
            cellspacing="0"
            cellpadding="0"
            border="0"
            bgcolor="#ffffff"
            style="border-radius:8px; overflow:hidden;"
          >
            <tr>
              <td style="padding:30px; font-family:Arial, sans-serif;">
                <!-- CONTENT GOES HERE -->
              </td>
            </tr>
          </table>
          <!-- End Container -->
        </td>
      </tr>
    </table>
  </body>
</html>
```

---

## Example Layouts

### 1. Hero Section with Background Image

```html
<tr>
  <td
    background="https://example.com/bg.jpg"
    bgcolor="#333333"
    style="background-size:cover; text-align:center; padding:60px; color:white;"
  >
    <h1
      style="margin:0; font-size:28px; font-family:Helvetica, Arial, sans-serif;"
    >
      Welcome to Our Community 🎉
    </h1>
    <p style="font-size:16px; margin:20px 0;">We’re glad you’re here.</p>
    <a
      href="https://example.com/login"
      style="display:inline-block; padding:14px 28px; background:#FF5722; color:#fff;
              text-decoration:none; border-radius:4px; font-weight:bold;"
    >
      Get Started
    </a>
  </td>
</tr>
```

### 2. Two-Column Layout (Features/Products)

```html
<tr>
  <td style="padding:20px;">
    <table width="100%" cellspacing="0" cellpadding="0" border="0">
      <tr>
        <td width="50%" valign="top" style="padding:10px;">
          <img
            src="https://example.com/feature1.png"
            width="100%"
            style="border-radius:6px;"
          />
          <h3 style="font-family:Arial; color:#333;">Feature One</h3>
          <p style="color:#666;">Quick description of this awesome feature.</p>
        </td>
        <td width="50%" valign="top" style="padding:10px;">
          <img
            src="https://example.com/feature2.png"
            width="100%"
            style="border-radius:6px;"
          />
          <h3 style="font-family:Arial; color:#333;">Feature Two</h3>
          <p style="color:#666;">Quick description of another benefit.</p>
        </td>
      </tr>
    </table>
  </td>
</tr>
```

### 3. Card-Based Section

```html
<tr>
  <td style="padding:30px; text-align:center;">
    <table
      role="presentation"
      cellpadding="0"
      cellspacing="0"
      border="0"
      align="center"
      width="100%"
    >
      <tr>
        <td
          style="padding:20px; background:#fafafa; border-radius:8px; font-family:Arial; color:#333;"
        >
          <h3 style="margin:0;">Weekly Update</h3>
          <p style="margin:10px 0;">Here’s what’s new this week...</p>
          <a
            href="https://example.com/update"
            style="color:#007BFF; text-decoration:none; font-weight:bold;"
            >Read More →</a
          >
        </td>
      </tr>
    </table>
  </td>
</tr>
```

### 4. Footer Section

```html
<tr>
  <td
    bgcolor="#333333"
    style="padding:20px; text-align:center; color:#ffffff; font-size:12px; font-family:Arial;"
  >
    <p style="margin:0;">&copy; 2025 Your Company</p>
    <p style="margin:5px 0;">
      <a
        href="https://example.com/unsubscribe"
        style="color:#bbbbbb; text-decoration:none;"
      >
        Unsubscribe
      </a>
    </p>
  </td>
</tr>
```

### the final email design look

![the final email template](https://abdulkadersafi.com/storage/d6yw2Dvue71biEvrPK7x5goL9maWRe4Byh8kybfg.png)

---

## SMTP Sending Examples

Now let’s plug in these templates in real code.

### .NET (C# + MailKit)

```csharp
var builder = new BodyBuilder();
builder.TextBody = "Welcome to our service! (Plain text fallback)";
builder.HtmlBody = File.ReadAllText("templates/welcome.html");
message.Body = builder.ToMessageBody();
```

### Node.js (Nodemailer)

```javascript
const htmlTemplate = require("fs").readFileSync(
  "templates/welcome.html",
  "utf8"
);

let info = await transporter.sendMail({
  from: '"Your App" <you@example.com>',
  to: "user@example.com",
  subject: "Welcome 🎉",
  text: "Plain text fallback here",
  html: htmlTemplate,
});
```

### PHP (PHPMailer)

```php
$mail->Body    = file_get_contents("templates/welcome.html");
$mail->AltBody = "Plain text fallback here";
```

---

## Developer Pro Tips

- ✅ Use template files (welcome.html, newsletter.html) and load them in your code.
- ✅ Keep reusable snippets for hero, footer, buttons, and columns.
- ✅ Always inline CSS, use Premailer or MJML.
- ✅ Test in Gmail, Outlook, Apple Mail, and iOS/Android.
- ✅ Minimize image weight and host them on HTTPS.

---

## Final Takeaway

Writing HTML email templates is more about discipline than creativity. Stick to tables, inline CSS, fallbacks, and modular snippets. Once you have a set of reusable sections (hero, two-column, footer, CTA buttons), you can mix-and-match to build newsletters, receipts, onboarding emails, or promotional campaigns.

When combined with SMTP in .NET, Node.js, or PHP, you’ll have full control and flexibility without relying on third-party services.

---

### 🤝 Need a Custom RSVP System or Dashboard?

I help businesses build tools that _actually work_ , even on tight deadlines.

Whether you're planning an event, need internal tools, or want a custom dashboard for your team , I can help.

### Reach out

📧 Email: [safi.abdulkader@gmail.com](mailto:safi.abdulkader@gmail.com) | 💻 LinkedIn: [@abdulkader-safi](https://www.linkedin.com/in/abdulkader-safi/) | 📱 Instagram: [@abdulkader.safi](https://www.instagram.com/abdulkader.safi/) | 🏢 [DSRPT](https://www.dsrpt.com.au/kw/contact)

_Drop me a line, I’m always happy to collaborate!_ 🚀
