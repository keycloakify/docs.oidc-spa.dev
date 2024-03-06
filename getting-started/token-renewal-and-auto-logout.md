# 🔁 Token renewal and auto logout

### Refreshing token

The token refresh is handled automatically for you, however you can manually trigger a token refresh with `oidc.renewTokens()`.(or `const { renewTokens } = useOidc({ assertUserLoggedIn: true }`);&#x20;
