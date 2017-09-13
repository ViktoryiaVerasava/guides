# Introduction

Imagine you have been given a new library to perform some date operations and you want to try it to see how it works.

What would you do?

Would you create a class with a main method in a new or an existing project just to test it?

A JUnit test?

Well, in Java 9, now you have another option to accomplish this task, JShell.

JShell is a Read-Evaluate-Print-Loop (REPL), a command line tool that allows you to enter Java statements (simple statements, compound statements, or even full methods and classes), evaluate them, and print the result.

This is something like what you can find in other JVM languages like [Scala](https://docs.scala-lang.org/overviews/repl/overview.html) and [Groovy](http://www.groovy-lang.org/groovysh.html), which means that you don't have to use the REPLs of these languages to test Java code.

In this tutorial, you'll learn the basics of working with JShell, from executing simple statements to defining classes, along with some commands to manage code snippets. Finally, we'll go through a practical example that will show you how to use JShell to explore [Guava](https://github.com/google/guava), a library that includes a lot of utilities and collection types for Java.

JShell comes with the Java 9 JDK distribution. So download it from [here](http://www.oracle.com/technetwork/java/javase/downloads/index.html) or [here](http://openjdk.java.net/install/) and follow install instructions. Alternatively, you can [use Docker and an OpenJDK image to run JShell](https://medium.com/@_purukaushik/start-using-java-9-shell-jshell-with-docker-888c91ff742c).

# The importance of a REPL
In a few words, a REPL provides a simple way of executing code.

So you might be thinking, why is a REPL better than using other methods? Like a class with a main method in an IDE?

Moreover, Java already has a few third-party REPL tools (like [Java REPL](http://www.javarepl.com)), without mentioning the capability of some IDEs to execute snippets of code.

The [JDK Enhancement-Proposal (JEP) for JShell](http://openjdk.java.net/jeps/222) provides two excellent reasons. Here's the first one:

> Immediate feedback is important when learning a programming language and its APIs. The number one reason schools cite for moving away from Java as a teaching language is that other languages have a "REPL" and have far lower bars to an initial "Hello, world!" program.

When you're learning something new, it's important to try out things with a minimal effort and have immediate feedback for more engaging. If you had to try out some code, even a  *Hello World*, you had to create a class, add a main method, add the line(s) of code, optionally add some dependencies, and then compile and (if there are not any errors) run the code. Even with an IDE, that's more complicated than using a REPL.

Of course, Java has alternatives to JShell, but they don't have all the features that JShell provides in one package, like full language support or preserved context and session history.

In addition, JShell also provides or an API to its evaluation engine, so it can be [integrated to IDEs](https://blogs.oracle.com/geertjan/jshell-integration-in-netbeans-ide) or other tools to further simplify our workflow.

Even experienced developers can benefit from a REPL. Here's the second reason:

> Exploration of coding options is also important for developers prototyping code or investigating a new API. [...] Without the ceremony of class Foo { public static void main(String[] args) { ... } }, learning and exploration is streamlined.

You can quickly prototype and experiment with libraries, like what's is the return type of a method, as well as language features, for example, what's the format pattern to show the time zone for a date.

Having explained the importance of a REPL, let's dig into the basics of JShell.

# JShell Basics
As said before, JShell is included with the JDK. Just go to the `bin` directory of your installation, for example:
```bash
cd C:\Program Files\Java\jdk-9\bin
```

Or make sure this directory is in your `PATH` environment variable and execute:
```bash
jshell
```

You should see something like the following:
```java
$ jshell
|  Welcome to JShell -- Version 9
|  For an introduction type: /help intro

jshell>
```

This will start an interactive stateful session, which means that it will remember previously-defined statements and modifications in the same session. 

You can input specific JShell commands or Java code (referred as *snippets*), in particular:
- Expressions
- Statements
- Class declarations
- Interface declarations
- Method declarations
- Field declarations
- Import declarations

So let's start with a simple expression:
```java
jshell> "Hello World"
$1 ==> "Hello World"
```

Notice that the result of the evaluation is shown and it's assigned to the variable named `$1`. You can check this by entering the name of the variable:
```java
jshell> $1
$1 ==> "Hello World"
```

A variable like this will be created implicitly when the result of an expression is not assigned to some variable. Obviously, if the expression doesn't have a return value (like a print statement), no variable will be created.

You can use this implicit variable like any other:
```java
jshell> System.out.println($1)
Hello World
```

Of course, you can also change the value it holds:
```java
jshell> $1 = $1 + " again"
$1 ==> "Hello World again"

jshell> System.out.println($1)
Hello World again
```

However, probably it's better to work with some meaningful identifiers:
```java
jshell> String hello = $1
hello ==> "Hello World again"

jshell> System.out.println(hello)
Hello World again
```

As you can see in the previous examples, it's not required to use a semicolon for single statements. Of course, this will not work if you try to execute more than one statements at the same time:
```java
jshell> int sum = 1 + 2 System.out.println(sum)
|  Error:
|  ';' expected
|  int sum = 1 + 2 System.out.println(sum);
|                 ^
```

You'll need a semicolon between statements (but not after the last one if you don't want):
```java
jshell> int sum = 1 + 2; System.out.println(sum)
sum ==> 3
3
```

Also, JShell won't execute a statement until it knows the statement is completed. For example, if you only type `if(sum > 0)`, the prompt will change to `...>`:
```java
jshell> if(sum >0)
   ...> 
```

And we'll be prompted to continue entering snippets until a valid statement or block is completed
```java
jshell> if(sum > 0)
   ...> System.out.print(sum)
3
```

This capability of typing multiline statements allows us to write complete methods and classes. For example, you can define a method to return the negative value of the argument:
```java
jshell> int getNegativeValue(int number) {
   ...> return -number;
   ...> }
|  created method getNegativeValue(int)
```

This way, you can call it like this:
```java
jshell> getNegativeValue(10)
$5 ==> -10
```

Sometimes, a method references variables or other methods that are defined later in a class. These are called forward references. Since in JShell the code is entered and evaluated sequentially, code that uses forward reference cannot be invoked until these references are evaluated. Take the following method declaration for example:
```java
jshell> int addMagicNumber(int number) {
   ...> return number + MAGIC_NUMBER;
   ...> }
|  created method addMagicNumber(int), however, it cannot be invoked until variable MAGIC_NUMBER is declared
```

You won't be able to use until `MAGIC_NUMBER` is defined:
```java
jshell> System.out.println(addMagicNumber(3))
|  attempted to call method addMagicNumber(int) which cannot be invoked until variable MAGIC_NUMBER is declared

jshell> int MAGIC_NUMBER = 1
MAGIC_NUMBER ==> 1

jshell> System.out.println(addMagicNumber(3))
4
```

JShell supports forward references in methods and in the type of return statements, parameters and variables (for example, when you return a *Book* object and that class hasn't been defined). However, it doesn't support forward references in variable initializers (maybe because they need to be executed at the moment the variable is defined):
```java
jshell> int MAGIC_NUMBER = getMagicNumber()
|  Error:
|  cannot find symbol
|    symbol:   method getMagicNumber()
|  int MAGIC_NUMBER = getMagicNumber();
|                     ^------------^
```

You can also declare classes, for example:
```java
jshell> class Book {
   ...> private String title;
   ...> public void setTitle(String title) { this.title = title; }
   ...> public String getTitle() { return title; }
   ...> }
|  created class Book
```

And use them like this:
```java
jshell> Book book = new Book()
book ==> Book@3dd4520b

jshell> book.setTitle("Java 9");

jshell> System.out.println(book.getTitle())
Java 9
```

Modifiers of methods and fields defined inside a class are applied just like Java:
```java
jshell> book.title
|  Error:
|  title has private access in Book
|  book.title
|  ^--------^
```

However, declarations outside a class or interface (and declarations of classes and interfaces by itself) are created under the following rules:
1. Access modifiers (`public`, `protected`, and `private`) are ignored. All declaration snippets are accessible to all other snippets:
  ```java
  jshell> private int x = 1;
  x ==> 1

  jshell> System.out.print(x) // private ignored
  1
  ```
2. The modifier `final` is ignored. Changes and inheritance are allowed:
  ```java
  jshell> final class A {void m() {} }
  |  Warning:
  |  Modifier 'final'  not permitted in top-level declarations, ignored
  |  final class A { void m() {} }
  |  ^---^
  |  created class A
  ```
3. The modifier `static` is ignored because there isn't a container class (at least none you can use):
  ```java
  jshell> static char letter = 'A'
  |  Warning:
  |  Modifier 'static'  not permitted in top-level declarations, ignored
  |  static char letter = 'A';
  |  ^----^
  letter ==> 'A'
  ```
4. The modifiers `default` and `synchronized` are not allowed:
  ```java
  jshell> synchronized void method() {}
  |  Error:
  |  Modifier 'synchronized'  not permitted in top-level declarations
  |  synchronized void method() {}
  |  ^----------^
  ```
5. The modifier `abstract` is allowed only on classes:
  ```java
  jshell> abstract void method();
  |  Error:
  |  Modifier 'abstract'  not permitted in top-level declarations
  |  abstract void method();
  |  ^------^
  ```

JShell automatically catches exceptions, display information about them and allows you to continue the session as if nothing has happened. This works for unchecked exceptions:
```java
jshell> 1/0
|  java.lang.ArithmeticException thrown: / by zero
|        at (#24:1)

jshell>
```

And checked exceptions without wrapping them in a try/catch statement:
```java
jshell> FileInputStream fis = new FileInputStream("nonexistentfile.txt")
|  java.io.FileNotFoundException thrown: nonexistentfile.txt (The system cannot find the file specified)
|        at FileInputStream.open0 (Native Method)
|        at FileInputStream.open (FileInputStream.java:196)
|        at FileInputStream.<init> (FileInputStream.java:139)
|        at FileInputStream.<init> (FileInputStream.java:94)
|        at (#25:1)
```

Finally, to end a session just type `/exit`:
```java
jshell> /exit
|  Goodbye

$
```

# Managing code snippets
As said before, JShell has a few commands on its own. You can see the complete list by typing `/help` or `/?`:
```java
jshell> /help
|  Type a Java language expression, statement, or declaration.
|  Or type one of the following commands:
|  /list [<name or id>|-all|-start]
|       list the source you have typed
| ...
```

The most useful commands are probably the ones that help you manage code snippets.

The first one is `list`, which lists by default, all the valid statements you have typed since starting the session:
```java
jshell> /list

   1 : "Hello World"
   2 : $1
   3 : System.out.println($1)
```

There's another command that does something similar, `/history`. The difference is that `/history` presents all the statements (valid or not) and commands you have typed:
```java
jshell> /history
/help
"Hello World"
$1
System.out.println($1)
Sys
/list
/history
```

You can also list only variables with `/var`, only methods with `/methods`, or only types with `/types`:
```java
jshell> /vars
|    String $1 = "Hello World"
```

However, the advantage of `/list` is that it shows you a numeric identifier for each snippet that you can use to rerun it with `/<id>`:
```java
jshell> /3
System.out.println($1)
Hello World
```

Delete it with the `/drop` command:
```java
jshell> /drop 2

jshell> /list

   1 : "Hello World"
   3 : System.out.println($1)
```

Or edit it:
```java
jshell> /edit 1
```

In this case, a window will open with a simple editor that will help you modify the snippet:
![01-default-editor](default jshell editor)

This editor comes in handy to add or modify multi line statements like classes or methods and it won't be closed until you click on the *Exit* (saving the changes) or *Cancel* (discarding the changes) buttons. 

However, in the case of implicit variables, notice that instead of modifying the variable, JShell creates another one with the new value:
```java
jshell> /edit 1
$4 ==> "Hello World modified"

jshell> /list

   1 : "Hello World"
   3 : System.out.println($1)
   4 : "Hello World modified";

jshell>
```

This doesn't happen with explicitly named variables or types. By the way, you can also use the name of the variable or type with these commands. For example, assuming the `Book` class defined in the previous section:
```java
jshell> class Book {
   ...> private String title;
   ...> public void setTitle(String title) { this.title = title; }
   ...> public String getTitle() { return title; }
   ...> }
|  created class Book

jshell> /list

   1 : "Hello World"
   3 : System.out.println($1)
   4 : "Hello World modified";
   5 : class Book {
       private String title;
       public void setTitle(String title) { this.title = title; }
       public String getTitle() { return title; }
       }

jshell>
```

We can edit it with the command:
```java
jshell> /edit Book
```

If we modify the class in this way:
```java
class Book {
private String title = "<NO TITLE>";
public void setTitle(String title) { this.title = title; }
public String getTitle() { return title; }
public String toString() { return "Book: " + title; }
}
```

And click on *Exit*, JShell will update the definition of the class (however, notice the ID also changed):
```java
jshell> /edit Book
|  replaced class Book

jshell> /list

   1 : "Hello World"
   3 : System.out.println($1)
   4 : "Hello World modified";
   6 : class Book {
       private String title = "<NO TITLE>";
       public void setTitle(String title) { this.title = title; }
       public String getTitle() { return title; }
       public String toString() { return "Book: " + title; }
       }
```

If you execute the `/edit` command without an argument, the editor will let you modify all the current statements in the session (and even add some if you want):
![02-editor-all-statements](editor with all the statements)

One problem we have is that all these statements are lost when we exit JShell. Luckily, there's a '/save' command, that can save all commands and snippets to a file. It has the following options:
- `/save <file>`. Save the commands and snippets currently active to the provided file path.
- `/save -all <file>`. Save all commands and snippets, including overwritten, start-up, and failed commands and snippets to the provided file path.
- `/save -history <file>`. Save all commands and snippets run to the provided file.
- `/save -start <file>`. Save all commands and snippets that initialized the JShell session to the provided path.

This way, if we execute:
```java
/save my-session.txt
```

We'll get a text file named *my-session.txt* saved in the current directory with the following content:
```java
"Hello World"
System.out.println($1)
"Hello World modified";
class Book {
private String title = "<NO TITLE>";
public void setTitle(String title) { this.title = title; }
public String getTitle() { return title; }
public String toString() { return "Book: " + title; }
}
```

This way, if we open a new JShell session and execute the command:
```java
/open my-session.txt
```

All the statements defined in the text file will be executed:
```java
jshell> /open my-session.txt
Hello World

jshell> /list

   1 : "Hello World"
   2 : System.out.println($1)
   3 : "Hello World modified";
   4 : class Book {
       private String title = "<NO TITLE>";
       public void setTitle(String title) { this.title = title; }
       public String getTitle() { return title; }
       public String toString() { return "Book: " + title; }
       }
```

By the way, `/open` also works with existing Java source files, for example:
```java
jshell> /open Book.java
´´´
 
# Using JShell to explore libraries
As mentioned before, JShell does an excellent job helping us explore new libraries. So let's assume we want to try out the [Guava](https://github.com/google/guava) library. 

You can download the JAR file of Guava Release 23.0 (the latest at this time) [here](https://github.com/google/guava/wiki/Release23).

In a JShell session, you get the following set of imports by default:
```java
jshell> /imports
jshell> /imports
|    import java.io.*
|    import java.math.*
|    import java.net.*
|    import java.nio.file.*
|    import java.util.*
|    import java.util.concurrent.*
|    import java.util.function.*
|    import java.util.prefs.*
|    import java.util.regex.*
|    import java.util.stream.*
```

You can import other packages, but how does JShell know where to look for custom packages like the following:
```java
jshell> import com.mycompany.*
```

What you have to do is add the packages to the JShell classpath with the `/env -class-path <path>` command (if we already started a JShell session) or the option `$jshell --class-path <path>` (on the command-line).

You can specify a directory (in this example the current directory):
```java
jshell> /env -class-path .
```

Or a list of directories, JARs, or ZIP archives to search for compiled class files (the list must be separated with `:` on Linux/Mac and `;` on Windows). You can find out more by executing `/help context`.

This way, assuming you have downloaded the Guava JAR in `c:\`, we can execute the following command:
```java
jshell> /env -class-path c:\guava-23.0.jar
|  Setting new options and restoring state.
```

As you can see from the message, it will restore the session with the new classpath setting, which means that it will run all valid snippets executed until that point.

If the library you want to test depends on other libraries, you have to download those dependencies and add them to the classpath too.

Now, you can import the classes from Guava's packages, and this is where JShell auto-completion comes in handy.

JShell's auto-completion capabilities allow you to view:
- Classes in a package
- Class members
- The parameters required by a method
- The list of overloads for a method
- Documentation of classes and their members

For example, you can start typing:
```java
jshell> import com.google.common.primitives.
```

And then press *Tab* so JShell can present some options:
```java
jshell> import com.google.common.primitives.
Booleans               Bytes                  Chars                  Doubles                Floats
ImmutableDoubleArray   ImmutableIntArray      ImmutableLongArray     Ints                   Longs
Primitives             Shorts                 SignedBytes            UnsignedBytes          UnsignedInteger
UnsignedInts           UnsignedLong           UnsignedLongs
```

Let's import the `Longs` class:
```java
jshell> import com.google.common.base.Longs
```

If we type `Longs.` and press *Tab*, JShell will show the class’s static members:
```java
jshell> Longs.
BYTES                         MAX_POWER_OF_TWO              asList(
class                         compare(                      concat(
constrainToRange(             contains(                     ensureCapacity(
fromByteArray(                fromBytes(                    hashCode(
indexOf(                      join(                         lastIndexOf(
lexicographicalComparator()   max(                          min(
stringConverter()             toArray(                      toByteArray(
tryParse(
```

The names that are not followed by parentheses (like `BYTES`) are static variables. All the other names are static methods:
- If the method name is followed by `()` (like `lexicographicalComparator()`), it means that the method doesn't require any arguments.
- If the method name is followed by an opening parenthesis, `(`, it means that the method requires at least one argument or that it is overloaded.

If you want to know more about the method `indexOf` type the name of this method and its opening parenthesis (the name can be auto-completed too), and press *Tab*:
```java
jshell> Longs.indexOf(
Signatures:
int Longs.indexOf(long[] array, long target)
int Longs.indexOf(long[] array, long[] target)

<press tab again to see documentation>
```

If you press *Tab* again, you'll see the documentation (Javadoc) of the first overload:
```java
jshell> Longs.indexOf(
int Longs.indexOf(long[] array, long target)
<no documentation found>

<press tab to see next documentation>
```

Well, in this case there's no documentation found. But you can keep pressing *Tab* to see the documentation of the next methods:
```java
jshell> Longs.indexOf(
int Longs.indexOf(long[] array, long[] target)
<no documentation found>

<press tab again to see all possible completions; total possible completions: 600>
```

You can do this kind of exploration with any class, even at class level:
```java
jshell> String
String                            StringBuffer                      StringBufferInputStream
StringBuilder                     StringIndexOutOfBoundsException   StringJoiner
StringReader                      StringTokenizer                   StringWriter
Strings

Signatures:
java.lang.String

<press tab again to see documentation>

jshell> String
java.lang.String
The String class represents character strings.All string literals in Java programs, such as
"abc", are implemented as instances of this class.
...
Unless otherwise noted, passing a null argument to a constructor or method in this class will

<press tab again to see next page>
```

Or with any object to show its instance members:
```java
jshell> "Hello".
charAt(                chars()                codePointAt(           codePointBefore(       codePointCount(
codePoints()           compareTo(             compareToIgnoreCase(   concat(                contains(
contentEquals(         endsWith(              equals(                equalsIgnoreCase(      getBytes(
getChars(              getClass()             hashCode()             indexOf(               intern()
isEmpty()              lastIndexOf(           length()               matches(               notify()
notifyAll()            offsetByCodePoints(    regionMatches(         replace(               replaceAll(
replaceFirst(          split(                 startsWith(            subSequence(           substring(
toCharArray()          toLowerCase(           toString()             toUpperCase(           trim()
wait(
```

# Conclusion
JShell is an important addition to the language. In an easy and quick way, you can verify things like return types, formatting options, or try out libraries or language features.

In this tutorial, you have learned how to work with various types of code snippets in JShell, some useful commands and about its auto-completion capabilities to explore a class’s members and documentation.

Of course, there's still a lot of things to learn for more advanced uses, like feedback modes, custom options (like setting a custom editor), and how to integrate JShell into your Java programs. A good resource to learn about some of these topics is [Robert Field's tutorial on JShell](http://cr.openjdk.java.net/~rfield/tutorial/JShellTutorial.html). Thanks for reading.