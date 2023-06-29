# JDB (The Java Debugger)
Connect to jvm with jdpa enabled:
````
jdb -attach 127.0.0.1:8000
````
List methods of the class:
````
methods org.apache.tomcat.websocket.server.WsFilter
````
You can find where the troubles happen, for example, in logs, in java stack traces, etc.. <br>

Set a breakpoint in a method (right in the constructor in this example):
````
stop in en.xxx.xxx.auth.AuthorizationException.<init>
````
Dump stack of the current thread:
````
where
...
  [1] en.xxx.xxx.AuthProvider.get (AuthProvider.java:31)
  [2] en.xxx.xxx.AuthChecker.getCallerOrFail (AuthChecker.java:81)
  [3] en.xxx.xxx.AuthChecker.check (AuthChecker.java:28)
  [4] en.xxx.xxx.AuthInterceptor.preHandle (AuthInterceptor.java:36)
  [5] org.springframework.web.servlet.HandlerExecutionChain.applyPreHandle (HandlerExecutionChain.java:136)
  [6] org.springframework.web.servlet.DispatcherServlet.doDispatch (DispatcherServlet.java:986)
  [7] org.springframework.web.servlet.DispatcherServlet.doService (DispatcherServlet.java:925)
  [8] org.springframework.web.servlet.FrameworkServlet.processRequest (FrameworkServlet.java:974)
  [9] org.springframework.web.servlet.FrameworkServlet.doGet (FrameworkServlet.java:866)
````
Here you can find what methods called the exceptions and set a breakpoint in them in next time. <br>
You can execute line by line with step command, and read variables with locals on each step to look for changes. <br>
The whole list of commands:
````
> help
** command list **
connectors                -- list available connectors and transports in this VM

run [class [args]]        -- start execution of application's main class

threads [threadgroup]     -- list threads
thread <thread id>        -- set default thread
suspend [thread id(s)]    -- suspend threads (default: all)
resume [thread id(s)]     -- resume threads (default: all)
where [<thread id> | all] -- dump a thread's stack
wherei [<thread id> | all]-- dump a thread's stack, with pc info
up [n frames]             -- move up a thread's stack
down [n frames]           -- move down a thread's stack
kill <thread id> <expr>   -- kill a thread with the given exception object
interrupt <thread id>     -- interrupt a thread

print <expr>              -- print value of expression
dump <expr>               -- print all object information
eval <expr>               -- evaluate expression (same as print)
set <lvalue> = <expr>     -- assign new value to field/variable/array element
locals                    -- print all local variables in current stack frame

classes                   -- list currently known classes
class <class id>          -- show details of named class
methods <class id>        -- list a class's methods
fields <class id>         -- list a class's fields

threadgroups              -- list threadgroups
threadgroup <name>        -- set current threadgroup

stop in <class id>.<method>[(argument_type,...)]
                          -- set a breakpoint in a method
stop at <class id>:<line> -- set a breakpoint at a line
clear <class id>.<method>[(argument_type,...)]
                          -- clear a breakpoint in a method
clear <class id>:<line>   -- clear a breakpoint at a line
clear                     -- list breakpoints
catch [uncaught|caught|all] <class id>|<class pattern>
                          -- break when specified exception occurs
ignore [uncaught|caught|all] <class id>|<class pattern>
                          -- cancel 'catch' for the specified exception
watch [access|all] <class id>.<field name>
                          -- watch access/modifications to a field
unwatch [access|all] <class id>.<field name>
                          -- discontinue watching access/modifications to a field
trace [go] methods [thread]
                          -- trace method entries and exits.
                          -- All threads are suspended unless 'go' is specified
trace [go] method exit | exits [thread]
                          -- trace the current method's exit, or all methods' exits
                          -- All threads are suspended unless 'go' is specified
untrace [methods]         -- stop tracing method entrys and/or exits
step                      -- execute current line
step up                   -- execute until the current method returns to its caller
stepi                     -- execute current instruction
next                      -- step one line (step OVER calls)
cont                      -- continue execution from breakpoint

list [line number|method] -- print source code
use (or sourcepath) [source file path]
                          -- display or change the source path
exclude [<class pattern>, ... | "none"]
                          -- do not report step or method events for specified classes
classpath                 -- print classpath info from target VM

monitor <command>         -- execute command each time the program stops
monitor                   -- list monitors
unmonitor <monitor#>      -- delete a monitor
read <filename>           -- read and execute a command file

lock <expr>               -- print lock info for an object
threadlocks [thread id]   -- print lock info for a thread

pop                       -- pop the stack through and including the current frame
reenter                   -- same as pop, but current frame is reentered
redefine <class id> <class file name>
                          -- redefine the code for a class

disablegc <expr>          -- prevent garbage collection of an object
enablegc <expr>           -- permit garbage collection of an object

!!                        -- repeat last command
<n> <command>             -- repeat command n times
# <command>               -- discard (no-op)
help (or ?)               -- list commands
version                   -- print version information
exit (or quit)            -- exit debugger

<class id>: a full class name with package qualifiers
<class pattern>: a class name with a leading or trailing wildcard ('*')
<thread id>: thread number as reported in the 'threads' command
<expr>: a Java(TM) Programming Language expression.
Most common syntax is supported.

Startup commands can be placed in either "jdb.ini" or ".jdbrc"
in user.home or user.dir

````
