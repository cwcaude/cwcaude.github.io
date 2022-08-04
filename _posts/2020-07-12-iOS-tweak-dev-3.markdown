---
layout: post
title:  "iOS Tweak Development : 3"
date:   2020-7-12 16:00:00 -0400
categories: iOS_Tweak_Development
---
# Note
This series of posts is not made as strictly a *tutorial* While I will be explaining how things work and giving instruction on iOS tweak development, the purpose of this posting is to **explain** and **document** my journey/path of learning to develop iOS tweaks. If you desire for strictly a tutorial in iOS Tweak Development check out these links here :
[Zane Helton](https://www.youtube.com/watch?v=uNXd4KLLjhk&list=PLFWEDfSyl7h_K8Ew4rwTzlUPgWU7nKYri) or [r/Jailbreak Tutorial](https://www.reddit.com/r/jailbreak/comments/839bnv/tutorial_how_to_get_into_tweak_development_for/)

# Building my **own** tweak
Now that I have researched some open source tweaks, it's time to attempt my own tweak. For this tweak I want to modify the Messages app to replace the word 'Messages' at the top with a different word. To do this during the creation of the tweak template using Theos/nic.pl I made sure to set the Mobile Substrate Bundle Identifier to *com.apple.MessageSMS*. This will allow me to hook into the Messages app process and modify what I desire.

# Finding *what* to modify
In order to begin developing the tweak I used FLEXing to study the Messages application and find what to modify. 

![FLEXing Select Screenshot](/assets/FlexSelect.jpg) 

Here I use the select function to select the part of the application I want to modify. From there I can view the *View* that I have selected and look at all the methods I can hook into. 

![FLEXing Methods Screenshot](/assets/UILabel.PNG) 

I found a method called *setText* that I could hook into to attempt to change the value. 

![setText Method](/assets/setTextSearch.PNG)

# Hooking *setText*
Within the Theos tweak I have setup I hook into the UILabel view. From here I call the *setText* method and change the argument to *"Intel"* as that is the text I am attempting to change it to. Here is the entire Tweak.x file to do that:

```objc
@interface UIInterface : UILabel
@end

%hook UILabel

		-(void)setText:(id)arg1 {
			%orig(@"Intel");
		}

%end
```
Now all I have to do is install the tweak and see what happens. 

![Hooking setText](/assets/IntelEverywhere.PNG)

Well it certainly changed some text to Intel, but definitely not what I expected to happen. Here we can see that every single instance of a UILabel except, ironically, for the *"Messages"* text has been changed to *"Intel"*. So obviously it's back to the drawing board on this one. 

# Exploring other options
In order to find other possible solutions to this I first went to the Internet. I was looking for a solution on how to access a specific UILabel. While browsing answers I found this [Reddit Post](https://www.reddit.com/r/jailbreakdevelopers/comments/gi2wvy/access_uilabel_from_view/) where the user was asking about a similar tweak idea but in the Mail application. A solution given is to hook the *MailboxPickerController* and set the title text within another method. This leads to me looking through FLEXing and [Limneos](https://developer.limneos.net/index.php?ios=13.1.3) to find an equivalent *MessagePickerController or SMSPickerController* and could not find anything for a really long time. I eventually give up on this and finally after way too long went back to using *Select* in FLEXing.

![View Above](/assets/ViewAbove.PNG) 

I found that one step above the UILabel that displays the *"Messages"* text there is a view called *_UINavigationBarLargeTitleView*. Within this view there is a method called *setTitle*. So I decided to try and hook into this view and access this method. 

# Let's try this again
Here again you can see me hook into the view, and access the setTitle Method. After I get
```objc
%hook _UINavigationBarLargeTitleView

		-(void)setTitle: (id)arg1 {
			%orig(@"Intel");
		}

%end
```

Let's install this tweak and see what happens. 

![Successful Intel](/assets/Success1.PNG)

As you can see I successfully changed the text! However I noticed when I scroll down and the text gets smaller, it shows *"Messages"* again. No problem, I'll just use FLEXing again and view what is above the UILabel view. This time I find *_UINavigationBarContentView*, also with the method *setTitle*. In order to modify this view I add this to the Tweak.x file and reinstall:
```objc
%hook _UINavigationBarContentView

		-(void)setTitle: (id)arg1 {
			%orig(@"Intel");
		}

%end
```
![Successful Intel 2](/assets/Success2.PNG)

Once again this worked and I achieved my goal! Though it may seem like a trivial accomplishment and amount of code, this success has given me confidence in the fact that I know *something* about what I am doing. This motivation will allow me to continue learning and developing more complicated tweaks. 

# What next?
Now that I have developed this simple tweak, I now want to learn how to make it customizable within the settings application. So that is what I will be developing for next week! Thanks for reading! View the next post [here](https://cwcaude.github.io/project/tutorial/2020/07/16/iOS-tweak-dev-4.html).