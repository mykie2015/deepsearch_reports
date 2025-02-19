# Best Practices for Measuring iOS App Size

This report outlines best practices for measuring the download and install size of your iOS app throughout its lifecycle, from daily development builds to the final version users download from the App Store. Understanding and managing app size is crucial for user experience, affecting download times, storage consumption, and potentially user acquisition.

## Key Concepts

*   **Download Size:** The compressed size of the app users download from the App Store. This is smaller than the install size due to compression.
*   **Install Size:** The actual storage space the app occupies on the user's device after installation. This is the uncompressed size.
*   **App Thinning:** A technology introduced with iOS 9 that optimizes app delivery by providing only the necessary resources and code for a specific device. This significantly reduces both download and install sizes. App thinning includes *slicing*, *bitcode*, and *on-demand resources*.
*   **Slicing:** The process of creating and delivering only the resources (images, etc.) needed for a specific device's screen resolution and capabilities.
*   **Bitcode:** An intermediate representation of the compiled app. When enabled, it allows Apple to re-optimize the app's binary for specific device architectures, potentially reducing size.
*   **On-Demand Resources (ODR):** Resources (like images, sounds, or levels in a game) hosted on the App Store and downloaded only when needed by the app. This significantly reduces the initial download size.

## App Thinning: The Key to Accurate Measurement

App Thinning, introduced with iOS 9, is crucial for optimized app delivery. It ensures only necessary resources and code for a specific device are downloaded. It comprises three main components:

*   **Slicing:** The App Store creates and delivers different app variants for different devices (e.g., iPhone 11, iPad Pro). Each variant contains only the resources (images, assets) and executable architecture needed for that device.
*   **Bitcode:** Allows Apple to re-optimize your app in the future without requiring you to submit a new version. The App Store can recompile your app from the bitcode to take advantage of new compiler improvements.
*   **On-Demand Resources (ODR):** Allows you to tag certain resources (e.g., levels in a game, less-frequently used features) to be downloaded only when needed. This significantly reduces the initial download and install size.

Because of App Thinning, the size of your development builds (even release builds) *will not* accurately reflect the final App Store size.

## Measurement at Different Stages

### 1. Daily Builds (Development)

*   **Best Practice:** Use Xcode's "App Thinning Size Report."

*   **Steps:**

    1.  In Xcode, Archive your app (Product > Archive).
    2.  Export the archived app as an "Ad Hoc," "Development," or "Enterprise" build.
    3.  In the distribution options, select "All compatible device variants" for app thinning. This is crucial to simulate the App Store's thinning process.
    4.  Optionally, enable "Rebuild from Bitcode" if you are using Bitcode.
    5.  Sign and export the app.
    6.  The output folder will contain a file named "App Thinning Size Report.txt". This report provides detailed size information for each device variant, including compressed (download) and uncompressed (install) sizes.

*   **Example (Bash):**

    ```bash
    app_path="Size.xcarchive/Products/Applications/Size.app"
    size_in_mb=$(echo "scale=2; <span class="math-inline">\(du \-sk "</span>{app_path}" | cut -f1) / 1024" | bc)
    echo "$size_in_mb MB"
    ```

*   **Bloaty McBloatface:** A size profiler for binaries. It can give you a detailed breakdown of what's contributing to your binary size.

    ```bash
    brew install bloaty
    ```

*   **Importance:** This method provides a close estimate of the sizes users will experience, accounting for app thinning. It's far more accurate than simply looking at the size of the `.app` bundle or `.ipa` file generated during regular development builds, as those include resources for all device types.

### 2. Release Builds (Pre-App Store Submission)

*   **Best Practice:** Continue using the "App Thinning Size Report" as described above. This remains the most accurate method *before* submitting to the App Store. You can also use TestFlight (see below), but keep in mind that TestFlight builds include extra data.

