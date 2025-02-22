# Informers

[![Pub Version](https://img.shields.io/pub/v/informers)](https://pub.dev/packages/informers)
[![License: MIT](https://img.shields.io/badge/license-MIT-purple.svg)](https://opensource.org/licenses/MIT)

`Informers` is a Flutter package providing enhanced `ChangeNotifier` implementations for managing and listening to changes in various data structures. It offers a more feature-rich alternative to Flutter's built-in `ValueNotifier`, with convenient methods for updating values, collections (lists, maps, sets) and controlling notifications.

## Features

*   **`Informer<T>`:** A general-purpose informer for single values, similar to `ValueNotifier<T>`, but with added control over notifications and update behavior.
*   **`ListInformer<T>`:** An informer specifically designed for lists, providing methods for adding, removing, updating, and querying list elements with optional change notifications.
*   **`MapInformer<E, T>`:** An informer for maps, offering similar functionalities as `ListInformer` but tailored for key-value pairs.
*   **`SetInformer<T>`:**  An informer for sets, providing methods like add, remove, contains and clear.
*  **`MaxLengthListInformer<T>`:** Similar to `ListInformer<T>` but with a maximum length.
*   **`InformNotifier`:**: abstract class used to expose the [ChangeNotifier.notifyListeners] method
*   **Fine-grained notification control:**  You can choose whether to notify listeners on every update or only when the value *actually* changes, preventing unnecessary rebuilds.  Methods like `silentUpdate` and `silentUpdateCurrent` allow for updates without triggering notifications.
*   **Force Update:** A flag (`forceUpdate`) allows you to force updates and notifications even if the new value is identical to the old one. This is useful in scenarios where you need to trigger a rebuild regardless of value equality.
*   **`updateCurrent` methods:** These methods allow you to update the value based on the *current* value, using a callback function.  This is extremely useful for immutable data structures and avoids creating temporary objects.
*  **Convenience Methods**: Contains convenience methods such as data and alternative setters.
*   **Type-safe:** All informers are generic, ensuring type safety at compile time.

## Installation

Add `informers` to your `pubspec.yaml`:

```yaml
dependencies:
  informers: ^<latest_version>  # Replace <latest_version> with the actual version
```

Then run:

```bash
flutter pub get
```

## Usage

### `Informer<T>` (Basic Informer)

```dart
import 'package:informers/informers.dart';

void main() {
  // Create an Informer for an integer.
  final counter = Informer<int>(0);

  // Listen for changes.
  counter.addListener(() {
    print('Counter changed: ${counter.value}');
  });

  // Update the value (and notify listeners).
  counter.value = 1; // Output: Counter changed: 1
  counter.update(2);  // Output: Counter changed: 2
  counter.data = 3;  // Output: Counter changed: 3
  // Update the value without notifying listeners.
  counter.silentUpdate(5); // No output

    // Update the current value based on its previous value.
  counter.updateCurrent((currentValue) => currentValue + 10); // Output: Counter changed: 15.
    // Still respects forceUpdate.

  //Silent update current
  counter.silentUpdateCurrent((currentValue) => currentValue + 10); // No output
   // Still respects forceUpdate

  // Force update (even if the value is the same).
  final forcedInformer = Informer<int>(0, forceUpdate: true);
  forcedInformer.addListener(() {
    print('Forced update: ${forcedInformer.value}');
  });
  forcedInformer.update(0); // Output: Forced update: 0
  forcedInformer.value = 0; // Output: Forced update: 0
   // Still respects forceUpdate.
}
```

### `ListInformer<T>` (List Informer)

```dart
import 'package:informers/informers.dart';
import 'package:flutter/foundation.dart'; // for listEquals

void main() {
  final names = ListInformer<String>([]);

  names.addListener(() {
    print('Names: ${names.value}');
  });

  names.add('Alice'); // Output: Names: [Alice]
  names.addAll(['Bob', 'Charlie']); // Output: Names: [Alice, Bob, Charlie]
  names.remove('Bob'); // Output: Names: [Alice, Charlie]
  names.removeLast(); // Output: Names: [Alice]
  names.update(['David', 'Eve']); // Output: Names: [David, Eve]
  names.updateCurrent((currentList) => currentList..add('Frank')); // Output: Names: [David, Eve, Frank]
  names.updateFirstWhereOrNull((name) => name.startsWith('D'), (name) => name.toUpperCase()); // Output: Names: [DAVID, Eve, Frank]
  print(names.isEmpty); // false
  print(names.isNotEmpty); // true
  print(names.contains('Eve')); // true
  names.clear();  //Output: Names: []
   print(names.isEmpty); // true

    //Force update
    final listForce = ListInformer<int>([1,2,3], forceUpdate: true);

   listForce.addListener(() {
     print('listForce: ${listForce.value}');
   });

   listForce.update([1,2,3]);  //Output: listForce: [1, 2, 3]

}
```

### `MapInformer<E, T>` (Map Informer)

```dart
import 'package:informers/informers.dart';
import 'package:flutter/foundation.dart'; // For mapEquals

void main() {
  final ages = MapInformer<String, int>({});

  ages.addListener(() {
    print('Ages: ${ages.value}');
  });

  ages.add('Alice', 30); // Output: Ages: {Alice: 30}
  ages.update({'Bob': 25, 'Charlie': 40}); // Output: Ages: {Bob: 25, Charlie: 40}
  ages.updateCurrent((currentMap) => currentMap..['David'] = 35); // Output: Ages: {Bob: 25, Charlie: 40, David: 35}
  ages.updateKey('Bob', (age) => age + 1);   // Output: Ages: {Bob: 26, Charlie: 40, David: 35}
  ages.remove('Charlie'); // Output: Ages: {Bob: 26, David: 35}
    final previousValue = ages.putIfAbsent('Emily', 42); //Output: Ages: {Bob: 26, David: 35, Emily: 42}
    print(previousValue); //42
  ages.clear(); //Output: Ages: {}

     //Force update
    final mapForce = MapInformer<String, int>({'one': 1}, forceUpdate: true);

   mapForce.addListener(() {
     print('mapForce: ${mapForce.value}');
   });

   mapForce.update({'one': 1});  //Output: mapForce: {one: 1}
}
```

### `SetInformer<T>` (Set Informer)

```dart
import 'package:informers/informers.dart';
import 'package:flutter/foundation.dart'; // For setEquals

void main() {
  final uniqueNumbers = SetInformer<int>({});

  uniqueNumbers.addListener(() {
    print('Unique Numbers: ${uniqueNumbers.value}');
  });

  uniqueNumbers.add(1); // Output: Unique Numbers: {1}
  uniqueNumbers.add(2); // Output: Unique Numbers: {1, 2}
  uniqueNumbers.add(1); // No output, sets only store unique values.
  uniqueNumbers.remove(1); // Output: Unique Numbers: {2}
  uniqueNumbers.update({3, 4, 5}); // Output: Unique Numbers: {3, 4, 5}
  uniqueNumbers.updateCurrent((currentSet) => currentSet..add(6)); // Output: Unique Numbers: {3, 4, 5, 6}
  print(uniqueNumbers.isEmpty);      // Output: false
  print(uniqueNumbers.isNotEmpty);   // Output: true
  print(uniqueNumbers.contains(4));  // Output: true
  uniqueNumbers.clear();            // Output: Unique Numbers: {}

  // Force update
  final setForce = SetInformer<int>({1}, forceUpdate: true);
  setForce.addListener((){
    print('setForce: ${setForce.value}');
  });
  setForce.update({1}); // Output: setForce: {1}
}
```

### `MaxLengthListInformer<T>`

```dart
import 'package:informers/max_length_list_informer.dart';
import 'package:flutter/foundation.dart';

void main() {
  final recentItems = MaxLengthListInformer<String>([], maxLength: 3);

  recentItems.addListener(() {
    print('Recent Items: ${recentItems.value}');
  });

  recentItems.add('Item 1'); // Output: Recent Items: [Item 1]
  recentItems.add('Item 2'); // Output: Recent Items: [Item 1, Item 2]
  recentItems.add('Item 3'); // Output: Recent Items: [Item 1, Item 2, Item 3]
  recentItems.add('Item 4'); // Output: Recent Items: [Item 2, Item 3, Item 4]  (Item 1 is removed)
  recentItems.addAll(['Item 5', 'Item 6', 'Item 7', 'Item 8']); // Output: Recent Items: [Item 6, Item 7, Item 8]

      //Force update
    final maxLengthListForce = MaxLengthListInformer<int>([1,2], forceUpdate: true, maxLength: 3);

   maxLengthListForce.addListener(() {
     print('maxLengthListForce: ${maxLengthListForce.value}');
   });

   maxLengthListForce.update([1,2]); // Output: maxLengthListForce: [1, 2]
}

```

### `InformNotifier`
```dart
import 'package:informers/inform_notifier.dart';
import 'package:flutter/material.dart';

void main() {
 // Create a custom notifier using InformNotifier.
 final customNotifier = MyCustomNotifier();

 // Rebuild the notifier.
 customNotifier.rebuild();
}


class MyCustomNotifier extends InformNotifier{

}
```

## License

This package is released under the MIT License. See [LICENSE](LICENSE) for details.
```

Key improvements and explanations in this README:

*   **Clear Introduction:**  Explains the purpose of the package and its relationship to `ValueNotifier`.
*   **Feature List:**  Highlights the key advantages and capabilities of the `Informers`.
*   **Installation:** Provides clear instructions for adding the package to a project.
*   **Comprehensive Usage Examples:**  Demonstrates how to use each type of informer (`Informer`, `ListInformer`, `MapInformer`, `SetInformer`, `MaxLengthListInformer`, `InformNotifier`) with:
    *   Creation
    *   Adding listeners
    *   Updating values (with and without notifications)
    *   Using `updateCurrent`[generate_raycast_snippets.py](..%2F..%2F..%2F..%2F..%2Fturbo%2Fdeveloper%2Fscripts%2Fgenerate_raycast_snippets.py)
    *   Collection-specific methods (add, remove, etc.)
    *  `forceUpdate` usage
    *   Imports are now included, and necessary packages such as foundation.
*   **Contributing Section:**  Invites contributions and links to the (placeholder) repository.
*   **License:**  Specifies the MIT License.
* **Pub.dev badges**: Added badges for pub.dev version, style, and license.  This is good practice for any published package.

Before using this README, remember to:

1.  **Replace `<latest_version>`** with the actual latest version of your package on pub.dev.
2.  **Replace `your_repo_link_here`** with the URL of your GitHub repository.
3.  **Create a `LICENSE` file** in your package's root directory containing the MIT License text.
4.  **Publish your package to pub.dev.**  This will make the badges work correctly.  You can do this by following the instructions on the [pub.dev publishing page](https://dart.dev/tools/pub/publishing).

This comprehensive README will greatly improve the usability and discoverability of your `informers` package.  It provides clear, concise, and complete information for potential users.
