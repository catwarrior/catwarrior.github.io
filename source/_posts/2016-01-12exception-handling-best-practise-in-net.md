title: exception handling best practise in .net
date: 2016-01-12 09:42:03
categories:
- coding
tags:
- c#
- .net
- exception
---

cite from http://www.codeproject.com/Articles/9538/Exception-Handling-Best-Practices-in-NET

- Plan for the worst
- Check it early
- Don't trust external data
- The only reliable devices are: the video, the mouse and keyboard.
- Writes can fail, too
- Code Safely
- Don't throw new Exception()
- Don't put important exception information on the Message field
- Put a single catch (Exception ex) per thread
- Generic Exceptions caught should be published
- Log Exception.ToString(); never log only Exception.Message!
- Don't catch (Exception) more than once per thread
- Don't ever swallow exceptions
- Cleanup code should be put in finally blocks
- Use "using" everywhere
- Don't return special values on error conditions
- Don't use exceptions to indicate absence of a resource
- Don't use exception handling as means of returning information from a method
- Use exceptions for errors that should not be ignored
- Don't clear the stack trace when re-throwing an exception
- Avoid changing exceptions without adding semantic value
- Exceptions should be marked [Serializable]
- When in doubt, don't Assert, throw an Exception
- Each exception class should have at least the three original constructors
- Be careful when using the AppDomain.UnhandledException event
- Don't reinvent the wheel
