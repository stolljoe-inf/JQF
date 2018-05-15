# JQF: A feedback-directed fuzz testing platform for Java

JQF is built on top of [junit-quickcheck](https://github.com/pholser/junit-quickcheck), which itself lets you write [Quickcheck](http://www.cse.chalmers.se/~rjmh/QuickCheck/manual.html)-like generators and properties in a [Junit](http://junit.org)-style test class. JQF enables better input generation using state-of-the-art fuzzing tools such as [AFL](http://lcamtuf.coredump.cx/afl). 

JQF has been successful in [discovering a number of bugs in widely used open-source software](https://github.com/rohanpadhye/jqf/wiki/Bug-trophy-case) such as OpenJDK, Apache Maven and the Google Closure Compiler.

## Quickstart

Write a Junit-like test class annotated with `@RunWith(JQF.class)` and write some test methods annotated with `@Fuzz`. The arguments to the test methods will be fuzzed by JQF:

```java
@RunWith(JQF.class)
public class DateFormatterTest {

    /* Input params will be generated by JQF, many times. */
    /* Exceptions listed in the "throws" clause are considered normal (tests will pass on throw) */
    @Fuzz
    public void fuzzSimple(Date date, String format) throws IllegalArgumentException {
        // Create a simple date formatter using the input format string
        // May throw IllegalArgumentException for invalid formats
        DateFormat df = new SimpleDateFormat(format);

        // Format the date using the constructed formatter
        df.format(date);
    }
}
```

Compile with JQF on the classpath (using the provided handy script):
```bash
$ javac -cp $(jqf/scripts/classpath.sh) DateFormatterTest.java
```


Fuzz the method `fuzzSimple` with AFL:
```bash
$ jqf/bin/jqf-afl-fuzz DateFormatterTest fuzzSimple
```

Grab a coffee while AFL does its thing:

![AFL status screen](https://rohanpadhye.github.io/jqf/images/AFL-DateFormatter.png)

Ooh! We found some crashes. Let's reproduce one such test case and see what the error was.

```
$ jqf/bin/jqf-repro DateFormatterTest fuzzSimple fuzz-results/crashes/id:000000
java.lang.ArrayIndexOutOfBoundsException: 127
	at java.text.SimpleDateFormat.subFormat(SimpleDateFormat.java)
	at java.text.SimpleDateFormat.format(SimpleDateFormat.java)
	at java.text.SimpleDateFormat.format(SimpleDateFormat.java)
	at java.text.DateFormat.format(DateFormat.java)
```

This shouldn't happen! `DateFormat.format()` does not specify that it will throw `ArrayIndexOutOfBoundsException`. Time to file a [bug report](https://github.com/rohanpadhye/jqf/wiki/Bug-trophy-case) :-)

## Building 

To build JQF, you need Java 8+, Maven (`mvn`) and GNU Make (`make`) installed and on your path. 

Clone the JQF repo and run `setup.sh`.

```bash
git clone https://github.com/rohanpadhye/jqf
jqf/setup.sh 
```

## Documentation

The [JQF wiki](https://github.com/rohanpadhye/jqf/wiki) contains lots more documentation including:
- [Writing a JQF test](https://github.com/rohanpadhye/jqf/wiki/Writing-a-JQF-test)
- [Fuzzing with AFL](https://github.com/rohanpadhye/jqf/wiki/Fuzzing-with-AFL)
- [Using a custom fuzz guidance](https://github.com/rohanpadhye/jqf/wiki/The-Guidance-interface)
- [Performance Benchmarks](https://github.com/rohanpadhye/jqf/wiki/Performance-benchmarks)


## Contact the developers

We want your feedback! (haha, get it? get it?) 

If you've found a bug in JQF or are having trouble getting JQF to work, please open an issue on the [issue tracker](https://github.com/rohanpadhye/jqf/issues). You can also use this platform to post feature requests.

If it's some sort of fuzzing emergency you can always send an email to the main developer: [Rohan Padhye](https://people.eecs.berkeley.edu/~rohanpadhye).
