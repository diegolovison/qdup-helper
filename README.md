# Run a simple test
```java
package io.hyperfoil.tools.qdup;

import io.hyperfoil.tools.qdup.cmd.Dispatcher;
import io.hyperfoil.tools.qdup.config.RunConfig;
import io.hyperfoil.tools.qdup.config.RunConfigBuilder;
import io.hyperfoil.tools.qdup.config.yaml.Parser;
import org.junit.Test;

import java.util.stream.Collectors;

import static org.junit.Assert.assertFalse;

public class MyTest {
    @Test
    public void wrongHost() {
        Parser parser = Parser.getInstance();
        RunConfigBuilder builder = new RunConfigBuilder();
        builder.loadYaml(parser.loadFile("example","""
            scripts:
              hello-qdup:
                - sh: echo hello-qdup
            hosts:
              local : ${{target-host}}
            roles:
              run-hello-qdup:
                hosts:
                  - local
                run-scripts:
                  - hello-qdup
        """));
        builder.forceRunState("target-host", System.getenv("USER") + "@localhost");
        RunConfig config = builder.buildConfig(parser);
        assertFalse("runConfig errors:\n" + config.getErrorStrings().stream().collect(Collectors.joining("\n")), config.hasErrors());

        Dispatcher dispatcher = new Dispatcher();
        Run doit = new Run("/tmp", config, dispatcher);
        doit.run();

        State state = config.getState();
    }
}
```

# How to understand what is happening with qDup
Apply the following
```text
diff --git a/src/main/java/io/hyperfoil/tools/qdup/shell/AbstractShell.java b/src/main/java/io/hyperfoil/tools/qdup/shell/AbstractShell.java
index 7cdcf82c..56862e37 100644
--- a/src/main/java/io/hyperfoil/tools/qdup/shell/AbstractShell.java
+++ b/src/main/java/io/hyperfoil/tools/qdup/shell/AbstractShell.java
@@ -356,7 +356,7 @@ public void sh(String command, BiConsumer<String,String> callback, Map<String, S
     }
     void sh(String command, boolean acquireLock, BiConsumer<String,String> callback, Map<String, String> prompt) {
         command = command.replaceAll("[\r\n]+$", ""); //replace trailing newlines
-        logger.trace("{} sh: {}, lock: {}", getHost(), command, acquireLock);
+        logger.info("{} sh: {}, lock: {}", getHost(), command, acquireLock);
         ShAction newAction = new ShAction(command,acquireLock,callback,prompt);
         lastCommand = command;
         if (command == null || commandStream == null) {
diff --git a/src/main/resources/log4j2.xml b/src/main/resources/log4j2.xml
index 05dcefae..9019d4ec 100644
--- a/src/main/resources/log4j2.xml
+++ b/src/main/resources/log4j2.xml
@@ -8,7 +8,7 @@
             </PatternLayout>
         </File>
         <Console name="STDOUT" target="SYSTEM_OUT" follow="true">
-            <PatternLayout disableAnsi="false" pattern="%highlight{%d{HH:mm:ss.SSS} %m%n}{FATAL=red blink, ERROR=red, WARN=yellow bold, INFO=white, DEBUG=green bold, TRACE=blue}"/>
+            <PatternLayout disableAnsi="false" pattern="%highlight{%d{HH:mm:ss.SSS} [%p] %m%n}{FATAL=red blink, ERROR=red, WARN=yellow bold, INFO=white, DEBUG=green bold, TRACE=blue}"/>
         </Console>
     </Appenders>
     <Loggers>
```