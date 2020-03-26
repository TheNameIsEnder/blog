+++
author = "Jacob Hell"
title = "The Chain of Responsibility Pattern Makes Hard Validation Easy"
date = "2020-03-25"
description = "Introducing the Chain of Responsibility Pattern."
tags = [
    "design",
    "chain of responsibility"
]

+++

I like my validation workflow the same way I like my IKEA tables. Easy to setup and with a cool name. Though the Chain of Responsibility is no [Godfjord](https://www.ikea.com/us/en/p/godfjord-bed-frame-gray-s99256172/), it is pretty easy to use.

<!--more-->

## Intro

Hello professional computer touchers.

A great [intro](https://refactoring.guru/design-patterns/chain-of-responsibility) has already been written for Chain of Responsibility. I highly recommend Refactoring.Guru for their other articles too.

## My Two Problems

I've got a backend written in Java that takes in a username and password. This backend authenticates users and returns a token.

I was happy to get it done and it was easy to implement. Unfortunately my happiness was short-lived. Because my site is now under **attack** by hackers.

If that wasn't bad enough, I forgot to pay the bill for the VPS hosting my database. Now they've downgraded my service.

## The Solutions

Here's what I've got to do: 

1. Prevent the usernames of known hackers from logging in.
2. Cache authenticated usernames for less DB reads.

## The POJOs

The POJO that stores the authentication request:

```java
public class UserAuthenticationRequest {

	private String username;
	private String password;
	
	public UserAuthenticationRequest(String username, String password) {
		this.username = username;
		this.password = password;
	}

	public String getUsername() {
		return username;
	}

	public String getPassword() {
		return password;
	}
}
```

The POJO that stores the authentication response:

```java
public class UserAuthenticationResult {
	private String token;
	private String username;
	private boolean authenticated;
	
	public String getUsername() {
		return username;
	}
	public void setUsername(String username) {
		this.username = username;
	}
	public String getToken() {
		return token;
	}
	public void setToken(String token) {
		this.token = token;
	}
	public boolean isAuthenticated() {
		return authenticated;
	}
	public void setAuthenticated(boolean authenticated) {
		this.authenticated = authenticated;
	}
	
	@Override
	public String toString()
	{
		return "Username: " + username 
				+ "\nToken: " + token
				+ "\nIsAuthenticated: " + authenticated;
	}
}

```

## Chain of Responsibility Implementation

The Chain of Responsibility pattern first needs an abstract `Handler` class. My other Handlers will extend from this class.

```java
public abstract class UserAuthenticationHandler {
	private UserAuthenticationHandler next;
	
	public UserAuthenticationHandler(UserAuthenticationHandler next)
	{
		this.next = next;
	}
	
	public UserAuthenticationResult handleUserCredentials(UserAuthenticationRequest request)
	{
		return next.handleUserCredentials(request);
	}
}
```

Should do the trick.

Let's write some code to deal with those pesky hackers. The handler should return a UserAuthenticationResult with authenticate set to `false` if they are up to no good.

```java
public class UsernameBannedHandler extends UserAuthenticationHandler {
	private List<String> bannedUsernames;
	
	public UsernameBannedHandler(UserAuthenticationHandler next) {
		super(next);
		
		bannedUsernames = List.of("jakehell", "bannedUser", "jakehell2");
	}
	
	@Override
	public UserAuthenticationResult handleUserCredentials(UserAuthenticationRequest request)
	{
		if(isUsernameBanned(request))
		{
			return getAuthenticationFailureResult(request);
		}
		
		return super.handleUserCredentials(request);
	}
	
	private boolean isUsernameBanned(UserAuthenticationRequest request)
	{
		return bannedUsernames.contains(request.getUsername());
	}
	
	private UserAuthenticationResult getAuthenticationFailureResult(UserAuthenticationRequest request)
	{
		UserAuthenticationResult result = new UserAuthenticationResult();
		result.setUsername(request.getUsername());
		result.setAuthenticated(false);
		
		return result;
	}
}
```

That takes care of my first problem.

Now to check if the username is cached. This handler should return the cached username and token.

```java
public class UserCachedHandler extends UserAuthenticationHandler {
	private Map<String, String> usernameToken;
	
	public UserCachedHandler(UserAuthenticationHandler next) {
		super(next);
		
		usernameToken = Map.of("cachedUser", "cachedUserToken", "jakehell4", "token5");
	}

	@Override
	public UserAuthenticationResult handleUserCredentials(UserAuthenticationRequest request)
	{
		String username = request.getUsername();
		if(usernameToken.containsKey(username))
		{
			return getSuccessfulUserAuthenticationResult(username);
		}
		
		return super.handleUserCredentials(request);
	}
	
	public UserAuthenticationResult getSuccessfulUserAuthenticationResult(String username)
	{
		UserAuthenticationResult result = new UserAuthenticationResult();
		result.setUsername(username);
		result.setToken(usernameToken.get(username));
		result.setAuthenticated(true);
		
		return result;
	}
}
```

Lastly, I need to actually try to authenticate the user.

```java
public class AuthenticateUserHandler extends UserAuthenticationHandler {

	public AuthenticateUserHandler(UserAuthenticationHandler next) {
		super(next);
	}
	
	@Override
	public UserAuthenticationResult handleUserCredentials(UserAuthenticationRequest request)
	{
		if(authenticateUser(request))
		{
			return createSuccessfulResult(request);
		}
		
		return super.handleUserCredentials(request);
	}
	
	private boolean authenticateUser(UserAuthenticationRequest request)
	{
		if(request.getUsername().equals("authenticatedUser") && request.getPassword().equals("password"))
		{
			return true;
		}

		return false;
	}
	
	private UserAuthenticationResult createSuccessfulResult(UserAuthenticationRequest request)
	{
		UserAuthenticationResult result = new UserAuthenticationResult();
		result.setAuthenticated(true);
		result.setUsername(request.getUsername());
		result.setToken(generateToken());
		
		return result;
	}
	
	private String generateToken()
	{
		return "token";
	}
}
```

For some reason, only `authenticatedUser` can login. Oh well, different problem for a different day.

## Creating the Chain

Here is the chain:

`UsernameBannedHandler -> UserCachedHandler -> UserAuthenticationHandler`

I also put `ForceAuthenticationFailureHandler` after `UserAuthenticationHandler`. Since authentication failed if it got to that handler.

```java
public static void main(String[] args) {
		ForceAuthenticationFailureHandler forceAuthenticationFailureHandler = new ForceAuthenticationFailureHandler(null);
		AuthenticateUserHandler authenticateUserHandler = new AuthenticateUserHandler(forceAuthenticationFailureHandler);
		UserCachedHandler userCachedHandler = new UserCachedHandler(authenticateUserHandler);
		UsernameBannedHandler usernameBannedHandler = new UsernameBannedHandler(userCachedHandler);
		
		UserAuthenticationRequest banned = new UserAuthenticationRequest("bannedUser", "password");
		UserAuthenticationRequest cached = new UserAuthenticationRequest("cachedUser", "password");
		UserAuthenticationRequest authenticated = new UserAuthenticationRequest("authenticatedUser", "password");
		UserAuthenticationRequest failure = new UserAuthenticationRequest("authenticateFailure", "password");
		
		UserAuthenticationResult bannedResult = usernameBannedHandler.handleUserCredentials(banned);
		UserAuthenticationResult cachedResult = usernameBannedHandler.handleUserCredentials(cached);
		UserAuthenticationResult authenticatedResult = usernameBannedHandler.handleUserCredentials(authenticated);
		UserAuthenticationResult failureResult = usernameBannedHandler.handleUserCredentials(failure);
		
		System.out.println(bannedResult.toString());
		System.out.println();
		System.out.println(cachedResult.toString());
		System.out.println();
		System.out.println(authenticatedResult.toString());
		System.out.println();
		System.out.println(failureResult.toString());
	}
```

Output:

```
Username: bannedUser
Token: null
IsAuthenticated: false

Username: cachedUser
Token: cachedUserToken
IsAuthenticated: true

Username: authenticatedUser
Token: token
IsAuthenticated: true

Username: authenticateFailure
Token: null
IsAuthenticated: false
```

## Conclusion

[Here](https://github.com/jakehell/Chain-of-Responsibility-Example) is the source of this code on GitHub.

Want to brag to your friends that you learned about the Chain of Responsibility?

Please share on your favorite social media site:

<!-- Sharingbutton Twitter -->
<a class="resp-sharing-button__link" href="https://twitter.com/intent/tweet/?text=The%20Chain%20of%20Responsibility%20Pattern%20Makes%20Hard%20Validation%20Easy&amp;url=http%3A%2F%2Fjakehell.github.io%2Fpost%2Fintro-chain-of-responsibility%2F" target="_blank" rel="noopener" aria-label="">
  <div class="resp-sharing-button resp-sharing-button--twitter resp-sharing-button--small"><div aria-hidden="true" class="resp-sharing-button__icon resp-sharing-button__icon--solidcircle">
    <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24"><path d="M12 0C5.38 0 0 5.38 0 12s5.38 12 12 12 12-5.38 12-12S18.62 0 12 0zm5.26 9.38v.34c0 3.48-2.64 7.5-7.48 7.5-1.48 0-2.87-.44-4.03-1.2 1.37.17 2.77-.2 3.9-1.08-1.16-.02-2.13-.78-2.46-1.83.38.1.8.07 1.17-.03-1.2-.24-2.1-1.3-2.1-2.58v-.05c.35.2.75.32 1.18.33-.7-.47-1.17-1.28-1.17-2.2 0-.47.13-.92.36-1.3C7.94 8.85 9.88 9.9 12.06 10c-.04-.2-.06-.4-.06-.6 0-1.46 1.18-2.63 2.63-2.63.76 0 1.44.3 1.92.82.6-.12 1.95-.27 1.95-.27-.35.53-.72 1.66-1.24 2.04z"/></svg>
    </div>
  </div>
</a>

<!-- Sharingbutton Reddit -->
<a class="resp-sharing-button__link" href="https://reddit.com/submit/?url=http%3A%2F%2Fjakehell.github.io%2Fpost%2Fintro-chain-of-responsibility%2F&amp;resubmit=true&amp;title=The%20Chain%20of%20Responsibility%20Pattern%20Makes%20Hard%20Validation%20Easy" target="_blank" rel="noopener" aria-label="">
  <div class="resp-sharing-button resp-sharing-button--reddit resp-sharing-button--small"><div aria-hidden="true" class="resp-sharing-button__icon resp-sharing-button__icon--solidcircle">
    <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24"><circle cx="9.391" cy="13.392" r=".978"/><path d="M14.057 15.814c-1.14.66-2.987.655-4.122-.004-.238-.138-.545-.058-.684.182-.13.24-.05.545.19.685.72.417 1.63.646 2.568.646.93 0 1.84-.228 2.558-.642.24-.13.32-.44.185-.68-.14-.24-.445-.32-.683-.18zM5 12.086c0 .41.23.78.568.978.27-.662.735-1.264 1.353-1.774-.2-.207-.48-.334-.79-.334-.62 0-1.13.507-1.13 1.13z"/><path d="M12 0C5.383 0 0 5.383 0 12s5.383 12 12 12 12-5.383 12-12S18.617 0 12 0zm6.673 14.055c.01.104.022.208.022.314 0 2.61-3.004 4.73-6.695 4.73s-6.695-2.126-6.695-4.74c0-.105.013-.21.022-.313C4.537 13.73 4 12.97 4 12.08c0-1.173.956-2.13 2.13-2.13.63 0 1.218.29 1.618.757 1.04-.607 2.345-.99 3.77-1.063.057-.803.308-2.33 1.388-2.95.633-.366 1.417-.323 2.322.085.302-.81 1.076-1.397 1.99-1.397 1.174 0 2.13.96 2.13 2.13 0 1.177-.956 2.133-2.13 2.133-1.065 0-1.942-.79-2.098-1.81-.734-.4-1.315-.506-1.716-.276-.6.346-.818 1.395-.88 2.087 1.407.08 2.697.46 3.728 1.065.4-.468.987-.756 1.617-.756 1.17 0 2.13.953 2.13 2.13 0 .89-.54 1.65-1.33 1.97z"/><circle cx="14.609" cy="13.391" r=".978"/><path d="M17.87 10.956c-.302 0-.583.128-.79.334.616.51 1.082 1.112 1.352 1.774.34-.197.568-.566.568-.978 0-.623-.507-1.13-1.13-1.13z"/></svg>
    </div>
  </div>
</a>

<!-- Sharingbutton Hacker News -->
<a class="resp-sharing-button__link" href="https://news.ycombinator.com/submitlink?u=http%3A%2F%2Fjakehell.github.io%2Fpost%2Fintro-chain-of-responsibility%2F&amp;t=The%20Chain%20of%20Responsibility%20Pattern%20Makes%20Hard%20Validation%20Easy" target="_blank" rel="noopener" aria-label="">
  <div class="resp-sharing-button resp-sharing-button--hackernews resp-sharing-button--small"><div aria-hidden="true" class="resp-sharing-button__icon resp-sharing-button__icon--solidcircle">
    <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 256 256"><path fill-rule="evenodd" d="M128 256c70.692 0 128-57.308 128-128C256 57.308 198.692 0 128 0 57.308 0 0 57.308 0 128c0 70.692 57.308 128 128 128zm-9.06-113.686L75 60h20.08l25.85 52.093c.397.927.86 1.888 1.39 2.883.53.994.995 2.02 1.393 3.08.265.4.463.764.596 1.095.13.334.262.63.395.898.662 1.325 1.26 2.618 1.79 3.877.53 1.26.993 2.42 1.39 3.48 1.06-2.254 2.22-4.673 3.48-7.258 1.26-2.585 2.552-5.27 3.877-8.052L161.49 60h18.69l-44.34 83.308v53.087h-16.9v-54.08z"/></svg>
    </div>
  </div>
</a>
