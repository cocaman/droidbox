# Introduction #

Android is upgrading in a fast speed. To avoid endless porting of DroidBox, we changed the way to do dynamic analysis. Instead of hooking systems, we interpose APIs in APK files and insert monitoring code. By running the repackaged APK, we can get API call logs and understand APK's behavior.


# Requirements #
  * Linux/Unix/MacOSX
  * JRE
  * python 2.7
  * command "jarsigner" must work in your system


# Usage #


To start with, you can have a try as follows:
```
$ cd APIMonitor-beta
$ ./apimonitor.py example/APKILTests.apk
```
Now your apk file will be repackaged.
Run your app in real device or emulator and check the logs with tag "DroidBox".

Too see complete usage, just use _-h_:

```
$ ./apimonitor.py -h
usage: apimonitor.py [-h] [-o, --output dirpath] [-l, --level level] [-a, --api apilist] [-v, --version] filename

Repackage apk to monitor arbitrary APIs.

positional arguments:
  filename              path of APK file

optional arguments:
  -h, --help            show this help message and exit
  -o, --output dirpath  output directory
  -l, --level level     target API level for instrumentation
  -a, --api apilist     config file of API list
  -v, --version         show program's version number and exit
```

Default API list for monitoring lies in _config/default\_api\_collection_. It may be incomplete for now, but it will be more classified and sufficient in the near future.

You can also specify APIs to monitor yourself. Just put your set of method signatures in the file.

There are 3 kinds of signatures:
  * Signature of one method
  * Signature of methods with the same name
  * Signature of all methods of the same class

One signature each line as follows(NOTICE:You should **NOT** add return value):
```
# Signature of one method
Landroid/content/ContextWrapper;->openFileInput(Ljava/lang/String;)

# Signature of methods with the same name 
Landroid/content/Intent;-><init>

# Signature of all methods of the same class 
Landroid/content/ContentResolver;
```

Don't worry about inheritance of APIs. Just specify any callable API signatures in the right format. APIMonitor will deal with the inheritance relationship.