---
layout: post
title:  "iOS Tweak Development : 4"
date:   2020-7-16 16:00:00 -0400
categories: Project Tutorial
---
# Note
This series of posts is not made as strictly a *tutorial* While I will be explaining how things work and giving instruction on iOS tweak development, the purpose of this posting is to **explain** and **document** my journey/path of learning to develop iOS tweaks. If you desire for strictly a tutorial in iOS Tweak Development check out these links here :
[Zane Helton](https://www.youtube.com/watch?v=uNXd4KLLjhk&list=PLFWEDfSyl7h_K8Ew4rwTzlUPgWU7nKYri) or [r/Jailbreak Tutorial](https://www.reddit.com/r/jailbreak/comments/839bnv/tutorial_how_to_get_into_tweak_development_for/)

# Adding Settings options to my Tweak
The next thing I wanted to learn is how to add a settings option to my tweak, this is how I did it.

# Creating a Preference Bundle
In the settings app, each app that is listed has what is called a *Preference Bundle*. This Preference Bundle is what determines how the settings app will look and behave. To create a preference bundle that is linked with your current tweak, run *$THEOS/bin/nic.pl* from within the directory of your tweak and create a *preference bundle modern*.

# Important Files in the Preference Bundle

I will not be covering every file within the directory, however I will be listing the important ones I needed to modify. 

#### entry.plist
Within this file, you can find the Label that will be seen from the main page of the settings application. As well as the name of the icon file that will be displayed on that page as well.

#### MSPRootListController*.h & .m*
It is within these files that you can create new methods to be used as actions within your preferences page. 

#### Resources/Root.plist
Here is where you will build your preferences page in XML. Each *part* of the preference page is called a cell, and within this file you can define the type of cell as well as modify properties of each cell. I will go into more detail on this later. 

#### Resources/Info.plist
Not much (if anything) in this file is required to be modified, however this is where the bundle identifier is stored and can be modified. This is used by your tweak to access your preferences, as well as used by the OS to create the bundle to be saved. 

# First steps
The first thing I did was begin building out the preferences page to include what I wanted for my tweak. I wanted: Enable Switch, Custom Text Switch, Custom Text Box, Website Link, and Donation Link. To add these sections to the preferences page I modified the Resources/Root.plist file. This file uses xml in order to build the page. In order to learn how to actually use this I used [Zane Helton's Tutorial](https://www.youtube.com/watch?v=iHEuM1e8V40) and [iPhoneDevWiki's Site](https://iphonedevwiki.net/index.php/Preferences_specifier_plist). To explain it quickly, each page is made up of *cells*, you can view the different types of cells from [iPhoneDevWiki's Site](https://iphonedevwiki.net/index.php/Preferences_specifier_plist). 

# Building the preferences page
The file is structured as an *array* of *dictionaries*, each dictionary specifies a cell. Here is an example of what a cell looks like:

```xml
	<dict>
		<key>cell</key>
		<string>PSSwitchCell</string>
		<key>default</key>
		<true/>
		<key>defaults</key>
		<string>com.ziegen.messageswappreferences</string>
		<key>key</key>
		<string>isEnabled</string>
		<key>label</key>
		<string>Enabled</string>
	</dict>
```
This cell type is a *PSSwitchCell*, this is denoted using the *<key>* **cell** and following it with a *<string>* of the cell type. Again these cell types can be found on [iPhoneDevWiki's Site](https://iphonedevwiki.net/index.php/Preferences_specifier_plist). 

Using the *default* key you can specify a default value for the cell, in the example above considering the cell type is a switch I set the default value to *<true/>*. However if the cell was a *PSEditTextCell* like I use for the custom text, I would set the default to a *<string>* value. 

In order to add persistence to the values of these cells you need to use the *defaults* key. You need to set this value to the bundle identifier you have specified in your **Info.plist** file, in my case it is *com.ziegen.messageswappreferences*. Directly following the *defaults* key, you need to set a *key* key which allows you to set a name to the value you are storing, you can think of it as naming the *variable* you are saving. You should set this name to be something unique that you can use to differentiate each value. In my example, because this switch controls whether or not the tweak is enabled or not, I set the name to *isEnabled*. 

Finally the last part of this example you can see the *label* key, this sets the *(you guessed it)* **label** shown on the settings page.

# Grouping Cells
In order to group cells within this array you can define a new cell of *PSGroupCell*, every cell following this cell until the next *PSGroupCell* is included within that group. Here is an example of a group cell:

```xml
	<dict>
		<key>cell</key>
		<string>PSGroupCell</string>
		<key>label</key>
		<string>Settings</string>
	</dict>
```

As you can see it's a rather simple cell, with only the cell specifier and a label to display above the cell.

# Adding **actions** to the buttons on the Settings page
In order to add actually functionality to the buttons on the settings page, you must create a method within the *MSPRootListController.m* file to be executed when the button is pressed. In my example, my buttons simply open websites. This is the code for that method to be included within the *MSPRootListController* implementation:

```objc
-(void)openWebsite{
	[[UIApplication sharedApplication]
	openURL:[NSURL URLWithString:@"https://cwcaude.github.io/"]
	options:@{}
	completionHandler:nil];
}
```

# Full File
Here I am going to paste the full *Root.plist* file and a screenshot of what it looks like so you can compare directly how each line affects the page. It may look overwhelming at first, but it is very easy to understand if you take a moment to analyze it.  

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
	<key>items</key>
	<array>
		<dict>
			<key>cell</key>
			<string>PSGroupCell</string>
			<key>label</key>
			<string>Settings</string>
		</dict>
		<dict>
			<key>cell</key>
			<string>PSSwitchCell</string>
			<key>default</key>
			<true/>
			<key>defaults</key>
			<string>com.ziegen.messageswappreferences</string>
			<key>key</key>
			<string>isEnabled</string>
			<key>label</key>
			<string>Enabled</string>
		</dict>
		<dict>
			<key>cell</key>
			<string>PSSwitchCell</string>
			<key>default</key>
			<false/>
			<key>defaults</key>
			<string>com.ziegen.messageswappreferences</string>
			<key>key</key>
			<string>isCustom</string>
			<key>label</key>
			<string>Custom Text</string>
		</dict>
		<dict>
			<key>cell</key>
			<string>PSGroupCell</string>
			<key>label</key>
			<string>Custom Text to Display</string>
		</dict>
		<dict>
			<key>cell</key>
			<string>PSEditTextCell</string>
			<key>default</key>
			<string>Custom</string>
			<key>defaults</key>
			<string>com.ziegen.messageswappreferences</string>
			<key>key</key>
			<string>displayText</string>
		</dict>
		<dict>
			<key>cell</key>
			<string>PSGroupCell</string>
			<key>label</key>
			<string>Credits</string>
		</dict>
		<dict>
			<key>cell</key>
			<string>PSButtonCell</string>
			<key>label</key>
			<string>Learn how I made this tweak</string>
			<key>action</key>
			<string>openWebsite</string>
		</dict>
				<dict>
			<key>cell</key>
			<string>PSButtonCell</string>
			<key>label</key>
			<string>Buy me a Coffee</string>
			<key>action</key>
			<string>openDonation</string>
		</dict>
	</array>
	<key>title</key>
	<string>Message Swap</string>
</dict>
</plist>
```
![Screenshot of Settings Page](/assets/iOS-tweak-dev-4-SettingsPage.jpeg)

# Modifying the tweak to use this Preference Bundle Data
In the last post I developed the *Tweak.xm* to modify the values with a hard-coded value. However with this new preferences page I need to add the ability to modify the tweak with different settings changed within the preference page. To do this I created some new functions to access this data from the *plist bundle*. 

```objc
#define PLIST_PATH @"/var/mobile/Library/Preferences/com.ziegen.messageswappreferences.plist"

inline bool GetPrefBool (NSString *key)
{
	return [[[NSDictionary dictionaryWithContentsOfFile:PLIST_PATH] valueForKey:key] boolValue];
}

NSString* GetPrefString (NSString *key)
{
	return [[[NSDictionary dictionaryWithContentsOfFile:PLIST_PATH] valueForKey:key] stringValue];
}
```

Here you can see I defined the *PLIST_PATH* so that it can be more easily used within the functions defined below it. 

Each of these functions queries the *plist bundle* file and accesses the preferences saved via the *defaults* we set up earlier. Each function returns a specific type of value, for example the *GetPrefBool* returns boolean values that could be given from a switch. While the *GetPrefString* returns a value that returns a string value from a text box. Another function I could make would be one that would return a float.

In order to get the value you want from these functions you would make a call like this `GetPrefBool(@"isEnabled")` Here you can see I am returning a Boolean value from the key *"isEnabled"*, this will of course return the status of our enabled switch. Here you can see both of the functions utilized with a hook to control the program using these values from the preference bundle:

```objc
%hook _UINavigationBarLargeTitleView

	-(void)setTitle: (id)arg1 {
	if (GetPrefBool(@"isEnabled"))
	{		
		if (GetPrefBool(@"isCustom"))	{
			%orig(GetPrefString(@"customText"));
		}
		else {
			%orig(@"Intel");
		}
	}
	else {
			%orig(@"Messages");
		}
}

%end
```

Here the *GetPrefBool* function is used as the argument in simple *if/else* statements to control the flow of the tweak since it will return a *true* or *false* value. The *GetPrefString* function is used as the argument to the *%orig()* call to give the string of the custom text stored in the specified key.

# How'd it turn out?
Screenshot with the tweak disabled:

![DisabledScreenshot](/assets/iOS-tweak-dev-4-disabled.PNG)

Screenshot with the tweak enabled but custom disabled:

![NoCustomScreenshot](/assets/iOS-tweak-dev-4-Enabled.PNG)

Screenshot with Custom enabled:

![CustomScreenshot](/assets/iOS-tweak-dev-4-Custom.PNG)

Obligatory Meme Screenshot:

![11Dollaridoos](/assets/iOS-tweak-dev-4-11Dollas.PNG)

# Want to install this tweak?
You can install this right now from Cydia from the repo here: https://cwcaude.github.io
Any issues you have with this tweak should be posted as an issue to the Github page. [Link Here](https://github.com/cwcaude/MessageSwap/issues)


# What next?
This tweak now looks like it's something presentable! However now I want to add more functionality to the tweak such as an unread messages counter, and controlling the color of that text. So that is what I'll be working on for next post! If you are enjoying these posts and would like to support me I have a donation link down below. Any questions or comments about this post can be sent to my email and I will try to respond quickly! 