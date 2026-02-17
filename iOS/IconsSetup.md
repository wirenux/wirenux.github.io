# 📱 Create your own iPhone icons pack (iOS)

## Intro

Did you ever want to get custom icons on your iPhone?
To do this you can do the `Shortcut` Method or the `.mobileconfig` (the best one I think).

*For me*, the `.mobileconfig` method is the best 'cause of those reasons:

- Add multiple icons at once (you don't need to open the shortcut app for every app)
- Getting rid of `Shortcut` banner (when you open every app)
- Works with `URL Scheme` that shortcut may block

## What you need ?

To follow this tutorial you need at least those things:

- A text editor (`VSCode` my choice)
- A Base64 converter (I recommend this one: [Base64 converter](https://jam.dev/utilities/image-to-base64))
- Some icons *(ideally in PNG and 180x180)*

## 1. Encode icons

Go to [Base64 converter](https://jam.dev/utilities/image-to-base64) and upload an image, for the test I will upload this one (it's the maps Emojie):

<img src="../Images/iOS/Maps.png" width="130" height="130" />

When you put your icon in an converter you will have something that starts like this:

```base64
data:image/png;base64,iVBORw0KGgoAAAANSUh 
```

So copy it and save it in a .txt file or whatever (we need it later).

## 2. Create the `.mobileconfig`

The `.mobileconfig` is just an `.xml`/`.plist` file but with a different extension.

First you need to create a file named : `YourPackIcon.mobileconfig` and open it in a text editor.

A `.mobileconfig` file has 2 main parts, the Profile Container (like an header) and The payloads (the part that contains icons, links, etc...)

So let's dive in. ദ്ദി(•̀ ᗜ <)

In first, you need to add the **header**:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN"
 "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
```

After that you need:

```xml
    <!-- 1. GENERAL PROFILE METADATA -->
    <key>PayloadContent</key>
    <array>
        <dict>

            <!-- 2. WEBCLIP PAYLOAD IDENTIFIER -->
            <key>PayloadType</key>
            <string>com.apple.webClip.managed</string>

            <!-- 3. PAYLOAD UUID (unique ID) -->
            <key>PayloadIdentifier</key>
            <string>com.example.webclip.YOURID</string>

            <key>PayloadVersion</key>
            <integer>1</integer>

            <!-- 5. PAYLOAD DISPLAY NAME -->
            <key>PayloadDisplayName</key>
            <string>Your App Name</string>

            <!-- 6. THE ACTUAL BOOKMARK URL -->
            <key>URL</key>
            <string>yourapp://</string>

            <!-- 7. ICON (BASE64) -->
            <key>Icon</key>
            <data>
            BASE64_ICON_DATA_HERE
            </data>

            <key>IsRemovable</key>
            <true/>
            <key>FullScreen</key>
            <false/>
            <key>Precomposed</key>
            <true/>

        </dict>
    </array>
```

Now that you’ve added your payload, you need to close the Profile Container.
This is the second half of the .mobileconfig and it contains metadata about the whole profile (not the icon).

Add this right after the previous block:

```xml
    <!-- 8. TOP-LEVEL PROFILE IDENTIFIER -->
    <key>PayloadIdentifier</key>
    <string>com.example.profile.YOURID</string>

    <!-- 9. PROFILE NAME (shown to user in Settings) -->
    <key>PayloadDisplayName</key>
    <string>Your Custom Icon</string>

    <!-- 10. PROFILE TYPE -->
    <key>PayloadType</key>
    <string>Configuration</string>

    <key>PayloadVersion</key>
    <integer>1</integer>

    <!-- 12. PROFILE UUID -->
    <key>PayloadUUID</key>
    <string>UNIQUE-PROFILE-UUID</string>

</dict>
</plist>
```

---

### How to choose your PayloadIdentifier

The value:

`<string>com.example.webclip.YOURID</string>`

is basically like a "namespace" for your icon.

`com.example` = your domain (can be anything)

`webclip` = category

`YOURID` = a unique name for this specific icon (ex: spotify, discord, etc.)

Examples:

```xml
com.wirenux.webclip.spotify
com.wirenux.webclip.discord
com.wirenux.webclip.youtube
```

Each icon must have its own PayloadIdentifier.

### How to generate a UUID

iOS requires every payload and every profile to have a unique UUID.

You can generate UUIDs from: [https://www.uuidgenerator.net](https://www.uuidgenerator.net)

Every icon = 1 UUID
The main profile also = 1 UUID

---

### Ready to use Template

Paste this into a new file named YourPackIcon.mobileconfig

⚠️ Replace:

`YOUR-ICON-ID`

`YOUR-PROFILE-ID`

`UUID-HERE`

`BASE64_GOES_HERE` (remove data:image/png;base64,! Only the raw Base64)

`yourapp://` in this example I will use `maps://` (who open Apple Maps)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN"
 "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>

    <!-- Payload Content: all icons go here -->
    <key>PayloadContent</key>
    <array>
        <dict>

            <!-- WebClip payload -->
            <key>PayloadType</key>
            <string>com.apple.webClip.managed</string>

            <key>PayloadIdentifier</key>
            <string>com.example.webclip.YOUR-ICON-ID</string>

            <key>PayloadUUID</key>
            <string>UUID-HERE</string>

            <key>PayloadVersion</key>
            <integer>1</integer>

            <key>Label</key>
            <string>Your App Name</string>

            <key>URL</key>
            <string>yourapp://</string>

            <key>Icon</key>
            <data>
            BASE64_GOES_HERE
            </data>

            <key>IsRemovable</key>
            <true/>
            <key>FullScreen</key>
            <true/>
            <key>Precomposed</key>
            <true/>

        </dict>
    </array>

    <!-- Profile metadata -->
    <key>PayloadIdentifier</key>
    <string>com.example.profile.YOUR-PROFILE-ID</string>

    <key>PayloadDisplayName</key>
    <string>My Custom Icons</string>

    <key>PayloadType</key>
    <string>Configuration</string>

    <key>PayloadUUID</key>
    <string>UUID-HERE</string>

    <key>PayloadVersion</key>
    <integer>1</integer>

</dict>
</plist>
```

This is mine [Download Here](iOS/Maps.mobileconfig)

## Install the `.mobileconfig`

1. Open the .mobileconfig file

    Just tap on the file

2. Tap "Allow"

    A popup appears:

    “This profile will be downloaded.”

3. Go to Settings → General → VPN & Device Management

    You’ll see:

    Downloaded Profile

4. Tap Install

    Enter your passcode.

5. Confirm Install

    The icons appear ON your home screen immediately.

## Conclusion

And that it ! You did it  ദ്ദി(˵ •̀ ᴗ - ˵ ) ✧
Now you know how to create your own iPhone icon pack using the `.mobileconfig` method. It’s faster and cleaner than the Shortcut method, and you can customize your home screen exactly how you like.

Have fun creating your icons and giving your iPhone a fresh, unique look!

## Boring Stuff

### Boring stuff

You can find some `.mobileconfig` file here : [W-Clan](https://wallpapers-clan.com/app-icons/)
And and `.mobileconfig` exemple file here (to get retro icon like iOS 6): [iOS 6 icons](https://wallpapers-clan.com/wp-content/uploads/app-icons/retro-ios-app-icons.mobileconfig)

**Developed with ❤️ by [@wirenux](https://github.com/wirenux)**
