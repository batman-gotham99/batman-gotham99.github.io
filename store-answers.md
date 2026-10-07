# Thabat: store privacy answers

Answers for Google Play's Data safety form and the Play Console app content pages, and Apple's App Privacy label for later. They match the app as of version 0.2.0 (2026-10-07) and `site/privacy.html`. Re-check them whenever an SDK, a permission or an event changes.

Taxonomy checked against Google's help page "Provide information for Google Play's Data safety section" (answer 10787469) on 2026-10-07.

## Privacy policy URL (do this first)

Both consoles need a public URL for the privacy policy. `site/` is plain static HTML and is not published anywhere yet. Host it yourself:

- **GitHub Pages:** the repo is private, and Pages on a private repo needs a paid plan. Either make a small public repo (for example `thabat-site`) holding only the contents of `site/`, then Settings → Pages → Deploy from branch `main`, folder `/ (root)`; or use any static host (Cloudflare Pages, Netlify) pointed at the `site/` folder.
- The URL becomes `https://<user>.github.io/thabat-site/privacy.html` (or your own domain).
- Paste it in **Play Console** → Policy and programs → App content → Privacy policy, and in **App Store Connect** → App Privacy → Privacy Policy URL. The landing page (`index.html`) can serve as the marketing / support URL.
- The policy says PostHog and Sentry run in the **EU** and keep no IP address. Before you publish it, make sure that is true: PostHog EU cloud with "Discard client IP data" on; a Sentry organization created in the EU region (sentry.io → data storage location: EU) with "Prevent Storing of IP Addresses" on in the project's security settings. Set retention to match the policy (events up to 12 months, crash reports up to 90 days) or change the policy's numbers.

## Google Play: Data safety

### Overview questions

| Question | Answer |
| --- | --- |
| Does your app collect or share any of the required user data types? | **Yes** |
| Is all of the user data collected by your app encrypted in transit? | **Yes** (PostHog and Sentry are HTTPS only) |
| Which of the following methods of account creation does your app support? | **My app does not allow users to create an account** |
| Do you provide a way for users to request that their data is deleted? | **Yes**: by email to batman.gothann@gmail.com (privacy policy, "Deleting your data"). Erase everything and uninstalling remove all on-device data. |

The "delete account URL" requirement does not apply: there are no accounts.

### Data types

Declare exactly these three. Every other category (Location, Personal info, Financial info, Health and fitness, Messages, Photos and videos, Audio, Files and docs, Calendar, Contacts, Web browsing) is **not collected**.

| Category → data type | Collected | Shared | Processed ephemerally | Required or optional | Purposes |
| --- | --- | --- | --- | --- | --- |
| App activity → **App interactions** | Yes | No | No | **Optional** | **Analytics** |
| App info and performance → **Crash logs** | Yes | No | No | **Optional** | **Analytics**, **App functionality** |
| App info and performance → **Diagnostics** | Yes | No | No | **Optional** | **Analytics**, **App functionality** |
| Device or other IDs → **Device or other IDs** | Yes | No | No | **Optional** | **Analytics** |

Why each one:

- **App interactions**: Google's definition is "how a user interacts with the app ... sections they tap on". The twelve PostHog events (`apps/mobile/src/analytics/catalogue.ts`) are exactly that: a routine logged, a day closed, a notification tapped, a reminder switched. Not "Other actions" (that is for gameplay, likes and the like) and not "Other user-generated content" (no text the user writes is ever sent).
- **Crash logs**: "stack traces, or other information directly related to a crash". Sentry crash reports (`apps/mobile/src/crash`).
- **Diagnostics**: "any technical diagnostics". A Sentry report carries device context (model, system version, memory and similar) and breadcrumbs; PostHog adds the device model, OS version and screen size to each event. Not "Other app performance data", since Diagnostics already covers it.
- **Device or other IDs**: Google lists "Firebase installation ID" as an example, i.e. an app-scoped install id counts. Thabat's install id is a random UUID made on the phone (`analytics.device` in kv), sent with every event and as the Sentry user. It is not a hardware or advertising id and it rotates when sharing is turned off, but it is still an identifier relating to an app install, so declare it.

Answers that apply to all four:

