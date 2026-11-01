# Tok Salone setup guide (simple steps)

Tok Salone is now its own app. It has its own sign up and log in, and no Claude name anywhere.
It uses Google's free Firebase service to store accounts and posts. A computer makes this much easier than a phone.

## Part 1: Make your Firebase project (free)
1. Go to console.firebase.google.com and sign in with a Google account.
2. Click **Add project**, name it `jesnap`, and finish the steps (you can turn off Analytics).
3. In the left menu click **Build > Authentication > Get started**. Choose **Email/Password**, turn it on, and Save.
4. Click **Build > Firestore Database > Create database**. Pick a location near your users and start in **production mode**.
5. In Firestore, open the **Rules** tab. Delete what is there and paste the rules from the box below. Click **Publish**.
6. Click the gear icon > **Project settings**. Under "Your apps" click the **web icon `</>`**, name it `jesnap`, and Register.
7. Copy the four values it shows: `apiKey`, `authDomain`, `projectId`, `appId`.

## Part 2: Put your settings in the app
Open `index.html` in any text editor. Find the line that starts with `const firebaseConfig=`.
Replace each `PASTE_HERE` with your real value, keeping the quote marks. Save.

## Part 3: Put it on the internet (free)
1. Go to **app.netlify.com/drop** (Netlify) and make a free account.
2. Drag the whole `jesnap` folder onto the page.
3. You get a link right away. Open it, sign up, and post your first photo.
4. In Firebase **Authentication > Settings > Authorized domains**, add your new Netlify domain so login works.

## Part 4: Your own web address (about 10 to 15 dollars a year)
Buy a name like jesnap.com from a seller such as Namecheap or Cloudflare. In Netlify open **Domain settings > Add a domain** and follow the steps. Then add that domain in Firebase Authorized domains too.

## Part 5: Phone apps (later)
When the website works well, I can help you turn it into an Android and iPhone app. Google Play has a one-time fee of about 25 dollars. Apple charges about 99 dollars a year.

## Firestore rules (paste in Part 1, step 5). If you already pasted old rules, replace them all with these and click Publish.
```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /users/{uid} {
      allow read: if request.auth != null;
      allow write: if request.auth != null && request.auth.uid == uid;
    }
    match /posts/{id} {
      allow read: if request.auth != null;
      allow create: if request.auth != null && request.resource.data.by == request.auth.uid;
      allow update: if request.auth != null
        && request.resource.data.diff(resource.data).affectedKeys().hasOnly(['likes','comments']);
      allow delete: if request.auth != null && resource.data.by == request.auth.uid;
    }
    match /chats/{cid} {
      allow read: if request.auth != null && request.auth.uid in resource.data.members;
      allow create: if request.auth != null && request.auth.uid in request.resource.data.members;
      allow update: if request.auth != null && request.auth.uid in resource.data.members
        && request.resource.data.members == resource.data.members;
      match /msgs/{mid} {
        allow read: if request.auth != null
          && request.auth.uid in get(/databases/$(database)/documents/chats/$(cid)).data.members;
        allow create: if request.auth != null && request.resource.data.by == request.auth.uid
          && request.auth.uid in get(/databases/$(database)/documents/chats/$(cid)).data.members;
      }
    }
  }
}
```

## Things to know before many people join
- Chats are saved in the database and are NOT end-to-end encrypted like WhatsApp. The app owner could read them. Be honest about this in your Privacy Policy.
- Not built yet: group chats, voice and video calls, read ticks, photo messages and phone notifications.
- Photos are stored as small compressed pictures inside the database. That is fine to start. When you have many users, we should move photos to proper file storage (this needs a paid Firebase plan).
- You need a Privacy Policy and Terms page before the app stores will accept it, and a way to report and block bad users. I can write all of these.
- Check that the name Tok Salone is free before you launch.
