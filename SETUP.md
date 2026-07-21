# Setup guide — student council voting app (Firebase + GitHub Pages/Netlify)

This version runs on a real backend (Firebase) instead of Claude's storage, so it works on any static host. It takes about 15 minutes to set up once.

## 1. Create a Firebase project (free)

1. Go to https://console.firebase.google.com and sign in with any Google account.
2. Click **Add project**, give it a name (e.g. "school-election"), and finish the wizard (you can skip Google Analytics).

## 2. Turn on Firestore (the database)

1. In the left sidebar, click **Build > Firestore Database**.
2. Click **Create database**. Choose a location close to your school, and start in **production mode**.
3. Once created, go to the **Rules** tab and replace the default rules with:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /config/election {
      allow read: if true;
      allow write: if request.auth != null;
    }
    match /roster/{id} {
      allow get: if true;
      allow list: if request.auth != null;
      allow write: if request.auth != null;
    }
    match /ballots/{id} {
      allow get: if true;
      allow list: if request.auth != null;
      allow create: if true;
      allow update, delete: if false;
    }
  }
}
```

4. Click **Publish**.

**What these rules do:** anyone can vote once per student ID (and it's locked after — no changing a vote). Only someone signed in as admin can edit candidates, manage the roster, or see the full list of votes/roster. Students can still check their own single roster entry to log in, but can't browse everyone else's.

## 3. Turn on admin login (Firebase Authentication)

1. In the sidebar, click **Build > Authentication**, then **Get started**.
2. Under **Sign-in method**, enable **Email/Password**.
3. Go to the **Users** tab, click **Add user**, and enter the email + password you (the election admin) want to log in with. This is your admin account — there's no "set up a passcode" step inside the app anymore, since this is real security.

## 4. Get your Firebase config

1. Click the gear icon near "Project Overview" > **Project settings**.
2. Scroll to **Your apps**, click the **</>** (web) icon to register a new web app (any nickname is fine).
3. Firebase shows you a `firebaseConfig` object — copy it.
4. Open `index.html` from this folder, find the section near the top marked:
   ```
   const firebaseConfig = {
     apiKey: "YOUR_API_KEY",
     ...
   };
   ```
   and paste your real values in.

## 5. Deploy it

**Option A — Netlify (easiest):**
1. Go to https://app.netlify.com/drop
2. Drag the folder containing `index.html` onto the page.
3. Netlify gives you a live URL immediately.

**Option B — GitHub Pages:**
1. Create a new GitHub repository.
2. Upload `index.html` to it (must be named exactly `index.html` at the root, or in a `/docs` folder).
3. Go to the repo's **Settings > Pages**, set the source to your branch/root, and save.
4. GitHub gives you a URL like `https://yourname.github.io/repo-name/`.

## 6. Test it before sending it to students

1. Open the live URL, click **Election admin**, sign in with the account from step 3.
2. Add a position and a couple of candidates.
3. Go to **Voter roster** and add yourself as a test voter (or paste a small CSV).
4. Go back, click **Vote now**, and cast a test vote with your test ID.
5. Check **Live results** to confirm it counted.

## Notes and limits

- **Free tier is generous** for a school election — Firebase's free (Spark) plan covers far more reads/writes than a single-school vote will use.
- **Roster passwords** are hashed before being stored, not kept in plain text — but this is still a lightweight system, not bank-grade security. Fine for a low-stakes student election.
- **Photos**: candidate photos are compressed and embedded directly in the candidate data (no separate file storage), so keep photos to reasonably small headshots — a handful of them per position is no problem.
- **The admin account is the real gatekeeper now.** Anyone who has that email/password can edit the whole election. Don't share it casually; create a second Firebase user later if you need a second admin.
