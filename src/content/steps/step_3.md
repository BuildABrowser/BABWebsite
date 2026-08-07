---
title: 'Step 3: A Very Basic DOM'
description: 'Get a feel for the browser DOM, creating a basic tree that will suffice for some basic HTML parsing.'
next-step: 'step_4'
hashes:
  release: '8ca071b319e8e695ace8739ff46955223b044dfa'
specs:
  - 'https://dom.spec.whatwg.org'
---

If you have done web development (and web development experience is surely a prerequisite to browser development),
you know that a web page is delivered to the browser in the form of HTML.

What does a browser do with that HTML?

It converts the HTML into nodes of a Document Object Model (DOM) tree.

Imagine this basic HTML snippet:

```html
<p>I am building a browser!</p>
```

If you save this snippet to an HTML file, or use the handy copy linked here [TODO: provide URL to server copy], you'll see that,
when opened in a browser, it shows a paragraph with the text "I am building a browser!".

In the example HTML snippet above, you might say that the `p` element "contains" some text.

Using a record for simplicity, we might then represent a paragraph as `record P(Text text);`.
So, to instatiate a pargraph with that text, you might do `new P(new Text("I am building a browser!"))`.

Of course, that's a bit of an oversimplification.

HTML has a ton of elements (like `span`, `div`, etc), not just `p`. We'll call these identifiers the name of the element.
With that, we can generalize our previous record to: `record Element(String name, Text text)`.


However, it is also possible to put other elements inside an element (think `<div><span></span></div>`)
or even to mix multiple `Element` nodes and `Text` nodes into the same parent.
We can make a superinterface of `Element` and `Text` called `Node`, allowing us to store both together in a `List<Node>`.

With that, we might derive a set of interfaces and records such as

```java
public interface Node {}

public record Element(String name, List<Node> children) {}

public record Text(String text) {}
```

For completeness, let's also add a "Document" to represent the containing page.

```java
public record Document(List<Node> children) implements Node {}
```

We shouldn't mix these into our main :Browser module, because it will make code reuse harder.

Let's make a module called `:DOM`.

Make sure that the file named `gradle/libs.versions.toml` has this content:

```toml
[versions]
junit = "6.1.0"

[libraries]
junit-jupiter = { group = "org.junit.jupiter", name = "junit-jupiter", version.ref = "junit" }
```

Replace the version number with the latest stable JUnit version.

Create a `build.gradle` in your project root:
```groovy
subprojects {
	apply plugin: 'java'

	repositories {
		mavenCentral()
	}

	dependencies {
		implementation "org.slf4j:slf4j-api:1.7.36"

		testImplementation libs.junit.jupiter
    testRuntimeOnly 'org.junit.platform:junit-platform-launcher'
	}

	tasks.withType(Test).configureEach {
		useJUnitPlatform()
	}
}
```

Create a folder in your project root named `DOM`, add a reference to it in `settings.gradle`, and create a `DOM/build.gradle`:

```groovy
plugins {
    id 'java-library'
}
```

Reload your IDE.

While the records shown earlier were easier to demonstrate with, we will need to store state later.
As such, for our actual codebase, let's use interfaces and classes instead.

```java
public interface Node {

  void forEachChild(Consumer<Node> itFunc);

}

public interface Element extends Node {

  String name();

  public static Element create(
    String name, List<Node> childNodes
  ) {
    return new ElementImp(name, childNodes);
  }

}

public interface Text extends Node {

  String data();

  static Text create(String text) {
    return new TextImp(text);
  }

}

public interface Document extends Node {

  static Document create(List<Node> childNodes) {
    return new DocumentImp(childNodes);
  }

}
```

We'll include toString methods that print the DOM as HTML (primitively, not accurately)
just so we can watch the DOM in action.

```java
public abstract class NodeImp implements Node {
  
  // This List is not good and will be addressed in a later step.
  private final List<Node> childNodes;

  public NodeImp(List<Node> childNodes) {
    this.childNodes = childNodes;
  }

  @Override
  public void forEachChild(Consumer<Node> itFunc) {
    childNodes.forEach(itFunc);
  }
  
}

public class ElementImp extends NodeImp implements Element {

  private final String name;

  public ElementImp(
    String name,
    List<Node> childNodes
  ) {
    super(childNodes);
    this.name = name;
  }

  @Override
  public String name() {
    return this.name;
  }
  
  @Override
  public String toString() {
    StringBuilder builder = new StringBuilder("<");
    builder.append(name);
    builder.append(">");
    forEachChild(child -> {
      builder.append(child.toString());
    });
    builder
      .append("</")
      .append(name)
      .append(">");
    
    return builder.toString();
  }
  
}

public class DocumentImp extends NodeImp implements Document {

  public DocumentImp(List<Node> childNodes) {
    super(childNodes);
  }

  @Override
  public String toString() {
    StringBuilder builder = new StringBuilder();
    forEachChild(child -> {
      builder.append(child.toString());
    });
    
    return builder.toString();
  }
  
}

public class TextImp extends NodeImp implements Text {

  private final String data;

  public TextImp(String data) {
    super(List.of());
    this.data = data;
  }

  @Override
  public String data() {
    return this.data;
  }

  @Override
  public String toString() {
    return this.data;
  }
  
}
```

With all of these classes, we now a simplistic representation of a browser's Document Object Model (DOM).

Let's put it to use and observe what we did!

In `Browser/build.gradle`, add this to the `dependencies` block:

```groovy
implementation project(':DOM')
```

Reload your IDE.

At the bottom of the `Main` class, add this:
```java
Document document = Document.create(List.of(
  Element.create("h1", List.of(
    Text.create("This is my document!"))),
  Element.create("p", List.of(
    Text.create("I just think it is "),
    Element.create("i", List.of(
      Text.create("really")
    )),
    Text.create(" cool!")
  ))
));

System.out.println(document);
```

When running, we do unfortunately have to pass some URL to prevent a crash.

After running `./gradlew run --args="https://example.com/"`, you should see this in console:

```html
<h1>This is my document!</h1><p>I just think it is <i>really</i> cool!</p>
```

As you can see, the DOM is primarily a tree of nodes (elements, text, etc).
If you've worked with parsers before, you can think of it like an AST but for a document.

Of course, this is hard coded, and does not at all match the page we passed as an argument.
If you want the DOM to match some HTML you pass in, you first need to parse that HTML.

Speaking of, let's start writing a simple parser in the next step!
