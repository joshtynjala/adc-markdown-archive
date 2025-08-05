# Developing cross-platform Adobe AIR applications

by Joe Ward

![Joe Ward](./img/1313179592546.jpg)

## Requirements

### User level

All

### Required products

- [Adobe AIR SDK](https://airsdk.dev/) or
  [Apache Flex SDK](https://flex.apache.org)

This article discusses some of the best practices for developing AIR
applications that work well on all supported platforms and operating systems.

AIR applications run on any platform and operating system that support the AIR
runtime, including Mac OS X, Linux, and Windows. The AIR runtime lets you
concentrate on creating the unique functionality of your application without
worrying too much about operating system and platform differences. However, when
dealing with those features that interact directly with the operating system,
you should consider the best practices discussed in this article to avoid
pitfalls that will prevent your application from working everywhere it could.

No two operating systems provide the exact same feature set. Although you can
create expressive applications using only those features of the operating system
available everywhere, there may be times when you want to take advantage of an
OS-specific feature. For example, window menus would seem very out of place to
users on Mac OS X. AIR enables you to go beyond the lowest common set of
features of the supported operating systems. But, when you use platform-specific
features, you should consider the best practices discussed here to make sure
that you don't accidentally limit the reach of your applications.

### Best practices

#### Test!

Even the most scrupulous adherence to best practices cannot replace the need for
testing. The most important areas to test for cross-platform issues are file
system access, window behavior, and networking. Hardware virtualization
technology such as _VMware_ or _Parallels_ can reduce the hardware costs of
cross-platform testing.

#### Detect capabilities, not operating systems

AIR provides several properties for detecting whether the computer on which your
application is running supports a particular feature or capability. Wherever
possible, you should use these properties rather than detecting the operating
system itself.

For example, Mac OS X supports application-style menus that are displayed on the
desktop menu bar. When you create a menu for your application, you should use
the `NativeApplication.supportsMenu` property to determine whether application
menus are supported.

The AIR properties to use for detecting operating system capabilities are:

- `NativeApplication.supportsDockIcon`
- `NativeApplication.supportsMenu`
- `NativeApplication.supportsSystemTrayIcon`
- `NativeWindow.supportsMenu`
- `NativeWindow.supportsTransparency`

You should use these properties rather than checking Capabilities.os or using
other means to detect a specific operating system. Logic such as that in the
following pseudo-code may seem to work, but it can lead to problems on Linux or
other operating systems:

    if (Windows)
    {
        // Use system tray
    }
    else
    {
        // Use dock
    }

#### Allow for future platforms and operating system versions

There are a few cases where detecting the operating system or platform is
desirable. For example, if you are implementing custom chrome in your
application, you may want to place the window buttons on the left side of the
window on Mac OS X and the right side on Windows and Linux. Adobe plans to
extend the reach of AIR to even more platforms in the future, so always make
sure that the application behaves reasonably when run on an operating system or
platform that you did not expect. This can be as simple as providing a fall-back
option when your platform detection code fails to identify a known operating
system, as illustrated by the following pseudo-code:

    if ( Windows ) {
        //Windows specific code
    } else if ( Mac ) {
        //Mac specific code
    } else if ( Linux ) {
        //Linux specific code
    } else {
        //Fall-back code
    }

#### Never write to the application directory

AIR does not let you write to the application directory by default, because the
directory is not writable to all user accounts on all operating systems. There
are ways to get around the AIR restriction, but if you use them, your code will
fail on systems on which the operating system itself enforces write protection.

If your application needs to install editable assets, such as a database file,
you should copy or move those assets from the application directory to the
application storage directory the first time your application is run.

#### Use the appropriate File property when referencing well-known folders

AIR provides several properties for referencing well-known, standard directories
on different operating systems. These include:

- `File.applicationDirectory`: The read-only directory in which your application
  is installed.
- `File.applicationStorageDirectory`: A directory for storing external
  application assets, such as writable files, downloaded images, and so on. This
  directory can be difficult for a user to locate through the file system, so
  user documents are better kept in the documents directory.
- `File.desktopDirectory`: The desktop folder.
- `File.documentsDirectory`: The user's documents folder. Files that a user
  expects to use outside your application, such as edited pictures or text
  files, should be stored in a suitable subdirectory of the documents folder.
- `File.userDirectory`: The user directory.

Each of these properties provides a File object referencing the appropriate
directory. Use the `resolvePath()` method to create or access a file within one
of these directories. For example, the following code statement creates a File
object referencing a file in the user's documents directory:

File.documentsDirectory.resolvePath("myFile.xyz");

You can also use the `File.createTempDirectory()` and `File.createTempFile()`
methods to create temporary folders and files. (Note that these files are not
automatically cleaned up, so your application should delete them from disk when
through with them.)

#### Always treat file systems as case sensitive

Although not all file systems are case sensitive, your code will not fail by
treating them as if they were. On the other hand, treating a case-sensitive file
system as non–case-sensitive can lead to severe bugs.

One way to catch errors in file case when you develop on a case-insensitive file
system is to check that the canonicalized name of a File object matches the
requested name. For example, the following function returns true if the
requested file name matches the canonicalized file name:

    public function checkFileCase( path:String ):Boolean
    {
        var file:File = new File( path );
        var requestedName:String = file.name;
        file.canonicalize();
        if ( file.name == requestedName )
        {
            return true;
        } else
        {
           return false; //Case is different
        }
    }

You could use a function such as this in debug code to detect when your
application writes and reads a file using mismatched file name case.

#### Use File URLs instead of native paths

Whenever possible, use URLs rather than native paths. The File class provides
both a `url` and a `nativePath` property.

Many AIR, JavaScript, HTML, and Flex properties and methods expect a URL string.
When setting such properties or parameters, be sure to use the File object `url`
property (or a properly formatted URL string) rather than the `nativePath`
property. If you use the `nativePath` property of a File object to set a URL, it
will be interpreted differently on different platforms. On Windows, a native
path used as a URL would be interpreted as an absolute URL. However, on Mac and
Linux, a native path used as a URL is indistinguishable from a relative URL.
This can lead to situations in which code that seems to work on Windows, fails
on Mac OS X and Linux.

#### Use File.separator when parsing or constructing paths

AIR provides the File.separator property so that your code can always use the
correct separator character when parsing or constructing paths. Use it when you
must work with native paths.

#### Use the File.getRootDirectories() method to get the root directories

`File.getRootDirectories` will give you an array containing File objects
referencing the root directories of the user's computer. On Mac OS X and Linux,
the array will contain the single object referencing the root directory ("`/`").
On Windows, the array will contain an object for each logical drive that is
assigned a letter ("`C:\`", "`E:\`", and so on). Other platforms, or even a new
version of an operating system on one of these platforms, might have a different
convention.

#### Use File.lineEnding when reading and writing files

AIR provides the `File.lineEnding` property so that your code can always use the
correct line ending character when parsing or writing files.

#### Treat changes to the size of native windows as asynchronous

In some environments, any changes to the size or position of a window, including
maximizing or minimizing the window, are completed asynchronously. This means
that if you set a property such as the window width in one line of code and read
it in the next, then the property will still reflect the old value.

If you need to take some action, such as laying out controls in the window based
on the new size or state of the window, you should call this code from the
handlers for the related events, such as the native window `resize`, `move`, or
`displayStateChange` events or the stage object's `resize` or `fullscreen`
events.

The native window operations that are asynchronous on some platforms and
synchronous on others are:

- Setting` x`, `y`, `width`, `height` and `bounds`
- `maximize()`
- `minimize()`
- `restore()`
- Setting the stage `displayState`
- `activate()`

#### Use window types appropriately

Different operating systems, and even different window managers on a single
system, impose different rules on native windows based on the type. Utility-type
windows in particular are often subject to differing rules about whether they
can be maximized and how they can be ordered among other windows. In general, if
you use the _normal_ type for standard application windows, the _utility_ type
for dialog boxes and tool palettes, and the _lightweight_ type for special
purposes or temporary display surfaces (such as tooltips, drop-down lists, and
so on), your windows will behave intuitively on all operating systems.

#### Use window transparency sparingly

Large transparent windows can consume an inordinate share of the available
processor resources. Some platforms perform better than others in this regard.
In particular, you should avoid implementing special effects or pseudo-windowing
code using an invisible full-screen window.

On Linux, even completely transparent areas block mouse clicks from reaching
other windows. Depending on the window manager and user settings, transparency
may not be supported at all and transparent areas will be composited against
black. You can detect whether transparency is supported using the
`NativeWindow.supportsTransparency` property.

#### Follow the menu conventions of each operating system

Menus are an area in which user expectations are strongly tied to operating
system conventions. When application menus are available
(`NativeApplication.supportsMenu` is `true`), you should create an application
menu conforming to Mac OS X guidelines. When window menus are available
(`NativeWindow.supportsMenu` is `true`), you should create window menus
following Microsoft or Linux conventions (which are very similar to each other).

Guidelines for menu design on the currently supported platforms are available in
the following documents:

- Mac OS X:
  [Apple Human Interface Guidelines](https://web.archive.org/web/20170718140841/http://developer.apple.com/documentation/UserExperience/Conceptual/AppleHIGuidelines/XHIGMenus/chapter_17_section_1.html#//apple_ref/doc/uid/TP30000356-TP6)
- Windows:
  [Windows Vista Experience Guide](https://web.archive.org/web/20170718140841/http://msdn.microsoft.com/en-us/library/aa511502.aspx)
- Linux:
  [KDE User Interface Guidelines](https://web.archive.org/web/20170718140841/http://developer.kde.org/documentation/standards/kde/style/menus/index.html)
  or
  [Gnome Human Interface Guidelines](https://web.archive.org/web/20170718140841/http://library.gnome.org/devel/hig-book/stable/menus.html.en)

#### AIR classes with platform-dependent behavior

The following list describes the AIR classes that either implement
platform-specific features or that behave differently on different platforms:

- **Clipboard:** On some operating systems, the clipboard contents can still be
  accessed after an application has closed. On others, the clipboard contents
  are no longer available.
- **DockIcon:** The DockIcon represents the application icon displayed on the
  desktop dock. Currently, AIR only supports dock icons on Mac OS X. Use
  `NativeApplication.supportsDockIcon` to detect whether dock icons are
  supported.
- **File:** Many of the cross-platform file system considerations have been
  discussed earlier in this article. The key point to remember is to avoid
  making assumptions about the syntax of file paths based on the operating
  system on which you develop. In addition, file URLs can usually be used
  instead of native paths and work on all platforms.
- **InvokeEvent:** Some operating systems dispatch a single InvokeEvent object
  referencing several File objects when a user opens more than one selected file
  of a type registered to an AIR application. Others dispatch a separate
  InvokeEvent object for each selected file. The best way to handle this is to
  always process every file in the InvokeEvent argument list.
- **NativeDragEvent:** `NativeDragUpdate` and `NativeDragOver` events are not
  dispatched by all operating systems. Since these events can be used to make a
  UI more responsive, you should use these events where appropriate to add to
  the user experience, but you must never rely on them.
- **NativeMenu:** There are two styles of menu. Application menus are displayed
  as part of the desktop. Window menus are displayed as part of the window. You
  can use the same menu code for both types of menus. However, this is an area
  where user expectations can be strongly tied to the operating system. The
  static properties `NativeApplication.supportsMenu` and
  `NativeWindow.supportsMenu` can be used to detect which type of menu is
  supported.
- **NativeWindow:** Many of the cross-platform file system considerations have
  been discussed earlier in this article. The most significant differences
  involve support for transparency and the asynchronous behavior encountered
  when setting window properties. In addition, on Linux, resizing, moving, and
  `displayStateChanging` events cannot be canceled.
- **SystemTrayIcon:** The SystemTrayIcon represents the application icon
  displayed on the taskbar dock, typically in the notification area. Currently,
  AIR supports system tray icons on Windows and Linux. Use
  `NativeApplication.supportsSystemTrayIcon` to detect whether system tray icons
  are supported.

### Where to go from here

Developing AIR applications that work on all platforms is generally very easy.
However, when you work with the file system and native windows you should avoid
making assumptions that apply only to the operating system on which you develop.
Likewise, when you implement features specific to a particular operating system,
you must make sure that your application is still usable without that feature
when run on a different operating system. Following the best practices discussed
in this article—and testing—will help give your applications the broadest reach
possible with the least effort.
