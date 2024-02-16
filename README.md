# Burpsuite Academy

## SQLi

### Lab: SQL injection vulnerability in WHERE clause allowing retrieval of hidden data

We can exploit an SQLi in category search input: `' OR 1=1 -- `

The query will be interpreted as: `SELECT * FROM products WHERE category = '' OR 1=1 -- ' AND released = 1`

cleaning it up gives: `SELECT * FROM products WHERE category = '' OR 1=1`

### Lab: SQL injection vulnerability allowing login bypass

We can head to `/login` route, send `administrator' -- ` in the username field. We'll be logged in as administrator.

## XSS

### Lab: Reflected XSS into HTML context with nothing encoded

Performing a search in the website, we can see that our input is reflected. We try `<script>alert(1)</script>` as our search payload & we get an XSS.

### Lab: Stored XSS into HTML context with nothing encoded

We can go to a post's details page, scroll down & write a new comment. We use `<script>alert(1)</script>` for the comment body & anything for the rest of the other fields. Posting this comment creates a stored XSS.

This alert will be executed to anyone that opens the post details later on.

### Lab: DOM XSS in document.write sink using source location.search

**Description**: This lab contains a DOM-based cross-site scripting vulnerability in the search query tracking functionality. It uses the JavaScript document.write function, which writes data out to the page. The document.write function is called with data from location.search, which you can control using the website URL.

Examining the page's source code:

```html
<script>
    function trackSearch(query) {
        document.write('<img src="/resources/images/tracker.gif?searchTerms='+query+'">');
    }
    var query = (new URLSearchParams(window.location.search)).get('search');
    if(query) {
        trackSearch(query);
    }
</script>
```

Well, that's our XSS. We can insert a `">` to break out of the `img` tag then inject our XSS payload. Full payload: `a"><script>alert(1)</script>`