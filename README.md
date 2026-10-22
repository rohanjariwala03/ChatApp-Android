# ChatApp-Android

A one-to-one chat app for Android, written in Java on top of Firebase. Users sign up with email and password, find each other by username, and chat in real time. New messages trigger a push notification.

## Features

- Register, log in and reset a forgotten password (Firebase Authentication).
- Three tabs after login:
  - Chats: people you've talked to, with the last message. The tab title shows how many unread messages you have.
  - Users: everyone registered, with a search box that matches on username.
  - Profile: change your username and bio, and upload a profile picture (stored in Firebase Storage).
- Chat screen with message times and a "Delivered" / "Seen" label under the last message when you sent it.
- Online / offline status, set when the main or chat screen is resumed or paused.
- Tap a user's picture in the list to see their profile in a bottom sheet.
- Push notifications for new messages through Firebase Cloud Messaging.

## How it works

App data lives in the Firebase Realtime Database (profile pictures go to Storage):

- `Users/<uid>`: username, lowercase `search` field, bio, `imageURL`, status
- `Chats`: one entry per message (`sender`, `receiver`, `message`, `time`, `isseen`)
- `Chatlist/<uid>/<otherUid>`: who each user has a conversation with
- `Tokens/<uid>`: the device's FCM token

When a message is sent, the app looks up the receiver's token and posts a notification to `https://fcm.googleapis.com/fcm/send` using Retrofit (`Fragments/APIService.java`). `Notifications/MyFirebaseMessaging` shows it on the receiving device, with a separate notification channel on Android 8+.

## Stack

- Java, Android Gradle Plugin 3.1.3, Gradle 4.4
- compileSdk / targetSdk 27, minSdk 16, Android Support Library 27.1.1
- Firebase Auth, Realtime Database, Storage and Cloud Messaging (2018-era SDKs)
- Retrofit 2.3 with Gson, Glide 4.8, CircleImageView, MaterialEditText

## Running it

1. Create a Firebase project and add an Android app with the package name `com.saharsh.chatapp`.
2. Enable Email/Password sign-in, the Realtime Database and Storage.
3. Download `google-services.json` into `app/`. It's in `.gitignore`, so it isn't in this repo.
4. Open the project in Android Studio, or run `./gradlew assembleDebug`.

The build setup is from 2018 (it still pulls from `jcenter()`), so expect to update the Gradle plugin and dependencies before it builds in a current Android Studio.

Push notifications use the legacy FCM HTTP API with a server key in the request header. Google has since shut that API down, so notifications won't send without moving to the FCM HTTP v1 API, which needs a backend to sign requests.

## Project structure

```
app/src/main/java/com/saharsh/chatapp/
  StartActivity, LoginActivity, RegisterActivity, ResetPasswordActivity
  MainActivity          tabs, unread count, online status
  MessageActivity       chat screen, sends messages and notifications
  ViewProfileActivity   profile bottom sheet (a BottomSheetDialogFragment)
  Fragments/            Chats, Users, Profile tabs and the FCM Retrofit interface
  Adapter/              RecyclerView adapters for users and messages
  Model/                User, Chat, Chatlist
  Notifications/        FCM service, token handling, notification helpers
```
