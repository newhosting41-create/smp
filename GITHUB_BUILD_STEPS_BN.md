# GitHub থেকে APK build করার ধাপ — Windows beginner guide

এই guide-এ আপনার v4 Android project GitHub-এ তুলে **Actions workflow দিয়ে APK build ও download** করার ধাপ আছে। **ZIP ফাইলটি সরাসরি repository-তে upload করবেন না**—ZIP extract করে ভেতরের project files-গুলো upload/push করতে হবে, যাতে `.github/workflows/build.yml` repository root-এ থাকে।

## যা লাগবে

- GitHub account
- Windows PC
- GitHub Desktop
- [SmmPaySync v4 auto-sync project ZIP](/home/ubuntu/SmmPaySync_v4_auto_sync_fixed.zip)

> Financial SMS সম্পর্কিত app code বলে GitHub repository-টি **Private** রাখুন। Device Token/HMAC Secret কোনো code file বা GitHub commit-এ লিখবেন না; app-এর Settings-এ রাখবেন।

## 1. ZIP extract করুন

1. উপরের ZIP link-এ click করে download করুন।
2. Windows Downloads folder-এ ZIP-এর ওপর right-click করে **Extract All…** চাপুন।
3. Extracted folder-এর ভেতরে `settings.gradle.kts`, `build.gradle.kts`, `app` folder এবং `.github` folder আছে কি না দেখুন। Project root হলো যে folder-এর ভেতরে `settings.gradle.kts` আছে।
4. `.github` hidden দেখা না গেলে File Explorer-এ **View → Show → Hidden items** চালু করুন। `.github/workflows/build.yml` file থাকতে হবে।

## 2. GitHub Desktop-এ repository বানান

1. GitHub Desktop install/open করে আপনার GitHub account-এ sign in করুন।
2. Menu থেকে **File → Add Local Repository…** বেছে নিন, তারপর extract করা project root folder নির্বাচন করুন।
3. Folder-টি আগে Git repository না হলে Desktop-এ **create a repository here** বা **Create Repository** option আসবে। সেটি চাপুন। যদি আলাদা dialog আসে, Local Path-এ এই project root-টাই বেছে নিন। Existing files-এর জন্য “initialize with README” জরুরি নয়।
4. Repository name `SmmPaySync` দিন। Repository publish করার সময় **Keep this code private** নির্বাচন করুন।
5. GitHub Desktop-এর **Changes** tab-এ source files এবং `.github/workflows/build.yml` আছে কি না দেখুন। Workflow file না থাকলে Publish করবেন না—ভুল folder নির্বাচন হয়েছে।
6. Summary-তে `Add SmmPaySync Android project` লিখে **Commit to main** চাপুন।
7. উপরে **Publish repository** চাপুন, Private setting নিশ্চিত করে publish করুন।

## 3. Actions দিয়ে build করুন

1. Browser-এ আপনার GitHub repository খুলুন।
2. **Actions** tab-এ যান। GitHub workflow চালু করতে বললে **I understand my workflows, go ahead and enable them** চাপুন।
3. **Build APK** workflow বেছে নিন। প্রথম push-এ এটি নিজে থেকেই run হওয়ার কথা। Run না হলে workflow page-এ **Run workflow** button থাকলে সেটি চাপুন, branch `main` বেছে **Run workflow** দিন।
4. Run-এর পাশে হলুদ ঘড়ি থাকলে build চলছে; লাল cross হলে build error হয়েছে। সবুজ check হওয়া পর্যন্ত অপেক্ষা করুন।
5. সবুজ check হওয়া run-টি খুলুন। নিচের দিকে **Artifacts** অংশে `SmmPaySync-debug-apk` click করে download করুন।
6. Download ZIP-টি Windows-এ extract করুন. `.apk` file-টাই Android app installer.

## 4. ফোনে APK install করুন

1. APK ফোনে copy করুন বা ফোনে download করুন।
2. APK tap করলে Android “Install unknown apps” চাইতে পারে। যে app দিয়ে খুলেছেন (Chrome বা Files), সেটিকে ওইবার install permission দিন, তারপর ফিরে এসে APK install করুন।
3. Install শেষ হলে app খুলে `RECEIVE_SMS` এবং inbox-recovery চাইলে `READ_SMS` permission দিন। Settings-এ সঠিক HTTPS endpoint URL, Device Token ও HMAC Secret দিন; bKash switch এবং background setting on রাখুন।
4. Battery settings-এ app-কে background-এ চলতে দিন। App-কে Android Settings-এ গিয়ে **Force stop** করলে user আবার app না খোলা পর্যন্ত Android background execution থামাতে পারে।

## 5. আগের app update করতে গেলে সতর্কতা

Project-এর application ID `com.smmpaysync.app` একই। কিন্তু GitHub Actions-এর প্রতিটি hosted build-এ debug signing key একই থাকবে এমন নিশ্চয়তা নেই। পুরোনো GitHub-built APK-এর ওপর নতুনটি install করতে Android “App not installed” বা signature conflict বলতে পারে। **Pending SMS বা credentials থাকা ফোন থেকে পুরোনো app uninstall করবেন না**—uninstall করলে local database/queue মুছে যেতে পারে। একই signing key দিয়ে build/sign করতে হবে। এ জন্য স্থায়ী keystore নিরাপদে নিজের কাছে রেখে, GitHub Actions-এ GitHub Secrets হিসেবে configure করা যায়; keystore/secret repository-তে commit করবেন না।

## 6. Build ব্যর্থ হলে কী দেবেন

GitHub Actions-এ লাল cross-যুক্ত run খুলে **Build APK → Build debug APK** step-এর প্রথম লাল error line-গুলো copy/screenshot দিন। Token, HMAC Secret, SMS body বা personal number ঢেকে দেবেন। Workflow file `.github/workflows/build.yml` project-এ আছে এবং `main`/`master`-এ push হলেই build শুরু করার জন্য তৈরি।

## এই project ZIP নিয়ে নোট

ZIP-এর project-এ বর্তমানে `SyncWorker`/`SyncEngine` class-সহ fixed v4 source আছে। আলাদা WorkManager guide-এ উদাহরণ হিসেবে দেখানো `PaymentSyncWorker` নতুন class-গুলো এই ZIP-এ আবার paste করবেন না—দুই implementation মিশলে duplicate/compile error হতে পারে। প্রথমে ZIP-এর project যেমন আছে তেমনই GitHub-এ build করুন।
