# Armitage Notes <!-- omit from toc -->
These are notes I have taken when debugging the Armitage system. This was done at the request of Prof. Fu to fix some bugs he observed during his classes. 

This document contains an overview of the systems and some observations made throughout. 

## Table of Contents <!-- omit from toc -->
- [Systems](#systems)
- [Log file locations](#log-file-locations)
- [Exploit Error Messages](#exploit-error-messages)
  - [OLD Kali](#old-kali)
  - [New Kali](#new-kali)
- [MSF Scan Error](#msf-scan-error)
    - [Normal Logs](#normal-logs)
- [Exploit \& Scanning Results](#exploit--scanning-results)


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

## Log file locations 
Metasploit: 
* The logs will be located at ```~/.msf4/logs/framework.log```, if you run armitage without sudo, they will be located in the user's ```.msf``` directory. If sudo is used then they will be located in the root user's home directory (```sudo su``` then the previous path will work)

Armitage:
* The "logs" if they can be called that are located at ```~/.armitage/<YearMothDay>/localhost/target/<attackType>.log```
  * The ```<YearMonthDay>``` is in the form of concatinated number for example 9/8/23 is ```230908```
  * Example Logfile ```~/.armitage/230908/localhost/10.0.2.4/auxiliary.log```
    * ```10.0.2.4``` can be replaced with "all"


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

See the [NMAP google Sheet](https://docs.google.com/spreadsheets/d/16HaFhDpn895UwkTSc3k9G0QJOdUyeW_SxGld3nmStmE/edit?usp=sharing) provided for more details. Mainly the NMAP scans did only failed once, this was due to system usage. 