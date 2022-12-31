---
layout: post
title: How To Integrate Stripe Card Scan In Your Android App
author: Carlos Molero Mata
description: Learn how you can integrate the stripe library for credit card scanning into your Android app with Kotlin and Java.
header-style: text
tags: [java, android, stripe, kotlin]
---

Scanning credit cards information to automatically fill out a payment form is an excellent option to streamline this task and provide a good user experience. It reduces errors and saves the user from having to go through the tedious task of entering card details one by one, indirectly influencing sales.

Stripe recently released a library that allows us to scan credit cards from an Android device and get the card number. I used to work with [card.io](https://github.com/card-io/card.io-Android-SDK) to scan credit cards but the repository appears to be archived and I think Stripe's scanning SDK is a robust and easy to integrate option (that you can use whether you use this payment gateway or not).

You will see how easy it is to integrate this library and how much you can benefit from it, let's get started.

Before anything else, be sure to set the `compileSdk` and `targetSdk` properties of your `build.gradle` file to **33**. Otherwise the library won't work:

```gradle
android {
  // ...
  compileSdk 33

  defaultConfig {
      targetSdk 33
      // ...
  }
}
```

## Kotlin example

If you go to [stripecardscan](https://github.com/stripe/stripe-android/tree/master/stripecardscan), inside the Stripe repository, you will find the following code snippet as an example:

```kotlin
class LaunchActivity : AppCompatActivity {

  override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    setContentView(R.layout.activity_launch)

    /**
     * Create a [CardScanSheet] instance with [ComponentActivity].
     *
     * This API registers an [ActivityResultLauncher] into the
     * [ComponentActivity], it must be called before the [ComponentActivity]
     * is created (in the onCreate method).
     */
    val cardScanSheet = CardScanSheet.create(
      from = LaunchActivity.this,
      stripePublishableKey = "stripe_key",
    )

    findViewById(R.id.scanCardButton).setOnClickListener { _ ->
      cardScanSheet.present(
        from = LaunchActivity.this,
      ) { cardScanSheetResult ->
        when (cardScanSheetResult) {
          is CardScanSheet.Completed -> {
            /*
             * The user scanned a card. The result of the scan can be found
             * by querying the stripe card image verification endpoint with
             * the CVI_ID, CVI_SECRET, and Stripe secret key.
             *
             * Details about the card itself are returned in the `scannedCard`
             * field of the result.
             */
            Log.i(cardScanSheetResult.scannedCard.pan)
          }
          is CardScanSheet.Canceled -> {
            /*
             * The scan was canceled. This could be because any of the
             * following reasons (returned as the
             * [CancellationReason] in the result):
             *
             * - Closed - the user pressed the X
             * - Back - the user pressed the back button
             * - UserCannotScan - the user is unable to scan this card
             * - CameraPermissionDenied - the user did not grant permissions
             */
            Log.i(cardScanSheetResult.reason)
          }
          is CardScanSheet.Failed -> {
            /*
             * The scan failed. The error that caused the failure is
             * included in the [Throwable] `error` field of the verification
             * result.
             */
            Log.e(cardScanSheetResult.error.message)
          }
        }
      }
    }
  }
}
```

If you were looking for the Kotlin example, here you go. To know how it is implemented in Java keep reading.

## Java example

### Creating a `CardScanSheet` object instance

The `CardScanSheet.create()` method overload that we are going to use accepts the following arguments:

- The `Activity` context
- The Stripe publishable key
- A callback method which will receive the `CardScanSheetResult`
- The `ActivityRegistry` of the given context (we will not use `onActivityResult` in this example, but it can be used instead of the callback method)

```java
@Override
protected void onCreate(Bundle savedInstanceState) {
  // ...

  final CardScanSheet cardScanSheet = CardScanSheet.create(this, "<STRIPE_PK>",
  this::onCardScanFinished, getActivityResultRegistry());
}
```

After this, create your callback method declaring a `CardScanSheetResult` type parameter in the method signature:

```java
private void onCardScanFinished(CardScanSheetResult result){
  // TODO: Implement
}
```

### Getting the scan result

Before getting the result implement an `onClick` listener where you call `cardScanSheet.isPresent()` and attach it to a button, calling this method will open the scanning GUI.

If we open the Kotlin file where the `CardScanSheetResult` interface is declared you will find this implementation:

```kotlin
sealed interface CardScanSheetResult : Parcelable {
  @Parcelize
  data class Completed(
      val scannedCard: ScannedCard
  ) : CardScanSheetResult

  @Parcelize
  data class Canceled(
      val reason: CancellationReason
  ) : CardScanSheetResult

  @Parcelize
  data class Failed(val error: Throwable) : CardScanSheetResult
}
```

We can see that this interface extends `Parcelable`, `Parcelable` is just the Android implementation of Java `Serializable`. It is processed relatively fast, compared to the standard Java serialization and used mainly to pass data between intents. It also holds 3 classes which extend from `CardScanSheetResult`: `Completed`, `Canceled` and `Failed`.

To access the result you must check to which subclass the instance you receive belongs, in the Kotlin example the instance check is done with the `is` keyword:

```kotlin
is CardScanSheet.Completed -> {
  // do something
}
```

In Java, we need to use `instanceof` to check the instance type of the result and cast and assign it to a correctly typed object variable. Then, we can get the specific instance variables and proceed with our logic.

```java
private void onCardScanFinished(CardScanSheetResult result){
  if(result instanceof CardScanSheetResult.Completed){
      CardScanSheetResult.Completed r = (CardScanSheetResult.Completed) result;
      String cardNumber = r.getScannedCard().getPan();
      // do something with the card number
  } else if(result instanceof CardScanSheetResult.Failed){
      CardScanSheetResult.Failed r = (CardScanSheetResult.Failed) result;
      System.out.println(r.getError().getMessage());
  } else if(result instanceof CardScanSheetResult.Canceled){
      CardScanSheetResult.Canceled r = (CardScanSheetResult.Canceled) result;
      // Check instance type of CancellationReason (UserCannotScan, Back, Closed, CameraPermissionDenied)
  }
}
```

That's all for today, I hope you were able to learn something new and see you in the next post. Don't forget to share any opinions or suggestions for improvement in the comments.
