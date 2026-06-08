# ⛽ Fuel Tracker — Support

Fuel Tracker is an internal tool for logging fuel usage across tanks, trailers, and service stations. Access is by invitation only — contact your administrator to get an account.

---

## Frequently Asked Questions

**I can't log in**
Make sure you're using the email and password set up by your administrator. If you've forgotten your password, tap **Forgot password?** on the login screen and a reset link will be sent to your email.

**A transaction I logged isn't showing up**
If your device was offline when you logged it, the transaction is saved locally and syncs automatically once you're back online. You'll see an orange banner on the dashboard while items are waiting to sync.

**How do I attach a receipt photo?**
When logging a transaction, scroll down to **Receipt Photo** and tap **Add receipt photo**. You can take a new photo with the camera or choose one from your photo library.

**What is a meter irregularity?**
Fuel Tracker checks the meter readings entered on each withdrawal. If readings go backwards, skip unexpectedly, or don't match the amount withdrawn, the entry is flagged. Admins can review and accept flagged entries from the dashboard.

**How do I export transactions?**
Admins can export a CSV from **Settings → Export Transactions**. Filter by tank, vehicle, and date range, then tap **Export as CSV** to share or save the file.

**Can I use the app without internet?**
Yes. Transactions logged without an internet connection are saved on your device and automatically uploaded when connectivity is restored.

**How do I add a new tank or user?**
Tank and user management is available to admins under **Settings → Manage Tanks** and **Settings → Manage Users**.

**What is a Service Station tank?**
Service station tanks have unlimited supply. The app logs withdrawals and tracks usage but does not deduct from a running level or show a fill percentage.

---

## Contact

For help, email [support@cfrandsen.com](mailto:christianfrandsen6@hotmail.com)

---

# Fuel Tracker — Setup Guide

## 1. Firebase Setup

1. Go to [Firebase Console](https://console.firebase.google.com) and create a new project.
2. Add an **iOS app** to the project. Use your app's Bundle ID (e.g. `com.yourname.FuelTracker`).
3. Download the `GoogleService-Info.plist` and drag it into your Xcode project target.
4. Enable **Authentication** → Email/Password sign-in.
5. Enable **Firestore Database** → Start in production mode, then set the rules below.

### Firestore Security Rules

These rules enforce role-based access. Admins can do everything; Operators can only log transactions and read data.

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {

    // Helper: get the current user's role from their profile doc
    function userRole() {
      return get(/databases/$(database)/documents/users/$(request.auth.uid)).data.role;
    }
    function isAdmin() {
      return request.auth != null && userRole() == 'admin';
    }
    function isOperator() {
      return request.auth != null && userRole() == 'operator';
    }
    function isSignedIn() {
      return request.auth != null;
    }

    // Users collection — anyone signed in can read their own doc
    // Admins can read/write all user docs
    match /users/{uid} {
      allow read: if isSignedIn();
      allow create: if isSignedIn() && request.auth.uid == uid;
      allow update, delete: if isAdmin();
    }

    // Tanks — admins can create/edit/delete; operators can read only
    match /tanks/{tankId} {
      allow read: if isSignedIn();
      allow create, update, delete: if isAdmin();
    }

    // Vehicles — same as tanks
    match /vehicles/{vehicleId} {
      allow read: if isSignedIn();
      allow create, update, delete: if isAdmin();
    }

    // Transactions — anyone signed in can create; read by all; delete by admin only
    match /transactions/{txId} {
      allow read: if isSignedIn();
      allow create: if isSignedIn();
      allow update: if isAdmin();
      allow delete: if isAdmin();
    }
  }
}
```

> **Note:** The tank level update that happens during a transaction write is done via a Firestore batch from the app. Because operators can write transactions but not tanks directly, you'll need to use a **Cloud Function** (or grant operators tank write access) for the atomic level update. The simplest option is to also allow operators to update the `currentLevelLiters` field on tanks — change the tanks rule to:
> ```
> allow update: if isSignedIn();
> ```
> and rely on app-level validation to keep it safe.

### Required Firestore Indexes

Create a composite index for the history filter to work:
- Collection: `transactions`
- Fields: `tankId` (Ascending), `timestamp` (Descending)

(Firestore will prompt you with a link to create this automatically when you first use the filter.)

---

## 2. Swift Package Dependencies

In Xcode: **File → Add Package Dependencies…**

Add: `https://github.com/firebase/firebase-ios-sdk`

Select these products:
- `FirebaseAuth`
- `FirebaseFirestore`
- `FirebaseFirestoreSwift`

---

## 3. Add Source Files to Xcode

All Swift files in the `Fuel App` folder need to be added to your Xcode target:

1. In Xcode, right-click your app group in the Project Navigator.
2. Choose **Add Files to "FuelTracker"…**
3. Select all the subfolders (`Models`, `ViewModels`, `Views`) and files.
4. Make sure **"Add to targets"** is checked for your app target.
5. Also add `FuelTrackerApp.swift` and `MainTabView.swift`.

> **Important:** Delete the default `ContentView.swift` Xcode created — it's replaced by `MainTabView.swift`.

---

## 4. App Entry Point

Make sure `FuelTrackerApp.swift` is set as the `@main` entry point. If Xcode created a default `<AppName>App.swift`, delete it and use the one in this folder instead.

---

## 5. Data Structure

Firestore collections used:
- `tanks` — fuel tank records
- `transactions` — all fuel withdrawal/refill logs
- `vehicles` — vehicles and equipment

---

## 6. Features

- **Multiple tanks** — stationary tanks and fuel trailers
- **Log withdrawals and refills** — with live before/after level preview
- **Vehicle/equipment linking** — associate fuel with a specific piece of equipment
- **Real-time sync** — all users see live updates via Firestore listeners
- **Multi-user** — Firebase Auth with email/password; all authenticated users share the same data
- **Transaction history** — filterable by tank, grouped by date
- **Low fuel warnings** — tanks below 20% are flagged red
