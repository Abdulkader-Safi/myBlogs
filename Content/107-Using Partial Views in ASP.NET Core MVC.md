# Using Partial Views in ASP.NET Core MVC

How to build reusable UI components with Partial Views

This article uses the [MVC_Components_test](https://github.com/Abdulkader-Safi/MVC_Components_test) repo by [Abdulkader Safi](https://abdulkadersafi.com) as a working example.

---

## TL;DR

Partial Views let you break up UI into modular, reusable pieces in MVC. Using them improves maintainability, consistency, and reduces duplication. In this example, we’ll build a Card component as a Partial View, driven by a strongly-typed ViewModel.

---

### Table of Contents

1. [What is a Partial View?](#1-what-is-a-partial-view)
2. [Why use Partial Views (benefits)](#2-why-use-partial-views-benefits)
3. [Create the sample project via CLI](#3-create-the-sample-project-via-cli)
4. [Structure of the sample “MVC_Components_test” project](#4-structure-of-the-sample-mvc_components_test-project)
5. [Code Walk-through: building a Card Partial View](#5-code-walk-through-building-a-card-partial-view)
6. [Using the Partial in a View](#6-using-the-partial-in-a-view)
7. [Conditional rendering & customization](#7-conditional-rendering--customization)
8. [Best practices](#8-best-practices)
9. [SEO / LLM readiness: making your UI friendly to search engines and AI readers](#9-seo--llm-readiness-making-ui-friendly-for-search-engines--ai)
10. [Next steps & advanced topics](#10-next-steps--advanced-topics)

---

## 1. What is a Partial View?

A Partial View in ASP.NET Core MVC is a Razor view that renders part of a page. Think of it like a “component” or a piece of UI you can embed inside other views. It helps you reuse UI markup & logic without duplicating code.

---

## 2. Why Use Partial Views (Benefits)

- **Reusability**: write the HTML once, use it in many places
- **Maintainability**: updates in one place propagate everywhere
- **Consistency**: same look & feel across app
- **Modularity**: easier to reason about small components
- **Strong typing**: using ViewModels decreases runtime errors
- **Easier testing and styling**: isolate component styling or behavior

---

## 3. Create the Sample Project via CLI

Here’s how you could start from scratch to mimic (or understand) the "[MVC_Components_test](https://github.com/Abdulkader-Safi/MVC_Components_test)" project using CLI commands.

```bash
# Create a new solution and project
dotnet new sln -o MVC_Components
cd MVC_Components

# Create an MVC web app
dotnet new mvc -n MVC_Components_test
dotnet sln add MVC_Components_test/MVC_Components_test.csproj

cd MVC_Components_test

# Optional: add Bootstrap or other CSS frameworks (if you're using CDN / package manager)
# For demo, assume built-in CSS + custom styles

# Restore and run
dotnet restore
dotnet build
dotnet run
```

This gives you the base ASP.NET Core MVC app scaffolded with Controllers, Views, wwwroot, etc.

---

## 4. Structure of the Sample “MVC_Components_test” Project

Looking at your repo, here are the important parts:

```bash
MVC_Components_test/
├─ Controllers/
│   └ HomeController.cs
├─ Models/
│   ├ CardViewModel.cs
│   └ ErrorViewModel.cs
├─ Views/
│   ├ Home/
│   │   ├ Index.cshtml
│   │   └ Privacy.cshtml
│   └ Shared/
│       ├ Components/
│       │   └ _Card.cshtml         ← your Partial View
│       ├ _Layout.cshtml
│       └ _ValidationScriptsPartial.cshtml
├─ wwwroot/
│   └ css/site.css
├ Program.cs
└ MVC_Components_test.csproj
```

Key point: **Views/Shared/Components/\_Card.cshtml** is where the Card Partial View lives; CardViewModel defines its properties. Your HomeController populates instances of CardViewModel and passes them to Index.cshtml where you render partials.

---

## 5. Code Walk-Through: building a Card Partial View

### 5.1 The ViewModel: CardViewModel

In Models/CardViewModel.cs you have something like:

```csharp
public class CardViewModel
{
    public string Title { get; set; } = string.Empty;
    public string Content { get; set; } = string.Empty;
    public string? Footer { get; set; }
    public string? ImageUrl { get; set; }
    public string? ActionUrl { get; set; }
    public string? ActionText { get; set; }
    public string CssClass { get; set; } = "card-default";
    public bool ShowBorder { get; set; } = true;
    public bool ShowShadow { get; set; } = true;
}
```

This defines what data or configuration your Card needs: text, optional image, optional action, styling flags etc. Strong typing here is very helpful.

### 5.2 The Partial View: \_Card.cshtml

Located under **Views/Shared/Components/\_Card.cshtml**. It might look like:

```csharp
@model MVC_Components_test.Models.CardViewModel

<div class="card @Model.CssClass @(Model.ShowBorder ? "card-border" : "") @(Model.ShowShadow ? "card-shadow" : "")">
    @if (!string.IsNullOrEmpty(Model.ImageUrl))
    {
        <img src="@Model.ImageUrl" class="card-img-top" alt="@Model.Title" />
    }
    <div class="card-body">
        <h5 class="card-title">@Model.Title</h5>
        <p class="card-text">@Model.Content</p>
        @if (!string.IsNullOrEmpty(Model.ActionUrl) && !string.IsNullOrEmpty(Model.ActionText))
        {
            <a href="@Model.ActionUrl" class="btn btn-primary">@Model.ActionText</a>
        }
    </div>
    @if (!string.IsNullOrEmpty(Model.Footer))
    {
        <div class="card-footer text-muted">@Model.Footer</div>
    }
</div>
```

This shows conditional rendering: only show image if ImageUrl is non-empty, only show footer etc.

---

## 6. Using the Partial in a View

In **Views/Home/Index.cshtml**:

```csharp
@model IEnumerable<MVC_Components_test.Models.CardViewModel>

@{
    ViewData["Title"] = "Home Page";
}

<h2>Card Component Demo</h2>

<div class="row">
    @foreach (var card in Model)
    {
        <div class="col-md-4">
            <partial name="Components/_Card" model="card" />
        </div>
    }
</div>
```

In **HomeController.cs**, you build up a list of CardViewModel instances, something like:

```csharp
public IActionResult Index()
{
    var cards = new List<CardViewModel>
    {
        new CardViewModel {
            Title = "Welcome Card",
            Content = "This is a sample card component demonstrating reusable UI in MVC.",
            Footer = "Last updated: " + DateTime.Now.ToString("MMM dd, yyyy"),
            CssClass = "card-success",
            ActionUrl = Url.Action("Privacy", "Home"),
            ActionText = "Learn More"
        },
        // ... more cards
    };

    return View(cards);
}
```

Then in Index.cshtml, you access `@model IEnumerable<CardViewModel>`.

---

## 7. Conditional Rendering & Customization

You’ve done good with:
- Only render Image section if ImageUrl not null or empty
- Only render Action button if both URL and Text are set
- Toggle CSS classes for border/shadow etc

These patterns are essential to make components flexible without cluttering callers with HTML.

---

## 8. Best Practices

Here are some suggestions / best practices when using Partials in MVC:
- Naming convention: prefix partials with \_ (underscore) and keep them in Shared/Components or another common folder
- Strongly-typed ViewModels for partials — avoid using ViewBag/ViewData unless necessary
- Keep partials focused: each partial should do one thing (e.g. Card, Modal, Alert)
- Parameterize behavior via ViewModel properties
- Minimize logic in view: conditional parts are okay, but avoid heavy business logic
- Style separation: put component-specific CSS / classes in your site CSS or component CSS files
- Performance: partials are part of the view rendering; overuse or deeply nested partials can affect performance—profile if needed
- Caching: for parts that don’t change often, you may explore output caching or using ViewComponents (next step)

---

## 9. SEO / LLM Readiness: Making UI Friendly for Search Engines & AI

To make your MVC app and partial components more discoverable / usable by search engines and LLMs (large language models, AI):
- **Semantic HTML**: use proper headings (`<h1>`, `<h2>` etc.), alt attributes on images, descriptive link text
- **Accessible markup**: alt text, aria tags if needed, semantic tags
- **Clean URLs** for links (e.g. Privacy, not weird hashed routes)
- **Consistent CSS class names**: helps when AI is parsing HTML + CSS for structure
- **Include meaningful metadata**: title, description in layout, perhaps Open Graph etc, though that’s more about pages than partials
- **Avoid inline styles where possible**: separate CSS helps readability + parsing

In your sample, you already have alt attributes via alt="@Model.Title" for image, which is good.

---

## 10. Next Steps & Advanced Topics

Once you’re comfortable with partials, consider:
- **ViewComponents**: more powerful: can have logic, DI, caching, etc. Better for components with behavior, not just pure rendering
- **Tag Helpers / Razor Components**: (if moving toward Blazor / Razor Pages)
- **Component Libraries / Shared UI**: NuGet packages for reuse across projects
- **Client-side interactivity**: add JS hooks for components (modals, drop downs, etc.)
- **Theming & themability**: allow per-component theme settings

---

## Putting it All Together: Full Example

Here’s a condensed end-to-end “from scratch → card component” flow, combining the code:

### 1. Create project:

```bash
dotnet new mvc -n MVC_Components_test
cd MVC_Components_test
```

### 2. Create ViewModel:

```csharp
// Models/CardViewModel.cs
namespace MVC_Components_test.Models
{
    public class CardViewModel
    {
        public string Title { get; set; } = string.Empty;
        public string Content { get; set; } = string.Empty;
        public string? Footer { get; set; }
        public string? ImageUrl { get; set; }
        public string? ActionUrl { get; set; }
        public string? ActionText { get; set; }
        public string CssClass { get; set; } = "card-default";
        public bool ShowBorder { get; set; } = true;
        public bool ShowShadow { get; set; } = true;
    }
}
```

### 3. Create Partial View:

```csharp
// Views/Shared/Components/_Card.cshtml
@model MVC_Components_test.Models.CardViewModel

<div class="card @Model.CssClass @(Model.ShowBorder ? "card-border" : "") @(Model.ShowShadow ? "card-shadow" : "")">
    @if (!string.IsNullOrEmpty(Model.ImageUrl))
    {
        <img src="@Model.ImageUrl" class="card-img-top" alt="@Model.Title" />
    }
    <div class="card-body">
        <h5 class="card-title">@Model.Title</h5>
        <p class="card-text">@Model.Content</p>
        @if (!string.IsNullOrEmpty(Model.ActionUrl) && !string.IsNullOrEmpty(Model.ActionText))
        {
            <a href="@Model.ActionUrl" class="btn btn-primary">@Model.ActionText</a>
        }
    </div>
    @if (!string.IsNullOrEmpty(Model.Footer))
    {
        <div class="card-footer text-muted">@Model.Footer</div>
    }
</div>
```

### 4. Controller:

```csharp
using Microsoft.AspNetCore.Mvc;
using MVC_Components_test.Models;
using System;
using System.Collections.Generic;

namespace MVC_Components_test.Controllers
{
    public class HomeController : Controller
    {
        public IActionResult Index()
        {
            var cards = new List<CardViewModel>
            {
                new CardViewModel {
                    Title = "Welcome Card",
                    Content = "This is a sample card component demonstrating reusable UI in MVC.",
                    Footer = "Last updated: " + DateTime.Now.ToString("MMM dd, yyyy"),
                    CssClass = "card-success",
                    ActionUrl = Url.Action("Privacy", "Home"),
                    ActionText = "Learn More"
                },
                // more cards...
            };

            return View(cards);
        }

        public IActionResult Privacy()
        {
            return View();
        }
    }
}
```

### 5. View:

```csharp
// Views/Home/Index.cshtml
@model IEnumerable<MVC_Components_test.Models.CardViewModel>

<h2>Card Component Demo</h2>
<div class="row">
    @foreach (var card in Model)
    {
        <div class="col-md-4">
            <partial name="Components/_Card" model="card" />
        </div>
    }
</div>
```

### 6. Styling (site.css or similar):

```css
.card-default {
  /* default look */
}
.card-border {
  border: 1px solid #ccc;
}
.card-shadow {
  box-shadow: 0 2px 5px rgba(0, 0, 0, 0.1);
}
.card-success {
  border-color: green;
}
/* etc */
```

---

## Conclusion

Using Partial Views in ASP.NET Core MVC (as in your MVC_Components_test example) is a powerful way to build modular, maintainable, and reusable UI. With strong ViewModels, conditional rendering, and consistent styling, you can greatly improve developer experience and code quality.

If you follow the practices above, your application will be:
- easier to maintain
- easier to expand with new components
- more consistent in UI
- more understandable by search engines and AI systems

---

### 🤝 Need a Custom RSVP System or Dashboard?

I help businesses build tools that _actually work_ , even on tight deadlines.

Whether you're planning an event, need internal tools, or want a custom dashboard for your team , I can help.

### Reach out

📧 Email: [safi.abdulkader@gmail.com](mailto:safi.abdulkader@gmail.com) | 💻 LinkedIn: [@abdulkader-safi](https://www.linkedin.com/in/abdulkader-safi/) | 📱 Instagram: [@abdulkader.safi](https://www.instagram.com/abdulkader.safi/) | 🏢 [DSRPT](https://www.dsrpt.com.au/kw/contact)

_Drop me a line, I’m always happy to collaborate!_ 🚀