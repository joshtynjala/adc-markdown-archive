# AIR compiler known issues

AIR 13 introduces a redesigned compiled packaging mode. It aims to considerably
reduce packaging time, for the same performance experience at runtime. The beta
version of the redesigned compiler is currently available to all developers.

Following are the known issues:

## Issue 1

While debugging, any changes to the variable's value is not reflected in
subsequent executions.

#### Steps to reproduce the issue

1.  Create an ActionScript mobile project in Flash Builder 4.7.
2.  Add a variable and print its value (for example,
    `var x:int = 10; trace ( x );`)
3.  Navigate to _Debug Configurations_ and in the _Customize Launch Parameters_
    dialog box, add `-useLegacyAOT=no` before `-provisioning-profile.`
4.  Place a breakpoint at the trace statement and start debugging on the device.
5.  After the breakpoint is hit, modify the value of the variable.

The trace statement prints the old value.

#### Workaround

None.

## Issue 2

If the base class function doesn't use any arguments and the overridden function
in the derived class does, then any function calls through the base class
reference can cause the application to terminate abruptly.

#### Sample Code

    class ClassA {
        public function func():Object {
            return null;
        }
    }
    class ClassB extends ClassA {
        public override function func():Object  {
            trace(arguments.length);
            return _staticObj;
        }
        private static var _staticObj:Object = {member:false};
    }
    var obj:ClassA = new ClassB();
    obj.func();

#### Workaround

Either use arguments in both the base and derived classes, or avoid using any
arguments in both the classes.

## Issue 3

The new AIR Compiler is not supported on Windows XP. Using the new compiler on
Windows XP throws the error, _compile-abc.exe is not a valid win32 application_.

> This work is licensed under a
> [Creative Commons Attribution-Noncommercial-Share Alike 3.0 Unported License](https://creativecommons.org/licenses/by-nc-sa/3.0/)
