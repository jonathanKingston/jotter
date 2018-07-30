title: "role=\"password\" is not wise"
categories:
  - Security
  - HTML
  - JavaScript
  - development
  - accessibility
data:
  updated: "2016-05-16 00:00"
---
**TL;DR:** [ARIA](https://www.w3.org/TR/wai-aria-1.1/) is a hack on real accessibility, reimplementing HTML in ARIA is silly, subverting password managers is evil, expecting the developers to start using role="password" likely wont happen, CSS replaced elements can't use pseudo elements and we should just fix that.

So initially [security people](https://lists.w3.org/Archives/Public/public-aria/2016May/0009.html) did not really raise issue with `role="password"` as it doesn't directly create security risks, however it is my view that it permits bad web authorship and promotes hurting users experience on the web.

## Use cases for role="password"

1. I am unable to style pseudo elements for [replaced elements](https://drafts.csswg.org/css2/conform.html#replaced-element) which [an input](https://html.spec.whatwg.org/multipage/rendering.html#replaced-elements)
2. I don't want password managers remembering my fields and autocomplete has been removed from me
3. I don't want other code impacting my input field

The only real discussion for and against the role I could find on the mailing list:

>> The reason for the new role is that authors are creating custom password  
>> input fields instead of the native HTML input type=“password”. By  
>> applying a role=“password” to the input element we can convey to the  
>> assistive technology that the author intended it to be a password field  
>> vs an ordinary text field. Since these are custom password fields it is  
>> up to the author and not the user agent to handle the obscuring of text.
>>
>> example: `<input type=“text” role=“password”>`
>
>This example conveys the primary problem I see - it doesn't include  
>anything that makes the input behave like a password.
>
>Issues as I understand them:
>
>1. Relying on web app authors to implement security seems like a terrible  
>idea, just because there are so many more of them to get it wrong.
>
>2. The primary audience for ARIA is screen reader users, and the primary  
>audience for screen readers are people who are blind. If their screen  
>reader tells them they are on a password field, they have no way to verify  
>beyond noticing it behave differently from a real one. Which means they  
>have to expose some of their password and notice different things happen.
>
>In the example above, they may be told that there was a password field,  
>and as they start to type the password it is likely that it would be  
>echoed back at them. Depending on their settings, perhaps as a complete  
>word - i.e. when they've put it all there.
>
>They may not be told, since the backward compatibility story is that the  
>input would seem to be a normal input. I think that is preferable, since  
>at least users can tell they are not in a "real" password field.
>
>3. There is currently no way for the role value to trigger common user  
>agent behaviour with passwords, such as blocking copy or storing things in  
>a password manager.
>
>4. Expecting the user to listen to characters when they start typing  
>passwords, as a way to determine whether they are visually exposing their  
>password, seems to be an anti-apttern from the point of view of user  
>experience, and inherently fragile from the perspective of protecting  
>security.
>
>A counter argument is that it is not clear what is a password input  
>anyway. There is a convention that characters entered are masked with "*"  
>but this is easy enough to mock up with a standard text input. So how much  
>are we increasing the attack surface?
>
>cheers
>
>Chaals
>
>> The browser would convey to the assistive technology that this is a  
>> password field vs. an ordinary text field. In this scenario the user  
>> should pay attention to the text rendered.
>>
>> Does your group believe this adds security risks? If so, what are those  
>> risks?
>>
>> Thank you,
>>
>> Rich Schwerdtfeger
>> ARIA Working Group Chair

## Expectation that authors do the right thing

So the sum total of why this is being added appears to be: web authors are behaving badly but those same people will do the right thing for assistive technologies.

The problem is that in marking up their document with `role="password"`:

- Web authors aid password managers by providing a role that can select the element to save the password (**use-case 2.**)
- Websites that decide to do wonky things will unlikely spend time doing anything new for assistive tech
- Assistive tech will take a long time to take advantage of this, promoting the right thing would happen faster

It all feels like closing the gate after the horse has bolted.

## Assistive tech will struggle to behave correct here

Conceptually assistive technologies are going to struggle to suggest that the user is safe from someone snooping on their passwords. It has been suggested that the technology needs to make a generic noise to mask the password being input into the field. However this doesn't inform the user if it is doing the same visually. So assistive technology could check the visual output to ensure the password is masked perhaps and warn the user, however this would likely be a fairly heavy weight implementation which will likely not be implemented consistently.

## ARIA wasn't supposed to be a vehicle to reimplement HTML

Web components have needed to define new behaviours in HTML and thus creating explanations to screen readers and other technologies makes sense. However lately it has become a way to reimplement the whole of HTML. This really is going to lead to web components doing lots of reinventing the wheel.

Password input fields have lots of technology assigned to them already which we are promoting is a good thing to reimplement:

- Form validation
- Password masking
- Autocomplete
- etc


Password fields today are already more accessible today because of the existing features, screen readers could choose to announce the requirements of a password (maxlength, minlength and pattern) when the user requests so.

Where possible real HTML should be used over ARIA.

## Password managers are good

Permitting web authors to circumvent password managers is a bad thing! We prevented [web authors using autocomplete badly](https://bugzilla.mozilla.org/show_bug.cgi?id=956906) in much the same manner. Adding new tools to let website owners circumvent password management seems to only validate this behaviour.

If we wanted to fix password field selection in the future it likely would require unmanageable heuristics for browsers to remember password fields.

Browsers should consider spending more time on making form population and password management much simpler such that users become the driving factor why they web authors shouldn't roll their own fields.

Harming users of the site over what authors want is against the [core design pattern 'Priority of Constituencies' of HTML](https://www.w3.org/TR/html-design-principles/#priority-of-constituencies).

> In case of conflict, consider users over authors over implementors over specifiers over theoretical purity. In other words costs or difficulties to the user should be given more weight than costs to authors; which in turn should be given more weight than costs to implementors; which should be given more weight than costs to authors of the spec itself, which should be given more weight than those proposing changes for theoretical reasons alone. Of course, it is preferred to make things better for multiple constituencies at once. 

Users lose having a well defined password field that behaves consistently and can be saved within a password manager. Websites subverting password managers should actively be called out either publicly or through the browser.

## Replaced elements won't allow ::before and ::after

**Use-case 1.**
Input is a replaced element which doesn't allow [pseudo elements such as ::before and ::after](http://codepen.io/anon/pen/LNvOeq) which some layouts want without creating wrappers. This should be made possible in CSS or have a new native element, preferentially the first. It's a widely desired feature and it doesn't seem a limitation on CSS itself.

**Update 2016/05/17:** This could be simulated with a custom element wrapping an input element reducing templates to not need a wrapping div. [As highlighted out by @simevidas](https://twitter.com/simevidas/status/732389037609209856)

## Users want to isolate their CSS

**Use-case 3.** - *"I would like CSS isolation on my input field"*

Use Shadow DOM in future.

## Existing HTML - Update: 2016/05/17

[@briankardell](https://twitter.com/briankardell) mentioned another use case that I had not covered (there are likely many).
I concentrated on uses that couldn't be replicated in HTML today. He mentioned the 'show password' use, which should be possible to simulate with an input element and JavaScript changing the type.
