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

## 6. Features

- **Multiple tanks** — stationary tanks and fuel trailers
- **Log withdrawals and refills** — with live before/after level preview
- **Vehicle/equipment linking** — associate fuel with a specific piece of equipment
- **Real-time sync** — all users see live updates via Firestore listeners
- **Multi-user** — Firebase Auth with email/password; all authenticated users share the same data
- **Transaction history** — filterable by tank, grouped by date
- **Low fuel warnings** — tanks below 20% are flagged red
