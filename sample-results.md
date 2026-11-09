# Sample results

All clients are fictional. Drafts are shown as Claude wrote them; nothing was sent. `[SETUP_GUIDE_LINK]` is a placeholder the CSM fills in.

## CSM alert (internal, sees the evidence)

Scenario C, day 3. The competitor was added only inside Google Tag Manager, so the evidence points to the GTM container, not the page HTML.

```
Level: high - Tidewell Kitchen
A script from OneSignal now loads on the site, seen on 2 scans in a row.
Evidence:
- OneSignal (GTM container) | file: gtm.js?id=GTM-TDW4K2P | fragment: <script src="https://cdn.onesignal.com/sdks/web/v16/OneSignalSDK.page.js">
- ExamplePush (page script) | file: homepage HTML | fragment: <script src="https://cdn.examplepush.example/sdk/v2/examplepush.js">
- ExamplePush (service worker) | file: /examplepush-sw.js | fragment: importScripts('https://cdn.examplepush.example/sdk/v2/sw.js');
Next step: Review the evidence and the client email draft before anything is sent.
```

Scenario E, day 3. Our script is gone and a competitor's script and service worker are in its place.

```
Level: urgent - Bluestone Bikes
Our script is no longer loaded on the homepage or through the tag container, and a Pushwoosh script now loads on the site. Both changes were seen on 2 scans in a row.
Evidence:
- file: homepage HTML | fragment: <script src="https://cdn.pushwoosh.com/webpush/v3/pushwoosh-web-notifications.js">
- file: /pushwoosh-service-worker.js | fragment: importScripts('https://cdn.pushwoosh.com/webpush/v3/pushwoosh-service-worker.js' + self.location.search);
Next step: Review the evidence and the client email draft before anything is sent.
```

## Evidence of absence (added later)

When our script is missing, the stored evidence now also says where the scanner looked. Scenario D, day 4, as saved by the scanner (no AI involved):

```
ExamplePush (ours, fictional) | service worker | /examplepush-sw.js | importScripts('https://cdn.examplepush.example/sdk/v2/sw.js');
ExamplePush (ours, fictional) | not found      | homepage HTML      | no runnable script containing cdn.examplepush.example/sdk/
```

The first line explains why a quick look at the server might mislead: the old service worker file is still there. The second line is the actual finding.

## Client email, urgent alert: before and after

### Run 1 (passed the code checks, failed my reading)

> **Subject:** Notifications not showing for your visitors
>
> Hi Bluestone Bikes team,
>
> Thank you for reaching out about web push notifications that have stopped showing for some of your visitors. We are sorry for the trouble and want to help you get them working again. ...

The client never contacted us. The email invents a report and apologises for a problem we have not confirmed with them.

### Run 2 (body fixed, subject still wrong)

> **Subject:** Notifications stopped showing for some visitors
>
> Hi Bluestone Bikes team,
>
> It looks like notifications may have stopped showing for some of your visitors. ...

The body now says "may have", but the subject still states the problem as a fact.

### Run 3 (final)

> **Subject:** Quick check on your notifications
>
> Hi Bluestone Bikes team,
>
> It looks like notifications may have stopped showing for some of your visitors. We want to help you get this back on track.
>
> Small changes on the site, such as a theme update, a new plugin or a new cookie banner, can sometimes affect how notifications work. Your web developer is best placed to check the installation. The setup guide at [SETUP_GUIDE_LINK] walks through each step they can review.
>
> Once you reply, we can check the settings on our side straight away.
>
> Has anything changed on your site recently, such as a theme update, a new plugin or a new cookie banner?
>
> Best regards,
> Customer Success team

No mention of the check, the competitor or "your code". The installation is named as the client's developer's task; our side is checking our own settings.

## Client email, competitor alert (value email)

Scenario B, day 4. The email does not mention the competitor at all. It gives one useful idea and a low-effort "reply yes".

> **Subject:** Back-in-stock alerts for your shoe shop
>
> Hi Harbor Lane Shoes team,
>
> Here is one idea for the coming weeks: back-in-stock alerts. When a popular shoe sells out, shoppers who wanted it can ask to hear when it returns. Many people leave a sold-out product and forget about it. A quick browser alert brings them back at the right moment, when the size they need is on the shelf again.
>
> Start with your best sellers and the sizes that sell fastest. Keep the message short, with the product name, the size, and one clear link to buy.
>
> I can send you a short checklist for setting this up if you would like one. Just reply "yes" and I will send it over.
>
> Which shoe line would you like to start with?
>
> Best regards,
> Customer Success team
