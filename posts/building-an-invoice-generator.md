---
title: Building an Invoice Tool
date: 2022-05-09
tags:
  - CSS
  - HTML
  - Javascript
layout: layouts/post.njk
---

As the final project from scrimba's great intro to javascript course, I'll be building a simple invoice generator tool. The project itself is _fairly_ basic with the requirements being: three possible services that can be rendered, each can only be charged for once, a total displayed, and a stretch goal of allowing each to be removed separately. I'm going to be augmenting the requirements a bit: the list of services rendered will start with the three defaults but an input field will allow for adding more, as a result both the buttons and rendered services will be stored in localStorage. Additionally, you will be able to add a recipient name and email address and the send button will format a useful message stored in a `mailto` link. Let's get started!

## Laying the HTML Foundation
I'm still a big believer in separation of concerns, the idea that each language is responsible for a specific aspect of a web experience and there should be minimal overlap between those responsibilities. HTML provides document structure, CSS provides style and legibility, and JS provides functionality. Let's start by structuring the document. 

The page is broken up into four major sections with additional content within them. The first is the header with the title and welcome message. Next is the form for adding new custom services as well as the buttons for each service available. Following that section is the main part of the page showing the table of services rendered and the invoice total. Last is the footer where you enter the recipient name and email address to populate the send invoice mailto link properties. It looks a bit like this: 

```html
<!DOCTYPE html>
<html>
    <head>
        <title>Demo - Invoice Generator</title>
        <link rel="stylesheet" href="../reset.css">
        <link rel="stylesheet" href="style.css">
    </head>
    <body>
      <header>
        <h1>Invoice Creator</h1>
        <p>Thanks for choosing GoodCorp, LLC!</p>
      </header>
      <section>
        <form action="/">
          <span>
            <label for="customServiceName">Custom Service</label>
            <input type="text" id="customServiceName">
          </span>
          <span>
            <label for="customServiceValue">Cost</label>
            <input type="text" id="customServiceValue" type="number">
          </span>
          <button>Add Service</button>
        </form>
        <ul>
          <li>
            <button>Wash Car: $10</button>
            <button>&times;<span>remove</span></button>
          </li>
        </ul>
      </section>
      <main>
        <table>
          <caption>Services rendered for {Client Name} by GoodCorp</caption>
          <thead>
          <tr>
            <th>Service</th>
            <th>Cost</th>
          </tr>
          </thead>
          <tbody>
          <tr>
            <td>
              <span>Service</span>
              <button>Remove</button>
            </td>
            <td>
              <span>$</span>
              <span>00</span>
            </td>
          </tr>
          </tbody>
        </table>
        <hr>
        <div>
          <section>
            <h2>Notes</h2>
            <p>We accept cash, credit card, or PayPal</p>
          </section>
          <section>
            <h2>Total Amount</h2>
            <p>$00</p>
          </section>
        </div>
      </main>
      <footer>
        <form action="/">
          <label for="recipientName">Recipient Name</label>
          <input type="text" id="recipientName">
          <label for="recipientEmail">Recipient Email</label>
          <input type="text" id="recipientEmail" type="email">
          <button>Update</button>
        </form>
        <a href="mailto:address@example.com">Send Invoice</a>
      </footer>

      <script type="text/javascript" src="script.js"></script>
    </body>
</html>
```
