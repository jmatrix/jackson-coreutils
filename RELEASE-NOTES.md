## 1.4

* Use msg-simple 0.9; do not rely on ServiceLoader anymore.
* Build file changes.
* Various code cleanups.
* Fix javadoc build (all package-info.java and overview.html were missing).

## 1.3

* Use Closer (from Guava) for better I/O resource handling.
* Improve performance of reference token parsing.
* Throw a RuntimeException when failing to deserialize JSON...

## 1.2

* Sanitize I/O resource handling when reading from a classpath resource.
* Use Jackson 2.2.x; change default JsonNodeFactory.
* Switch to gradle; leave old pom.xml in place.
* README updates.

## 1.1

* JSON Patch and JSON Pointer are now RFCs: update links.
* `JsonPointer`: add `.parent()` method (Randy Watler).
* pom.xml is now OSGi-enabled (Matt Bishop).
* Use msg-simple.
* Use JSR 305 annotations.

## 1.0

* First version: code migrated from json-schema-core.

