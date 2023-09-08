# Armitage Notes <!-- omit from toc -->
These are notes I have taken when debugging the Armitage system. This was done at the request of Prof. Fu to fix some bugs he observed during his classes. 

This document contains an overview of the systems and some observations made throughout. 

## Table of Contents <!-- omit from toc -->
- [Systems](#systems)
- [Armitage Communications](#armitage-communications)
- [Log file locations](#log-file-locations)
- [Exploit Error Messages](#exploit-error-messages)
  - [OLD Kali](#old-kali)
  - [New Kali](#new-kali)
- [MSF Scan Error](#msf-scan-error)
    - [Normal Logs](#normal-logs)
- [Exploit \& Scanning Results](#exploit--scanning-results)
- [Armitage Code](#armitage-code)
  - [Sleep](#sleep)
  - [Java Code](#java-code)
  - [Build Process](#build-process)
- [Notes](#notes)


## Systems
**Class Kali Linux**: 
* CPU - 1 
* RAM - 2048
* Memory - 25 GB
  * 46% Free 
* Version - 2022.1
* Metasploit Version 
  * Framework: 6.1.35-dev
  * Console  : 6.1.35-dev

**DevKali Linux**: 
* CPU - 1 
* RAM - 2048
* Memory - 25 GB 
  * 46% Free 
* Version - 2023.2
* Metasploit Version 
  * Framework: 6.3.27-dev
  * Console  : 6.3.27-dev

**Windows**: 
* CPU - 1 
* RAM - 2048
* Windows 10

## Armitage Communications 
Armitage communicates with a Metasploit RPC server that can be hosted locally or remotely. Prof. Fu's class will always host the Metasploit instance Locally. In this way we can capture the packets between the Armitage Client and Metasploit RPC server easily. The hard part appears to be 1. Decoding the messages between Armitage and the MSF RPC server as they communicate with JSON structures encoded with the *MessagePack* encoding scheme, and 2. Capturing the right traffic as there is *a lot* of *http* traffic between Armitage and the MSF RPC server.   


As alluded to before, the Armitage Client communicates with Metasploit using the Metasploit RPC server. There are a few sources detailing the [MSF RPC API](https://docs.rapid7.com/metasploit/rpc-api/), however they tend to lack alot of the desired detail that we may want, there is an older [PDF](https://blog.ehcgroup.io/wp-content/uploads/2017/08/metasploit-rpc-api-guide-1.pdf) with a decent amount of details, this was also downloaded for posterity or something. As the MessagePack API is used, the communcations will be encoded. This means we will need to decode it with a [WireShark plugin](https://github.com/linear-rpc/msgpack-rpc-dissector), [cli tool](https://github.com/ludocode/msgpack-tools), or [web application](https://msgpack.solder.party/). Regardless, this is a tedious task. A general outline of the communications between the Armitage and MSF RPC Server is shown below.

<img src=Images/Armitage-MSF.jpg>

* Simply the client makes a request to the RPC API endpoint
* The Endpoint responds with a MessagePack encoded JSON object 
* The Object is then parsed by Armitage and used to update the screen's display. 

## Log file locations 
Metasploit: 
* The logs will be located at ```~/.msf4/logs/framework.log```, if you run armitage without sudo, they will be located in the user's ```.msf``` directory. If sudo is used then they will be located in the root user's home directory (```sudo su``` then the previous path will work)

Armitage:
* The "logs" if they can be called that are located at ```~/.armitage/<YearMothDay>/localhost/target/<attackType>.log```
  * The ```<YearMonthDay>``` is in the form of concatinated number for example 9/8/23 is ```230908```
  * Example Logfile ```~/.armitage/230908/localhost/10.0.2.4/auxiliary.log```
    * ```10.0.2.4``` can be replaced with "all"

The Armitage Logs are just a list of commands and their resulting output that are sent to the Server, if there are error or event  based logging I have not seen it.


## Exploit Error Messages
The following are error messages that were observed from the MSF Framework during the execution of Exploits
### OLD Kali 
On the Old Kali the MSF Framework has generated the following errors.
* IOErros - This seemed to be the most common when the system fails   
    ```
    [09/07/2023 14:23:26] [d(0)] core: monitor_rsock: EOF in rsock
    [09/07/2023 14:23:26] [w(0)] core: monitor_rsock: exception during read: Errno::EAGAIN Resource temporarily unavailable
    [09/07/2023 14:23:26] [d(0)] core: HistoryManager.pop_context name: :msfconsole
    [09/07/2023 14:23:26] [e(0)] core: Thread Exception: WebConsoleShell  critical=false    source:
        /usr/share/metasploit-framework/lib/msf/ui/web/web_console.rb:84:in `initialize'
        /usr/share/metasploit-framework/lib/msf/ui/web/driver.rb:62:in `new'
        /usr/share/metasploit-framework/lib/msf/ui/web/driver.rb:62:in `create_console'
        /usr/share/metasploit-framework/lib/msf/core/rpc/v10/rpc_console.rb:28:in `rpc_create'
        /usr/share/metasploit-framework/lib/msf/core/rpc/v10/service.rb:143:in `block in process'
        /usr/lib/ruby/3.0.0/timeout.rb:97:in `block in timeout'
        /usr/lib/ruby/3.0.0/timeout.rb:35:in `block in catch'
        /usr/lib/ruby/3.0.0/timeout.rb:35:in `catch'
        /usr/lib/ruby/3.0.0/timeout.rb:35:in `catch'
        /usr/lib/ruby/3.0.0/timeout.rb:112:in `timeout'
        /usr/share/metasploit-framework/lib/msf/core/rpc/v10/service.rb:143:in `process'
        /usr/share/metasploit-framework/lib/msf/core/rpc/v10/service.rb:81:in `on_request_uri'
        /usr/share/metasploit-framework/lib/msf/core/rpc/v10/service.rb:62:in `block in start'
        /usr/share/metasploit-framework/lib/rex/proto/http/handler/proc.rb:38:in `on_request'
        /usr/share/metasploit-framework/lib/rex/proto/http/server.rb:369:in `dispatch_request'
        /usr/share/metasploit-framework/lib/rex/proto/http/server.rb:303:in `on_client_data'
        /usr/share/metasploit-framework/lib/rex/proto/http/server.rb:162:in `block in start'
        /usr/share/metasploit-framework/vendor/bundle/ruby/3.0.0/gems/rex-core-0.1.27/lib/rex/io/stream_server.rb:42:in `on_client_data'
        /usr/share/metasploit-framework/vendor/bundle/ruby/3.0.0/gems/rex-core-0.1.27/lib/rex/io/stream_server.rb:185:in `block in monitor_clients'
        /usr/share/metasploit-framework/vendor/bundle/ruby/3.0.0/gems/rex-core-0.1.27/lib/rex/io/stream_server.rb:184:in `each'
        /usr/share/metasploit-framework/vendor/bundle/ruby/3.0.0/gems/rex-core-0.1.27/lib/rex/io/stream_server.rb:184:in `monitor_clients'
        /usr/share/metasploit-framework/vendor/bundle/ruby/3.0.0/gems/rex-core-0.1.27/lib/rex/io/stream_server.rb:64:in `block in start'
        /usr/share/metasploit-framework/lib/rex/thread_factory.rb:22:in `block in spawn'
        /usr/share/metasploit-framework/lib/msf/core/thread_manager.rb:105:in `block in spawn'
        /usr/share/metasploit-framework/vendor/bundle/ruby/3.0.0/gems/logging-2.3.0/lib/logging/diagnostic_context.rb:474:in `block in create_with_logging_context' - IOError stream closed in another thread
    ```
* Type Conversion Errors 
    ```
    [09/07/2023 14:23:25] [d(0)] core: monitor_rsock: EOF in rsock
    [09/07/2023 14:23:26] [e(0)] core: Thread Exception: WebConsoleShell  critical=false    source:
        /usr/share/metasploit-framework/lib/msf/ui/web/web_console.rb:84:in `initialize'
        /usr/share/metasploit-framework/lib/msf/ui/web/driver.rb:62:in `new'
        /usr/share/metasploit-framework/lib/msf/ui/web/driver.rb:62:in `create_console'
        /usr/share/metasploit-framework/lib/msf/core/rpc/v10/rpc_console.rb:28:in `rpc_create'
        /usr/share/metasploit-framework/lib/msf/core/rpc/v10/service.rb:143:in `block in process'
        /usr/lib/ruby/3.0.0/timeout.rb:97:in `block in timeout'
        /usr/lib/ruby/3.0.0/timeout.rb:35:in `block in catch'
        /usr/lib/ruby/3.0.0/timeout.rb:35:in `catch'
        /usr/lib/ruby/3.0.0/timeout.rb:35:in `catch'
        /usr/lib/ruby/3.0.0/timeout.rb:112:in `timeout'
        /usr/share/metasploit-framework/lib/msf/core/rpc/v10/service.rb:143:in `process'
        /usr/share/metasploit-framework/lib/msf/core/rpc/v10/service.rb:81:in `on_request_uri'
        /usr/share/metasploit-framework/lib/msf/core/rpc/v10/service.rb:62:in `block in start'
        /usr/share/metasploit-framework/lib/rex/proto/http/handler/proc.rb:38:in `on_request'
        /usr/share/metasploit-framework/lib/rex/proto/http/server.rb:369:in `dispatch_request'
        /usr/share/metasploit-framework/lib/rex/proto/http/server.rb:303:in `on_client_data'
        /usr/share/metasploit-framework/lib/rex/proto/http/server.rb:162:in `block in start'
        /usr/share/metasploit-framework/vendor/bundle/ruby/3.0.0/gems/rex-core-0.1.27/lib/rex/io/stream_server.rb:42:in `on_client_data'
        /usr/share/metasploit-framework/vendor/bundle/ruby/3.0.0/gems/rex-core-0.1.27/lib/rex/io/stream_server.rb:185:in `block in monitor_clients'
        /usr/share/metasploit-framework/vendor/bundle/ruby/3.0.0/gems/rex-core-0.1.27/lib/rex/io/stream_server.rb:184:in `each'
        /usr/share/metasploit-framework/vendor/bundle/ruby/3.0.0/gems/rex-core-0.1.27/lib/rex/io/stream_server.rb:184:in `monitor_clients'
        /usr/share/metasploit-framework/vendor/bundle/ruby/3.0.0/gems/rex-core-0.1.27/lib/rex/io/stream_server.rb:64:in `block in start'
        /usr/share/metasploit-framework/lib/rex/thread_factory.rb:22:in `block in spawn'
        /usr/share/metasploit-framework/lib/msf/core/thread_manager.rb:105:in `block in spawn'
        /usr/share/metasploit-framework/vendor/bundle/ruby/3.0.0/gems/logging-2.3.0/lib/logging/diagnostic_context.rb:474:in `block in create_with_logging_context' - TypeError no implicit conversion of nil into String
    ```
    * Appears to be generated when closing a CONSOLE TAB
* Thread Generation 
    ```
    [09/07/2023 14:03:20] [e(0)] core: Thread Exception: WebConsoleShell  critical=false    source:
        /usr/share/metasploit-framework/lib/msf/ui/web/web_console.rb:84:in `initialize'
        /usr/share/metasploit-framework/lib/msf/ui/web/driver.rb:62:in `new'
        /usr/share/metasploit-framework/lib/msf/ui/web/driver.rb:62:in `create_console'
        /usr/share/metasploit-framework/lib/msf/core/rpc/v10/rpc_console.rb:28:in `rpc_create'
        /usr/share/metasploit-framework/lib/msf/core/rpc/v10/service.rb:143:in `block in process'
        /usr/lib/ruby/3.0.0/timeout.rb:97:in `block in timeout'
        /usr/lib/ruby/3.0.0/timeout.rb:35:in `block in catch'
        /usr/lib/ruby/3.0.0/timeout.rb:35:in `catch'
        /usr/lib/ruby/3.0.0/timeout.rb:35:in `catch'
        /usr/lib/ruby/3.0.0/timeout.rb:112:in `timeout'
        /usr/share/metasploit-framework/lib/msf/core/rpc/v10/service.rb:143:in `process'
        /usr/share/metasploit-framework/lib/msf/core/rpc/v10/service.rb:81:in `on_request_uri'
        /usr/share/metasploit-framework/lib/msf/core/rpc/v10/service.rb:62:in `block in start'
        /usr/share/metasploit-framework/lib/rex/proto/http/handler/proc.rb:38:in `on_request'
        /usr/share/metasploit-framework/lib/rex/proto/http/server.rb:369:in `dispatch_request'
        /usr/share/metasploit-framework/lib/rex/proto/http/server.rb:303:in `on_client_data'
        /usr/share/metasploit-framework/lib/rex/proto/http/server.rb:162:in `block in start'
        /usr/share/metasploit-framework/vendor/bundle/ruby/3.0.0/gems/rex-core-0.1.27/lib/rex/io/stream_server.rb:42:in `on_client_data'
        /usr/share/metasploit-framework/vendor/bundle/ruby/3.0.0/gems/rex-core-0.1.27/lib/rex/io/stream_server.rb:185:in `block in monitor_clients'
        /usr/share/metasploit-framework/vendor/bundle/ruby/3.0.0/gems/rex-core-0.1.27/lib/rex/io/stream_server.rb:184:in `each'
        /usr/share/metasploit-framework/vendor/bundle/ruby/3.0.0/gems/rex-core-0.1.27/lib/rex/io/stream_server.rb:184:in `monitor_clients'
        /usr/share/metasploit-framework/vendor/bundle/ruby/3.0.0/gems/rex-core-0.1.27/lib/rex/io/stream_server.rb:64:in `block in start'
        /usr/share/metasploit-framework/lib/rex/thread_factory.rb:22:in `block in spawn'
        /usr/share/metasploit-framework/lib/msf/core/thread_manager.rb:105:in `block in spawn'
        /usr/share/metasploit-framework/vendor/bundle/ruby/3.0.0/gems/logging-2.3.0/lib/logging/diagnostic_context.rb:474:in `block in create_with_logging_context' - ThreadError can't alloc thread
    ```
    * This occurred when closing the Armitage APP
* Module Loading (non-fatal) 
    ```
    [09/07/2023 14:23:25] [w(0)] core: The following modules could not be loaded!
    [09/07/2023 14:23:25] [w(0)] core:      /usr/share/metasploit-framework/modules/auxiliary/scanner/msmail/exchange_enum.go
    [09/07/2023 14:23:25] [w(0)] core:      /usr/share/metasploit-framework/modules/auxiliary/scanner/msmail/onprem_enum.go
    [09/07/2023 14:23:25] [w(0)] core:      /usr/share/metasploit-framework/modules/auxiliary/scanner/msmail/host_id.go
    ```

### New Kali
The new kali has not generated errors as far as I have been able to see. 

* Module Loading (non-fatal)
    ```
    [09/08/2023 10:48:32] [e(0)] core: /usr/share/metasploit-framework/modules/auxiliary/scanner/msmail/onprem_enum.go failed to load - LoadError Failed to execute external Go module. Please ensure you have Go installed on your environment.
    [09/08/2023 10:48:32] [w(0)] core: The following modules could not be loaded!
    [09/08/2023 10:48:32] [w(0)] core:      /usr/share/metasploit-framework/modules/auxiliary/scanner/msmail/host_id.go
    [09/08/2023 10:48:32] [w(0)] core:      /usr/share/metasploit-framework/modules/auxiliary/scanner/msmail/exchange_enum.go
    [09/08/2023 10:48:32] [w(0)] core:      /usr/share/metasploit-framework/modules/auxiliary/scanner/msmail/onprem_enum.go
    ```
* Thread Generation
    ```
    [09/08/2023 10:06:29] [e(0)] core: Thread Exception: WebConsoleShell  critical=false    source:
        /usr/share/metasploit-framework/lib/msf/ui/web/web_console.rb:84:in `initialize'
        /usr/share/metasploit-framework/lib/msf/ui/web/driver.rb:62:in `new'
        /usr/share/metasploit-framework/lib/msf/ui/web/driver.rb:62:in `create_console'
        /usr/share/metasploit-framework/lib/msf/core/rpc/v10/rpc_console.rb:28:in `rpc_create'
        /usr/share/metasploit-framework/lib/msf/core/rpc/v10/service.rb:143:in `block in process'
        /usr/share/metasploit-framework/vendor/bundle/ruby/3.1.0/gems/timeout-0.4.0/lib/timeout.rb:186:in `block in timeout'
        /usr/share/metasploit-framework/vendor/bundle/ruby/3.1.0/gems/timeout-0.4.0/lib/timeout.rb:41:in `handle_timeout'
        /usr/share/metasploit-framework/vendor/bundle/ruby/3.1.0/gems/timeout-0.4.0/lib/timeout.rb:195:in `timeout'
        /usr/share/metasploit-framework/lib/msf/core/rpc/v10/service.rb:143:in `process'
        /usr/share/metasploit-framework/lib/msf/core/rpc/v10/service.rb:81:in `on_request_uri'
        /usr/share/metasploit-framework/lib/msf/core/rpc/v10/service.rb:62:in `block in start'
        /usr/share/metasploit-framework/lib/rex/proto/http/handler/proc.rb:38:in `on_request'
        /usr/share/metasploit-framework/lib/rex/proto/http/server.rb:369:in `dispatch_request'
        /usr/share/metasploit-framework/lib/rex/proto/http/server.rb:303:in `on_client_data'
        /usr/share/metasploit-framework/lib/rex/proto/http/server.rb:162:in `block in start'
        /usr/share/metasploit-framework/vendor/bundle/ruby/3.1.0/gems/rex-core-0.1.31/lib/rex/io/stream_server.rb:42:in `on_client_data'
        /usr/share/metasploit-framework/vendor/bundle/ruby/3.1.0/gems/rex-core-0.1.31/lib/rex/io/stream_server.rb:185:in `block in monitor_clients'
        /usr/share/metasploit-framework/vendor/bundle/ruby/3.1.0/gems/rex-core-0.1.31/lib/rex/io/stream_server.rb:184:in `each'
        /usr/share/metasploit-framework/vendor/bundle/ruby/3.1.0/gems/rex-core-0.1.31/lib/rex/io/stream_server.rb:184:in `monitor_clients'
        /usr/share/metasploit-framework/vendor/bundle/ruby/3.1.0/gems/rex-core-0.1.31/lib/rex/io/stream_server.rb:64:in `block in start'
        /usr/share/metasploit-framework/lib/rex/thread_factory.rb:22:in `block in spawn'
        /usr/share/metasploit-framework/lib/msf/core/thread_manager.rb:105:in `block in spawn' - ThreadError can't alloc thread
    ```

## MSF Scan Error 
```
[09/08/2023 11:58:18] [e(0)] core: Thread Exception: WebConsoleShell  critical=false    source:
    /usr/share/metasploit-framework/lib/msf/ui/web/web_console.rb:84:in `initialize'
    /usr/share/metasploit-framework/lib/msf/ui/web/driver.rb:62:in `new'
    /usr/share/metasploit-framework/lib/msf/ui/web/driver.rb:62:in `create_console'
    /usr/share/metasploit-framework/lib/msf/core/rpc/v10/rpc_console.rb:28:in `rpc_create'
    /usr/share/metasploit-framework/lib/msf/core/rpc/v10/service.rb:143:in `block in process'
    /usr/lib/ruby/3.0.0/timeout.rb:97:in `block in timeout'
    /usr/lib/ruby/3.0.0/timeout.rb:35:in `block in catch'
    /usr/lib/ruby/3.0.0/timeout.rb:35:in `catch'
    /usr/lib/ruby/3.0.0/timeout.rb:35:in `catch'
    /usr/lib/ruby/3.0.0/timeout.rb:112:in `timeout'
    /usr/share/metasploit-framework/lib/msf/core/rpc/v10/service.rb:143:in `process'
    /usr/share/metasploit-framework/lib/msf/core/rpc/v10/service.rb:81:in `on_request_uri'
    /usr/share/metasploit-framework/lib/msf/core/rpc/v10/service.rb:62:in `block in start'
    /usr/share/metasploit-framework/lib/rex/proto/http/handler/proc.rb:38:in `on_request'
    /usr/share/metasploit-framework/lib/rex/proto/http/server.rb:369:in `dispatch_request'
    /usr/share/metasploit-framework/lib/rex/proto/http/server.rb:303:in `on_client_data'
    /usr/share/metasploit-framework/lib/rex/proto/http/server.rb:162:in `block in start'
    /usr/share/metasploit-framework/vendor/bundle/ruby/3.0.0/gems/rex-core-0.1.27/lib/rex/io/stream_server.rb:42:in `on_client_data'
    /usr/share/metasploit-framework/vendor/bundle/ruby/3.0.0/gems/rex-core-0.1.27/lib/rex/io/stream_server.rb:185:in `block in monitor_clients'
    /usr/share/metasploit-framework/vendor/bundle/ruby/3.0.0/gems/rex-core-0.1.27/lib/rex/io/stream_server.rb:184:in `each'
    /usr/share/metasploit-framework/vendor/bundle/ruby/3.0.0/gems/rex-core-0.1.27/lib/rex/io/stream_server.rb:184:in `monitor_clients'
    /usr/share/metasploit-framework/vendor/bundle/ruby/3.0.0/gems/rex-core-0.1.27/lib/rex/io/stream_server.rb:64:in `block in start'
    /usr/share/metasploit-framework/lib/rex/thread_factory.rb:22:in `block in spawn'
    /usr/share/metasploit-framework/lib/msf/core/thread_manager.rb:105:in `block in spawn'
    /usr/share/metasploit-framework/vendor/bundle/ruby/3.0.0/gems/logging-2.3.0/lib/logging/diagnostic_context.rb:474:in `block in create_with_logging_context' - IOError closed stream
```
* Scan Failed 


Error but Scan did not fail. This was seen once over 28 scans 
```
[09/08/2023 12:13:12] [d(0)] core: HistoryManager.push_context name: :msfconsole
[09/08/2023 12:13:12] [w(0)] core: monitor_rsock: exception during read: Errno::EAGAIN Resource temporarily unavailable
[09/08/2023 12:13:12] [d(0)] core: HistoryManager.pop_context name: :msfconsole
[09/08/2023 12:13:12] [e(0)] core: Thread Exception: WebConsoleShell  critical=false    source:
    /usr/share/metasploit-framework/lib/msf/ui/web/web_console.rb:84:in `initialize'
    /usr/share/metasploit-framework/lib/msf/ui/web/driver.rb:62:in `new'
    /usr/share/metasploit-framework/lib/msf/ui/web/driver.rb:62:in `create_console'
    /usr/share/metasploit-framework/lib/msf/core/rpc/v10/rpc_console.rb:28:in `rpc_create'
    /usr/share/metasploit-framework/lib/msf/core/rpc/v10/service.rb:143:in `block in process'
    /usr/lib/ruby/3.0.0/timeout.rb:97:in `block in timeout'
    /usr/lib/ruby/3.0.0/timeout.rb:35:in `block in catch'
    /usr/lib/ruby/3.0.0/timeout.rb:35:in `catch'
    /usr/lib/ruby/3.0.0/timeout.rb:35:in `catch'
    /usr/lib/ruby/3.0.0/timeout.rb:112:in `timeout'
    /usr/share/metasploit-framework/lib/msf/core/rpc/v10/service.rb:143:in `process'
    /usr/share/metasploit-framework/lib/msf/core/rpc/v10/service.rb:81:in `on_request_uri'
    /usr/share/metasploit-framework/lib/msf/core/rpc/v10/service.rb:62:in `block in start'
    /usr/share/metasploit-framework/lib/rex/proto/http/handler/proc.rb:38:in `on_request'
    /usr/share/metasploit-framework/lib/rex/proto/http/server.rb:369:in `dispatch_request'
    /usr/share/metasploit-framework/lib/rex/proto/http/server.rb:303:in `on_client_data'
    /usr/share/metasploit-framework/lib/rex/proto/http/server.rb:162:in `block in start'
    /usr/share/metasploit-framework/vendor/bundle/ruby/3.0.0/gems/rex-core-0.1.27/lib/rex/io/stream_server.rb:42:in `on_client_data'
    /usr/share/metasploit-framework/vendor/bundle/ruby/3.0.0/gems/rex-core-0.1.27/lib/rex/io/stream_server.rb:185:in `block in monitor_clients'
    /usr/share/metasploit-framework/vendor/bundle/ruby/3.0.0/gems/rex-core-0.1.27/lib/rex/io/stream_server.rb:184:in `each'
    /usr/share/metasploit-framework/vendor/bundle/ruby/3.0.0/gems/rex-core-0.1.27/lib/rex/io/stream_server.rb:184:in `monitor_clients'
    /usr/share/metasploit-framework/vendor/bundle/ruby/3.0.0/gems/rex-core-0.1.27/lib/rex/io/stream_server.rb:64:in `block in start'
    /usr/share/metasploit-framework/lib/rex/thread_factory.rb:22:in `block in spawn'
    /usr/share/metasploit-framework/lib/msf/core/thread_manager.rb:105:in `block in spawn'
    /usr/share/metasploit-framework/vendor/bundle/ruby/3.0.0/gems/logging-2.3.0/lib/logging/diagnostic_context.rb:474:in `block in create_with_logging_context' - IOError stream closed in another thread
```
#### Normal Logs

The Following logs are generated when a Armitage MSF Console is created
```
09/08/2023 12:07:22] [w(0)] core: The following modules could not be loaded!
[09/08/2023 12:07:22] [w(0)] core:      /usr/share/metasploit-framework/modules/auxiliary/scanner/msmail/exchange_enum.go
[09/08/2023 12:07:22] [w(0)] core:      /usr/share/metasploit-framework/modules/auxiliary/scanner/msmail/onprem_enum.go
[09/08/2023 12:07:22] [w(0)] core:      /usr/share/metasploit-framework/modules/auxiliary/scanner/msmail/host_id.go
[09/08/2023 12:07:22] [d(0)] core: HistoryManager.push_context name: :msfconsole
```


The Following Logs are generated when a Armitage Console Is closed 
```
[09/08/2023 12:09:45] [d(0)] core: HistoryManager.pop_context name: :msfconsole
[09/08/2023 12:09:45] [d(0)] core: monitor_rsock: EOF in rsock
```

## Exploit & Scanning Results
See the [exploit google sheet](https://docs.google.com/spreadsheets/d/15_0rrRfq31w_6zAp9RyuIbCWcmZXz0Ex1_gfmTfyYoM/edit?usp=sharing) provided for more details. The main thing to note is out of 20 times Armitage was run on the New Kali with New MSF system if failed 0 times, and errors in the MSF logs were observed 0 times. 

See the [MSF Scan google sheet](https://docs.google.com/spreadsheets/d/1q09IqhhHdoiVH2intBxbvQAaEAoxcz1m6NXZ3LtKM9I/edit?usp=sharing) provided for more details. The main thing to note is that the MSF scan failed 3 times out of 20 (With 4 observed errors) in the Old kali, with it failing 0 times and no observed errors in the new Kali and MSF system. I can also note that later uses of the MFS scan failed, showing poorer performance over time.

See the [NMAP google Sheet](https://docs.google.com/spreadsheets/d/16HaFhDpn895UwkTSc3k9G0QJOdUyeW_SxGld3nmStmE/edit?usp=sharing) provided for more details. Mainly the NMAP scans succeded but took over 40 seconds on average when the VulnServer application was running on the target. This is because NMAP attempts to connect to the system multiple times as the ```-sV``` flag signals that nmap should attempt to determine the version of the applications running on the system. Overall NMAP only failed once, this was due to system usage. 


## Armitage Code 
This is quite... pain inducing. The code had and still has limited comments, and is a mess of tangled dependencies with hard to read code held together by a niche scripting language the author created. That said it manages to work surprisingly well.

### Sleep
[Sleep](http://sleep.dashnine.org/) is a interpreted scripting language, It combines Pearl, and Objective-C syntax, That seem to result in a Bash and Java like syntax too. This makes one very hard to read language when there is a lack of **Comments**. The Arguments are referenced like Bash function arguments, that is they are referenced based on position, not name. Additionally you do not provide the number of expected arguments in the function definition meaining you have to trace back to the first function call for any context if no comments are left. 

One of the more painful things is that when a script is loaded by an engine, the functions, and more annoyingly the global variables seem to be accessible to *all* other scripts loaded in the same context (engine) this is how Armitage combines 27 scripts ranging from 100 to 2000 lines together so that the author could glue together a few tens of thousands of Java code lines. Almost all of this code is uncomments which does not help in the debugging and understanding process . 

You can find more information about sleep on it's site [http://sleep.dashnine.org/](http://sleep.dashnine.org/) or assuming it goes the same way Armitage's site went on [Internet Archive](https://web.archive.org/web/20230412064619/http://sleep.dashnine.org/) I downloaded the documentation as well which can be found in [PDFs](./PDFs/sleep21manual.pdf).


### Java Code 

This will mainly be a list of areas I though were notable at one time 
* [https://www.gitlab.com/ChrisM09/armitage/blob/4648028ca880b387da62b4269af9029bbd206d10/armitage/src/main/java/msf/RpcConnectionImpl.java#L19-L20](https://www.gitlab.com/ChrisM09/armitage/blob/4648028ca880b387da62b4269af9029bbd206d10/armitage/src/main/java/msf/RpcConnectionImpl.java#L19-L20)
  * This is the location that defines the execute methods used in the Sleep Code to send requests 
    * [https://www.gitlab.com/ChrisM09/armitage/blob/4648028ca880b387da62b4269af9029bbd206d10/armitage/src/main/resources/scripts/armitage.sl#L189](https://www.gitlab.com/ChrisM09/armitage/blob/4648028ca880b387da62b4269af9029bbd206d10/armitage/src/main/resources/scripts/armitage.sl#L189)
    * This is a class that is extended by MsgRpcImpl used in the sleep code
* [https://www.gitlab.com/ChrisM09/armitage/blob/4648028ca880b387da62b4269af9029bbd206d10/armitage/src/main/java/armitage/ConsoleQueue.java#L139](https://www.gitlab.com/ChrisM09/armitage/blob/4648028ca880b387da62b4269af9029bbd206d10/armitage/src/main/java/armitage/ConsoleQueue.java#L139)
  * This is where the processing of commands starts. 
* [https://www.gitlab.com/ChrisM09/armitage/blob/4648028ca880b387da62b4269af9029bbd206d10/armitage/src/main/java/armitage/ArmitageMain.java#L275](https://www.gitlab.com/ChrisM09/armitage/blob/4648028ca880b387da62b4269af9029bbd206d10/armitage/src/main/java/armitage/ArmitageMain.java#L275)
  * Entrypoint of the program
  * You can see which sleep scripts will be run in this file


There are alot of Queues used by Armitage, the most important is the [Console Queue](https://www.gitlab.com/ChrisM09/armitage/blob/4648028ca880b387da62b4269af9029bbd206d10/armitage/src/main/java/armitage/ConsoleQueue.java#L13) as that seems to fire off events, and uses the many other queues defined in the project... 

### Build Process 
I use a workaround since the CLI Gradle was not working all that well. 

1. Install [IntelliJ IDEA IDE](https://www.jetbrains.com/idea/download/?section=windows)
2. Open the Armitage Project
3. Run the Gradle Build from the IDE -- this works
   * I had an odd amount of trouble running the packaging script which uses the CLI Gradle commands
4. Comment out Gradle commands in the packaging script
5. Run the Packaging Script
6. Navigate to armitage/release/unix and you are all set


## Notes
In general the project can be improved with comments, and general refactoring of the code to increase readability, and improve performance (If that is possible). Mostly focused on the readability aspects and noting down what a function actually needs and does...

We can also update the terminology, add additional NMap options, among other possibilities. 
