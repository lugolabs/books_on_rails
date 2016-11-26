# Styling the layout

In the [first part](http://lugolabs.com/lugoland/articles/75-building-a-ruby-on-rails-app-part-1-the-app) of our *Build a web app with Ruby on Rails* series we created a layout view, as well as a header and footer. In this post we will add a paint of style to them.

Layout
--

It's important we keep our stylesheets well organized in separate files, based on the styles or elements they are connected. I usually start my apps with this `application.css` skeleton:

```css
/*
 *= require reset
 *= require layout
 *= require header
 *= require footer
*/
```

I still use Meyer's `reset.css` file - with some tweaks - that you can find aat [github repo](). It removes margins and paddings, resets new HTML5 elements in older browsers, etc.

The *layout.css* file is occupied with the high level elements or classes used accross the application:

```css
/* app/assets/layout.scss */

body {
  font-family: 'Helvetica Neue', Helvetica, sans-serif;
  font-size: 15px;
  background: #f2f5f8;
  font-weight: 300;
}

a {
  text-decoration: none;
}

.app-cont {
  max-width: 1080px;
  margin: 0 auto;
}

.app-cont-thin {
  max-width: 650px;
  background: #fff;
  padding: 1em 0;
  border-radius: 3px;
}
```

Please note how we prefix the application-wide elements with *app-*, so that they differentiate from the same elements in internal pages.

Let's add the logo styles here too as they'll be used in both *header* and *footer*:

```sass
/* app/assets/layout.scss */

.app-logo {
  a {
    font-family: 'Helvetica Neue', Helvetica, sans-serif;
    font-size: 1.4em;
    color: rgba(255, 255, 255, 0.71);
    text-transform: uppercase;
    letter-spacing: 5px;
    text-shadow: 0 0 10px rgba(218, 62, 62, 0.32);

    b {
      color: rgba(255, 153, 153, 0.83);
    }
  }
}
```

We've placed the *on* of *Roby on Rails* logo inside a `b` element, which has allowed us to style it differently, this spicying it up a little.


Header
--

Next we style the header:

```sass
/* app/assets/header.scss */

.app-header {
  margin-bottom: 3em;
  padding: 1.5em 0;
  background: rgba(53, 144, 202, 0.95);
  position: relative;

  .img {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background: url('<%= asset_path "library.jpg" %>') center center no-repeat;
    background-size: cover;
    z-index: -1;
  }
}
```

The header has a semi transparent blue background, letting through an image of books to give the app a bit of context. The image is taken from the free image library [Pixabay](http://www.pixabay.com) and prepared for the web in Photoshop.

Footer
--

The footer uses the same structure as the header, but a bit bigger:

```sass
/* app/assets/footer.scss */

.app-footer {
  margin-top: 6em;
  padding: 4em 0;
  background: rgba(53, 144, 202, 0.95);
  position: relative;

  .img {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background: url('<%= asset_path "library.jpg" %>') center center no-repeat;
    background-size: cover;
    z-index: -1;
  }

  .copy {
    padding-top: 2em;
    color: rgba(255, 255, 255, 0.6);

    a {
      color: rgba(255, 255, 255, 0.6);
      text-decoration: underline;

      &:hover {
        color: #fff;
      }
    }
  }
}
```

Conclusion
--

In this post we added some needed styles to our *Books on Rails* app. We accomplished that by creating separate stylesheets for sepearate concerns, e.g. layout, header, footer, and logo.

In the [next post](/lugoland/articles/77-building-a-ruby-on-rails-app-part-3-importing-book-details-from-amazon) we will connect to the Amazon API to get the details about our books.

You can check out the previous posts in this series in:

* [Books on Rails](/booksonrails) live example
