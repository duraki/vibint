Request when adding new phone number (via MANIP#1):
---
GET /media/user_photo?dlid=0-03-05-55b192a60bbc40f059f33279f404b7644e3622ff7be3b346f5e8caa54482cccf&fltp=jpg&rqvr=1&sdcc=387&styp=95&udid=63dc88e6884f926b160d54cc25a403c511db74f0&vcpv=51&xuat=00000000165DD8FD&xuav=13.9.1.14-521f694 HTTP/1.1
Host: media.cdn.viber.com
Accept: */*
Cookie: language=en
User-Agent: Viber/13.9.1.14 CFNetwork/1121.2.2 Darwin/19.2.0
Accept-Language: en-us
Accept-Encoding: gzip, deflate
Connection: close


Response when adding new phone number (via MANIP#1):
---
{
  "Media": {
    "Expires": "2020-10-11T07:18:19.739Z",
    "Download": {
      "Url": "https://dl-media.viber.com/5/share/2/long/user/def/image/0x0/cccf/55b192a60bbc40f059f33279f404b7644e3622ff7be3b346f5e8caa54482cccf.jpg?Expires=1602400699&Signature=QKzhSq9XDFgHoZvBSOB4AItC0L241NY9PCYWb6tppAIG9iV9mKUsY3rqWqr~x8T0h53zjnse8yREOyB9YRnJ6bJzbifXVmK4MVc0xVBZqsKPxiNMDEuZGl-YqOaA4YA3rXv4QqJ3Aqopm3DWH6p3EtLdgWZmpHsKl6uuV6nbPOejEHDXcS~~6UAjVanH724PSGmYMLfbHzG0zP1msKx4ZTw3G5nBNvY1aGuIFvs5Jr0ihCQNk7i5kRnFL0RXnlSshqbfqIs3hRgZ5wcjcgUuxxFdbG1K-wmh22eZwkwpHhemoyxGc5-vqc2HT2FAbT2k3pR0K13gZXOKJSKVR20PFg__&Key-Pair-Id=APKAJ62UNSBCMEIPV4HA"
    }
  },
  "Trace": {
    "Call-Id": "05.0004.007a.73b98e01.00005a701dc3a082",
    "Call-Result": "Success_0",
    "Call-Ticks": 1222522
  }
}

Lets split request calls based on trial-and-error value check in sent request one by one. This way we can investigate deeper which of the request values are responsible for user data. As we can see, it's a GET request with number of params.

GET /media/user_photo?dlid=0-03-05-55b192a60bbc40f059f33279f404b7644e3622ff7be3b346f5e8caa54482cccf&fltp=jpg&rqvr=1&sdcc=387&styp=95&udid=63dc88e6884f926b160d54cc25a403c511db74f0&vcpv=51&xuat=00000000165DD8FD&xuav=13.9.1.14-521f694 HTTP/1.1

URL	dlid	0-03-05-55b192a60bbc40f059f33279f404b7644e3622ff7be3b346f5e8caa54482cccf
This is the parameter we are interested in reversing. The last part of the above dlid value is 64bit hash responsible for particular phone number we entered in MANIP#1 or MANIP#2. It identifies the phone number so no simple fuzzing can be exploited. I'm not sure yet what is 0-03-05.

URL	fltp	jpg
This is the file-type parameter responsible for image content-type response. Putting any other value but .jpg yields an error in the response.

URL	rqvr	1
This is the parameter that identifies the request version. I suspect this is related to HTTP RFC responsible for accepting request on server-side. Must be of value 1, always.

URL	sdcc	387
This is the parameter referencing country phone code. For Bosnia-Herzegovina, that is <+387>, therfore the 387 in value. Can be anything, will always return appropriate profile media/avatar. We can conclude that this parameter is not used when retrieving the media, rather, the dlid is fully used.

URL	styp	95
Phone number type. Not sure what it's used for. Can be anything.

URL	udid	63dc88e6884f926b160d54cc25a403c511db74f0
UDID of the caller. This is mobile-phone identifier, aka my iPhone I'm testing on. Can be anything. Might be used for limit bypass.

URL	vcpv	51
Not sure what it is. Can be anything.

URL	xuat	00000000165DD8FD
Perhaps (xu)ApplicationType - xuat (ie. Mobile, Desktop, iOS). Can be anything.

URL	xuav	13.9.1.14-521f694
Perhaps (xu)ApplicationVersion - xuav. Can be anything.





Now lets play with response data a little bit. Lets see what we got here, I'll comment things inline:

{
 "Media":{
  "Expires":"2020-10-11T07:18:19.739Z", // When does the request expires?
  "Download":{ // What to download
   "Url":"https://dl-media.viber.com/5/share/2/long/user/def/image/0x0/cccf/55b192a60bbc40f059f33279f404b7644e3622ff7be3b346f5e8caa54482cccf.jpg?Expires=1602400699&Signature=QKzhSq9XDFgHoZvBSOB4AItC0L241NY9PCYWb6tppAIG9iV9mKUsY3rqWqr~x8T0h53zjnse8yREOyB9YRnJ6bJzbifXVmK4MVc0xVBZqsKPxiNMDEuZGl-YqOaA4YA3rXv4QqJ3Aqopm3DWH6p3EtLdgWZmpHsKl6uuV6nbPOejEHDXcS~~6UAjVanH724PSGmYMLfbHzG0zP1msKx4ZTw3G5nBNvY1aGuIFvs5Jr0ihCQNk7i5kRnFL0RXnlSshqbfqIs3hRgZ5wcjcgUuxxFdbG1K-wmh22eZwkwpHhemoyxGc5-vqc2HT2FAbT2k3pR0K13gZXOKJSKVR20PFg__&Key-Pair-Id=APKAJ62UNSBCMEIPV4HA"
  }
 },
 "Trace":{ // Metadata
 	...
 }
}


As you can see, the Url parameter in Download JSON key is our image. The good thing is we don't need additional parameters, so we can actually get our picture at the location of:

https://dl-media.viber.com/5/share/2/long/user/def/image/0x0/cccf/55b192a60bbc40f059f33279f404b7644e3622ff7be3b346f5e8caa54482cccf.jpg

Also, the 64bit value in the URL is same as one in the request. This means we don't need to send double requests to get image for appropriate number; we need to enumerate possible number to hash and just inject the resulted hash in given URL. Thats good, but, how do we get the key?

Lets hold for a bit and observe the flow of adding new phone number via Viber:

	* User opens Viber and goes to Calls
	* User enters the phone number
	* User press "Add" button
	* Viber hashes the phone number
	* Viber initiates request
	* Viber gets response
	* The data is filled in Application View

Something is missing here, after all. Where is the First name / Last Name medatada? Well, thats a good question, and we are yet to find that out by reversing the Viber logic.

---

So with Objection, the first thing we can do is get the entire view hierarchy. That is, just get an array of all views displayed. You, of course, need a Frida server running on the device. I'm using Jailbroken iPhone 7 for this.

I ran the following script by attaching JavaScript file which prints out View Hierarchy to get a better picture of what I'm looking on via screen. 

==== START ====
root@kali:/# cat ui-hierarchy.js 
var get_all_views = function() {
    var root = ObjC.classes.UIWindow.keyWindow(); // The root of the view hierarchy
    var buff = [root];                            // The views left to traverse
    var visited = [];                             // The views we have already traversed

    while (buff.length > 0) {
        var node = buff.shift();

        // Make sure we don't traverse a view twice
        if (visited.indexOf(node) >= 0)
            continue;

        visited.push(node);

        // Iterate over all the view's subviews
        var children = node.subviews();

        for (var i=0; i<children.count(); i++) {
            var child = children['- objectAtIndex:'](i);

            if (visited.indexOf(child) == -1)
                buff.push(child);
        }
    }

		visited.forEach(printView);

    return visited;
}

function printView(item) {
	console.log(item);
}

get_all_views();
==== END ====


root@kali:/# frida -U -l ui-hierarchy.js -no-pause -p 14349
     ____
    / _  |   Frida 12.11.1 - A world-class dynamic instrumentation toolkit
   | (_| |
    > _  |   Commands:
   /_/ |_|       help      -> Displays the help system
   . . . .       object?   -> Display information about 'object'
   . . . .       exit/quit -> Exit
   . . . .
   . . . .   More info at https://www.frida.re/docs/home/
Attaching...                                                            
<Viber.MainWindow: 0x10a420f80; baseClass = UIWindow; frame = (0 0; 375 667); autoresize = W+H; tintColor = <UIDynamicProviderColor: 0x28091b1e0; provider = <__NSMallocBlock__: 0x2807f6430>>; gestureRecognizers = <NSArray: 0x2806e5cb0>; layer = <UIWindowLayer: 0x2808496c0>>
<UITransitionView: 0x10a428df0; frame = (0 0; 375 667); autoresize = W+H; layer = <CALayer: 0x28084b020>>
<UITransitionView: 0x10a4e2690; frame = (0 0; 375 667); autoresize = W+H; layer = <CALayer: 0x2809152e0>>
<UIDropShadowView: 0x10a429cf0; frame = (0 0; 375 667); clipsToBounds = YES; autoresize = W+H; layer = <CALayer: 0x28084b080>>
<UIView: 0x105e32d00; frame = (0 0; 375 667); autoresize = W+H; layer = <CALayer: 0x2808f3420>>
<UIView: 0x10a44f5f0; frame = (0 0; 375 667); autoresize = W+H; layer = <CALayer: 0x28091c2a0>>
<UIButton: 0x10a4b4d90; frame = (10 69.5; 38 38); opaque = NO; layer = <CALayer: 0x28091c4e0>>
<UIButton: 0x112832be0; frame = (324 67.5; 42 38); opaque = NO; layer = <CALayer: 0x280918ae0>>
<CopyMenuLabel: 0x112836fa0; baseClass = UILabel; frame = (44 69.5; 285 34); text = '0 62 424 256'; userInteractionEnabled = NO; layer = <_UILabelLayer: 0x282b835c0>>
<UILabel: 0x11283e9c0; frame = (67.5 101.5; 240 17); text = ''; userInteractionEnabled = NO; layer = <_UILabelLayer: 0x282b837a0>>
<UIButton: 0x11283ec30; frame = (152.5 395; 70 70); opaque = NO; layer = <CALayer: 0x2808f32a0>>
<UIButton: 0x10a4aaad0; frame = (60.5 140; 70 70); opaque = NO; tag = 1; layer = <CALayer: 0x28091d380>>
<UIButton: 0x10a4c6b30; frame = (152.5 140; 70 70); opaque = NO; tag = 2; layer = <CALayer: 0x28091d240>>
<UIButton: 0x10a4d85b0; frame = (244.5 140; 70 70); opaque = NO; tag = 3; layer = <CALayer: 0x28091d7c0>>
<UIButton: 0x10a4da2c0; frame = (60.5 225; 70 70); opaque = NO; tag = 4; layer = <CALayer: 0x28091d200>>
<UIButton: 0x10a4d9ec0; frame = (152.5 225; 70 70); opaque = NO; tag = 5; layer = <CALayer: 0x28091d7e0>>
<UIButton: 0x10a4dacf0; frame = (244.5 225; 70 70); opaque = NO; tag = 6; layer = <CALayer: 0x28091d860>>
<UIButton: 0x10a4dc180; frame = (60.5 310; 70 70); opaque = NO; tag = 7; layer = <CALayer: 0x28091d7a0>>
<UIButton: 0x10a4de610; frame = (152.5 310; 70 70); opaque = NO; tag = 8; layer = <CALayer: 0x28091d720>>
<UIButton: 0x10a4df320; frame = (244.5 310; 70 70); opaque = NO; tag = 9; layer = <CALayer: 0x28091d6a0>>
<UIButton: 0x10a4e0e10; frame = (60.5 395; 70 70); opaque = NO; tag = 10; layer = <CALayer: 0x28091d680>>
<UIButton: 0x10a4e32a0; frame = (244.5 395; 70 70); opaque = NO; tag = 11; layer = <CALayer: 0x2808f7fe0>>
<UIButton: 0x10a4e41d0; frame = (152.5 485; 70 70); opaque = NO; layer = <CALayer: 0x2808cffe0>>
<UIView: 0x10a4e6850; frame = (0 596; 375 71); clipsToBounds = YES; layer = <CALayer: 0x2808b5400>>
<UIButton: 0x10a4ebfd0; frame = (256 549; 100 30); opaque = NO; layer = <CALayer: 0x2808773e0>>
<UIImageView: 0x112800000; frame = (10 10; 18 18); clipsToBounds = YES; opaque = NO; userInteractionEnabled = NO; layer = <CALayer: 0x2808762a0>>
<UIImageView: 0x11282f740; frame = (11 10.5; 20 17); clipsToBounds = YES; opaque = NO; userInteractionEnabled = NO; layer = <CALayer: 0x280840ae0>>
<UIImageView: 0x1128409c0; frame = (0 0; 70 70); clipsToBounds = YES; opaque = NO; userInteractionEnabled = NO; layer = <CALayer: 0x2808b5ca0>>
<UIImageView: 0x11280c370; frame = (0 0; 70 70); clipsToBounds = YES; opaque = NO; userInteractionEnabled = NO; layer = <CALayer: 0x2808cfe40>>
<UIImageView: 0x11280e140; frame = (0 0; 70 70); clipsToBounds = YES; opaque = NO; userInteractionEnabled = NO; layer = <CALayer: 0x2808f2be0>>
<UIImageView: 0x112823de0; frame = (0 0; 70 70); clipsToBounds = YES; opaque = NO; userInteractionEnabled = NO; layer = <CALayer: 0x2808f20c0>>
<UIImageView: 0x112803a90; frame = (0 0; 70 70); clipsToBounds = YES; opaque = NO; userInteractionEnabled = NO; layer = <CALayer: 0x28091ad60>>
<UIImageView: 0x1128124f0; frame = (0 0; 70 70); clipsToBounds = YES; opaque = NO; userInteractionEnabled = NO; layer = <CALayer: 0x2809180c0>>
<UIImageView: 0x112811b10; frame = (0 0; 70 70); clipsToBounds = YES; opaque = NO; userInteractionEnabled = NO; layer = <CALayer: 0x28091ac20>>
<UIImageView: 0x1128385e0; frame = (0 0; 70 70); clipsToBounds = YES; opaque = NO; userInteractionEnabled = NO; layer = <CALayer: 0x28091b760>>
<UIImageView: 0x112843fa0; frame = (0 0; 70 70); clipsToBounds = YES; opaque = NO; userInteractionEnabled = NO; layer = <CALayer: 0x28091bca0>>
<UIImageView: 0x11280da60; frame = (0 0; 70 70); clipsToBounds = YES; opaque = NO; userInteractionEnabled = NO; layer = <CALayer: 0x280918ea0>>
<UIImageView: 0x11281aba0; frame = (0 0; 70 70); clipsToBounds = YES; opaque = NO; userInteractionEnabled = NO; layer = <CALayer: 0x280918b00>>
<UIImageView: 0x1128096c0; frame = (0 0; 70 70); clipsToBounds = YES; opaque = NO; userInteractionEnabled = NO; layer = <CALayer: 0x2809189e0>>
<UIImageView: 0x11282b4c0; frame = (0 0; 70 70); clipsToBounds = YES; opaque = NO; userInteractionEnabled = NO; layer = <CALayer: 0x280918960>>
<UILabel: 0x10a4e76d0; frame = (20 13; 223 31.5); text = 'Call any landline or mobi...'; userInteractionEnabled = NO; layer = <_UILabelLayer: 0x282b567b0>>
<ViberCoreUI.HighlightedButton: 0x10a4e7940; baseClass = UIButton; frame = (277 13; 79 32); opaque = NO; layer = <CALayer: 0x280888060>>
<UIButtonLabel: 0x10a4ec280; frame = (63 6.5; 37 17); text = 'Close'; opaque = NO; userInteractionEnabled = NO; layer = <_UILabelLayer: 0x282b83ca0>>
<UIButtonLabel: 0x10a4e7c00; frame = (14 8; 51 16); text = 'Try Now'; opaque = NO; userInteractionEnabled = NO; layer = <_UILabelLayer: 0x282b09d10>>
[iOS Device::PID::14349]->  


> ui-hierarchy-1.jpg

From above picture, we can see that there are total 13 buttons between 0x11283ec30 and 0x10a4e41d0. These are buttons 1,2,3...0,# + Call button as seen in the figure above. On top of each button we have UIImageView which is the overlay graphics above the buttons.

Looking further, I can disclose that 2xUIButton at address 0x10a4b4d90 and 0x112832be0 is responding to "Add" button and "Delete" button at the top. Lets say that UIButton at address 0x10a4b4d90 is what we are interested in.

We can further check and filter what we need to lookup the UIButton we are interested in with the following (from with-in Frida Console):

views = get_all_views(); labels = views.filter(function(v) { return v.$className.indexOf('Button') > 0; });


==== START ====
[iOS Device::PID::14349]-> views = get_all_views(); labels = views.filter(function(v) { return v.$className.indexOf('Button') > 0; });
<Viber.MainWindow: 0x10a420f80; baseClass = UIWindow; frame = (0 0; 375 667); autoresize = W+H; tintColor = <UIDynamicProviderColor: 0x28091b1e0; provider = <__NSMallocBlock__: 0x2807f6430>>; gestureRecognizers = <NSArray: 0x2806e5cb0>; layer = <UIWindowLayer: 0x2808496c0>>
<UITransitionView: 0x10a428df0; frame = (0 0; 375 667); autoresize = W+H; layer = <CALayer: 0x28084b020>>
[REDACTED]
<UIButtonLabel: 0x10a4e7c00; frame = (14 8; 51 16); text = 'Try Now'; opaque = NO; userInteractionEnabled = NO; layer = <_UILabelLayer: 0x282b09d10>>
[
    {
        "handle": "0x10a4b4d90"
    },
    {
        "handle": "0x112832be0"
    },
    {
        "handle": "0x11283ec30"
    },
    {
     	[REDACTED] ...
 	}
]

==== END ====

We can get the whole view hierarchy for just the choosen UIButton (The "Add" button). This is the View Stack and logic controllers from the background. The code is:

root@kali:/# cat hierarchy.js 

[SNIP]

var get_view_stack = function(view) {
	var stack = [];

	while (view.superview() != null) {
		stack.push(view);
		view = view.superview();
	}

	return stack;
}

Now to trace this hierarchy, lets run the script again and execute the same commands plus get_view_stack of the object we are interested in:

[iOS Device::PID::14349]-> var views = get_all_views();
[iOS Device::PID::14349]-> var buttons = views.filter(function(v) { return v.$className.indexOf('Button') >= 0; }); 
[iOS Device::PID::14349]-> stack = get_view_stack(buttons[0]); // point at ix 0, we are interested to "Add"
[iOS Device::PID::14349]-> stack.forEach(function(v, i) { return console.log(i, v.$className); 
0 UIButton
1 UIView
2 UIView
3 UITransitionView

### Tracing the UIButton Tap

Now that we've found the method we want to trace, we need to actually trace it with Frida. This can be done with Frida Stalker. 

First, we need to calculate the ASLR slide:

var impl = ObjC.classess.


The application is stored in: /private/var/containers/Bundle/Application/F1C05EEB-F3FD-4561-9E33-648598C803EF/Viber.app on my iPhone. I will transfer this directory to my pentesting box and fire up Hopper.

I can do that via SSH+Rsync:

PwnBox-Mac:~ h4x0r$ rsync -r root@172.22.4.103:/private/var/containers/Bundle/Application/F1C05EEB-F3FD-4561-9E33-648598C803EF/Viber.app $HOME/Desktop/Viber-Extract
root@172.22.4.103's password: ******


=== SNIP === [Viber Desktop]

  loc_100301a15:
0000000100301a15         lea        rdi, qword [aDlid]                          ; "dlid", CODE XREF=sub_1003015f0+903, sub_1003015f0+919, sub_1003015f0+1031, sub_1003015f0+1040
0000000100301a1c         mov        esi, 0x4
0000000100301a21         call       imp___stubs___ZN7QString17fromLatin1_helperEPKci ; QString::fromLatin1_helper(char const*, int)
0000000100301a26         mov        qword [rbp+-48], rax
0000000100301a2a         lea        rdx, qword [r12+8]
0000000100301a2f         lea        rsi, qword [rbp+-48]                        ; End of try block started at 0x1003019c7, Begin of try block (catch block at 0x1003022d9)
0000000100301a33         mov        rdi, r13
0000000100301a36         call       sub_100304120                               ; sub_100304120
0000000100301a3b         mov        rdi, qword [rbp+-48]                        ; End of try block started at 0x100301a2f, Begin of try block
0000000100301a3f         mov        eax, dword [rdi]
0000000100301a41         cmp        eax, 0xffffffff
0000000100301a44         je         loc_100301a62