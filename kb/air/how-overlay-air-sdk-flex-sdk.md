# Overlay AIR SDK on Flex SDK for Flash Builder

Applies to:

- Adobe AIR SDK
- Flash Builder 4.7

The instructions below reference specific versions of the Flex and AIR SDKs:
Flex SDK 4.6.0 and AIR SDK 3.4. The instructions, however, are general. You can
follow these instructions to overlay any version of the AIR SDK on any version
of the Flex SDK.

> Note: If the Flex SDK is newer than the AIR SDK, it can rely on AIR
> functionality that isn't present, resulting in compile-time or runtime errors.

1.  Exit Flash Builder.

2.  (Optional) Back up the Flex SDK by copying the entire directory.

    In Flash Builder, for example, copy the directory at:

    - Windows: C:/Program Files/Adobe/Flash Builder 4.7/sdks/4.6.0

    - Mac OS: /Applications/Adobe Flash Builder 4.7/sdks/4.6.0

3.  Download the appropriate AIR SDK file for your operating system from
    [https://airsdk.dev/](https://airsdk.dev/), and save it to the root
    directory of the Flex SDK.

    - Windows: AdobeAIRSDK.zip

    - Mac OS: AdobeAIRSDK.dmg

4.  Extract the contents of the AIR SDK archive and overwrite the existing SDK
    files.

    - Windows: Right-click the ZIP file and select Extract All, or use a
      decompression tool of your choice.

    - Mac OS: In Terminal, run these commands:

      - `hdiutil attach AdobeAIRSDK.dmg`

      - `cp -rf /Volumes/AIR\\ SDK/\* /path-to-empty-FLEXSDK-directory`

    - If you have trouble overwriting files due to file permissions, try these
      commands:

      - `sudo hdiutil attach AdobeAIRSDK.dmg`

      - `sudo cp -rf /Volumes/AIR\\ SDK/\* /path-to-empty-FLEXSDK-directory`

5.  (Optional) To access the new AIR 3.4 APIs, update your application
    descriptor file to the 3.4 namespace.

    To update the namespace, change the xmlns attribute in your application
    descriptor to:
    `<application xmlns="http://ns.adobe.com/air/application/3.4">`

6.  (Optional) To ensure that the output SWF file targets SWF version 17, pass
    an additional compiler argument: `-swf-version=17`.

> This work is licensed under a
> [Creative Commons Attribution-Noncommercial-Share Alike 3.0 Unported License](https://creativecommons.org/licenses/by-nc-sa/3.0/)
