# Live polling setup (Google Meet or Zoom)

This adds a separate **poll.html** page to the existing presentation. The same participant link works throughout the meeting. The presenter can launch one of seven prepared questions at a time and show live percentages.

**Not live until the steps below are completed.** GitHub Pages cannot store shared votes on its own.

## 1. Create a free Firebase project
1. Sign in at https://console.firebase.google.com/ and create a project. You do not need Google Analytics for this.
2. Add a **Web app** under Project settings. Copy the web app configuration.
3. Build → **Realtime Database** → Create database. Note its exact `databaseURL`. Choose a location and initially set **locked mode**.
4. Build → **Authentication** → Sign-in method → enable **Anonymous** and **Email/Password**.
5. Under Authentication → Users, add a presenter user with an email and a strong password. Copy that user's **UID** from the Users table.
6. In Authentication settings → Authorized domains, if needed add `sathyaaus555.github.io`.

## 2. Add the Firebase web configuration
Edit `poll-config.js` in this repository. Replace `REPLACE_ME` with the Firebase Web app fields, including the actual Realtime Database URL. These *web configuration fields* are intended for browser use; **do not add a service-account JSON or password** to GitHub.

## 3. Set database security rules
In Realtime Database → Rules, paste the following rules and replace `YOUR_PRESENTER_UID` with the presenter user's UID from step 1, then Publish.

```json
{
  "rules": {
    ".read": false,
    ".write": false,
    "session": {
      "current": {
        ".read": "auth != null",
        ".write": "auth != null && auth.uid === 'YOUR_PRESENTER_UID'"
      }
    },
    "votes": {
      "$pollId": {
        ".read": "auth != null",
        "$userId": {
          ".write": "auth != null && auth.uid === $userId && !data.exists() && root.child('session/current/id').val() === $pollId && root.child('session/current/status').val() === 'open'",
          ".validate": "newData.isString() && newData.val().matches(/^[0-3]$/)"
        }
      }
    }
  }
}
```

**Notes:** These rules block presenter controls for everyone but the UID you specify, and allow each anonymous Firebase account to vote once per poll. Participants may still vote again by clearing browser storage or changing browser/device; this is a feedback poll, not a high-stakes vote. Because participants need to read the vote counts to display results, anonymous vote identifiers and answer choices are database-readable to other authenticated visitors (though no names or emails are collected in vote records). Do not put personal or student information in the polls.

## 4. Test before training
1. Wait for GitHub Pages to deploy your changes.
2. Open `https://sathyaaus555.github.io/ModernEvaluationandAssessment/poll.html?presenter=1` in your browser and sign in with your presenter email/password.
3. Open `https://sathyaaus555.github.io/ModernEvaluationandAssessment/poll.html` in a different browser/private window or second device.
4. Click **Launch this poll** for Topic 01 on the presenter page.
5. Choose an option and **Submit vote** from the participant page.
6. Confirm the vote count and percentages update in your presenter page. Click **Show results to participants** if you want the audience to see results.
7. Click **End voting**, then launch Topic 02. Keep both windows open.

## 5. During the meeting
Send this **same link once**, in Google Meet or Zoom chat:

https://sathyaaus555.github.io/ModernEvaluationandAssessment/poll.html

Say: “Please open this poll page and keep it available during our session. After each topic I'll open the next poll. Just select one option and click Submit vote. You'll use the same link for the whole session.”

Keep the presenter link private. Only the configured presenter account is permitted to launch polls. Avoid screen-sharing the presenter login or password.

## Troubleshooting
- **Not connected:** `poll-config.js` hasn't been filled with the real web app config.
- **Permission denied:** Recheck database rules, UID, authentication providers, and the Realtime Database URL.
- **Poll not appearing:** Reload the participant page; ensure the user is signed in anonymously, then confirm the presenter launched a question.
- **Polls don't update:** Check browser developer console for network errors or Firebase authorization problems.
- **Repeated practice votes:** Old votes are retained; each test browser account votes once per topic. For a fresh training, clear test votes under `votes` in Firebase Console *only after you have confirmed the exact test data to remove*. Do not delete the database.
