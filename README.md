[MNG-7804](https://issues.apache.org/jira/browse/MNG-7804) plugin execution priority test
===========

By default, plugins executions are consolidated and sorted as defined in XML.

With MNG-7804, optional `priority` attribute of `<execution>` permits customization:

```
$ mvn 
...
[INFO] --- buildplan:2.2.2:list (default-cli) @ aid ---
[INFO] Build Plan for aid: 
---------------------------------------------------------------------------------------------------
PHASE                  | PLUGIN                   | VERSION | GOAL          | EXECUTION ID         
---------------------------------------------------------------------------------------------------
process-resources      | exec-maven-plugin        | 3.1.0   | exec          | run0                 
process-resources      | maven-resources-plugin   | 3.3.1   | resources     | default-resources    
process-resources      | exec-maven-plugin        | 3.1.0   | exec          | run1                 
initialize             | buildnumber-maven-plugin | 3.1.0   | create        | run2                 
process-resources      | exec-maven-plugin        | 3.1.0   | exec          | run3                 
initialize             | buildnumber-maven-plugin | 3.1.0   | create        | run4                 
compile                | maven-compiler-plugin    | 3.11.0  | compile       | default-compile      
process-test-resources | maven-resources-plugin   | 3.3.1   | testResources | default-testResources
test-compile           | maven-compiler-plugin    | 3.11.0  | testCompile   | default-testCompile  
test                   | maven-surefire-plugin    | 3.1.2   | test          | default-test         
package                | maven-jar-plugin         | 3.3.0   | jar           | default-jar          
install                | maven-install-plugin     | 3.1.1   | install       | default-install      
deploy                 | maven-deploy-plugin      | 3.1.1   | deploy        | default-deploy       
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
...
```

if you look at [`pom.xml`](pom.xml), you'll see that the different `run*` execution ids:

- don't follow XML declaraiton order

- exec and buildnumber executions are interleaved

- `run0` even executes before `default-resources` that is provided by the `jar` lifecycle

All that happens even if XML order remains as usual:

```
$ mvn help:effective-pom -Dverbose | grep priority -B4 -A5
  <version>1.0-SNAPSHOT</version>  <!-- gid:aid:1.0-SNAPSHOT, line 9 -->
  <name>aid</name>  <!-- gid:aid:1.0-SNAPSHOT, line 11 -->
  <url>http://www.example.com</url>  <!-- gid:aid:1.0-SNAPSHOT, line 13 -->
  <scm>
    <connection>scm:git:https://github.com/hboutemy/priority.git</connection>  <!-- gid:aid:1.0-SNAPSHOT, line 21 -->
    <developerConnection>scm:git:https://github.com/hboutemy/priority.git</developerConnection>  <!-- gid:aid:1.0-SNAPSHOT, line 22 -->
    <url>https://github.com/hboutemy/priority</url>  <!-- gid:aid:1.0-SNAPSHOT, line 23 -->
  </scm>
  <properties>
    <maven.compiler.release>11</maven.compiler.release>  <!-- gid:aid:1.0-SNAPSHOT, line 17 -->
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>  <!-- gid:aid:1.0-SNAPSHOT, line 16 -->
  </properties>
--
        <groupId>org.codehaus.mojo</groupId>  <!-- gid:aid:1.0-SNAPSHOT, line 89 -->
        <artifactId>exec-maven-plugin</artifactId>  <!-- gid:aid:1.0-SNAPSHOT, line 90 -->
        <version>3.1.0</version>  <!-- gid:aid:1.0-SNAPSHOT, line 42 -->
        <executions>
          <execution priority="3">
            <id>run3</id>  <!-- gid:aid:1.0-SNAPSHOT, line 60 -->
            <phase>process-resources</phase>  <!-- gid:aid:1.0-SNAPSHOT, line 61 -->
            <goals>
              <goal>exec</goal>  <!-- gid:aid:1.0-SNAPSHOT, line 63 -->
            </goals>
            <configuration>
              <executable>date</executable>  <!-- gid:aid:1.0-SNAPSHOT, line 44 -->
            </configuration>
          </execution>
          <execution priority="1">
            <id>run1</id>  <!-- gid:aid:1.0-SNAPSHOT, line 93 -->
            <phase>process-resources</phase>  <!-- gid:aid:1.0-SNAPSHOT, line 94 -->
            <goals>
              <goal>exec</goal>  <!-- gid:aid:1.0-SNAPSHOT, line 96 -->
            </goals>
            <configuration>
              <executable>date</executable>  <!-- gid:aid:1.0-SNAPSHOT, line 44 -->
            </configuration>
          </execution>
          <execution priority="-2000">
            <id>run0</id>  <!-- gid:aid:1.0-SNAPSHOT, line 100 -->
            <phase>process-resources</phase>  <!-- gid:aid:1.0-SNAPSHOT, line 101 -->
            <goals>
              <goal>exec</goal>  <!-- gid:aid:1.0-SNAPSHOT, line 103 -->
            </goals>
--
        <groupId>org.codehaus.mojo</groupId>  <!-- gid:aid:1.0-SNAPSHOT, line 69 -->
        <artifactId>buildnumber-maven-plugin</artifactId>  <!-- gid:aid:1.0-SNAPSHOT, line 70 -->
        <version>3.1.0</version>  <!-- gid:aid:1.0-SNAPSHOT, line 50 -->
        <executions>
          <execution priority="4">
            <id>run4</id>  <!-- gid:aid:1.0-SNAPSHOT, line 73 -->
            <phase>process-resources</phase>  <!-- gid:aid:1.0-SNAPSHOT, line 74 -->
            <goals>
              <goal>create</goal>  <!-- gid:aid:1.0-SNAPSHOT, line 76 -->
            </goals>
          </execution>
          <execution priority="2">
            <id>run2</id>  <!-- gid:aid:1.0-SNAPSHOT, line 80 -->
            <phase>process-resources</phase>  <!-- gid:aid:1.0-SNAPSHOT, line 81 -->
            <goals>
              <goal>create</goal>  <!-- gid:aid:1.0-SNAPSHOT, line 83 -->
            </goals>
...
```