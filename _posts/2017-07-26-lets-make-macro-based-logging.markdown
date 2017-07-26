---
published: true
title: Lets make a macro based Logging
layout: post
tags: [CPP, macro, logging]
categories: [Automatic]
---

# Lets make a macro based Logging

I have been attached to a software project for an Embedded Linux system.
Currently, our project is infested with  a lot of logs.
These logs are "printed" on both stdout and a log file on target system.
Also, log outputs are being put to "output" always.
So I tried to find a way to keep current logging functions almost intact while extending the current functionality.

Right now, our `WriteLogMsg` function and `VDPRINTF` & `DPRINTF` macros looks like this:

    #define DPRINTF(fmt, ...)
    #define VDPRINTF(fmt, ...)

    //...
    
    void WriteLogMsg(const char *format, ...) {
        struct tm *tm_tm;
        char tmBuf[64] = {0};
        time_t ltimes;
        va_list args;
    
        time(&ltimes);
        tm_tm = localtime(&ltimes);
        strftime(tmBuf, sizeof(tmBuf) - 1, "%Y %b %d %H:%M:%S", tm_tm);
    
        DPRINTF("%s ", tmBuf);
    
        va_start(args, format);
    
        VDPRINTF(format, args);
    
        va_end(args);
    }
    
This function along with two macros are sufficient for our current stage, e.g. Development.
But as we move toward Release and Production the urge for a clean and understandable log format arise.

As you can see from the codes, output of these log messages lack some crucial details such as source code file, function name and line.
These are most common needed details and they help a lot when debugging embedded applications.

So, back to our goals.
First let's see if we can introduce a method to write log messages without doing major changes to available codes.
From top of my head, two approaches are available:
Either modify implementation of `WriteLogMsg` or use Macros.
I haven't worked with macros although they are used in our project, so I took the challenge of implementing our goal through macros.

Supposedly, future implementation of `WriteLogMsg` will look like this:

    void WriteLogMsg(const char *format, ...) {
        LOG_MACRO(format, args); // psudo-code
    }
    
Let's dive in. I will define LOG_MACRO to start.
We will use `__FILE__`, `__FUNCTION__` & `__LINE__` predefined macros to get the name of source file, function & line number, accordingly:

    #define LOG_MACRO(msg) printf("[%s->%s->%d] %s", __FILE__, __FUNCTION__, __LINE__, msg)
    
To test the output I have created a sample app as follows:

    #include <stdio.h>

    #define LOG_MACRO(msg) printf("[%s->%s->%d] %s", __FILE__, __FUNCTION__, __LINE__, msg)
    
    using namespace std;
    
    int main(int argc, char *argv[]) {
        LOG_MACRO("Hello, Log!");
        return 0;
    }

This is the output:

>[main.cpp->main->12] Hello, Log!

Not bad, I have the source file name, function name and line number along with log message. Mind the "12" which is the line that actually called `LOG_MACRO("Hello, Log!");`.

_Please note: Value of "\_\_LINE\_\_" might be different depending on your code file._

Now, another question arise:
What if I want to log formatted messages?
Meaning, I would pass formatted message along with arguments to our macro.
This is a common case for logging,
something along `LOG_MACRO("Return value: %d", rv);`. 

To achieve this we need to modify our macro to accept 2 parameters: formatted message and arguments.

    #define LOG_MACRO(msg, ...) printf("[%s->%s->%d] %s", __FILE__, __FUNCTION__, __LINE__, msg,__VA_ARGS__)
    
`__VA_ARGS__` stand for anything passed inside `...`

Lets find out if our desired output is actually printed:

    //...
    int main(int argc, char *argv[]) {
        unsigned int rv = 256;
        LOG_MACRO("Return value: %d", rv);
        return 0;
    }

This is the output:

>[main.cpp->main->12] Return value: %d

