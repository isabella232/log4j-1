Log4j2 for dotCMS
=================

This is a log4j2 fork used on dotCMS.  This fork is needed for repackaging purposes.
Log4j2 uses a static cache file for plugins, this file is written using the writeUTF() method
wich saves the String as a {size}{String} pair.  This can not be replaced as a plain String
as the size portion should change as well; if only the String is changed then the reading
of the text could fail.

The be able to read the cache file after the repackaging is done, the way the cache file is
written changed to comply with dotCMS classpath repackaging standard.

The change is on 
```
org.apache.logging.log4j.core.config.plugins.processor.PluginCache#writeCache()
```

Difference on the this branch.
```
--- a/log4j-core/src/main/java/org/apache/logging/log4j/core/config/plugins/processor/PluginCache.java
+++ b/log4j-core/src/main/java/org/apache/logging/log4j/core/config/plugins/processor/PluginCache.java
@@ -81,7 +81,7 @@ public class PluginCache {
                 for (final Map.Entry<String, PluginEntry> entry : m.entrySet()) {
                     final PluginEntry plugin = entry.getValue();
                     out.writeUTF(plugin.getKey());
-                    out.writeUTF(plugin.getClassName());
+                    out.writeUTF("com.dotcms.repackage." + plugin.getClassName());
                     out.writeUTF(plugin.getName());
                     out.writeBoolean(plugin.isPrintable());
                     out.writeBoolean(plugin.isDefer());
```

Basically all it does is to append dotCMS repackage prefix packaging names to the current
package structure.  Only the core jar is affected by this, all other jars can be taken from
the oficial binary sources.

How to compile
==============

Change into the lastest branch version (2.3 at this moment)

This branch should contain a commit with the change needed for the repackaging process.

Run the following command to obtain the jars needed.

```
mvn clean package -Dmaven.test.skip=true
```

Skipping the tests is needed since the change on the cache file will impact the tests.  The
tests will try to use the cache file to load the plugins but the packages names won't fit
the existing ones, not until the repackage process is run.

The only jars affected are **log4j-core-2.3.jar** and **log4j-web-2.3.jar**.  Those jars are located under

```
 <repository>/log4j-core/target/
 ```
 
And
 
 ```
 <repository>/log4j-web/target/
```