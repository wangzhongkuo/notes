### compile the tomcat-8.5.8 resource and import into eclipse

- install ant
- download the tomcat resource form the  [Apache Tomcat Home](http://archive.apache.org/dist/tomcat/tomcat-8/v8.5.8/src/)
- open your cmd terminal, and uppackage the apache-tomcat-8.5.8-src.zip , then cd apache-tomcat-8.5.8-src folder,mv "build.properties.default" "build.properties", open the build.properties and set the base.path property, the value is where you want to save something downloaded when you compile the tomcat code. then execute the commond "ant" to compile your tomcat resource(in "apache-tomcat-8.5.8-src") directory. and you could see a lot of logs bilibili..., if the "BUILD SUCCESSFUL" displayed at the end, congratulations!
- now you could find two new folders, they are "output"、"tomcat-build-libs",the first is major, the second is what you set in previous step, i haven't use it anyhow...
- create a maven project in your ide, copy the "jave" and "test" folders into your new project, then a lot of errors will occur, don't worry, be happy.
- copy the following dependencies into your pom.xml
```xml
<dependencies>
		<dependency>
			<groupId>junit</groupId>
			<artifactId>junit</artifactId>
			<version>4.12</version>
			<scope>test</scope>
		</dependency>
		<dependency>
			<groupId>org.apache.ant</groupId>
			<artifactId>ant</artifactId>
			<version>1.10.1</version>
		</dependency>
		<dependency>
			<groupId>javax.xml.rpc</groupId>
			<artifactId>javax.xml.rpc-api</artifactId>
			<version>1.1.1</version>
		</dependency>
		<dependency>
			<groupId>org.eclipse.birt.runtime.3_7_1</groupId>
			<artifactId>javax.wsdl</artifactId>
			<version>1.5.1</version>
		</dependency>
		<dependency>
			<groupId>org.eclipse.jdt</groupId>
			<artifactId>org.eclipse.jdt.core</artifactId>
			<version>3.12.3</version>
		</dependency>
		<dependency>
			<groupId>org.easymock</groupId>
			<artifactId>easymock</artifactId>
			<version>3.4</version>
			<scope>test</scope>
		</dependency>
	</dependencies>
```
- now, most of the errors have been resolved, but you could find the CookieFilter.java is a missing class in your project, copy this.

```java
/*
 * Licensed to the Apache Software Foundation (ASF) under one or more
 * contributor license agreements.  See the NOTICE file distributed with
 * this work for additional information regarding copyright ownership.
 * The ASF licenses this file to You under the Apache License, Version 2.0
 * (the "License"); you may not use this file except in compliance with
 * the License.  You may obtain a copy of the License at
 *
 *     http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */
package util;

import java.util.Locale;
import java.util.StringTokenizer;

/**
 * Processes a cookie header and attempts to obfuscate any cookie values that
 * represent session IDs from other web applications. Since session cookie names
 * are configurable, as are session ID lengths, this filter is not expected to
 * be 100% effective.
 *
 * It is required that the examples web application is removed in security
 * conscious environments as documented in the Security How-To. This filter is
 * intended to reduce the impact of failing to follow that advice. A failure by
 * this filter to obfuscate a session ID or similar value is not a security
 * vulnerability. In such instances the vulnerability is the failure to remove
 * the examples web application.
 */
public class CookieFilter {

    private static final String OBFUSCATED = "[obfuscated]";

    private CookieFilter() {
        // Hide default constructor
    }

    public static String filter(String cookieHeader, String sessionId) {

        StringBuilder sb = new StringBuilder(cookieHeader.length());

        // Cookie name value pairs are ';' separated.
        // Session IDs don't use ; in the value so don't worry about quoted
        // values that contain ;
        StringTokenizer st = new StringTokenizer(cookieHeader, ";");

        boolean first = true;
        while (st.hasMoreTokens()) {
            if (first) {
                first = false;
            } else {
                sb.append(';');
            }
            sb.append(filterNameValuePair(st.nextToken(), sessionId));
        }


        return sb.toString();
    }

    private static String filterNameValuePair(String input, String sessionId) {
        int i = input.indexOf('=');
        if (i == -1) {
            return input;
        }
        String name = input.substring(0, i);
        String value = input.substring(i + 1, input.length());

        return name + "=" + filter(name, value, sessionId);
    }

    public static String filter(String cookieName, String cookieValue, String sessionId) {
        if (cookieName.toLowerCase(Locale.ENGLISH).contains("jsessionid") &&
                (sessionId == null || !cookieValue.contains(sessionId))) {
            cookieValue = OBFUSCATED;
        }
        return cookieValue;
    }
}
```
- ensure all of the code errors have been resolved, let't build a tomcat project. copy the "apache-tomcat-8.5.8-src/webapps"、"apache-tomcat-8.5.8-src/output/build/lib"、"apache-tomcat-8.5.8-src/output/build/conf" into your project's root directory.
- the great task has been completed. run the org.apache.catalina.startup.Bootstrap.main, open your brower, access the address "http://localhost:8080", hope you can see tom =_=.
