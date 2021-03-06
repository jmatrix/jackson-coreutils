## Read me first

The license of this project is LGPLv3 or later. See file `src/main/resources/LICENSE` for the full
text.

This project uses [Gradle](http://www.gradle.org) as a build system. See file `BUILD.md` for
details.

Credits where they are due: other people have contributed to this project, and this project would
not have reached its current state without them. Please refer to the `CONTRIBUTORS.md` file in this
project for details.

## What this is

This package is meant to be used with Jackson 2.2.x. It provides the three following features:

* its `JsonLoader` class brings you a convenient way for loading JSON data from
  a variety of sources: a string, an existing `InputStream` or `Reader`, etc;
* it uses Guava's
  [Equivalence](http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/base/Equivalence.html)
  over Jackson's
  [JsonNode](http://fasterxml.github.com/jackson-databind/javadoc/2.1.1/com/fasterxml/jackson/databind/JsonNode.html)
  to provide a means to compare JSON number values mathematically;
* it implements [JSON Pointer](http://tools.ietf.org/html/rfc6901)
  over Jackson's `TreeNode`, and has a dedicated implementation (`JsonPointer`)
  over Jackson's `JsonNode`.

## Versions

The current verson is **1.4**. Its Javadoc is [available
online](http://fge.github.io/jackson-coreutils/index.html).

## Using in Gradle/Maven

With Gradle:

```groovy
dependencies {
    compile(group: "com.github.fge", name: "jackson-coreutils", version: "yourVersionHere");
}
```

With Maven:

```xml
<dependency>
    <groupId>com.github.fge</groupId>
    <artifactId>jackson-coreutils</artifactId>
    <version>yourVersionHere</version>
</dependency>
```

## Why

### Numeric equivalence

When reading JSON into a `JsonNode`, Jackson will serialize `1` as an `IntNode` but `1.0` as a
`DoubleNode` (or a `DecimalNode`).

Understandably so, Jackson <b>will not</b> consider such nodes to be equal, since they do not share
the same class. But, understandably so as well, some uses of JSON out there, including [JSON
Schema](http://tools.ietf.org/html/draft-zyp-json-schema-04) and [JSON
Patch](http://tools.ietf.org/html/rfc6902), want to consider such nodes as equal.

And this is where this package comes in. It allows you to consider that two numeric JSON values are
mathematically equal -- recursively so. That is, JSON values `1` and `1.0` _will_ be considered
equivalent; but so will be all possible JSON representations of mathematical value 1 (including, for
instance, `10e-1`). And evaluation is recursive, which means that:

```json
[ 1, 2, 3 ]
```

will be considered equivalent to:

```json
[ 10e-1, 2.0, 0.3e1 ]
```

### JSON Pointer

JSON Pointer is an IETF draft which allows to unambiguously address any value into a JSON document
(including the document itself, with the empty pointer). It is used in several IETF drafts:

* [JSON Reference](http://tools.ietf.org/html/draft-pbryan-zyp-json-ref-03) (as the fragment part);
* [JSON Patch](http://tools.ietf.org/html/rfc6902).

The implementation in this package applies to all `TreeNode`s as of Jackson 2.2.x.

## Usage

### Numeric equivalence

For accuracy, it is **highly recommended** that you use this library's `JacksonUtils` class to
obtain a mapper/reader with the ability to read arbitrarily large numeric instances from JSON
values; if you don't, you may be (badly) surprised by the results of using the below feature.

The recommended way to read any `JsonNode` instance is therefore to use the `JsonLoader` class, or,
if you need to, grab an `ObjectMapper` preconfigured for dealing with arbitrarily large numbers:

```java
// Load a JsonNode with all decimals read as DecimalNode, from a file
final JsonNode node = JsonLoader.fromFile("/path/to/file.json");
// Get a preconfigured reader
final ObjectReader reader = JacksonUtils.getReader();
// Get a preconfigured ObjectMapper
final ObjectMapper mapper = JacksonUtils.newMapper();
```

Given two `JsonNode` instances which you want to be equivalent if their JSON number values are the
same, you can use:

```java
if (JsonNumEquals.getInstance().equivalent(node1, node2))
    // do something
```

You can also use this package to add `JsonNode` instances to a set:

```java
final Equivalence<JsonNode> eq = JsonNumEquals.getInstance();
// Note: uses Guava's Sets to create the set
final Set<Equivalence.Wrapper<JsonNode>> set
    = Sets.newHashSet();

// Insert values
set.add(eq.wrap(node1));
set.add(eq.wrap(node2));
// etc
```

### JSON Pointer

This section concentrates on the `JsonNode` specific JSON Pointer implementation: `JsonNode`.

There are several ways you can build one:

```java
// Build from an input string -- potentially throws JsonPointerException on malformed inputs
final JsonPointer ptr = new JsonPointer("/foo/bar");
// Build from a series of raw tokens
final JsonPointer ptr = JsonPointer.of("foo", "bar", 1); // Yields pointer "/foo/bar/1"
// Get another pointer's parent:
final JsonPointer parent = ptr.parent();
```

Note that `JsonPointer` (and, for that matter, `TreePointer` as well) is **immutable**:

```java
// DON'T DO THAT: value of "ptr" will not change
ptr.append("foo");
// Do that instead
ptr = ptr.append("foo");
```

