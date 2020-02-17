# HowTo: Symbolicating iOS Crashlog (VS + Xamarin)

1. Enable the "Build..IPA" option for iOS project and rebuild it (Not sure whether this steps is required).

![iOS IPA Options](https://raw.githubusercontent.com/sergbuk/docu/master/iOS%20IPA%20Options.png)

When you build your iOS project, you should have the .app file in the buil directory on your Mac.

**Sample path to the build directory**: /Users/serg/Library/Caches/Xamarin/mtbs/builds/TheMobileApp.iOS/4a31f70da5bded32547e1048624495c1/bin/iPhone/Debug/ThereforeGo.app

> Important: You need to use the same .app file that generated the crash report. This means you would have to use the .app from the package you installed on the device that generated the crash report. Hopefully, you archived that or saved it somewhere. Simply rebuilding the project to get a new .app will not match up with the .crash file during symbolication and will not work. If you don't have access to that, you will need to publish again and this time keep the .app around for the next time you get a .crash to analyze.

2. Copy *.app, *.app.DSYM and the crashlog text file to some folder on the Mac.

3. **Create an Alias**. Open Terminal and run one of these commands for your version of Xcode:
>alias symbolicate="/Applications/Xcode.app/Contents/SharedFrameworks/DVTFoundation.framework/Versions/A/Resources/symbolicatecrash -v"

4. **Update the Developer Directory**. Run this command:
>export DEVELOPER_DIR="/Applications/Xcode.app/Contents/Developer"

5. **Symbolicate**. Open Terminal again and cd to the directory where you placed your files in the step above. Run the symbolicate command we aliased before with your .crash and .app files as the parameters like this:
>symbolicate -o "symbolicatedCrash.txt" "crashlog_text.crash" "MyAppName.app"

This will symbolicate the crash file and spit out the result in a new file named "symbolicatedCrash.txt". Make sure that correct the file names from my example to match yours.

see also: https://stackoverflow.com/questions/38579117/how-to-symbolicate-crash-error-logs-from-a-xamarin-forms-ios-project
