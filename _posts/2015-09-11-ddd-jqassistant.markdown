---
layout: post
title:  "DDD Structural validation using JQAssistant and Maven " 
date:   2015-09-10 23:59:00
Tags: [maven, java, jqassistant]
Categories: [DDD]
---

Hello,


If you are familiar with [Domain Driven Design (DDD)](https://en.wikipedia.org/wiki/Domain-driven_design) you might have already tried to validate the structure of your application during your build process. By validating I mean making sure that no dependency was introduced between layers of your application that should not access eachothers.

In the following article I will show you how to do that using [JQAssistant](http://www.jqassistant.org) and maven for a Java application.

#DDD structure

Typicaly if you are using DDD you have defined layers inside your backend code. Those layers can vary a little bit between intrepretation of the DDD concept, as dependencies between layers.

In my case here are the layers of my application backend. Arrows represent authorized dependencies between layers. For example Interfaces depend on application. It means you can use application objects inside an object of the interfaces layer.

<img class="center" src="/img/dddlayers.jpg" alt="ddd layers" width="400px">


#JQAssistant

JQAssistant typical flow is composed of 2 steps : scanning and analysing. 

During the scanning phase JQAssistant is able to flag your files with some labels. It uses [Neo4j](http://www.neo4j.com) as the underlying database to save those information. Neo4j is a graph database. So each of your file is in fact seen as a Node in the Neo4j database. It is also able to create relationships between those nodes. Typically JQAssistant is able to scan your classes files and create dependency relationships between them. This is the relationship we will use to validate our DDD structure.

## pom.xml

JQAssistant can be used inside maven (hopefully for the poor JAVA developers we are). Here is an extract of my pom.xml

Then important thing to note are the groups. Inside each group, you will be able to define certain rules to be tested on your scanned files.

{% highlight xml %}
<plugin>
  <groupId>com.buschmais.jqassistant.scm</groupId>
  <artifactId>jqassistant-maven-plugin</artifactId>
  <version>${jqassistant.version}</version>
  <executions>
    <execution>
      <goals>
        <goal>scan</goal>
        <goal>analyze</goal>
      </goals>
      <configuration>
        <failOnViolations>true</failOnViolations>
        <groups>
          <group>default</group>
          <group>ddd</group>
        </groups>
      </configuration>
    </execution>
  </executions>
</plugin>
{% endhighlight %}


### rules.xml

Now is the time to define our rules. They should be defined inside an xml file inside the JQAssistant directory that should be located under your app directory (next to your pom.xml).

### DDD concepts

The first thing we want to do is to flag our classes depending on the layer they belong to. This is pretty easy, we just need to create a concept. Here is the concept that will add the flag `DDD_Interfaces` to my class if its package is part of the packages `interfaces`.

{% highlight xml %}
<concept id="ddd:Interfaces" severity="blocker">
  <description>Interfaces layer of DDD architecture</description>
  <cypher><![CDATA[
      MATCH (t:Type) WHERE t.fqn =~ ".*interfaces.*" SET t:DDD_Interfaces return t
  ]]></cypher>
</concept>
{% endhighlight %}

We create the other concepts for the application, domain and infrastructure layers.

### Constraints

Constraints are the rules that will be tested on your files (nodes).

Again it is quite easy. Here is the constraint that checks if some classes inside the interfaces layer (flagged as DDD_Interfaces thanks to our previous concept) depends on classes inside the domain layer. If we find some, it means our ddd structure has been violated. Because we set the flag `failOnViolations` to true inside our `pom.xml` the maven phase will fail and your maven build will fail as well.

{% highlight xml %}
<constraint id="ddd:TestInterfacesDomain" severity="blocker">
  <requiresConcept refId="ddd:Interfaces" />
  <requiresConcept refId="ddd:Domain" />
  <description>No class from Interfaces can have a dependency on Domain</description>
  <cypher><![CDATA[
      MATCH
          (t:Type:DDD_Interfaces)-[DEPENDS_ON]->(f:Type:DDD_Domain)
      RETURN
          t AS DDD_InvalidDependency
  ]]></cypher>
</constraint>
{% endhighlight %}


##Sum Up

Here is my final `rules.xml` if you are too lazy to write it yourself :) Note that I have another rule part of the default group. It is just to illustrate the notion of group.

It is probably possible to use only one constraint to test everything. But I had no time to try to improve my rule.xml. It also makes it more readable for a beginner like me in cypher.


{% highlight xml %}
<jqa:jqassistant-rules xmlns:jqa="http://www.buschmais.com/jqassistant/core/analysis/rules/schema/v1.0">

    <concept id="ddd:Interfaces" severity="blocker">
        <description>Interfaces layer of DDD architecture</description>
        <cypher><![CDATA[
            MATCH (t:Type) WHERE t.fqn =~ ".*interfaces.*" SET t:DDD_Interfaces return t
        ]]></cypher>
    </concept>
    
    <concept id="ddd:Application" severity="blocker">
        <description>Application layer of DDD architecture</description>
        <cypher><![CDATA[
            MATCH (t:Type) WHERE t.fqn =~ ".*application.*" SET t:DDD_Application return t
        ]]></cypher>
    </concept>
    
    <concept id="ddd:Domain" severity="blocker">
        <description>Domain layer of DDD architecture</description>
        <cypher><![CDATA[
            MATCH (t:Type) WHERE t.fqn =~ ".*domain.*" SET t:DDD_Domain RETURN t
        ]]></cypher>
    </concept>
    
    <concept id="ddd:Infrastructure" severity="blocker">
        <description>Infrastructure layer of DDD architecture</description>
        <cypher><![CDATA[
            MATCH (t:Type) WHERE t.fqn =~ ".*infrastructure.*" SET t:DDD_Infrastructure RETURN t
        ]]></cypher>
    </concept>

    <constraint id="ddd:TestInterfacesDomain" severity="blocker">
        <requiresConcept refId="ddd:Interfaces" />
        <requiresConcept refId="ddd:Domain" />
        <description>No class from Interfaces can have a dependency on Domain</description>
        <cypher><![CDATA[
            MATCH
                (t:Type:DDD_Interfaces)-[DEPENDS_ON]->(f:Type:DDD_Domain)
            RETURN
                t AS DDD_InvalidDependency
        ]]></cypher>
    </constraint>
    
    <constraint id="ddd:TestInterfacesInfrastructure" severity="blocker">
        <requiresConcept refId="ddd:Interfaces" />
        <requiresConcept refId="ddd:Infrastructure" />
        <description>No class from Interfaces can have a dependency on Infrastructure</description>
        <cypher><![CDATA[
            MATCH
                (t:Type:DDD_Interfaces)-[DEPENDS_ON]->(f:Type:DDD_Infrastructure)
            RETURN
                t AS DDD_InvalidDependency
        ]]></cypher>
    </constraint>
    
    
    
    <constraint id="ddd:TestApplicationInterfaces" severity="blocker">
        <requiresConcept refId="ddd:Application" />
        <requiresConcept refId="ddd:Interfaces" />
        <description>No class from Application can have a dependency on Interfaces</description>
        <cypher><![CDATA[
            MATCH
                (t:Type:DDD_Application)-[DEPENDS_ON]->(f:Type:DDD_Interfaces)
            RETURN
                t AS DDD_InvalidDependency
        ]]></cypher>
    </constraint>
    
    <constraint id="ddd:TestApplicationInfrastructure" severity="blocker">
        <requiresConcept refId="ddd:Application" />
        <requiresConcept refId="ddd:Infrastructure" />
        <description>No class from Application can have a dependency on Infrastructure</description>
        <cypher><![CDATA[
            MATCH
                (t:Type:DDD_Application)-[DEPENDS_ON]->(f:Type:DDD_Infrastructure)
            RETURN
                t AS DDD_InvalidDependency
        ]]></cypher>
    </constraint>
    
    
    
    <constraint id="ddd:TestDomainApplication" severity="blocker">
        <requiresConcept refId="ddd:Domain" />
        <requiresConcept refId="ddd:Application" />
        <description>No class from Domain can have a dependency on Application</description>
        <cypher><![CDATA[
            MATCH
                (t:Type:DDD_Domain)-[DEPENDS_ON]->(f:Type:DDD_Application)
            RETURN
                t AS DDD_InvalidDependency
        ]]></cypher>
    </constraint>
    
    <constraint id="ddd:TestDomainInterfaces" severity="blocker">
        <requiresConcept refId="ddd:Domain" />
        <requiresConcept refId="ddd:Interfaces" />
        <description>No class from Domain can have a dependency on Interfaces</description>
        <cypher><![CDATA[
            MATCH
                (t:Type:DDD_Domain)-[DEPENDS_ON]->(f:Type:DDD_Interfaces)
            RETURN
                t AS DDD_InvalidDependency
        ]]></cypher>
    </constraint>
    
     <constraint id="ddd:TestDomainInfrastructure" severity="blocker">
        <requiresConcept refId="ddd:Domain" />
        <requiresConcept refId="ddd:Infrastructure" />
        <description>No class from Domain can have a dependency on Infrastructure</description>
        <cypher><![CDATA[
            MATCH
                (t:Type:DDD_Domain)-[DEPENDS_ON]->(f:Type:DDD_Infrastructure)
            RETURN
                t AS DDD_InvalidDependency
        ]]></cypher>
    </constraint>
    
    
    
    <constraint id="ddd:TestInfrastructureApplication" severity="blocker">
        <requiresConcept refId="ddd:Infrastructure" />
        <requiresConcept refId="ddd:Application" />
        <description>No class from Infrastructure can have a dependency on Application</description>
        <cypher><![CDATA[
            MATCH
                (t:Type:DDD_Infrastructure)-[DEPENDS_ON]->(f:Type:DDD_Application)
            RETURN
                t AS DDD_InvalidDependency
        ]]></cypher>
    </constraint>
    
    <constraint id="ddd:TestInfrastructureInterfaces" severity="blocker">
        <requiresConcept refId="ddd:Infrastructure" />
        <requiresConcept refId="ddd:Interfaces" />
        <description>No class from Infrastructure can have a dependency on Interfaces</description>
        <cypher><![CDATA[
            MATCH
                (t:Type:DDD_Infrastructure)-[DEPENDS_ON]->(f:Type:DDD_Interfaces)
            RETURN
                t AS DDD_InvalidDependency
        ]]></cypher>
    </constraint>
    
    
     <constraint id="my-rules:TestClassName">
        <requiresConcept refId="junit4:TestClass" />
        <description>All JUnit test classes must start with "Test" or "IT".</description>
        <cypher><![CDATA[
            MATCH
                (t:Junit4:Test:Class)
            WHERE NOT
                t.name =~ "(Test|IT).*"
            RETURN
                t AS InvalidTestClass
        ]]></cypher>
    </constraint>
    
  <group id="ddd">
        <includeConstraint refId="ddd:TestInterfacesDomain" />
        <includeConstraint refId="ddd:TestInterfacesInfrastructure" />
        
        <includeConstraint refId="ddd:TestApplicationInterfaces" />
        <includeConstraint refId="ddd:TestApplicationInfrastructure" />
        
        <includeConstraint refId="ddd:TestDomainInterfaces" />
        <includeConstraint refId="ddd:TestDomainApplication" />
        <includeConstraint refId="ddd:TestDomainInfrastructure" />
        
        <includeConstraint refId="ddd:TestInfrastructureInterfaces" />
        <includeConstraint refId="ddd:TestInfrastructureApplication" />
    </group>
    
    <group id="default">
        <includeConstraint refId="my-rules:TestClassName" />
    </group>

</jqa:jqassistant-rules>
{% endhighlight %}


Hope it will help !