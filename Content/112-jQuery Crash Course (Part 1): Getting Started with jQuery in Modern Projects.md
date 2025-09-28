# jQuery Crash Course (Part 1): Getting Started with jQuery in Modern Projects

jQuery has been around since 2006, and while modern JavaScript (ES6+) and frameworks like React or Vue dominate today’s landscape, jQuery still powers millions of websites. Its simplicity, cross-browser compatibility, and robust plugin ecosystem make it a valuable skill for developers working with legacy projects, or when you just need something quick and lightweight.

#### This Part 1 of our jQuery Crash Course will walk you through:

- What jQuery is and why it’s still useful.
- Setting up a modern project with Vite + jQuery.
- Core jQuery concepts: selectors, DOM manipulation, and events.
- Hands-on code examples.

#### In Part 2, we’ll dive into AJAX with jQuery.

---

## What is jQuery?

jQuery is a fast, lightweight JavaScript library designed to simplify HTML DOM traversal, event handling, animations, and AJAX. Its motto, “Write less, do more,” perfectly captures why it became so popular.

For example:

### Vanilla JS

```javascript
document.getElementById("myBtn").addEventListener("click", function () {
  alert("Button clicked!");
});
```

### jQuery

```javascript
$("#myBtn").click(() => alert("Button clicked!"));
```

With just one line, jQuery streamlines what used to be verbose and cross-browser tricky code.

---

## Setting Up a Project with Vite + jQuery

Although jQuery was originally used by just linking a CDN script in HTML, let’s modernize the setup using Vite, a blazing-fast frontend tool.

### Step 1: Initialize a Vite Project

#### Create a new project

```bash
npm create vite@latest jquery-crash-course
```

#### Navigate into it

```bash
cd jquery-crash-course
```

#### Install dependencies

```bash
npm install
```

### Step 2: Install jQuery

```bash
npm install jquery
```

### Step 3: Import jQuery in Your Project

Inside **main.js** (or index.js depending on your setup):

```javascript
import $ from "jquery";

$(document).ready(function () {
  console.log("jQuery is working with Vite!");
});
```

### Step 4: Run the Dev Server

```bash
npm run dev
```

Open the project in your browser, you’re officially running jQuery with Vite. 🚀

---

## jQuery Basics with Code Examples

### 1. jQuery Selectors

Selectors let you grab elements quickly.

```javascript
// By ID
$("#title").text("Hello jQuery!");

// By Class
$(".btn").css("color", "blue");

// By Element
$("p").hide();
```

### 2. DOM Manipulation

Change or add content dynamically.

```javascript
// Change text
$("#title").text("Welcome to the jQuery Crash Course");

// Add HTML
$("#content").html("<strong>Learning made easy!</strong>");

// Append new element
$("#list").append("<li>New Item</li>");
```

### 3. Event Handling

jQuery makes handling events elegant.

```javascript
// Click event
$("#myBtn").on("click", () => {
  alert("You clicked the button!");
});

// Hover event
$("#myDiv").on("mouseenter", () => {
  $("#myBtn").css("background-color", "lightblue");
});

$("#myDiv").on("mouseleave", () => {
  $("#myBtn").css("background-color", "white");
});
```

### 4. Simple Animations

jQuery comes with built-in effects.

```javascript
// Fade out element
$("#box").fadeOut(1000);

// Slide toggle
$("#menuBtn").on("click", () => {
  $("#menu").slideToggle();
});
```

---

## Why Use jQuery in 2025?

- **Legacy Projects**: Still widely used in WordPress, Drupal, Shopify, and older apps.
- **Quick Prototyping**: Easy to get things running without large frameworks.
- **Plugin Ecosystem**: Thousands of prebuilt plugins for UI widgets, sliders, validation, etc.

While you wouldn’t build a large-scale app with jQuery today, it remains relevant when speed and compatibility matter.

---

### What’s Next (Part 2 Teaser 🚀)

In Part 2 of this jQuery Crash Course, we’ll cover AJAX with jQuery, how to make asynchronous requests, fetch APIs, and dynamically update your webpage without reloading.

Stay tuned, it’s where the real magic happens.

---

## Conclusion

You’ve just set up a modern jQuery project with Vite, learned selectors, DOM manipulation, event handling, and even basic animations.

---

### 🤝 Need a Custom RSVP System or Dashboard?

I help businesses build tools that _actually work_ , even on tight deadlines.

Whether you're planning an event, need internal tools, or want a custom dashboard for your team , I can help.

### Reach out

📧 Email: [safi.abdulkader@gmail.com](mailto:safi.abdulkader@gmail.com) | 💻 LinkedIn: [@abdulkader-safi](https://www.linkedin.com/in/abdulkader-safi/) | 📱 Instagram: [@abdulkader.safi](https://www.instagram.com/abdulkader.safi/) | 🏢 [DSRPT](https://www.dsrpt.com.au/kw/contact)

_Drop me a line, I’m always happy to collaborate!_ 🚀
