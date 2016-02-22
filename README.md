# Log Format Enforcer

The goal of this maven plugin teams with a way of enforcing a certain logging message style in their prjects. I often 
find myself working in projects where certain log guidelines were agreed upon. In my opinion this is awesome because it 
makes it easier for parsing these kind of logs, it introduces consistency amongst the logs created by every team member,
etc, etc.

The only problem with this approach is that usually there's no way for enforcing these "agreed rules" which leads to 
problems of all sorts:
* typos in the log structure: "strings should be enclosed in double quotes" but someone missed the close double quote 
* missing agreed log data: the team agreed that every log should have the 'userId' but someone forgot to add it in one 
particular log invocation
* new team members don't actually know what the agreed rules are
* you spend more time checking for log correctness in your code-reviews than actually reviewing interesting bits of the
code logic
* if the team realizes that the log format should change a bit, it's already too late because you would have to go back 
to update every single log entry in your code base to make everything consistent again
* ...

All these are problems that this library tries to address.

## Before
```java
private static final Logger log = LoggerFactory.getLogger(MyClass.class); // or something similar
...
log.info("msg='This is a message', op=someMethod, someValue='{}'", someValue);
...
log.trace("msg='Other message', op=otherMethod, input='{}', output='{}'", input, output);
...
```
This is an actual example of some messages a team where I worked before agreed upon: 'msg' and 'op' would always be 
present. For some cases (tipically trace messages) we would log 'input' and 'output'. Every message could have any 
number of unknown fields.
###### Note 
>Even with these simple rules there was always problems: some people would use 'message' instead of 'msg' or 'operation'
>instead of 'op', quotes were commonly forgotten, same for commas between each field

## After
In your class:
```java
private static final LogFormatEnforcer log = LogFormatEnforcer.loggerFor(MyClass.class); 
...
log.log(INFO, b -> b.msg("This is a message").op("someMethod").other("someValue", someValue));
...
log.log(TRACE, b-> b.msg("Other message").op("otherMethod").input(input).output(output));
...
```
And I dare you trying to break the agreed rules now. 
###### Note 
>If you do, please let me know; this is still an early prototype ;)

In the pom.xml:
```xml
<plugin>
    <groupId>com.leandronunes85.lfe</groupId>
    <artifactId>log-format-enforcer-maven-plugin</artifactId>
    <version>1.0-SNAPSHOT</version>
    <configuration>
        <packageName>com.leandronunes85.tests</packageName>
        <mandatoryFields>
            <msg />
            <op />
        </mandatoryFields>
        <optionalFields>
            <input />
            <output />
        </optionalFields>
    </configuration>
    <executions>
        <execution>
            <goals>
                <goal>create</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```
Now, if you want to enclose your values in square brackets (because it's easier for you to parse it) and use "->" to 
seperate keys and values (because you fancy Java 8)? Just update your pom, like this:
```xml
<plugin>
    <groupId>com.leandronunes85.lfe</groupId>
    <artifactId>log-format-enforcer-maven-plugin</artifactId>
    <version>1.0-SNAPSHOT</version>
    <configuration>
        <packageName>com.leandronunes85.tests</packageName>
        <mandatoryFields>
            <op />
            <msg />
        </mandatoryFields>
        <optionalFields>
            <input />
            <output />
        </optionalFields>
        <valueDelimiterPrefix>[</valueDelimiterPrefix>
        <valueDelimiterSuffix>]</valueDelimiterSuffix>
        <keyValueSeparator>-></keyValueSeparator>
    </configuration>
    <executions>
        <execution>
            <goals>
                <goal>create</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```
Rebuild it, release it and that's it!

## Wrap up
As I said, this is still a prototype, there's still bugs that I'm aware of and many other that I'm not, for sure. 
Your comments, suggestions, concerns, etc, are all welcome so please drop me a line!