- **Shared: No.** PostHog and Sentry process the data on our behalf as service providers, which Google exempts from "sharing".
- **Ephemeral: No.** Both services store events.
- **Optional: Yes.** Every user, on every device and in every region, can turn it off in Settings → Data → Share anonymous usage, and that stops both PostHog and Sentry. (It is on by default during the beta; "optional" in Google's sense means the user can opt out, which they can.)
- **Not used for:** Advertising or marketing, Personalization, Account management, Fraud prevention, Developer communications.

### What is deliberately not declared

- **Feedback (text, screenshots, the user's email address).** The app does not transmit feedback. It opens the user's own mail app with a draft, and the user sends it from there (`apps/mobile/src/feedback/send.ts`). Nothing leaves the device through Thabat's code, so it is not "collected" by the app. If a reviewer disagrees, the conservative alternative is: Personal info → Email address, App activity → Other user-generated content, Photos and videos → Photos, all Optional, purpose Developer communications and App functionality.
- **Export a copy.** A user-initiated file through the system share sheet; the developer never receives it.
- **Notifications.** Local only (expo-notifications, no push token, no server).
- **App updates (expo-updates).** A download request to Expo's update service carrying the runtime version and platform, no user data.

## Play Console: App content

| Page | Answer |
| --- | --- |
| **App access** | All functionality is available without special access. No login, no account, no restricted areas. |
| **Ads** | No, the app does not contain ads. |
| **Content rating** (IARC questionnaire) | Category: Utility, Productivity, Communication or Other. No violence, sexual content, profanity, drugs, gambling or crude humour. Users interact or exchange content: **No** (notes stay on the phone; feedback is a private email). Shares the user's location: **No**. Allows purchases: **No**. Expected result: Everyone / PEGI 3 / IARC 3+. The faith starting points (prayers, Quran reading) are routine names only; the questionnaire has no question they trigger. |
| **Target audience and content** | **18 and over** only. Not designed to appeal to children. (The privacy policy says it is not directed at under 13.) |
| **News app** | No |
| **COVID-19 contact tracing and status apps** | No (my app is not a publicly available COVID-19 contact tracing or status app) |
| **Data safety** | As above |
| **Government apps** | No, not developed by or on behalf of a government |
| **Financial features** | My app doesn't provide any financial features |
| **Health apps** | My app does not have any health features. Thabat tracks routines the user defines (reading, a walk, prayer); it reads no health or fitness sensors and offers no medical function. |
| **Advertising ID** | No, the app does not use an advertising ID. Check the merged manifest of the release build has no `com.google.android.gms.permission.AD_ID`; neither posthog-react-native nor @sentry/react-native should add it. |
| **Photo and video permissions** (only if Play shows it) | expo-image-picker attaches feedback screenshots through the system photo picker. If the merged manifest carries `READ_MEDIA_IMAGES`, Play asks for this declaration: answer that access is one-off (a screenshot attached to feedback) and the system photo picker covers it, or remove the permission. |

## Apple: App Privacy ("nutrition label"), for later

Data Used to Track You: **None**. Thabat does not track: no advertising id, no data broker, no linking with other companies' data.

Data Linked to You: **None**. There is no account, no name, no email address and no hardware id in what is sent.

Data Not Linked to You:

| Apple category → type | Purposes | Linked to identity | Used for tracking |
| --- | --- | --- | --- |
| Usage Data → **Product Interaction** | Analytics | No | No |
| Diagnostics → **Crash Data** | Analytics, App Functionality | No | No |
| Diagnostics → **Performance Data** | Analytics, App Functionality | No | No |
| Diagnostics → **Other Diagnostic Data** | Analytics, App Functionality | No | No |
| Identifiers → **Device ID** | Analytics | No | No |

Notes:

- **Device ID** is the conservative choice for the random install id. Apple's examples are advertising and device-level ids; ours is app-scoped and rotates, so some apps leave it out. Declaring it costs nothing and cannot cause a "label mismatch" rejection.
- **Performance Data**: Sentry's tracing is off (`tracesSampleRate: 0`), but crash events carry device state such as memory. If you prefer the narrower label, keep Crash Data and Other Diagnostic Data only.
- **Feedback** sent from the user's own Mail app is not collected by the app. If you later replace mailto with an endpoint (plan §19), add Contact Info → Email Address (if asked for), User Content → Customer Support and Photos, all optional disclosure candidates under Apple's rules.
- Apple asks for a Privacy Policy URL (required) in App Store Connect → App Privacy; use the same hosted `privacy.html`.