*   **Xcode's App Thinning Size Report:** This is the recommended method for estimating App Store sizes *before* submitting to App Store Connect.

    1.  **Archive:** In Xcode, select "Product" > "Archive".
    2.  **Export:** In the Organizer (Window > Organizer), select your archive and click "Distribute App".
    3.  **Choose Distribution Method:** Select "Ad Hoc," "Development," or "Enterprise."
    4.  **App Thinning Options:** Crucially, choose "**All compatible device variants**" for app thinning. This generates separate IPAs for each device type, simulating the App Store's slicing process. Also, ensure "Rebuild from Bitcode" is selected (though this option might not be present in newer Xcode versions).
    5.  **Export and Review:** Sign and export the app. The output folder will contain a file named `App Thinning Size Report.txt`. This report provides compressed (download) and uncompressed (install) sizes for each device variant.

### 3. TestFlight (Beta Testing)

*   **App Store Connect File Sizes:** This is the *most accurate* way to determine the actual download and install sizes that users will experience.

*   **Considerations:**

    *   TestFlight builds *are* thinned, but they contain additional data not included in App Store releases. This means the TestFlight build will be larger than the final App Store version.
    *   The size displayed in the TestFlight app is the *download* size of the TestFlight build, *not* the final App Store download size.
    *   Some sources indicate significant discrepancies between TestFlight sizes and App Store sizes. This is likely due to the extra data included in TestFlight builds and potential differences in compression.

*   **Steps:**

    1.  **Upload:** Upload your build to App Store Connect (either for TestFlight or for App Store review).
    2.  **View File Sizes:** In App Store Connect, navigate to your app, then "TestFlight" (for TestFlight builds) or "Activity" (for App Store builds). You'll find a section called "App Store File Sizes" or similar, which lists the download and install sizes for various device types and iOS versions. This information is generated *after* Apple has processed your build with App Thinning.

### 4. App Store Connect (Post-Submission)

*   **Best Practice:** Use App Store Connect for the *most accurate* size information.

