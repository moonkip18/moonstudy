# Firebase Setup Guide for Study Tracker

## Step 1: Create a Firebase Project

1. Go to [Firebase Console](https://console.firebase.google.com)
2. Click **Create Project**
3. Enter project name (e.g., "moonstudy")
4. Disable Analytics (optional)
5. Click **Create Project**

## Step 2: Set Up Firestore Database

1. In Firebase Console, navigate to **Build** → **Firestore Database**
2. Click **Create Database**
3. Select **Start in test mode** (or production with rules below)
4. Choose region (e.g., `us-central1`)
5. Click **Create**

### Security Rules (Production)

Replace the default rules with:

```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /users/{userId} {
      // Users can only read/write their own data
      allow read, write: if request.auth.uid == userId;
    }
    
    match /leaderboard/{doc=**} {
      // Everyone can read leaderboard
      allow read: if true;
      // Authenticated users can write their own score
      allow write: if request.auth != null && 
                      get(/databases/$(database)/documents/users/$(request.auth.uid)).data.username == resource.data.username;
    }
  }
}
```

## Step 3: Enable Anonymous Authentication

1. In Firebase Console, go to **Build** → **Authentication**
2. Click **Get Started**
3. Select **Anonymous** provider
4. Click **Enable**
5. Click **Save**

## Step 4: Get Your Credentials

1. Go to **Project Settings** (gear icon)
2. Navigate to **Your apps** section
3. Click the Web app (or create one if needed)
4. Copy the following from the SDK snippet:
   - **API Key** (appears in config object as `apiKey`)
   - **Project ID** (from config object as `projectId`)

## Step 5: Configure the App

On first load of the Study Tracker:

1. Click **Get started**
2. Enter your name
3. Paste your **API Key** and **Project ID**
4. Click **Get started**

The app will now sync all study data to Firestore!

## Data Structure

Firestore automatically creates:

### `/users/{userId}`
```json
{
  "username": "Alex",
  "log": {
    "Math": 3600,
    "Physics": 1800
  },
  "subjects": [
    { "name": "Math", "color": "#C8FF00", "goal_h": 6 }
  ],
  "todayTotalSec": 5400,
  "updatedAt": "timestamp"
}
```

### `/leaderboard/{userId}`
```json
{
  "username": "Alex",
  "todayTotalSec": 5400,
  "weekTotalSec": 25200,
  "allTimeTotalSec": 518400,
  "updatedAt": "timestamp"
}
```

## Offline Support

Firestore offline persistence is automatically enabled. The app will:
- Cache all data locally
- Continue working without internet
- Auto-sync when connection returns

## Leaderboard Queries

The app uses efficient Firestore queries:

```javascript
db.collection('leaderboard')
  .orderBy('todayTotalSec', 'desc')
  .limit(10)
  .onSnapshot(snapshot => { /* render */ });
```

## Troubleshooting

**"Sync Error" shown?**
- Check API Key and Project ID are correct
- Ensure Firestore Database is created
- Verify Anonymous Authentication is enabled

**Data not syncing?**
- Check browser DevTools Console for errors
- Verify Firestore Security Rules allow writes
- Check network tab for failed requests

**Offline mode not working?**
- Ensure browser allows IndexedDB (check privacy settings)
- Clear browser cache/cookies and retry
