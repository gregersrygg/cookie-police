# Cookie Police :cop:

**NB! This is still a proof of concept. It has not been used in production yet.**

Protects `document.cookie` for you!

It's almost impossible to have control over what cookies third parties store on your domain. Since Safari started blocking third party cookies, most third parties have moved to store *their* third party cookies on your domain. So the number of cookies and the total size of cookies have started to become a problem. Because cookies are sent on every request to your server, it can slow down the user experience. You might also get problems if your infrastructure can't handle more than X cookies or the requests get so big that your server automatically responds with *400 Bad Request*!

If you have a large site, it might be hard to track down where every cookie come from. They often have bad cookie names, which doesn't make the search any easier. Maybe the cookies are from an old third party that is not in use anymore, or is it from a rarely visited page?

By providing a white-list of allowed cookies, Cookie Police can throw an error or call a callback whenever someone tries to set an unknown cookie. This way you can track it down locally, or create a way to report breaches from real users.

Ironically, in the most privacy-concerned browser Safari, it is not possible to protect document.cookie (8.0.8 on OS X and iOS 9.0.2). WebKit nightly acts the same way as Safari, so it doesn't seem to get fixed anytime soon. But this project can still give value by warning developers (that rarely use Safari) when adding new third party scripts, or by reporting breaches to a server. So 100% browser coverage is not necessary.

## API

### cookiePolice(options)
Available options:
+ `whiteList`: Array of allowed cookie-names. A breach will happen if anyone tries to set a cookie not in the list.
+ `ignoreList`: Array of cookie-names to ignore. It will silently ignore them without throwing an Error.
+ `onViolation`: Function to handle a violation. Arguments: `errorMessage`, `cookieName`. Default handler throws an error.

## Examples

For all examples, you first have to require cookie-police:

    var cookiePolice = require('cookie-police');

**Report unknown cookies**

    var knownCookiesArr = ['our', 'known', 'cookies'];
    cookiePolice({ whiteList: knownCookiesArr, onViolation: function (msg, cookieName) {
        new Image(1,1).src = '//my.domain.com/report-cookie-violation/' + encodeURIComponent(msg);
    }});

    // or with TrackJS
    cookiePolice({ whiteList: knownCookiesArr, onViolation: function (msg, cookieName) {
        try {
            // create an error with stack trace
            throw new Error(msg);
        } catch (e) {
            if (window.trackJs) { trackJs.track(e); }
        }
    }});


**You have a list of allowed cookies**

    cookiePolice({ whiteList: ['myCookie', 'thirdPartyCookie'] });

    document.cookie = 'myCookie=test'; // stored
    document.cookie = 'unknownThirdParty=foo'; // Throws Error: Cookie "unknownThirdParty" is not in the whitelist. Blocked.

**You have some allowed cookies and some you would like to dissallow without throwing**

    cookiePolice({ whiteList: ['myCookie'], ignoreList: ['thirdPartyCookie'] });

    document.cookie = 'myCookie=test'; // stored
    document.cookie = 'thirdPartyCookie=foo'; // silently ignored
