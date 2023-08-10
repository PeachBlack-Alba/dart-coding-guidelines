# dart-coding-guidelines
A few conventions for naming and ordering packages, classes, methods and variables which are in line with best practices.

# Use brackets for if-else statements
// Do

```
if (...) {
  // if-code
} else {
  // else-code
}
```

// Don't

```
if (condition)
  // if-code
else
  // else-code
```

Exception: When you have an if statement with no else clause and the whole if statement fits on one line, you can omit the braces if you prefer:
```
if (arg == null) return defaultValue;
```

If the body wraps to the next line, though, use braces:
// Do

```
if (overflowChars != other.overflowChars) {
  return overflowChars < other.overflowChars;
}
```

// Don’t

```
if (overflowChars != other.overflowChars)
  return overflowChars < other.overflowChars;
```

# “dart:” imports must come before other imports.

```
import ‘dart:async’;
import ‘dart:html’;
import ‘package:model/UserModel.dart’;
import ‘package:network/HttpRequest.dart’
```

# “export:” should be after other imports.

```
import ‘package:dltwallet/src/appflow/app_coordinator.dart’;
import ‘package:dltwallet/src/appflow/main_flow/main_tab.dart’;
import ‘package:dltwallet/src/appflow/splash_screen.dart’;
export ‘package:async/src/error.dart’;
```

# Use the ternary operator for single-line if-else statements

// Do

```
condition ? true-case : false-case;
```

// Don't

```
if (condition) {
  // true-case
} else {
  // false-case
}
```

# Use doc comments i.e. /// for methods and classes

// Do

```
/// This is a doc comment describing the purpose of the class
class A {

}
```

// Don't

```
// This is a normal comment describing the purpose of the class
class A {

}
```

# Always use trailing commas for fold statements

// Do

```
final failureOrResult = await getIt<FeatureUseCase>().call();
yield* failureOrResult.fold(
  (failure) => ErrorState(failure.message),
  (result) => SuccessState(result),
); 
```

// Don't

```
final failureOrResult = await getIt<FeatureUseCase>().call();
yield* failureOrResult.fold((failure) => ErrorState(failure.message), (result) => SuccessState(result)); 
```

# Prefer positive English conditions as they are easier to read

// Do

```
x == null ? doA() : doB();
```

// Don't

```
x != null ? doB() : doA();
```

# BLoC event names should describe the event, not the result of the event.

// Do

```
class AddToCartButtonPressedEvent extends CartEvent {}
```

// Don't

```
class CacheCartInLocalStorageEvent extends CartEvent {}
```

// Do

```
class OrdersListPageCreatedEvent extends OrdersListEvent {}
```

// Don't

```
class GetOrdersListEvent extends OrdersListEvent {}
```

This is in line with what [this example]([http://example.com](https://bloclibrary.dev/#/flutterbloccoreconcepts?id=counter_pagedart)) in the official Flutter BLoC documentation uses as well:

At this point we have successfully separated our presentational layer from our business logic layer. Notice that the CounterPage widget knows nothing about what happens when a user taps the buttons. The widget simply tells the CounterBloc that the user has pressed either the increment or decrement button.

# Do not prefix a BLoC event name with "On"

// Do

```
class AddToCartButtonPressedEvent extends CartEvent {}
```

// Don't

```
class OnAddToCartButtonPressedEvent extends CartEvent {}
```

# Always specify types for Blocs. Helps with Bloc lookup.

// Do

```
BlocConsumer<LoginBloc, LoginState>(
  bloc: ...,
)
```

// Don't

```
BlocConsumer(
  bloc: ...,
)
```

// Do

```
BlocProvider<LoginBloc>(
  create: ...,
)
```

// Don't

```
BlocProvider(
  create: ...,
)
```

# Use => only when the statement fits the single line length as well. Otherwise, use block body.

// Do

```
bool get isSupplierInfoValid {
  return name.isNotBlank() &&
      EmailValidator.validate(email) &&
      PhoneNumberValidator.validate(phone) &&
      invoiceImages.isNotEmpty;
}
```

// Don't

```
bool get isSupplierInfoValid =>
    name.isNotBlank() &&
    EmailValidator.validate(email) &&
    PhoneNumberValidator.validate(phone) &&
    invoiceImages.isNotEmpty;
```

* 2 entities within the same layer should never talk to each other:
* 2 usecases should never talk to each other
* 2 repositories should never talk to each other
* 2 data sources should never talk to each other

# // GetUserSettingsUseCase (get_user_settings_use_case.dart)

// Do 

```
Future<Either<Failure, UserSettings>> call(NoParams params) async {
  final String entityId = await _entityRepository.getCurrentEntityId();

  final String countryCode = await _entityRepository.getCurrentCountryCode();

  final List<InvitedUser> invitedUsers = await _teamRepository.getInvitedUsers();

  ...
}
```

// Don't

```
Future<Either<Failure, UserSettings>> call(NoParams params) async {
  final String entityId = await _entityRepository.getCurrentEntityId();

  final String countryCode = await _entityRepository.getCurrentCountryCode();

  final List<InvitedUser> invitedUsers = await getIt<GetInvitedUsersUseCase>().call(); // GetUserSettingsUseCsae shouldn't be talking to GetInvitedUsersUseCase directly

  ...
}
```

# When to use something via getIt directly:

* Use only Services in usecases/blocs via getIt.
* Use only Usecases in blocs via getIt.
* Everything else should be injected via constructors.

// Do

```
// in a use-case
final bool enableGroupChats = getIt<RemoteConfigService>().shouldShowGroupChats; // service used via getIt
final List<InvitedUser> invitedUsers = _teamRepository.getInvitedUsers(); // repository injected via constructor

// in a bloc
final bool enableGroupChats = getIt<RemoteConfigService>().shouldShowGroupChats; // service used via getIt
final List<InvitedUser> invitedUsers = getIt<GetInvitedUsersUseCase>().call(); // repository used via getIt`
```

// Don't 
```
// in a repository
final bool enableGroupChats = getIt<RemoteConfigService>().shouldShowGroupChats;  // service used via getIt BUT in a repository 

// in a use-case
final List<InvitedUser> invitedUsers = getIt<TeamRepository>().getInvitedUsers();  // repository used via getIt

// in a bloc
final List<InvitedUser> invitedUsers = getInvitedUsersUseCase.call(); 

// usecase injected via constructor [We don't want `getIt` to start appearing in the widget layer]
```


# Use widget-classes, not widget-methods:

// Do
``` 
class _ProductsList extends StatelessWidget {
   @override
   Widget build(BuildContext context) {
	  return ListView.builder(...);
   }
}
```
// Don’t

```
_getProductsList() {
   return ListView.builder(...);
}
```
 
// Read more here: https://stackoverflow.com/a/53234826/5066615


# Use async/await
For raw future functions, use aync/await. Any function you want to run asynchronously must have the async modifier added to it. When you are adding the await modifier, the code is explicitly saying: “don’t go further until my future is completed”.
For example:

```
function () async {
var data = await loadData();
// Do something…
}
```
