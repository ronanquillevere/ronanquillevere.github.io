---
layout: post
title:  "GWT and Rest : Part 1 - Server Side (Jersey JAX-RS)"
date:   2013-03-24 20:19:00
Tags: [Google Web Toolkit, GWT, JAX-RS, Jersey, REST, Restful, restyGWT, service, web]
Categories: [gwt]
---

Hello everyone,

In this article I will talk about  [REST](http://en.wikipedia.org/wiki/Representational_state_transfer) and how to create a REST application using GWT (and  [RestyGWT](http://restygwt.fusesource.org/)) for the client and  [JAX-RS](http://en.wikipedia.org/wiki/Java_API_for_RESTful_Web_Services) (with  [Jersey](http://jersey.java.net/)) for the server.

This part 1 is focused on the server side.

Part 2 will focus on the client side with GWT and RestyGWT :[GWT and Rest :  Part 2 - Client (RestyGWT)](/2013/07/03/gwt-rest-jersey-client.html)

##Why REST ?


In GWT there are many ways to make your client and your server talk to each other: [GWT-RPC](https://developers.google.com/web-toolkit/doc/latest/tutorial/RPC), [Request Factory](https://developers.google.com/web-toolkit/doc/latest/DevGuideRequestFactory), standard servlet mechanism, REST... Which one should I use ?

GWT-RPC is probably the most common way to do it and quite an easy way to start with. Nevertheless it is not an "open" solution. By "open" I mean that if you want to write a new client in php or in another language, you will have to modify your backend communication layer.

As the futur of GWT is not clear today, I do not want to have to re-write my backend code and architecture if I decide to make a new version of my client in php. I also want third parties to be able to build thier own solutions based on my API.

That's why Facebook, LinkedIn https://developer.linkedin.com/rest etc. are building web APIs based on rest principles.

I strongly recommend to read [Crafting interfaces that developers love](http://info.apigee.com/Portals/62317/docs/web api.pdf) by Brian Mulloy if you want to build a nice REST API.

##Rest Backend


For the following you will need the following dependencies

jersey-server
jersey-servlet
jersey-json
jersey-guice

guice
guice-servlet


To build my rest backend I will use Jersey which is the Reference Implementation for building RESTful Web services on a Java backend. I will also use [Guice](https://code.google.com/p/google-guice/) to handle the dependency injection.

In the rest approach you will ask your server for resources. A resource can be a book for example.

{% highlight java%}
public class Book
{

 public Book(String isbn, String author)
 {
  this.isbn = isbn;
  this.author = author;
 }

 public String getAuthor()
 {
  return author;
 }

 public void setAuthor(String author)
 {
  this.author = author;
 }

 public String getIsbn()
 {
  return isbn;
 }

 public void setIsbn(String isbn)
 {
  this.isbn = isbn;
 }

 private String isbn;
 private String author;
}
{% endhighlight %}


When asking you server for a book, for example the book whose isbn is 1,  you will probably make a request that will look like this :

[/api/v1/books/1](/api/v1/books/1)

On your server you must declare your books resource class. Here is what it should look like. Our resource will produce json output and will consume json (if we want to pass parameters in the payload). In this example I am creating books directly in the BookResource class but of course objects should come from a repository...

{% highlight java%}
@Singleton
@Path("v1/books")
@Produces(MediaType.APPLICATION_JSON)
@Consumes(MediaType.APPLICATION_JSON)
public class BookResource
{
    public BookResource()
    {
         books= new HashMap<String, Book>();
         Book book1 = new Book("1","Brian");
         books.put("1", book1);

         Book book2 = new Book("2","David");
         books.put("2", book2);
    }

    //example : http://127.0.0.1:8888/api/v1/books
    @GET
    public Collection<Book> getBooks()
    {
        return books.values();
    }
 
    //example : http://127.0.0.1:8888/api/v1/books/1
    @GET
    @Path("{isbn}")
    public Book getPage(@PathParam("isbn") String isbn)
    {
        return books.get(isbn);
    }

    private HashMap<String, Book> books;
}
{% endhighlight %}

Now you should declare your resource class in your servlet container and tell your server which url should be mapped to this class. To do so I will use a filter in my web.xml and map it to a GuiceServletContextListener.

{% highlight xml%}
<filter>
  <filter-name>guiceFilter</filter-name>
  <filter-class>com.google.inject.servlet.GuiceFilter</filter-class>
 </filter>

 <filter-mapping>
  <filter-name>guiceFilter</filter-name>
  <url-pattern>/*</url-pattern>
 </filter-mapping>

 <listener>
  <listener-class>com.rq.resty.server.GuiceServletConfig</listener-class>
 </listener>
{% endhighlight %}


Below is the code of the filter. The REST_INTERFACE_PACKAGE should point to your server side package where the resource class are declared. You can see below that every query starting by /api/ will be treated by the GuiceContainer. It will be responsible for deploying root resource classes with Guice integration.

Also we will use the POJO mapping feature to serialize our objects without having to create a specific serializer.


{% highlight java%}
public class GuiceServletConfig extends GuiceServletContextListener
{
    @Override
    protected Injector getInjector()
    {
        final Map<String, String> params = new HashMap<String, String>();

        params.put(JERSEY_CONFIG_PROPERTY_PACKAGES, REST_INTERFACE_PACKAGE);
        params.put(JERSEY_API_JSON_POJO_MAPPING_FEATURE, "true");

        return Guice.createInjector(
            new ServletModule()
            {
                @Override
                protected void configureServlets()
                {
                    serve("/api/*").with(GuiceContainer.class, params);      
                }
            }
        );
    }

private static final String JERSEY_CONFIG_PROPERTY_PACKAGES = "com.sun.jersey.config.property.packages";
private static final String JERSEY_API_JSON_POJO_MAPPING_FEATURE = "com.sun.jersey.api.json.POJOMappingFeature";
private static final String REST_INTERFACE_PACKAGE = "com.rq.resty.server";
}
{% endhighlight %}

If everything is working you should be able to call your server and get all your books like this :

[http://127.0.0.1:8888/api/v1/books](http://127.0.0.1:8888/api/v1/books)

Your server should send you back the following json :
{% highlight json%}
[{"isbn":"2","author":"David"},{"isbn":"1","author":"Brian"}]
{% endhighlight %}

Or you can ask directly for a particular book if you know its isbn

[http://127.0.0.1:8888/api/v1/books/1](http://127.0.0.1:8888/api/v1/books/1)

Your server should send you back the following json :
{% highlight json%}
{"isbn":"1","author":"Brian"}
{% endhighlight %}

Check out this article if you do not want to use a Guice filter to declare your resource classes : [gwt-rest](http://blog.javaforge.net/post/30469901979/gwt-rest)
