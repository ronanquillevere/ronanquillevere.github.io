---
layout: post
title:  "RestyGWT and basic authentication"
date:   2014-04-11 23:59:00
Tags: [GWT,  RestyGWT, basic Auth, Authentication, security]
Categories: [GWT]
---

Hello,

In this article I will be talking about how to secure you REST application using RestyGWT and basic authentication. 

What I will do in this article : 

* Provides some advices and code snippets to use basic authentication with RestyGWT for authenticated calls. 

What I will not do : 

* Discuss wether you should or should not use basic authentication.
* Gives you advices on how to handle basic auth on the backend side.
* Explain a sign-in scenario.

# HTTPS

[Basic authentication](http://technet.microsoft.com/en-us/library/cc784037.aspx) only implies that you will encode the username and the password of your user using a [base64](http://en.wikipedia.org/wiki/Base64) encoding. So everybody sniffing the network will be able to decrypt the login/password of your user. So it is mandatory to setup HTTPS.

Of course HTTPS comes with drawbacks espcially for caching. But this is not a discussion we are going to have in this article. Setting up HTTPS is pretty straightforward with tomcat. You can have a look [here](https://tomcat.apache.org/tomcat-6.0-doc/ssl-howto.html) for more information on the subject.


# Basic Authentication Header

As stated by [wikipedia](http://en.wikipedia.org/wiki/Basic_access_authentication) following the [Basic Authentication Scheme](http://tools.ietf.org/html/rfc1945#section-11.1)

When the user agent wants to send the server authentication credentials it may use the Authorization header

The Authorization header is constructed as follows:

* Username and password are combined into a string "username:password"
* The resulting string literal is then encoded using the RFC2045-MIME variant of Base64, except not limited to 76 char/line
* The authorization method and a space i.e. "Basic " is then put before the encoded string.

For example, if the user agent uses 'Aladdin' as the username and 'open sesame' as the password then the header is formed as follows:.

    Authorization: Basic QWxhZGRpbjpvcGVuIHNlc2FtZQ==

# Storing the credentials

Here I am supposing that your GWT application is a one page application using RestyGWT to send your AJAX requests. On the backend side, for every request, your server will check the validity of the username and password and will return a 2XX answer if the login/password provided are correct or a 4XX answer if they are not. So for every request you will need add the basic authentication header. 

In your application you will probably have a login activity/page where your user will enter his username and password. You need to save the login and password of your user inside your application to be able to add them for evey request (consider using a hash instead of the real password).

To do so you can create this kind of singleton class

{% highlight java %}
public enum UserCredentials {
	
	INSTANCE ("anonymous", null);
	
	private String userName;
	private String password;

	private UserCredentials(String userName, String password) {
		this.setUserName(userName);
		this.setPassword(password);
	}

	public String getUserName() {
		return userName;
	}

	public String getPassword() {
		return password;
	}

	public void setUserName(String userName) {
		this.userName = userName;
	}

	public void setPassword(String password) {
		this.password = password;
	}
}
{% endhighlight %}


## RestyGWT Dispatcher

RestyGWT comes with a `Dispatcher` interface. 

{% highlight java %}
public interface Dispatcher {

    public Request send(Method method, RequestBuilder builder) throws RequestException;

}
{% endhighlight %}

This will enable you to intercept your request before it is sent to the backend (and even cancel it if needed).

I suggest to use the `FilterawareDispatcher` that will enable you to add some more behavior afterward.

{% highlight java %}
public interface FilterawareDispatcher extends Dispatcher {

    public void addFilter(DispatcherFilter filter);

}
{% endhighlight %}


Let's create a global dispatcher for all my calls. I will then create a filter responsible for creating the basic authentication header.

{% highlight java %}
public class MyDispatcher extends DefaultFilterawareDispatcher {
	
	public MyDispatcher() {
		addFilter(new BasicAuthHeaderDispatcherFilter());
	}
}
{% endhighlight %}

# Basic Authentication Filter

To encode the userName anmd password in Base64 I uses [gwt-crypto](https://code.google.com/p/gwt-crypto/).

{% highlight xml %}
<inherits name="com.googlecode.gwt.crypto.Crypto"/>
{% endhighlight %} 

{% highlight xml %}
<dependencies>
    <dependency>
        <groupId>com.googlecode.gwt-crypto</groupId>
        <artifactId>gwt-crypto</artifactId>
        <version>2.3.0</version>
    </dependency>
</dependencies>
{% endhighlight %}



{% highlight java %}
final class BasicAuthHeaderDispatcherFilter implements DispatcherFilter {

	public static final String AUTHORIZATION_HEADER = "Authorization";

	public boolean filter(Method method, RequestBuilder builder) {
		try {

			String basicAuthHeaderValue = createBasicAuthHeader(
					UserCredentials.INSTANCE.getUserName(),
					UserCredentials.INSTANCE.getPassword());
			
			builder.setHeader(AUTHORIZATION_HEADER, basicAuthHeaderValue);
			
		} catch (UnsupportedEncodingException e) {
			return false;
		}
		return true;
	}

	private String createBasicAuthHeader(String userName, String password)
			throws UnsupportedEncodingException {
		String credentials = userName + ":" + password;
		String encodedCredentials = new String(Base64.encode(credentials
				.getBytes()), "UTF-8");

		return AUTHORIZATION_HEADER + ": Basic " + encodedCredentials;
	}
}
{% endhighlight %}

# Setting up the dispatcher


{% highlight java %}
Defaults.setDispatcher(new MyDispatcher());
	  
UserCredentials.INSTANCE.setUserName("ronan");
UserCredentials.INSTANCE.setPassword("password");
{% endhighlight %}

If you inspect your query in chrome you should now see a line like this, inside your request headers

    Authorization:Authorization: Basic cm9uYW46cGFzc3dvcmQ=

That's it, now your backend will look into this header (you can use a servlet filter to do so) for every request and send back a 4xx answer depending on the level of authorization your user have.

Hope it helped !

