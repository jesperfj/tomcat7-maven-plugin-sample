This project generated with Spring Roo 1.2.0.RELEASE using:

    $ roo.sh script --file petclinic.roo

(this is commit c1484df2cc8289a75f92b05968c5299a04e26f3a)

then modified `pom.xml` to add Tomcat7 Maven Plugin. Added plugin repository:

	<pluginRepositories>
		...
	    <pluginRepository>
	        <id>spring-roo-repository</id>
	        <name>Spring Roo Repository</name>
	        <url>http://spring-roo-repository.springsource.org/release</url>
	    </pluginRepository>
	</pluginRepositories>

and added plugin definition:

	<plugins>
		...
	    <plugin>
	        <groupId>org.apache.tomcat.maven</groupId>
	        <artifactId>tomcat7-maven-plugin</artifactId>
	        <version>2.0-SNAPSHOT</version>
	        <executions>
	            <execution>
	                <id>tomcat-run</id>
	                <goals>
	                    <goal>exec-war-only</goal>
	                </goals>
	                <phase>package</phase>
	                <configuration>
	                    <path>/</path>
	                </configuration>
	            </execution>
	        </executions>
	    </plugin>
	</plugins>

(this is commit 5016dab75602f811e306c7cdd17101ac3217fa7d)

Now build with:

	$ mvn package

and run with:

	$ java -jar target/petclinic-0.1.0.BUILD-SNAPSHOT-war-exec.jar 

Everything looks good in the log output (omitted since it's really long).

Now hit <http://localhost:8080>. This will produce a broken page because static resources could not be loaded. Inspect the page and you'll find image URLs such as this one:

	<img src="//resources/images/banner-graphic.png">

Which the browser translates to `http://resources/images/banner-graphic.png`. If you look at the underlying JSP file, this URl is generated with this tag:

	$ ack banner-graphic.png
	src/main/webapp/WEB-INF/views/header.jspx
	5:  <spring:url var="banner" value="/resources/images/banner-graphic.png" />

Looks like this spring tag prepends the context path to the URL provided in the value attribute. Unfortunately, if I set the `path` parameter in the tomcat7 plugin configuration to "":

	<plugin>
	    <groupId>org.apache.tomcat.maven</groupId>
	    <artifactId>tomcat7-maven-plugin</artifactId>
	    <version>2.0-SNAPSHOT</version>
	    <executions>
	        <execution>
	            <id>tomcat-run</id>
	            <goals>
	                <goal>exec-war-only</goal>
	            </goals>
	            <phase>package</phase>
	            <configuration>
	                <path></path>
	            </configuration>
	        </execution>
	    </executions>
	</plugin>

then I get this output after doing a `mvn clean package`, manually deleting `.extract/` and restarting the server with the same command:

	$ java -jar target/petclinic-0.1.0.BUILD-SNAPSHOT-war-exec.jar 
	Jan 31, 2012 9:08:32 AM org.apache.catalina.core.StandardContext setPath
	WARNING: A context path must either be an empty string or start with a '/'. The path [petclinic] does not meet these criteria and has been changed to [/petclinic]
	...

Petclinic now runs just fine, but it's on the `/petclinic` context path. The same happens if I completely remove the `path` parameter from the configuration.

How to get petclinic running on the ROOT context path and serving static resources correctly?
