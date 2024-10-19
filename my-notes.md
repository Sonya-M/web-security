## Cookies

- accessing cookies from the browser:

```js
document.cookie = 'username=bobytables'; // out: 'username=bobytables'
document.cookie = 'password=papayawhip'; // out: 'password=papayawhip'
document.cookie; // out: 'username=bobytables; password=papayawhip'
```

- think of cookies as key-value store, but w/ some special properties
- encrypting cookie values

### sessions & httpOnly

- session: just an abstraction, separate user from session - on login, create unique identifier for session;
  - can sign out of individual sessions
- set additional attributes on cookie, e.g. `{httpOnly: true}` to keep it away from js
- now cookie is in header, included in requests, but it's not available from js - so `document.cookie` returns empty string - but still going over the wire in plain text (here not using https - to use https, add `secure: true` or `secure: process.env.NODE_ENV === 'production'`)

```js
app.post('/login', async (req, res) => {
  const { username, password } = req.body;

  const user = await db.get(
    'SELECT * FROM users WHERE username = ? AND password = ?',
    [username, password]
  );

  if (user) {
    res.cookie('username', username, {
      httpOnly: true,
      secure: process.env.NODE_ENV === 'production',
    });
    res.redirect('/profile');
  } else {
    res.status(403).redirect('/login?error=Invalid login credentials.');
  }
});
```

### signing cookies

- value + secret:
  `app.use(cookieParser(process.env.COOKIE_SECRET));`
- cookieParser will sign the cookies, and browser will be able to read the cookies and know if their value was changed; we are sending 2 things: value of cookie and the proof that we know the secret
- not all cookies should be signed, e.g. dark mode cookie etc., so they can be accessed from js
- `req.cookie` vs `req.signedCookie`
