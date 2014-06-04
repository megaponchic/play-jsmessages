# Play JsMessages library [![Build Status](https://travis-ci.org/julienrf/play-jsmessages.png?branch=master)](https://travis-ci.org/julienrf/play-jsmessages)

This library allows you to compute localized messages on client-side, in Play! projects.

Basically, play-jsmessages takes the i18n messages of your Play! application, sends them to the client-side as a JSON object and defines a JavaScript function returning a message value from a given language and the message key and arguments.

Take a look at the [Scala](/sample-scala) and [Java](/sample-java) samples to see it in action.

## Installation (using sbt)

Add a dependency on the following artifact:

```scala
libraryDependencies += "org.julienrf" %% "play-jsmessages" % "1.6.2"
```

The current 1.6.2 version is compatible with Play! 2.3.

Previous versions are available here:
 * [`1.6.1`](https://github.com/julienrf/play-jsmessages/tree/1.6.1) for play-2.2 ;
 * [`1.5.2`](https://github.com/julienrf/play-jsmessages/tree/1.5.2) for play-2.2 ;
 * [`1.5.0`](https://github.com/julienrf/play-jsmessages/tree/1.5.0) for play-2.1 ;
 * [`1.3`](https://github.com/julienrf/play-jsmessages/tree/403dc8d7248c965c827b70edeff55016ae274bef) for play-2.0 ;
 * [`1.2.1`](https://github.com/julienrf/play-jsmessages/tree/cc52b23dae9997b77da3cbccf6d22b60a557c2ee) for play-2.0.

## API Documentation

You can browse the online [scaladoc](http://julienrf.github.io/play-jsmessages/1.6.2/api/), which contains the documentation of both the Scala and Java APIs.

## Quick start

### Create a `JsMessages` instance

On server-side, create an instance of the `jsmessages.api.JsMessages` class (or `jsmessages.JsMessages` for Java users):

```scala
import jsmessages.api.JsMessages
import play.api.Play.current

val messages = JsMessages.default
```

### Generate a JavaScript asset for the client’s language

Then you typically want to define an action returning a JavaScript resource containing all the machinery to compute localized messages from client-side:

```scala
Action { implicit request =>
  Ok(messages(Some("window.Messages")))
}
```

Or in Java:

```java
final static jsmessages.JsMessages messages = jsmessages.JsMessages.create(play.Play.application());

public static Result jsMessages() {
    return ok(messages.generate("window.Messages"));
}
```

The above code creates a Play! action that returns a JavaScript fragment containing the localized messages of the application for the client language and defining a global function `window.Messages`. This function returns a localized message given its key and its arguments:

```javascript
console.log(Messages('greeting', 'Julien')); // will print e.g. "Hello, Julien!" or "Bonjour Julien!"
```

The JavaScript function can also be supplied alternative keys that will be used if the main key is not defined. Keys will be tried in order until a defined key is found:

```javascript
  alert(Messages(['greeting', 'saluting'], 'Julien'));
```

The JavaScript function stores the messages map in a `messages` property that is publicly accessible so you can update the messages without reloading the page:

```javascript
// Update a single message
Messages.messages['greeting'] = 'Hi there {0}!';
// Update all messages
Messages.messages = {
  'greeting': 'Hello, {0}!'
}
```

### Generate a JavaScript asset for all the languages

Alternatively, use the `all` method (`generateAll` in Java) to generate a JavaScript fragment containing all the messages of the application instead of just those of the client’s current language, making it possible to switch the language from client-side without reloading the page. In this case, the generated JavaScript function takes an additional parameter corresponding to the language to use:

```javascript
console.log(Messages('en', 'greeting', 'Julien')); // "Hello, Julien!"
console.log(Messages('fr', 'greeting', 'Julien')); // "Bonjour Julien!"
```

In that case the `messages` property of the JavaScript function is a map of languages indexing maps of messages.

The JavaScript function can also be partially applied to fix its first parameter:

```javascript
val messagesFr = Messages('fr'); // Use only the 'fr' messages
console.log(messagesFr('greeting', 'Julien')); // "Bonjour Julien!"
```

Note: if you pass `undefined` as the language parameter, it will use the default messages.

### Generate a JavaScript asset with a subset of your messages

You can select which messages you want to export on client-side by using the `subset` factory method:

```scala
val messages = JsMessages.subset(
  "error.required",
  "error.number"
)
```

Or you can select the messages to keep by using a filter predicate:

```scala
val messages = JsMessages.filtering(_.startsWith("error."))
```

## Changelog

* v1.6.2
  - Play 2.3.x compatibility.

* v1.6.1
  - Fix crash when `undefined` is passed as the language parameter (thanks to Paul Dijou).

* v1.6.0
  - Big changes in the API ;
  - Make it possible to return messages of all languages (thanks to Paul Dijou) ;
  - Discard methods returning HTML, keep just JavaScript.

* v1.5.2
  - Export the `messages` property on client-side (thanks to Paul Dijou).

* v1.5.1
  - Play 2.2.x compatibility.

* v1.5.0
  - Fix the `subset` method of the Java API (thanks to Seppel Hardt) ;
  - Refactor the whole API in order to make it more extensible (thanks to Jacques Bachellerie).

## License

This content is released under the [MIT License](http://opensource.org/licenses/mit-license.php).
