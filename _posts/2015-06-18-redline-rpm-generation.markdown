---
layout: post
title:  "Generate rpm with redline (java) and maven" 
date:   2015-06-18 23:59:00
Tags: [rpm,  redline]
Categories: [redline]
---

A quick post as a simple tutorial to generate a rpm using [redline](http://redline-rpm.org/) and maven.


#Main

Simply create a Main class, inside the main method here is some basic info

{% highlight java %}
public static void main(String[] args) throws NoSuchAlgorithmException, IOException, URISyntaxException
{
	org.redline_rpm.Builder builder = new Builder();

	File directory = new File(".");
	builder.setType(RpmType.BINARY);
	builder.setPlatform(Architecture.X86_64, Os.LINUX);
	builder.setPackage("name", "1", "1");
	builder.setDescription("Description");
	builder.setSummary("Summary");

	builder.build(directory);
}
{% endhighlight %}

#pom.xml

Specify jar for packaging. Then add the dependencies. In your pom you will need redline, slf4j and a logger.

{% highlight xml %}
<dependency>
	<groupId>org.redline-rpm</groupId>
	<artifactId>redline</artifactId>
	<version>1.2.1</version>
</dependency>
<dependency>
	<groupId>org.slf4j</groupId>
	<artifactId>slf4j-api</artifactId>
	<version>1.7.12</version>
</dependency>
<dependency>
	<groupId>ch.qos.logback</groupId>
	<artifactId>logback-classic</artifactId>
	<version>1.1.3</version>
</dependency>
{% endhighlight %}

To build an executable jar add the following plugin to your pom. Note the `jar-with-dependencies` that will create a second jar with all the dependencies so that it will be easier to execute the jar

{% highlight xml %}
<plugin>
	<artifactId>maven-assembly-plugin</artifactId>
	<configuration>
		<archive>
			<manifest>
				<mainClass>put.your.package.and.yourMainCLass</mainClass>
			</manifest>
		</archive>
		<descriptorRefs>
			<descriptorRef>jar-with-dependencies</descriptorRef>
		</descriptorRefs>
	</configuration>
	<executions>
		<execution>
			<phase>package</phase>
			<goals>
				<goal>single</goal>
			</goals>
		</execution>
	</executions>
</plugin>
{% endhighlight %}

Run `mvn clean install` this will create your jar in your target repository.

Go to the target directory.

#Generate the rpm

Launch

`java -jar yourArtefact-with-dependencies.jar`

This will create the rpm

Then to install the rpm (on linux), you may have to use sudo ...

`rpm -Uvh yourrpm.rpm`

Now you can go on by adding some preInstall scripts for example !
