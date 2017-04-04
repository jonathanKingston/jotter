---
layout: post
title: Containers website assignment
date: 2017-04-04 00:00
updated: 2017-04-04 00:00
published: true
categories: ["Security", "HTML", "JavaScript", "development", "containers", "mozilla", "planet mozilla"]
---

At Mozilla we have been working on a feature called [containers](https://testpilot.firefox.com/experiments/containers), which gives users the ability to separate their lives online to prevent being tracked.

![](/images/containers/intro.gif)

Since [launching containers in Nightly](https://blog.mozilla.org/tanvi/2016/06/16/contextual-identities-on-the-web/) and now in [Test Pilot](https://hacks.mozilla.org/2017/03/containers-come-to-test-pilot/), the most requested feature for containers was the ability to assign a website to a container. When a user assigns a website to a container the browser will then load that website in said container whenever the website is requested.

*To get the features mentioned in this article [download Containers in Test Pilot](https://testpilot.firefox.com/experiments/containers)*


# How it works

Launching **today** you will be able to assign websites to your containers using the following flow:

Open a bbc.com in a News container:

![Screenshot of opening bbc.com in a container using a context menu](/images/containers/open.png)

Assign the container to the link by picking "Always Open in This Container":

![Screenshot of a context menu assigning website to a container](/images/containers/assign.png)

When you click a link to bbc.com you will see this prompt asking you to confirm opening in the container you asked for:

![Screenshot of user prompt to confirm opening website](/images/containers/prompt.png)

When you want to remove the assignment use the same context menu:
![Screenshot of context menu to unassign website to a container](/images/containers/unassign.png)

Using the example we can see how a user will be able to prevent their browsing history being shared from advertisers, news and shopping sites whilst only adding minimal effort to their browsing flow.

# Hard decisions

For a long time we put off implementing this feature due to it's user interface complexity and we likely will still have to tweak it after we launch. *The feature itself wasn't very complex, with the code being only ~300 lines of code.*

I was initially hesitant to implement this feature due to the following three issues:

1. [The ability to CSRF yourself.](#the-ability-to-csrf-yourself)
2. [The inability to use more than one container for a website.](#the-inability-to-use-more-than-one-container-for-a-website)
3. [The inability to grasp why containers are isolated.](#the-inability-to-grasp-why-containers-are-isolated)


# 1. The ability to CSRF yourself

Websites often have paths or pages that supply capabilities that other pages can call.
These capabilities could be **paying someone** or **resetting a password**.
A [Cross Site Request Forgery](https://en.wikipedia.org/wiki/Cross-site_request_forgery) is an attack when a user unknowingly visits one of these paths without intending to.
CSRF exploits usually require that a user is already authenticated to a website and then unknowingly gets sent to one of these capability URLs that performs a privileged action on the authenticated users behalf.

For example:

1. User visits *example.bank*.
2. User logs in.
3. User then visits *evil.example.com*.
4. *evil.example.com* redirects the user to *example.bank/makepayment?username=evil&amount=1000* in an hidden frame that they can't see.
5. Assuming *example.bank* doesn't protect against CSRF the user will have the money taken from their account.


There are [lots](https://www.owasp.org/index.php/Cross-Site_Request_Forgery_(CSRF)_Prevention_Cheat_Sheet) of [ways](https://scotthelme.co.uk/csrf-is-dead/) a website can protect users from these forms of attacks, however in reality they still exist on the web today.

Containers actually fix this issue so long as you log into services within a new container, lets take another look at that example:

1. User opens a Finance container and logs into *example.bank*.
2. User visits *bad.com* in the default "No container" new tab.
3. *bad.com* redirects user to *example.bank/makepayment?username=evil&amount=1000* in an hidden frame that they can't see.
4. *example.bank* in "No container" isn't logged in and doesn't know about the user so refuses to pay evil user.

Now if we had assigned the bank to go into the Finance container we would be not protected either.

Furthermore single use [capability URLs can actually leak in the referrer header too](https://www.w3.org/TR/capability-urls/#risk-of-exposure).

# 2. The inability to use more than one container for a website

Users won't get the advantage of having multiple mail clients in different containers (Work and Personal mail)

# 3. The inability to grasp why containers are isolated

My other issue with container assignment  would be that a user wouldn't see the immediate advantage of using containers if they always get teleported into a certain container.

Users might suffer from their search engine and mail provider all being lumped into the same container.
This would mean the website could still track across all these properties.
One of the design goals of containers was to limit the amount of tracking users had whilst browsing the internet and helping users shatter their [filter bubble](https://en.wikipedia.org/wiki/Filter_bubble).

If a website can force a user into a container for another site they potentially can continue to track the user.

Lets take a look at an example of a staff member leaking his information across the web:

1. User visits *https://work.example.com* in a work container and assigns it to always open in work.
2. User opens a new tab and opens *https://example.clinic*.
3. User searches on *example.clinic* for something they don't want their work to know about.
4. *horrible.example.marketing* sends a cookie to the browser with a unique identifier **"hey-we-heard-you-had-medical-issues"**.
5. *https://example.clinic* forces the user to load *https://work.example.com/?tracking-id=**"hey-we-heard-you-had-medical-issues"***.
6. Browser redirects work.example.com "No container" into a work container with the same path.
7. *horrible.example.marketing* script is also loaded on *work.example.com* domain which resends a cookie based upon the GET tracking-id parameter.
  - *horrible.example.marketing* are now able to link together the two sites across a container.
8. *work.example.com* decides to not promote staff that may push up insurance premiums.

Whenever you assign an origin into a container you risk leaking your browsing history like this unsuspecting person did. This might seem extreme but there is however [evidence to suggest medical information often leaks from one website to another](https://www.eff.org/deeplinks/2015/01/healthcare.gov-sends-personal-data).

## How we solved these issues

1. We decided to warn users with a prompt which can be dismissed with a checkbox.
- I also prevented form submission across containers unless they are using a GET method.
  - GET method forms shouldn't have capabilities like withdrawing money.
  - This will cause some breakage however it should prevent the worst of these attacks.

2. Issue two is currently left unresolved due to wanting the users to have the ability to hide the warning.

3. After checking out many URLs we decided that websites still often split their services enough by subdomain that we could assign containers based upon the [Same Origin Policy](https://en.wikipedia.org/wiki/Same-origin_policy).
  - This means *mail.example.com* and *www.example.com* would be treated differently.
  - Assigning sites based upon the Same Origin Policy also gave us isolation against HTTP based attacks. Unless users had explicitly assigned the insecure version too, however we had to decide if users would want this behaviour...

### Deciding on HTTP website assignment

We had a hard balance between separating HTTP vs HTTPS assignment of a website.

#### Advantages

- This would prevent ***http**://example.bank/makepayment?username=evil&amount=1000* redirecting to ***https**://example.bank/makepayment?username=evil&amount=1000*.
  - Which would harm some websites that don't use a redirect however we now provide [insecure password warnings for these](https://developer.mozilla.org/en-US/docs/Web/Security/Insecure_passwords) so users should be less tempted to login to the HTTP version of a website now.
- Further down rating of HTTP traffic.


#### Disadvantages

- Users might click a link to a HTTP version of the website expecting not to have their history leak into the container they are currently in.
- Users are unsure why they have to assign both HTTP and HTTPS, this wouldn't be easy to explain in UI.
- Display of the protocol for both *https://example.com* and *http://example.com* would add further UI barriers.

#### Decision

Despite the risk of HTTP over insecure networks we decided that there was enough websites this would break right now and the usability disadvantages were pretty bad. The containers project as a whole has been a hard balance of usability and stricter security settings. There are already places where containers are imperfect for preventing tracking such as:

- [OCSP](https://en.wikipedia.org/wiki/Online_Certificate_Status_Protocol)
- [HSTS flags](https://en.wikipedia.org/wiki/HTTP_Strict_Transport_Security) see [HSTS supercookies](http://www.radicalresearch.co.uk/lab/hstssupercookies) for tracking technique.
- [Security exceptions for invalid certs](https://bugzilla.mozilla.org/show_bug.cgi?id=1249348).

Containers isn't considered the most robust security and privacy measure we have within the browser, it's a compromise of usability and robust security. At risk users should consider the patches Mozilla is working on as part of the [Tor uplift project](https://wiki.mozilla.org/Security/Tor_Uplift) namely "first party isolation" and the anti fingerprinting techniques Tor browser uses.

# Outstanding questions

One of the advantages of us running the test pilot experiment is it allows us to measure the usage of containers to gauge how people are using it, we often struggle to do wild experiments within Firefox like this because we can't gather as much information as we likely would want.

- Do users end up with too many of the same containers?
- Should we remove the ctrl/cmd + click links from being in the same container?
- With the ability to assign websites to be in a container, opening links in a new tab likely should be in "No container" to reduce the leakage of authenticated internet back into third party sites.
- Should we use instead/as well use `rel="noreferrer"`, `rel="noopener"`, `rel="nofollow"` links be treated as an indicator for a "No container"?
  - When a website indicates it isn't a strong relationship to each other perhaps we should consider it to be a different container.

# Feedback and bugs welcome

Feedback is always welcome at [containers@mozilla.com](mailto:containers@mozilla.com) where all the other developers involved in the experiment and the underlying platform changes will be notified.

- [Bugs via our GitHub repo are always welcome too](https://github.com/mozilla/testpilot-containers/issues/new)
- [Bugs about the platform feature which is in nightly](https://bugzilla.mozilla.org/enter_bug.cgi?assigned_to=nobody%40mozilla.org&blocked=ContextualIdentity&bug_file_loc=http%3A%2F%2F&bug_ignored=0&bug_severity=normal&bug_status=NEW&cc=tanvi%40mozilla.com&cf_blocking_b2g=---&cf_blocking_fennec=---&cf_feature_b2g=---&cf_fx_iteration=---&cf_fx_points=---&cf_status_b2g_2_0=---&cf_status_b2g_2_0m=---&cf_status_b2g_2_1=---&cf_status_b2g_2_1_s=---&cf_status_b2g_2_2=---&cf_status_b2g_2_2r=---&cf_status_b2g_2_5=---&cf_status_b2g_2_6=---&cf_status_b2g_master=---&cf_status_firefox47=---&cf_status_firefox48=---&cf_status_firefox49=---&cf_status_firefox50=---&cf_status_firefox_esr38=---&cf_status_firefox_esr45=---&cf_status_thunderbird_esr38=---&cf_status_thunderbird_esr45=---&cf_tracking_b2g=---&cf_tracking_e10s=---&cf_tracking_firefox47=---&cf_tracking_firefox48=---&cf_tracking_firefox49=---&cf_tracking_firefox50=---&cf_tracking_firefox_esr38=---&cf_tracking_firefox_esr45=---&cf_tracking_firefox_relnote=---&cf_tracking_relnote_b2g=---&cf_tracking_thunderbird_esr38=---&cf_tracking_thunderbird_esr45=---&component=DOM%3A%20Security&contenttypemethod=autodetect&contenttypeselection=text%2Fplain&defined_groups=1&flag_type-203=X&flag_type-37=X&flag_type-4=X&flag_type-41=X&flag_type-5=X&flag_type-607=X&flag_type-720=X&flag_type-721=X&flag_type-737=X&flag_type-781=X&flag_type-787=X&flag_type-799=X&flag_type-800=X&flag_type-803=X&flag_type-835=X&flag_type-846=X&flag_type-855=X&flag_type-863=X&flag_type-864=X&flag_type-875=X&flag_type-889=X&flag_type-892=X&flag_type-901=X&flag_type-905=X&flag_type-908=X&form_name=enter_bug&maketemplate=Remember%20values%20as%20bookmarkable%20template&op_sys=Unspecified&priority=--&product=Core&rep_platform=Unspecified&status_whiteboard=%5BuserContextId%5D%5Bdomsecurity-backlog%5D&target_milestone=---&version=unspecified)

Feel free to reach out on twitter [@KingstonTime](https://twitter.com/kingstonTime)


I want to highlight, despite this being a feature I worked on many others have been involved with the experiment.

The team responsible for getting the Test pilot live:

- [Andrea Marchesini](https://github.com/bakulf)
- [John Gruen](https://twitter.com/johnmgruen) and the [Test Pilot team](https://twitter.com/FxTestPilot)
- [Kamil Jozwiak](https://twitter.com/KamilJozwiak)
- [Luke Crouch](https://twitter.com/groovecoder)
- [Tanvi Vyas](https://twitter.com/TanviHacks)

Existing platform team responsible for working on getting [containers](https://wiki.mozilla.org/Security/Contextual_Identity_Project/Containers) and origin attributes into Nightly.

- [Andrea Marchesini](https://github.com/bakulf)
- [Bram Pitoyo](https://twitter.com/brampitoyo)
- [Christoph Kerschbaumer](http://christophkerschbaumer.com/)
- [Dave Huseby](https://twitter.com/dwhuseby)
- [Ethan Tsengo](https://twitter.com/AhsinTseng)
- Jonathan Hao
- [Kamil Jozwiak](https://twitter.com/KamilJozwiak)
- [Paul Theriault](https://twitter.com/creativemisuse)
- [Steven Englehardt](https://twitter.com/s_englehardt)
- [Tanvi Vyas](https://twitter.com/TanviHacks)
- Tim Huang
- Yoshi Huang

Past Research:

- [http://www.ieee-security.org/TC/W2SP/2013/papers/s1p2.pdf](http://www.ieee-security.org/TC/W2SP/2013/papers/s1p2.pdf)
- [https://web.archive.org/web/20161107220901/https://blog.mozilla.org/ladamski/2010/07/contextual-identity/](https://web.archive.org/web/20161107220901/https://blog.mozilla.org/ladamski/2010/07/contextual-identity/)

[Carolyn Jaeger](https://twitter.com/Cejaeger) who created the stunning gif in this article.

I'm sure many others have been missed too!