*   **Steps:**

    1.  Upload your build to App Store Connect (either for TestFlight or for App Store release).
    2.  Go to your app in App Store Connect.
    3.  Navigate to the "Activity" or "TestFlight" tab (depending on your build's purpose).
    4.  Select the specific build you want to analyze.
    5.  Look for "App Store File Sizes" or a similar option. This will display the download and install sizes for various device types.

*   **Importance:** App Store Connect provides the *definitive* sizes after Apple has processed the app, including any final optimizations or additions (like DRM).

### 5. User's Cellphone (Post-App Store Download)

*   The install size on the user's device will match the "uncompressed" size reported in App Store Connect for their specific device type.
*   **iPhone Storage:** User can check the app install size from "Settings -> General -> iPhone Storage."
*   The download size will match the "compressed" size.
*   The user can see the install size prominently displayed in the App Store. The download size is displayed if the app exceeds the cellular download limit (currently 200MB) and the user is not on Wi-Fi. Users can configure their settings to always show download size warnings.

## RASP Impact Analysis

Runtime Application Self-Protection (RASP) adds security measures directly into your app to detect and prevent attacks in real-time. This *can* impact app size and performance.

*   **Size Increase:** RASP typically involves adding libraries and code to monitor the app's behavior, which increases the overall binary size. The extent of the increase depends on the specific RASP solution and its features.
*   **Performance Overhead:** Real-time monitoring requires computational resources (CPU, memory). Poorly implemented RASP can lead to noticeable slowdowns. However, well-optimized RASP solutions aim to minimize this impact.
*   **Measuring RASP Impact:**
    *   **Before and After:** Generate App Thinning Size Reports (as described above) *before* and *after* integrating RASP. Compare the sizes to quantify the increase.
    *   **Performance Profiling:** Use Xcode's Instruments (specifically the "Time Profiler" and "Allocations" templates) to measure the CPU and memory usage of your app with and without RASP enabled. This helps identify any performance bottlenecks introduced by RASP.

## Using TestFairy for App Size Monitoring

TestFairy is a mobile testing platform that can help with app size monitoring, although it's not its primary focus.

*   **App Distribution:** TestFairy facilitates distributing your app builds to testers.
*   **Session Recordings:** TestFairy records user sessions, including CPU and memory usage. While not a direct size measurement, this can help you identify if size optimizations (or RASP integration) are affecting performance.
*   **Crash Reports:** TestFairy provides crash reports, which can be helpful if size-related issues (e.g., excessive memory usage) are causing crashes.
*   **Integration:** TestFairy integrates with other tools like JIRA and Slack.
*   **Auto Updates:** TestFairy can help ensure users are on the latest version.
*   **Adding Events:** TestFairy allows adding custom events to provide insights into how testers use your apps.
*   **Attaching Files:** TestFairy allows developers to attach files to sessions (up to five files to a given session, with a maximum size of 15MB per file).

### Important Considerations

*   **TestFairy and App Thinning:** TestFairy itself doesn't directly perform App Thinning. You still need to use Xcode's App Thinning Size Report or App Store Connect for accurate size measurements. TestFairy primarily helps with distribution and monitoring *during* testing.
*   **UDID Registration:** For iOS, Testfairy uses a device management profile to send UDIDs to the Test Fairy servers.

## Best Practices for Reducing App Size

Beyond measurement, here are key strategies to *reduce* app size:

*   **Asset Catalogs:** Use asset catalogs to manage your images, textures, and other assets. Tag assets with appropriate metadata (device type, scale) to enable efficient slicing.
*   **Image Optimization:**
    *   Use appropriate image formats (HEIC for iOS 11+, WebP where supported).
    *   Compress images without sacrificing visual quality (tools like ImageOptim).
    *   Use resizable images (define a resizable center area) where possible.
*   **On-Demand Resources (ODR):** Use ODR for content that isn't needed immediately.
*   **Code Optimization:**
    *   **Remove Unused Code:** Use tools like `lint` (for unused resources) and compiler optimizations (Link-Time Optimization) to eliminate dead code.
    *   **Optimize Libraries:** Carefully evaluate the size of third-party libraries. Consider alternatives or modularize them if possible.
    *   **Code Refactoring:** Refactor code to reduce IPA file size.
    *   **Compiler Optimization:** Use compiler optimization to reduce IPA file size.
*   **App Thinning:** Always leverage App Thinning (slicing, bitcode, ODR).
*   **Modularize:** Break down large apps into smaller, independent modules.
*   **Optimize Updates:** The App Store creates an update package instead of always downloading the whole app.

## Continuous Monitoring

App size should be monitored continuously throughout the development lifecycle. Integrate size checks into your CI/CD pipeline (e.g., using the `xcodebuild` command to generate App Thinning Size Reports automatically). This helps catch size regressions early.

## Additional Tips and Considerations

*   **Cellular Download Limit:** Be mindful of the cellular download limit (200MB). App Store Connect will warn you if your app exceeds this limit.
*   **On-Demand Resources (ODR):** Use ODR to significantly reduce initial download size by hosting non-essential resources on the App Store and downloading them only when needed.
*   **Optimize Images and Media:** Use efficient image formats (WebP, HEIC) and compress images appropriately. Consider using vector graphics where possible.
*   **Remove Unused Resources:** Regularly review your project and remove any unused assets, code, or libraries. Xcode's `lint` tool and third-party tools like FengNiao (for iOS) can help identify unused resources.
*   **Code Optimization:** Use code shrinking and obfuscation tools (like ProGuard for Android, though less directly applicable to iOS, the principles are similar). Link-Time Optimization (LTO) in Xcode can help remove unused code.
*   **Modularize and Lazy Load:** Consider breaking your app into modules and loading them only when needed.
*   **App Updates:** iOS 13 and later have improved app update sizes, potentially reducing them by up to 50%. However, this refers to the *download* size of the update, not the overall install size.
*   **Differences between Finder size and App Store Connect:** There can be discrepancies between the size reported by Finder (even for the thinned `.app`) and the size reported by App Store Connect. This is due to factors like file system allocation units and additional processing done by Apple. App Store Connect is the definitive source.

By following these best practices, you can accurately measure and optimize your app's size, leading to a better user experience and potentially increased adoption. Remember that App Store Connect provides the most accurate size information after your app has been processed by Apple.