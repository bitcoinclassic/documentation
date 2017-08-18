# Log Levels standards

In the 1.3 (July 2017) series Classic introduced a new logging framework that makes a much more fine-grained logging possible. We expect other clients to steal this approach in due time.

Logging in Bitcoin has historically been for the developer, and the operator could benefit in a "rising tide raises all boats" kind of way.
This is meant to change with the logging framework which makes a very clear distinction between the software developers needs and and the operators needs.
But we need to actually be consistent in usage in order to reap the benefits.


## Logging for the developer

Developers need lots of info with minimal work.
There are several features I suggest developers use;

* The developer should consider to configure using --enable-dev-setup
  The result of this is that the executable will contain line-numbers and method names.
* The developer should configure using --enable-debug (implied by the previous line)
  The result of this is that debug-level log messages will actually show up.
* Create a logs.conf file that shows the sections you are interested in.
Remember that altering the file and sending the SIGHUP signal to your running process will re-load the config.

Developers are expected to be able to add logging lines to the code and are expected to finely tweak the logs they are interested in seeing using the config file.
All this results in the developer seeing much more info than anyone else.

## Logging for the (node) operator

Operators will likely run a node that has been successfully setup with much less verbose logging.
They can selectively alter the logs.conf to show sections with higher verbosity. But never debug-level.

The assumption is made that a node operator will not use a released version to debug and develop the actual node software.


# Log-Levels

Software developers have to choose the log level for every message they log. To have a consistent way of logging, here is a guideline on what people expect to see at a certain log level.

Remember, a user selecting to see, for instance, Warning level will see all messages of higher severity as well.

* Debug
Not present in the release binary at all.
As such purely for debugging purposes. Is allowed to be extra verbose.
This is exclusively for the software developer. Node operators never see this.
* Warning
This is used when something minor went wrong.
This is the lowest level that is shipped in the 'release' executable.
* Info
Both Warning and Info are the log levels a node operator will use to find out what is going on and why is something going wrong.
* Critical
This level should be seen as the default level a node can operate on, for people that don't really care
how it works, just that it does its work.
Users logging in and not getting a connection is going to this level, the network having issues as well.
This is the level of the car-dashboard. The 'handbrake on' light goes here, miles driven goes here as well.
* Fatal
Practically speaking this is limited to startup and shutdown messages, especially when the shutdown was due to an error.


# Log-Sections

Each log line is part of a 'section'. Sections are numbered and we separate functionality by sections. For instance we have a 'Net' section for the p2p part of Bitcoin. We have a 'Validation', a 'mempool' one etc.

The idea is that a developer or node operator can selectively turn on sections. For instance, we realise we consistently ban a node shortly after connecting to it. That means we can show more log-levels of the 'net' section to allow troubleshooting.

The log-levels an operator wants to see in the output are configurable per section. You can set one to quiet and another to debug level.

Guidance for developers in choosing the sections for logging;

* The 'Global' section is exclusively for developers. (0-999)
This is a bit messy as much of the old code still uses this, but the goal is to move them all to their respective sections.
Practically speaking this means we should not have a single warning/info or higher severity log-level in the code using any section under 1000.

* We have 'major' sections. "Validation", "Networking" , "Wallet" etc. Developers that assign a new section for their new code should do so in the appropriate major section.  
Not all sections need names, a number is perfectly fine. But please do register it somewhere so we avoid reuse.

* Do not pick sections over 20999, we do not initialise them and their behaviour is undefined.

* Log sections are currently documented only in the Logger.h, we need proper documentation either in the documentation repo or on the website (or both).
