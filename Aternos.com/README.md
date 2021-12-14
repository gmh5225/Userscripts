# A crash course into bypassing the Anti-Adblock

<details open><summary>Open/Close</summary>	
<p>
	
*(This section was last updated 12.12.2021, and was made to help you build your own scripts by understanding the structure better.)*

* [Basic structure](#basic-structure)
* [Bypassing the Anti-Adblock](#bypassing-the-anti-adblock)
	* [A look inside the Base64 encoded JavaScript file](#a-look-inside-the-base64-encoded-javascript-file)
* [Enabling all the buttons](https://github.com/Hakorr/Userscripts/tree/main/Aternos.com/AntiAntiAdblock#enabling-all-the-buttons)
* [Finding the obfuscated element's name and removing it](#finding-the-obfuscated-elements-name-and-removing-it)
* [Result](#result)


### Basic structure
	
<details open><summary>Open/Close</summary>	
<p>
	
![](https://github.com/Hakorr/Userscripts/blob/main/Aternos.com/Images/network.jpg)

Above us, you can see the files Aternos' server page loads, and their order. Most of the files do not contain anything harmful, but the [Base64 encoded JavaScript file](https://github.com/Hakorr/Userscripts/blob/main/Aternos.com/Other/Anti-Adblock-Obfuscated.js) is what starts the Anti-Adblock message. It also deletes the content page's elements, disables all the buttons and so fourth. It also enables everything afterwards.

So, let's take a look at the site's HTML structure, shall we? Here's a screenshot of the HTML, ***when the user has yet to clear the Anti-Adblock message.*** Feel free to take a closer look by opening the image on a new tab.

<img src="https://github.com/Hakorr/Userscripts/blob/main/Aternos.com/Images/structure2.jpg" alt="pageContentScreenshot" width="500"/>

The element inside the blue box is Aternos' page content element, which is basically the area below.

<img src="https://github.com/Hakorr/Userscripts/blob/main/Aternos.com/Images/aternosPageContent.jpg" alt="pageContentScreenshot" width="500"/>

The red area is where the page "hides" those elements you see on the above screenshot, inside the blue box. Once the user presses the "Continue with adblock" button and waits 3 seconds, those elements will be copied into the blue box, and then the red box removed. More about this later...

We can see the loaded files correspond with their order in the HTML (duh).  You can spot the [Base64 encoded JavaScript file](https://github.com/Hakorr/Userscripts/blob/main/Aternos.com/Other/Anti-Adblock-Obfuscated.js) we talked about earlier on the bottom. It's the core of the Anti-Adblock.

<img src="https://github.com/Hakorr/Userscripts/blob/main/Aternos.com/Images/structure.jpg" alt="pageContentScreenshot" width="500"/>

You can also see a similar Base64 source attribute on the top, on class body's element. That's a fake one, created by the Aternos' developers in response to my script. *(I once had a function that deleted every element with a Base64 encoded Javascript. Their change made my userscript delete the body element, which rendered the script useless for a while.)*

Again, this is the reason you wouldn't make your AntiAntiAdblock script public, the developers do stuff like this to prevent it functioning properly. Really annoying, but eh, I'll give them some respect for being so active.

---

</p>
</details>

### Bypassing the Anti-Adblock
	
<details open><summary>Open/Close</summary>	
<p>

<center><img src="https://github.com/Hakorr/Userscripts/blob/main/Aternos.com/Images/step1.jpg"></center>

Anyway, where were we... Ah yes, the [Base64 encoded file](https://github.com/Hakorr/Userscripts/blob/main/Aternos.com/Other/Anti-Adblock-Obfuscated.js) at the bottom. Currently at the time of writing this, **to bypass the Anti-Adblock, you need to stop that script from running.** I've done this by capturing all script execute(s), and then blocking all that request a [Base64 encoded file](https://github.com/Hakorr/Userscripts/blob/main/Aternos.com/Other/Anti-Adblock-Obfuscated.js).

Here's a code snippet.

```js
/* onbeforescriptexecute - https://github.com/Ray-Adams/beforeScriptExecute-Polyfill */
(() => {
    'use strict';

    if ('onbeforescriptexecute' in document) return;

    const BseEvent = new Event(beforeScriptExecute, {
        bubbles: true,
        cancelable: true
    });

    const observerCallback = (mutationsList) => {
        for (let mutationRecord of mutationsList) {
            for (let node of mutationRecord.addedNodes) {
                if (node.tagName !== 'SCRIPT') continue;

                // Adds functionality to document.onbeforescriptexecute
                if (typeof document.beforeScriptExecute === 'function') {
                    document.addEventListener(
                        beforeScriptExecute,
                        document.beforeScriptExecute,
                        { once: true }
                    );
                };

                // Returns false if preventDefault() was called
                if (!node.dispatchEvent(BseEvent)) {
                    node.remove();
                };
            };
        };
    };

    const scriptObserver = new MutationObserver(observerCallback);
    scriptObserver.observe(document, { childList: true, subtree: true });
})();

//A new web request initiated
document.beforeScriptExecute = (e) => {
    //If it requests a selected file
    if (e.target.src.includes('data:text/javascript;base64,')) {
        //Block it
        e.preventDefault();
    }
}
```

If the loaded script's source has the `data:text/javascript;base64` tag, it'll block it. If you're going to use the code in your own script, do remember to change the ```beforeScriptExecute``` event's name, otherwise the Aternos' developers might have made that variable null, like they did to one of my older variables. Screenshot of that below.

![](https://github.com/Hakorr/Userscripts/blob/main/Aternos.com/Images/blockedVariable.jpg)

Great, so we've stopped the [Anti-Adblock's JS file](https://github.com/Hakorr/Userscripts/blob/main/Aternos.com/Other/Anti-Adblock-Obfuscated.js) from loading, we should be good now, right? Not quite, the [Base64 encoded JavaScript file](https://github.com/Hakorr/Userscripts/blob/main/Aternos.com/Other/Anti-Adblock-Obfuscated.js) has important pieces of code that return everything back to normal. No buttons work without it. It also hides the fullscreen red Anti-Adblock element. We'll need to copy some code from the script to our userscript.

[You can find more methods to block the Anti-Adblock here.](https://github.com/Hakorr/Userscripts/blob/main/Aternos.com/Methods/BypassAntiAdblock.md)
	
---
	
</p>
</details>
	
### A look inside the Base64 encoded JavaScript file

<details open><summary>Open/Close</summary>	
<p>

Here's a code snippet from the Base64 encoded JavaScript. This is the "Continue with adblocker anyway" button's click function. Do remember that the whole script and all the class/id names would normally be obfuscated, like they are in the bottom of the HTML.

```js
//"Continue with adblocker anyway" button clicked - I've edited the variable, normally this would be randomized, such as ".FEKDMdmoAZVCNVEMXHADnandACCXZvmIIcxni"
$(".ContinueAnywayButton").click(function () {
    //"btn" variable is the "Continue with adblocker anyway" button element
    var btn = $(this);

    //If the button has already been pressed (And is counting down)
    if (btn.hasClass("btn-working")) {
        //Don't continue
        return false;
    }

    //Add a class to detect if the button has already been pressed
    btn.addClass("btn-working");

    //How many seconds to wait
    var secondsToWait = 3;

    //Append the number to a countdown element next to the button (It'll go 3, 2, 1...)
    $(".counterElem").html(secondsToWait);

    //Take the current UNIX time (Countdown start UNIX time)
    //UNIX is probably used to bypass any time modification scripts
    let t = Date.now();

    //Every 200ms interval
    var twohundredMsInterval = setInterval(function () {
        //Calculate the remaining time
        secondsToWait = Math.max(3 - Math.floor((Date.now() - t) / 1e3), 0);

        $(".counterElem").html(secondsToWait);

        //No more time to be waited (3 seconds elapsed)
        if (secondsToWait === 0) {
            //Remove the 200ms interval
            clearInterval(twohundredMsInterval);

            //Remove the "Continue with adblocker anyway" button
            btn.removeClass("btn-working");

            //Copy the page content's elements from a hidden element to the visible real element
            $(hiddenPageContentPlaceholderElement).children().appendTo($(".page-content"));

            //Remove the hidden element, because we just copied the stuff from it, it's not needed anymore.
            $(hiddenPageContentPlaceholderElement).remove();

            //Set the body and header class visible
            $(".body, .header").each(function () {
                this.style.setProperty("display", "");
                this.style.setProperty("height", "");
            });

            //Hide the fullscreen red Anti-Adblock element
            $("#mainAntiBlockElem")[0].style.display = "none";

            //Remove the countdown element
            $(".counterElem").html("");

            //Make the start button functional
            $("#start").each(function () {
                this._ready = true;
            });
        }
  }, 200);
});
```

Most of that code is not helpful to us though, as the class/id names are obfuscated. We'll get back to this.
	
---
	
</p>
</details>
	
### Enabling all the buttons
	
<details open><summary>Open/Close</summary>	
<p>
	
<center><img src="https://github.com/Hakorr/Userscripts/blob/main/Aternos.com/Images/step2.jpg"></center>
	
On the bottom of the [Base64 encoded JavaScript file](https://github.com/Hakorr/Userscripts/blob/main/Aternos.com/Other/Anti-Adblock-Obfuscated.js), is a function that enables many of the buttons. What I've done is add a couple functions from the last "Continue with adblocker anyway" button's click function to it. The result is a function that enables every button that was disabled. This can be used in your userscript.

```js
$(document).ready(function () {
    //Added from the "Continue with adblocker anyway" button's function
    $(".body, .header").each(function () {
        this.style.setProperty("display", "");
        this.style.setProperty("height", "");
    });
    //Added from the "Continue with adblocker anyway" button's function
    $("#start").each(function () {
        this._ready = true;
    });
    $("#userdropdown-toggle").click(function (e) {
        if ($(window).width() <= 1e3) {
        e.preventDefault();
        $(".userdropdown").slideToggle(100);
        }
    });
    $(".logout").click(function () {
        aget("/panel/ajax/account/logout.php", function () {
        location.href = "/go/";
        });
    });
    $(".navigation-toggle").click(function () {
        var cookieValue = 0;
        if ($(".navigation").hasClass("toggled")) {
        $(".navigation").removeClass("toggled");
        } else {
        $(".navigation").addClass("toggled");
        cookieValue = 1;
        }
        document.cookie = COOKIE_PREFIX + "_NAVIGATION_TOGGLED=" + cookieValue + ";path=/;max-age=31536000";
    });
    $(".friend-access-count-dropdown").click(function () {
        var dropdown = $(".friend-access-dropdown");
        if (dropdown.css("display") === "none") {
        dropdown.slideDown(100);
        $(".friend-access-count-dropdown i").fa("fas", "fa-caret-up");
        } else {
        dropdown.slideUp(100);
        $(".friend-access-count-dropdown i").fa("fas", "fa-caret-down");
        }
    });
    $(".js-friends-access").click(friendAccess);
    $(".js-friends-leave").click(friendLeave);
    $(".hamburger").click(function () {
        if ($(".navigation").css("left") == "-200px") {
        $(".navigation").animate({left: "0px"});
        } else {
        $(".navigation").animate({left: "-200px"});
        }
    });
});
```

Right, so now the buttons work and the fullscreen red Anti-Adblock screen is not visible, what else? Well, the fullscreen red Anti-Adblock screen is not visible, but it's still there, because the [Base64 encoded JavaScript file](https://github.com/Hakorr/Userscripts/blob/main/Aternos.com/Other/Anti-Adblock-Obfuscated.js) couldn't take it away (because we didn't load it). You need to have a function that finds and deletes this invisible element.

[You can find more methods to enable the buttons here.](https://github.com/Hakorr/Userscripts/blob/main/Aternos.com/Methods/FixButtons.md)

---
		
</p>
</details>	
		
### Finding the obfuscated element's name and removing it
				    
<details open><summary>Open/Close</summary>	
<p>
	
<center><img src="https://github.com/Hakorr/Userscripts/blob/main/Aternos.com/Images/step3.jpg"></center>

What I've done is decode the Base64 Javascript file, look for the variable name in a known location, then extract the element's id and remove it. This method is really vulnerable and will break by the slightest changes by the dev team.

There are many, many other ways to remove the red fullscreen element, you'll need to make your own or edit mine a bit. It's not hard.

Here's a code snippet of my method,
```js
function isBase64(r){ try { return btoa(atob(r)) == r} catch(r) { return!1}}

//A new web request initiated
document.beforeScriptExecute = (e) => {
    //If it requests a selected file
    if (e.target.src.includes('data:text/javascript;base64,')) {
        //Block it
        e.preventDefault();

        e.target.getAttribute("src").split("data:text/javascript;base64,").forEach(str => {
            //If the string is Base64
            if(isBase64(str)) {
                //Decode the Base64 string
                let beforeScriptExecute = atob(str);
                
                beforeScriptExecute.split(" ").forEach(x => {
                    //_0x4c04= is a variable name inside the Base64 encoded JavaScript file
                    if(x.includes("_0x4c04=")) {
                        x.split("'").forEach(x => {
                            if(x.includes("#")) {
                                if(x.length != 0 && $(x)) $(x)[0].innerHTML = "";
                            }
                        })
                    } 
                })
            }
        });
    }
}
```

At the time of writing, the method above works. They have just changed the `_0x4c04=` variable name to the new one. If you decode the Base64 JavaScript file and look for a variable similar to `var _0x4c04 = ["#mainAntiBlockElem", "css"];`, but just with a different name, you can replace the `_0x4c04=` with the new variable name.

[You can find other methods to find and remove the warning here.](https://github.com/Hakorr/Userscripts/blob/main/Aternos.com/Methods/FindWarning.md)

---
	
</p>
</details>
	
### Result
	
<details open><summary>Open/Close</summary>	
<p>
	
<center><img src="https://github.com/Hakorr/Userscripts/blob/main/Aternos.com/Images/done.jpg"></center>

The final result of the userscript is below,

```js
// ==UserScript==
// @name        script name
// @namespace   name
// @match       https://aternos.org/*
// @grant       none
// @version     1.0
// @author      name
// @description some description
// @require     https://ajax.googleapis.com/ajax/libs/jquery/2.2.0/jquery.js
// @run-at      document-start
// ==/UserScript==

//Enable all buttons
$(document).ready(function () {
    //Added from the "Continue with adblocker anyway" button's function
    $(".body, .header").each(function () {
        this.style.setProperty("display", "");
        this.style.setProperty("height", "");
    });
    //Added from the "Continue with adblocker anyway" button's function
    $("#start").each(function () {
        this._ready = true;
    });
    $("#userdropdown-toggle").click(function (e) {
        if ($(window).width() <= 1e3) {
        e.preventDefault();
        $(".userdropdown").slideToggle(100);
        }
    });
    $(".logout").click(function () {
        aget("/panel/ajax/account/logout.php", function () {
        location.href = "/go/";
        });
    });
    $(".navigation-toggle").click(function () {
        var cookieValue = 0;
        if ($(".navigation").hasClass("toggled")) {
        $(".navigation").removeClass("toggled");
        } else {
        $(".navigation").addClass("toggled");
        cookieValue = 1;
        }
        document.cookie = COOKIE_PREFIX + "_NAVIGATION_TOGGLED=" + cookieValue + ";path=/;max-age=31536000";
    });
    $(".friend-access-count-dropdown").click(function () {
        var dropdown = $(".friend-access-dropdown");
        if (dropdown.css("display") === "none") {
        dropdown.slideDown(100);
        $(".friend-access-count-dropdown i").fa("fas", "fa-caret-up");
        } else {
        dropdown.slideUp(100);
        $(".friend-access-count-dropdown i").fa("fas", "fa-caret-down");
        }
    });
    $(".js-friends-access").click(friendAccess);
    $(".js-friends-leave").click(friendLeave);
    $(".hamburger").click(function () {
        if ($(".navigation").css("left") == "-200px") {
        $(".navigation").animate({left: "0px"});
        } else {
        $(".navigation").animate({left: "-200px"});
        }
    });
});

//Stop the Base64 encoded JavaScript file from loading
(() => {
    'use strict';

    if ('onbeforescriptexecute' in document) return;

    const BseEvent = new Event(beforeScriptExecute, {
        bubbles: true,
        cancelable: true
    });

    const observerCallback = (mutationsList) => {
        for (let mutationRecord of mutationsList) {
            for (let node of mutationRecord.addedNodes) {
                if (node.tagName !== 'SCRIPT') continue;

                // Adds functionality to document.onbeforescriptexecute
                if (typeof document.beforeScriptExecute === 'function') {
                    document.addEventListener(
                        beforeScriptExecute,
                        document.beforeScriptExecute,
                        { once: true }
                    );
                };

                // Returns false if preventDefault() was called
                if (!node.dispatchEvent(BseEvent)) {
                    node.remove();
                };
            };
        };
    };

    const scriptObserver = new MutationObserver(observerCallback);
    scriptObserver.observe(document, { childList: true, subtree: true });
})();

function isBase64(r){ try { return btoa(atob(r)) == r} catch(r) { return!1}}

//A new web request initiated
document.beforeScriptExecute = (e) => {
    //If it requests a selected file
    if (e.target.src.includes('data:text/javascript;base64,')) {
        //Block it
        e.preventDefault();

        e.target.getAttribute("src").split("data:text/javascript;base64,").forEach(str => {
            //If the string is Base64
            if(isBase64(str)) {
                //Decode the Base64 string
                let beforeScriptExecute = atob(str);
                
                beforeScriptExecute.split(" ").forEach(x => {
                    //_0x4c04= is a variable name inside the Base64 encoded JavaScript file
                    if(x.includes("_0x4c04=")) {
                        x.split("'").forEach(x => {
                            if(x.includes("#")) {
                                if(x.length != 0 && $(x)) $(x)[0].innerHTML = "";
                            }
                        })
                    } 
                })
            }
        });
    }
}
```
	
---
	
</p>
</details>
	
It is probable that the methods above will get patched and the structure changed. Don't worry though, only small edits should get you far, as long as you don't share it. Try to use this as advice, not as a copypaste tutorial, the floor is yours!

So, TL;DR, all you have to do is,
1. [Select a method to Bypass the Anti-Adblock here](https://github.com/Hakorr/Userscripts/blob/main/Aternos.com/Methods/BypassAntiAdblock.md)
	* Modify the method a bit so it works!
2. [Select a method to find and remove the warning here](https://github.com/Hakorr/Userscripts/blob/main/Aternos.com/Methods/FindWarning.md)
	* Modify the method a bit so it works!
3. [Select a method to fix the buttons here](https://github.com/Hakorr/Userscripts/blob/main/Aternos.com/Methods/FixButtons.md)
	* Modify the method a bit so it works!
4. Put them all together as an userscript!
	
If you'd like to see the Base64 encoded JavaScript file decoded and slightly deobfuscated you can see it [here](https://github.com/Hakorr/Userscripts/blob/main/Aternos.com/Other/Anti-Adblock-Decoded.js).

</p>
</details>