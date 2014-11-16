---
layout: post
title: "How to: Deploy to weblogic using maven"
comments: false
categories: 
- How to 
- Weblogic
- Maven
---

<p>
	I have always liked Maven due to its simplicity and being so powerful. One of the recent requirement needed deploying to a remote Weblogic server. Since I was using maven for dependency management and all other purposes maven is well known for, I decided to use it for remote deployment as well.
	Following are the steps one need to follow to make this tick.
</p>
<h5> Step 1: Getting prerequisites ready</h5>
<p>
	1. A working maven installation. If you don't have, get that <a href="http://maven.apache.org/download.html">here</a> <br/>
    2. Weblogic server installation with a valid domain. Get that <a href="http://www.oracle.com/technetwork/middleware/ias/downloads/wls-main-097127.html">here</a>    
</p>

<h5> Step 2: Generate Weblogic maven plugin</h5>
{% codeblock %}
cd to Oracle Middleware installation directory
For eg: cd MW_HOME/wlserver_XX.X/server/lib/
java -jar wljarbuilder.jar -profile weblogic-maven-plugin
{% endcodeblock %}
<h5> Step 3: Get the pom file to the weblogic lib directory</h5>
{% codeblock %}
jar xvf MW_HOME/wlserver_XX.X/server/lib/weblogic-maven-plugin.jar META-INF/maven/com.oracle.weblogic/weblogic-maven-plugin/pom.xml
{% endcodeblock %}
<p>and then</p> 
{% codeblock %}
cp MW_HOME/wlserver_XX.X/server/lib/META-INF/maven/com.oracle.weblogic/weblogic-maven-plugin/pom.xml MW_HOME/wlserver_XX.X/server/lib
{% endcodeblock %}
<h5> Step 4: Configure the shortened command line goal</h5>
Add plugin groups to maven settings.xml (one can find this in conf directory of maven installation. It is apache-maven-3.0.4/conf/ for me) similar to below.
{% codeblock %}
<pluginGroups>
    <pluginGroup>com.oracle.weblogic</pluginGroup>
</pluginGroups>
{% endcodeblock %}

then edit the pom.xml in MW_HOME/wlserver_XX.X/server/lib/ similar to the below one.

{% codeblock %}
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.oracle.weblogic</groupId>
  <artifactId>weblogic-maven-plugin</artifactId>
  <packaging>maven-plugin</packaging>
  <version>12.1.1.0</version>
  <name>Maven Mojo Archetype</name>
  <url>http://maven.apache.org</url>
  <dependencies>
    <dependency>
      <groupId>org.apache.maven</groupId>
      <artifactId>maven-plugin-api</artifactId>
      <version>3.0.4</version>
    </dependency>
 </dependencies>
</project>
{% endcodeblock %}
execute the mvn install from MW_HOME/wlserver_XX.X/server/lib/ directory
{% codeblock %}
mvn install
{% endcodeblock %}

configure the Weblogic maven plugin to be used in local maven installation
{% codeblock %}
mvn install:install-file -Dfile=MW_HOME/wlserver_XX.X/server/lib/weblogic-maven-plugin.jar -DpomFile=pom.xml
{% endcodeblock %}

Now the plugin is ready to use.
<h5>Step 5: Add build goal to your pom.xml</h5>

To configure the build goal edit your project pom.xml like the below.
{% codeblock %}
<build>
	<plugins>
		<plugin>
			<groupId>com.oracle.weblogic</groupId>
			<artifactId>weblogic-maven-plugin</artifactId>
			<version>12.1.1.0</version>
			<configuration>
				<!-- Provide the ip address of the remote server if remote deployment-->
				<adminurl>t3://localhost:7001</adminurl>
				<user>weblogic_admin_user_id</user>
				<password>admin_password</password>
				<upload>true</upload>
				<action>deploy</action>
				<!-- Change to true if remote deployment-->
				<remote>false</remote>
				<verbose>true</verbose>
				<source>${project.build.directory}/${project.build.finalName}.${project.packaging}</source>
				<name>${project.build.finalName}</name>
			</configuration>
			<executions>
				<execution>
					<phase>install</phase>
					<goals>
						<goal>deploy</goal>
					</goals>
				</execution>
			</executions>
		</plugin>
	</plugins>
</build>
{% endcodeblock %}
<p>
Bingo! You are done.
<br/>
Start your Weblogic server and one can invoke following goal from project directory.
</p>
{% codeblock %}
mvn clean install
{% endcodeblock %}


