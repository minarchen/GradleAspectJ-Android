# GradleAspectJ-Android

A Gradle plugin which enables AspectJ for Android builds.
Supports writing code with AspectJ-lang in `.aj` files which then builds into annotated java class.
Full support of Android product flavors and build types.

Compilation order:
```groovy
  if (hasRetrolambda)
    retrolambdaTask.dependsOn(aspectCompileTask)
  else
    javaComplieTask.finalizedBy(aspectCompileTask)
```
This workaround is friendly with <a href="https://bitbucket.org/hvisser/android-apt" target="_blank">APT</a> (Android Annotation Processing Tools) and <a href="https://github.com/evant/gradle-retrolambda/" target="_blank">Retrolambda</a> project.
<a href="https://github.com/excilys/androidannotations" target="_blank">AndroidAnnotations</a>, <a href="https://github.com/square/dagger" target="_blank">Dagger</a> are also supported and works fine.
<a href="https://github.com/JakeWharton/butterknife" target="_blank">Butterknife</a> now doesn't support ad could works with bugs and errors. WIP on that problem.

This plugin based on <a href="https://github.com/uPhyca/gradle-android-aspectj-plugin/" target="_blank">uPhyca's plugin</a>.

Usage
-----

First add a maven repo link into your `repositories` block of module build file:
```groovy
mavenCentral()
maven { url 'https://github.com/Archinamon/GradleAspectJ-Android/raw/master' }
```
Don't forget to add `mavenCentral()` due to some dependencies inside AspectJ-gradle module.

Add the plugin to your `buildscript`'s `dependencies` section:
```groovy
classpath 'com.archinamon:AspectJ-gradle:1.1.0'
```

Apply the `aspectj` plugin:
```groovy
apply plugin: 'com.archinamon.aspectj'
```

To tune logging and error processing just add an extension:
```groovy
aspectj {
  weaveInfo true //turns on debug weaving information
  ignoreErrors false //explicitly ignores all aspectJ errors, could break a build
  addSerialVersionUID false //adds serialUID for Serializable interface inter-type injections
  logFileName "ajc_details.log" //custom name of default weaveInfo file
}
```

Now you can write aspects using annotation style or native (even without IntelliJ IDEA Ultimate edition).
Let's write simple Application advice:
```java
import android.app.Application;
import android.app.NotificationManager;
import android.content.Context;
import android.support.v4.app.NotificationCompat;

aspect AppStartNotifier {

    pointcut postInit(): within(Application) && execution(* Application.onCreate());

    after() returning: postInit() {
        Application app = (Application) thisJoinPoint.getTarget();
        NotificationManager nmng = (NotificationManager) app.getSystemService(Context.NOTIFICATION_SERVICE);
        nmng.notify(9999, new NotificationCompat.Builder(app).setTicker("Hello AspectJ")
                                                             .setContentTitle("Notification from aspectJ")
                                                             .setContentText("privileged aspect AppAdvice")
                                                             .setSmallIcon(R.drawable.ic_launcher)
                                                             .build());
    }
}
```

Changelog
-------
#### 1.1.0 -- Refactoring
* includes all previous progress;
* updated aspectjtools and aspectjrt to 1.8.7 version;
* now has extension configuration;
* all logging moved to the separate file in `app/build/ajc_details.log`;
* logging, log file name, error ignoring now could be tuned within the extension;
* more complex and correct way to detect and inject source sets for flavors, buildTypes, etc;

#### 1.0.17 -- Cleanup
* !!IMPORTANT!! now corectly supports automatically indexing and attaching aspectj sources within any buildTypes and flavors;
* workspace code refactored;
* removed unnecessary logging calls;
* optimized ajc logging to provide more info about ongoing compilation;

#### 1.0.16 -- New plugin routes
* migrating from corp to personal routes within plugin name, classpath;

#### 1.0.15 -- Full flavor support
* added full support of buld variants within flavors and dimensions;
* added custom source root folder -- e.g. `src/main/aspectj/path.to.package.Aspect.aj`;

#### 1.0.9 -- Basic flavors support
* added basic support of additional build varians and flavors;
* trying to add incremental build //was removed due to current implementation of ajc-task;

#### 1.0 -- Initial release
* configured properly compile-order for gradle-Retrolambda plugin;
* added roots for preprocessing generated files (needed to support Dagger, etc.);
* added MultiDex support;
 
#### Known limitations
* Plugin doesn't support direct speach into AspectJ code if project uses preprocessor (e.g. calling aspect class from java);
* No incremental aj-compilation;
* Doesn't support gradle-experimental plugin;

All these limits are fighting on and I'll be glad to introduce new build as soon as I solve these problems.

License
-------

    Copyright 2015 Eduard "Archinamon" Matsukov.

    Licensed under the Apache License, Version 2.0 (the "License");
    you may not use this file except in compliance with the License.
    You may obtain a copy of the License at

       http://www.apache.org/licenses/LICENSE-2.0

    Unless required by applicable law or agreed to in writing, software
    distributed under the License is distributed on an "AS IS" BASIS,
    WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
    See the License for the specific language governing permissions and
    limitations under the License.
