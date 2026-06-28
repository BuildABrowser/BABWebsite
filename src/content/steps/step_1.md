---
title: 'Step 1: Initial Commit'
description: 'Learn more about BuildABrowser. Initialize your repository.'
next-step: 'step_2'
hashes:
  release: '0a04a7803e884913384ca8c20c96690203a2b20b'
---

## Prelude

Hello! And welcome to BuildABrowser, where you - you guessed it - build a browser!

Specifically, you rebuild BuildABrowser Browser, a Java-Based browser.
If you haven't tried it yet, I would recommend downloading it and trying it out.
It's nice to know what you're going to build before you build it.

A browser, of course, allows you to browse the web.

There are a number of terms we use when speaking about web browsers, and they can
be easy to get mixed up. As such, here is a quick briefing on some browser terminology.
* **Browser** - An application that allows you to navigate to and view websites. This is what we will be building.
* **Website**/**Web Page** - Consists of a bundle of HTML/CSS/JS, and is typically navigated to by a URL (e.g. https://example.com/).
  This is **not** what we are building in this tutorial.
* **Rendering Engine** - This is the part of a browser responsible for displaying a web page.
* **Browser chrome (lowercase c)** - This is the part of the browser containing navigation controls, such as the URL bar, tab bar, download pane, menus, etc.
* **The Browser Chrome (uppercase C)** - Refers to the web browser named Google Chrome. To distinguish between it and browser chrome, pay attention to surrounding context.

To better illustrate the difference between these, a graphic is provided below:
[TODO: Insert Graphic]
Alt: A picture illustrating the difference between various browser related terms.

The rendering engine refers to the technologies that then retrieves and parses the HTML and CSS, as well as lays out
(determine the sizes and positions of) and paints (shows on screen) any UI elements found in those files.

A rendering engine has a number of responsibilities:
* It must retrieve a number of resources (such as HTML, CSS, and image files) associated with a web page.
* It must parse or decode the resources that it has retrieved (converting HTML to a DOM, CSS to a CSSOM, etc).
* It must generate 'boxes' to represent the elements in the DOM.
* It must 'lay out' the boxes, determining sizes and positions of elements and text.
* It must 'paint' fragments to the screen, so that the viewer can actually see the document.
* And more

Responsibilities of a rendering engine may even extend beyond this.
For example, a scriptable rendering engine might include bindings to a scripting engine.
An accessible rendering engine might construct an A11Y tree and communicate with OS accessibility technologies.

At this time, BuildABrowser does not yet support scripting (but it is on the roadmap).

Rendering engines can often be used for non-browser use cases.
For example, you can embed Blink (Chromium's rendering engine) into an Android project via WebView.
Alternatively, you might choose to embed a rendering engine into a PDF generation utility, allowing you to save or print web pages.

BuildABrowser covers both the rendering engine and the browser chrome, but spends more time focusing on the rendering engine
because rendering engines are historically that more complicated part to build.

When following this tutorial, it is expected that the reader should already be very comfortable with programming.
Browsers are complex, so this tutorial will be difficult to follow if you don't have as much experience.

Also, there is a *lot* of work involved regardless.

Most tutorials of this nature include code blocks for each new feature added, such that if you read only the text segments, you can compose
any contained code blocks into a coherent program. While this tutorial will certainly include code blocks for points of interest, the finished
product is multiple tens of thousands lines of code, and as such it is not practical to include every line in the markdown files. Instead, this tutorial opts for
a step-by-step commit reference repository. Each page of this tutorial will include a hyperlink to the diff between two commits, and that diff will
include the total code changes from the step. At times, you may even desire to try your own hand at implementing something before referencing the diffs
(just reset if it goes awry, and be aware that deviations may cause merge conflicts).

Before attempting to write a browser, please consider why you want to write a browser:
* If you want to learn how a browser works, great! This tutorial is for you.
* If you want to add a feature to the browser chrome, it would be better to use an existing rendering engine or fork an existing browser.

Again, it is good to know what you're building before you begin building. The BuildABrowser website includes a download for the most up-to-date BuildABrowser
build, which serves as a target goal in this tutorial. Feel free to give it a try.

Do please note that currently progress on the final engine is much further along than writing for this tutorial. Keep an eye out for new steps being published!

Lastly, there are of course already other browser tutorials such as https://browser.engineering/ that you may wish to also check out. It could be
good to get multiple perspectives on engine development.

Where I intend for BuildABrowser to differ from other tutorials is its focus on spec compliance and (currently static) web compatibility.
Due to this, however, the browser built in BuildABrowser may be significantly larger than those built in other tutorials.

To keep this manageable, BuildABrowser is split into many smaller contigous steps.
At any time, if you get lost, feel free to checkout a copy of the repo commit (linked at the top of each step) for the step you were on.
Alternatively, if you're interested in something specific, you may wish to checkout an arbitrary commit (just don't get lost on any concepts you skipped!).

Because browsers are infinitely growing, what is currently the last step may not be the last step in the future.
Don't feel obliged to complete these steps to the end - simply complete as much of the tutorial as you wish to,
and for what you do attempt, take it step-by-step.

Don't look ahead too far, and don't rush.

Oh, also: Be prepared to read.

But enough talking. Let's begin!

## Overall setup

This project is tested on Linux (graphical) and Windows.
If you have a different OS, you may need to skip some native integration steps.

Make sure you have these technologies installed:
* Gradle 9
* JDK 24 or 25
* A Java profiler, like VisualVM or JProfiler
* git

[TODO: Include website links and sdkman commands]

**Want to skip this step**? Clone the commit hash at the top of the page
```bash
git clone https://github.com/BuildABrowser/BuildABrowser
git checkout 0a04a7803e884913384ca8c20c96690203a2b20b
```

## Gradle Setup

We can start with the super basic steps.

Create a project directory, and run:
```bash
gradle init
# Select application, Java, 24, buildabrowser, Single Application, Groovy, JUnit Jupiter, no
git init
```

Clone the main BAB repo, and copy the following files/directories:
`LICENSE.txt`, `licenses`, `README.md`, `AUTHORS.txt`, `.gitignore`
Don't worry, these are not code, they're just metadata files and directories.
Be sure to keep them up to date over time! Some of these files are important because they
give the ability to use and redistribute code from the tutorial.

Rename the `app` folder (generated by Gradle) to `Browser`.
Delete the `org` folder nested in `Browser/src/main/java/`.
Also delete the `org` folder nested in `Browser/src/test/java/`.
In `Browser/build.gradle`, delete any unncessary auto-generated comments and the Guava lib.
Change `mainClass`'s value to `net.buildabrowser.babbrowser.browser.Main`.

Also delete unneccessary comments and foojay from `settings.gradle`. Change the include to point to `Browser`.

In future steps, this guide will not refer to file paths.
It is expected that you will look at the git diffs linked at the top of each step for these paths.

Create `net.buildabrowser.babbrowser.browser.Main`.
It is assumed that you are using your IDE to automatically create classes and manage imports/packages.

Add the following method:
```java
public static void main(String[] args) {
  System.out.println("Hello World!");
}
```

Reload your IDE.

Finally, run `./gradlew run`. It should print "Hello World!" if everything is working correctly.

In Git, add your work and commit. Generally, it is assumed you will do so at the end of each substep (or part, for steps broken into parts).

In the next step, we will look at writing an extremely simplified browser pipeline.
