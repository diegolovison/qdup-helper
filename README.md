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