OK, our macro cannot process passed arguments.
As you saw in the beginning of this post, `WriteLogMsg` accepts two arguments: a formatted message and its arguments.
We need to find a way to process formatted log messages.
We can solve this by introducing a second macro,
generate initial formatted output and then pass it to secondary macro which will produce final output:

    #define NEWLINE "\n"
    #define LOG_FMT "[%s->%s:%d] "
    #define LOG_ARGS __FILE__, __FUNCTION__, __LINE__
    #define PRINT_LOG(fmt, ...) printf(fmt, __VA_ARGS__)
    #define LOG_MACRO(s, args...) PRINT_LOG(LOG_FMT s NEWLINE, LOG_ARGS, ## args)
    using namespace std;
    
    int main(int argc, char *argv[]){
        unsigned int rv = 256;
        LOG_MACRO("Return value: %d", rv);
        return 0;
    } 

I have added three new macros, `NEWLINE`, `LOG_FMT` & `PRINT_LOG`.
`NEWLINE`, `LOG_FMT` are just placeholder for new line char and our format.
`PRINT_LOG` is the macro that will produce final output.
Also, I have modified `LOG_MACRO` to do the initial formatting.

This is the output:

>[main.cpp->main:17] Return value: 256

We have achieved our goal.

At this point I realize that I have missed another crucial detail: _Time_.
Unfortunately, there is no predefined macro for getting time.
We can get time using an inline function and use it along with our macro:

    #include <time.h>
    #include <string.h>
    static inline char *timenow();
    #define NEWLINE "\n"
    #define LOG_FMT "%s [%s->%s:%d] "
    #define LOG_ARGS timenow(), __FILE__, __FUNCTION__, __LINE__
    #define PRINT_LOG(fmt, ...) printf(fmt, __VA_ARGS__)
    #define LOG_MACRO(s, args...) PRINT_LOG(LOG_FMT s NEWLINE, LOG_ARGS, ## args)
    static inline char *timenow() {
        static char buffer[64];
        time_t rawtime;
        struct tm *timeinfo;

        time(&rawtime);
        timeinfo = localtime(&rawtime);

        strftime(buffer, 64, "%Y-%m-%d %H:%M:%S", timeinfo);

        return buffer;
    }
    
    #include <stdio.h>
    
    using namespace std;
    
    int main(int argc, char *argv[]){
        unsigned int rv = 256;
        LOG_MACRO("Return value: %d", rv);
        return 0;
    }
    
This is the output:

>2017-07-26 15:11:30 [main.cpp->main:12] Return value: 256

Great. but, this solution only covers the source file that our macro is present.
To make available to other source files we can move logging parts to a header file and include it where needed.

#### Contents of miniLog.h
    #ifndef MINILOG_H
    #define MINILOG_H
        #include <time.h>
        #include <string.h>
        static inline char *timenow();
        #define NEWLINE "\n"
        #define LOG_FMT "%s [%s->%s:%d] "
        #define LOG_ARGS timenow(), __FILE__, __FUNCTION__, __LINE__
        #define PRINT_LOG(fmt, ...) printf(fmt, __VA_ARGS__)
        #define LOG_MACRO(s, args...) PRINT_LOG(LOG_FMT s NEWLINE, LOG_ARGS, ## args)
        static inline char *timenow() {
            static char buffer[64];
            time_t rawtime;
            struct tm *timeinfo;

            time(&rawtime);
            timeinfo = localtime(&rawtime);

            strftime(buffer, 64, "%Y-%m-%d %H:%M:%S", timeinfo);

            return buffer;
        }
    #endif //MINILOG_H
    
#### Usage

    #include <stdio.h>
    #include "miniLog.h"
    
    using namespace std;
    
    int main(int argc, char *argv[]){
        unsigned int rv = 256;
        LOG_MACRO("Return value: %d", rv);
        return 0;
    }
    
## Refereences
* https://coderwall.com/p/v6u7jq/a-simplified-logging-system-using-macros
* https://stackoverflow.com/questions/3384912/define-log-msg-for-debugging
