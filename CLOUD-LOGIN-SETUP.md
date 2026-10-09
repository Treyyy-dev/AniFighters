# Turn on "Login with Google" (cloud save)

The game already has the login buttons (Settings → LOGIN / CLOUD SAVE). They start working once you connect a free Firebase project. Progress then saves to the player's account automatically and loads on any phone, tablet or computer they log in on.

## 1. Create the Firebase project (free)
1. Go to https://console.firebase.google.com and click **Add project**. Any name works (for example `anifighters`). Google Analytics is optional.
2. The free **Spark** plan is enough.

## 2. Turn on Google login
1. In the left menu: **Build → Authentication → Get started**.
2. **Sign-in method** tab → **Google** → turn it on → pick a support email → **Save**.

## 3. Create the cloud save database
1. **Build → Firestore Database → Create database** → pick a location → **Start in production mode**.
2. Open the **Rules** tab, replace everything with this, and click **Publish**:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /saves/{uid} {
      allow read: if request.auth != null && request.auth.uid == uid;
      allow write: if request.auth != null && request.auth.uid == uid
                   && request.resource.data.data is string
                   && request.resource.data.data.size() < 900000;
    }
  }
}
```
Each player can only read and write their own save.

## 4. Connect the game
1. Firebase **Project settings** (gear icon) → **Your apps** → click the **Web** icon `</>` → give it a name → **Register app**.
2. Firebase shows a `firebaseConfig` with `apiKey`, `authDomain`, `projectId`, `appId` and more.
3. Open `index.html` in this folder, find `window.ANI_FIREBASE = null;` near the top, and replace it with your values:

```
window.ANI_FIREBASE = {
  apiKey: "AIza....",
  authDomain: "your-project.firebaseapp.com",
  projectId: "your-project",
  appId: "1:1234567890:web:abcdef"
};
```
These values are meant to be public. The database rules above are what keep saves private.

## 5. Allow your website
**Authentication → Settings → Authorized domains** → **Add domain** → the address where you host the game (for example `yourname.github.io` or `anifighters.netlify.app`).

## Best option for iPhone: host the game on Firebase Hosting (free)
Logging in from the installed iPhone app works most reliably when the game is hosted on the same Firebase project:
1. Install the Firebase tools on a computer (`npm install -g firebase-tools`), run `firebase login`, then in this folder run `firebase init hosting` (public folder: `.`, single-page app: No) and `firebase deploy`.
2. Your game is then at `https://your-project.web.app`. Set `authDomain` in `index.html` to `your-project.web.app` and deploy again.

## How it works for players
- Settings → **LOGIN WITH GOOGLE**. Works on iPhone, Android, tablets and computers.
- If the account has no save yet, this device's progress is uploaded.
- If the account already has progress (from another device), the game asks which one to keep.
- After that, progress saves automatically a few seconds after every change, and when the app is closed. **SAVE NOW** forces a save. **SIGN OUT** keeps the progress on that device.
- Friends, chat history, player level, fighters, gems and settings are all part of the save, so they follow the account.
