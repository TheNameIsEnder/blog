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
