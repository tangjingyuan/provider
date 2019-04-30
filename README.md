[![Build Status](https://travis-ci.org/rrousselGit/provider.svg?branch=master)](https://travis-ci.org/rrousselGit/provider)
[![pub package](https://img.shields.io/pub/v/provider.svg)](https://pub.dartlang.org/packages/provider) [![codecov](https://codecov.io/gh/rrousselGit/provider/branch/master/graph/badge.svg)](https://codecov.io/gh/rrousselGit/provider)

A dependency injection system built with widgets for widgets. `provider` is mostly syntax sugar for `InheritedWidget`,
to make common use-cases straightforward.

## Usage

### Exposing a value

To expose a variable using `provider`, simply wrap any widget into one of the provider widgets from this packages
and pass it your variable. Then, all descendants of the newly added provider widget can access this variable.

A naive example would be to wrap the entire application into a `Provider` widget and pass it our variable:

```dart
Provider<String>.value(
  value: 'Hello World',
  child: MaterialApp(
    home: Home(),
  )
)
```

Alternatively for complex objects, most provider expose a constructor that takes a function to create the value.
The provider will cal that function only once, when the widget is inserted in the tree, and expose the result.
This is perfect to expose a complex object that never change over time without writing a `StatefulWidget`.

The following creates and expose a `MyComplexClass`. And in the event where `Provider` is removed from the widget tree,
the instantiated `MyComplexClass` will be disposed.

```dart
Provider<MyComplexClass>(
  builder: (context) => MyComplexClass(),
  dispose: (context, value) => value.dispose()
  child: SomeWidget(),
)
```

### Reading a value

The easiest way to read a value is using the a static method `Provider.of<T>(BuildContext context)`. This method will look
up in widget tree starting from the widget associated to the `BuildContext` passed and it will return the nearest variable
of type `T` found (or throw if nothing if found).

TODO: link

Combined with the first example of [exposing a value](), this widget will read the exposed `String` and render "Hello World".

```dart
class Home extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Text(
      /// Don't forget to pass the type of the object you want to obtain to `Provider.of`!
      Provider.of<String>(context)
    );
  }
}
```

Alternatively instead of using `Provider.of`, we can use the `Consumer` widget.

This can be useful for performances optimisations or when it is difficult to obtain a `BuildContext` descendant of the provider.

```dart
Provider<String>.value(
  value: 'Hello World',
  child: Consumer<String>(
    builder: (context, value) => Text(value);
  ),
)
```

---

Note that you can freely use multiple providers with different types together:

```dart
Provider<int>(
  value: 42,
  child: Provider<String>(
    value: 'Hello World',
    child: // ...
  )
)
```

And obtain their value independently:

```dart
var value = Provider.of<int>(context);
var value2 = Provider.of<String>(context);
```

### MultiProvider

In big applications where many values are injected, `Provider` can rapidly become pretty nested:

```dart
Provider<Foo>(
  value: foo,
  child: Provider<Bar>(
    value: bar,
    child: Provider<Baz>(
      value: baz,
      child: someWidget,
    )
  )
)
```

In that situation, we can use `MultiProvider` to improve the readability:

```dart
MultiProvider(
  providers: [
    Provider<Foo>(value: foo),
    Provider<Bar>(value: bar),
    Provider<Baz>(value: baz),
  ],
  child: someWidget,
)
```

The behavior of both examples is strictly the same. `MultiProvider` only changes the appearance of the code.
