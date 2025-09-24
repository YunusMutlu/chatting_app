
# FlutterChat (Realtime Chat App)

Lightweight cross‑platform realtime chat application built with Flutter & Firebase (Auth, Firestore, Cloud Messaging, Storage, Cloud Functions).


## ✨ Features

- Email & password authentication (Firebase Auth)
- Persistent login (auth state stream)
- Profile image upload (Firebase Storage + image picker)
- Realtime messaging (Cloud Firestore) with live stream updates
- Server side push notifications (FCM) via a Firestore trigger Cloud Function
- Topic based notification broadcast (`topic: chat`)
- Adaptive UI (Material 3 seed color, responsive layout basics)
- Graceful splash / loading screen while auth state resolves
- Clean separation of screens & widgets

## 🗂️ Project Structure (trimmed)


lib/
	main.dart                # App entry, Firebase init, auth state routing
	screens/
		auth.dart              # Sign up / login logic & UI
		chat.dart              # Chat screen (messages + input)
		splash.dart            # Waiting screen during initialization
	widgets/
		user_image_picker.dart # Select & preview user avatar
		chat_messages.dart     # (Expected) list view of messages
		new_messages.dart      # (Expected) input & send form
functions/
	index.js                 # Firestore onCreate trigger -> FCM notification
android/ ios/ web/ ...     # Platform specific code


## 🧱 Tech Stack

| Layer               | Technology                         |
| ------------------- | ---------------------------------- |
| UI                  | Flutter (Material)                 |
| State (lightweight) | Widget state / Streams             |
| Auth                | Firebase Authentication            |
| Database            | Cloud Firestore (realtime)         |
| Storage             | Firebase Storage (profile images)  |
| Notifications       | Firebase Cloud Messaging (FCM)     |
| Backend Logic       | Firebase Cloud Functions (2nd gen) |

## 🚀 Getting Started

### 1. Prerequisites

- Flutter SDK installed (`flutter --version`)
- Firebase CLI (`firebase --version`)
- FlutterFire CLI (`dart pub global activate flutterfire_cli`)
- A Firebase project created in the Firebase console

### 2. Clone

bash
git clone https://github.com/YunusMutlu/chatting_app.git
cd chatting_app


### 3. Install Dependencies

bash
flutter pub get


### 4. Firebase Configuration

This repository intentionally **ignores sensitive Firebase config files**:

Ignored in `.gitignore`:


lib/firebase_options.dart
android/app/google-services.json
ios/Runner/GoogleService-Info.plist


Regenerate them locally with:

bash
flutterfire configure


Select your Firebase project and target the platforms you need (at minimum Android or iOS or Web). This will create `lib/firebase_options.dart` and place the required platform config files.

### 5. Run the App

bash
# Android emulator / physical device
flutter run -d android

# iOS (macOS only)
flutter run -d ios

# Web
flutter run -d chrome


### 6. Deploy Cloud Function (Notifications)

Make sure you have billing enabled if required for certain advanced features (basic Firestore trigger + FCM usually works on the free tier):

bash
cd functions
npm install   # (if not already)
cd ..
firebase deploy --only functions


If you see an Eventarc permission warning on first deploy (2nd gen functions), wait a few minutes and redeploy:

bash
firebase deploy --only functions --force


### 7. Test Push Notifications

1. Run the app on an Android device
2. Accept notification permission (Android 13+ shows runtime prompt)
3. Send a new message in the chat
4. Cloud Function should broadcast a push notification to the `chat` topic

## 🔐 Security Notes

- Never commit `firebase_options.dart` or service account JSON keys.
- For production, add Firestore Security Rules + Storage Rules tailored to your data model.
- Limit Cloud Function IAM roles (principle of least privilege).
- Consider using App Check for additional abuse protection.

## 🧪 Potential Enhancements (Roadmap)

- Typing indicators
- Message reactions & read receipts
- Image / file message attachments
- Message pagination + lazy loading
- User presence (online / last seen)
- Pin / star messages
- Theming (dark mode toggle)
- Localization (i18n)

## 🛠️ Development Scripts

bash
# Format & analyze
flutter format lib
flutter analyze

# Run tests
flutter test


## 🧩 Cloud Function (Overview)

The `functions/index.js` listens to Firestore document create events under `chat/{messageId}` and sends a topic notification using FCM. Example payload:

js
exports.myFunction = functions.onDocumentCreated(
  "chat/{messageId}",
  (event) => {
    const data = event.data.data();
    return admin.messaging().send({
      notification: {
        title: data["username"],
        body: data["text"],
      },
      data: { click_action: "FLUTTER_NOTIFICATION_CLICK" },
      topic: "chat",
    });
  }
);



## 🤝 Contributing

PRs & suggestions welcome! Please open an issue first for significant changes.

## 🙋 Support

If you run into setup issues:

- Clear `build/` and `.dart_tool/` then retry
- Ensure Firebase project is selected: `firebase use --add`
- Re-run: `flutterfire configure`
- For Cloud Functions logs: `firebase functions:log`

Happy coding! 🚀
