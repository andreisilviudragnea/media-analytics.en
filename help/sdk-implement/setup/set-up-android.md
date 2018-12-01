---
seo-title: Set up Android
title: Set up Android
uuid: 3ffe3276-a104-4182-9220-038729e9f3d5
index: y
internal: n
snippet: y
---

# Set up Android{#set-up-android}

* **Obtain valid configuration parameters for the Media SDK -** These parameters can be obtained from an Adobe representative after you set up your analytics account. 
* **Implement ADBMobile for Android in your application -** For more information about the Adobe Mobile SDK documentation, see [Android SDK 4.x for Experience Cloud Solutions](https://marketing.adobe.com/resources/help/en_US/mobile/android/). 

* **Provide the following capabilities in your media player:**

    * *An API to subscribe to player events* - The Media SDK requires that you call a set of simple APIs when events occur in your player. 
    * *An API that provides player information* - This information includes details such as the media name and the play head position.

1. Add your [downloaded](../../sdk-implement/download-sdks.md#section_551A10AD7880426BB29AE52482BB4211) Media SDK to your project.

    1. Expand the Android zip file (e.g., [!DNL MediaSDK-android-v2.*.zip]). 
    1. Verify that the [!DNL MediaSDK.jar] file exists in the [!DNL libs/] directory. 
    
    1. Add the library to your project.

       **IntelliJ IDEA:**

        1. Right click your project in the **[!UICONTROL Project navigation]** panel. 
        1. Select **[!UICONTROL Open Module Settings]**. 
        1. Under **[!UICONTROL Project Settings]**, select **[!UICONTROL Libraries]**. 
        
        1. Click **[!UICONTROL +]** to add a new library. 
        1. Select **[!UICONTROL Java]** and navigate to the `MediaSDK.jar` file. 
        
        1. Select the modules in which you plan to use the mobile library. 
        1. Click **[!UICONTROL Apply]** and then **[!UICONTROL OK]** to close the Module Settings window.

       **Eclipse:**

        1. In the Eclipse IDE, right-click on the project name. 
        1. Click  **[!UICONTROL Build Path]** > **[!UICONTROL Add External Archives]** . 
        1. Select [!DNL MediaSDK.jar]. 
        1. Click **[!UICONTROL Open]**. 
        1. Right-click the project again, and click  **[!UICONTROL Build Path]** > **[!UICONTROL Configure Build Path]** . 
        1. Click the **[!UICONTROL Order]** and **[!UICONTROL Export]** tabs. 
        
        1. Ensure that the [!DNL MediaSDK.jar] file is selected.

1. Import the library.

   ```java
   import com.adobe.primetime.va.simple.MediaHeartbeat; 
   import com.adobe.primetime.va.simple.MediaHeartbeat.MediaHeartbeatDelegate; 
   import com.adobe.primetime.va.simple.MediaHeartbeatConfig; 
   import com.adobe.primetime.va.simple.MediaObject; 
   
   ```

1. Create the `MediaHeartbeatConfig` instance.

   Here is a sample `MediaHeartbeatConfig` initialization:

   ```java
   // Media Heartbeat Initialization 
   config.trackingServer =  
<i><SAMPLE_HEARTBEAT_TRACKING_SERVER></i>; 
   config.channel =  
<i><SAMPLE_HEARTBEAT_CHANNEL></i>; 
   config.appVersion =  
<i><SAMPLE_HEARTBEAT_SDK_VERSION></i>; 
   config.ovp =  
<i><SAMPLE_HEARTBEAT_OVP_NAME></i>; 
   config.playerName =  
<i><SAMPLE_PLAYER_NAME></i>; 
   config.ssl =  
<i><true/false></i>; 
   config.debugLogging =  
<i><true/false></i>; 
   
   ```

1. Implement the `MediaHeartbeatDelegate` interface.

   ```java
   public class VideoAnalyticsProvider  
     implements Observer, MediaHeartbeatDelegate {}
   ```

   ```java
   // Replace <bitrate>, <startupTime>, <fps>, and  
   // <droppeFrames> with the current playback QoS values.  
   @Override 
   public MediaObject getQoSObject() { 
       return MediaHeartbeat.createQoSObject(<bitrate>,  
                                             <startupTime>,  
                                             <fps>,  
                                             <droppedFrames>); 
   } 
    
   //Replace <currentPlaybackTime> with the video player current playback time 
   @Override 
   public Double getCurrentPlaybackTime() { 
       return <currentPlaybackTime>; 
   }
   ```

1. Create the `MediaHeartbeat` instance.

   Use the `MediaHeartbeatConfig` instance and the `MediaHertbeatDelegate` instance to create the `MediaHeartbeat` instance.

   ```java
   // Replace <MediaHertbeatDelegate> with your delegate instance 
   MediaHeartbeat _heartbeat =  
     new MediaHeartbeat(<MediaHeartbeatDelegate>, config);
   ```

   >[!IMPORTANT]
   >
   >Make sure that your `MediaHeartbeat` instance is accessible and *does not get deallocated until the end of the session*. This instance will be used for all of the following tracking events.

**Adding app permissions**

Your app using the Media SDK requires the following permissions to send data in tracking calls:

* `INTERNET` 
* `ACCESS_NETWORK_STATE`

To add these permissions, add the following lines to your [!DNL AndroidManifest.xml] file in the application project directory:

* `<uses-permission android:name="android.permission.INTERNET" />` 
* `<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />`

**Migrating from version 1.x to 2.x in Android**

In versions 2.x, all of the public methods are consolidated into the `com.adobe.primetime.va.simple.MediaHeartbeat` class to make it easier on developers. Also, all configs are now consolidated into the `com.adobe.primetime.va.simple.MediaHeartbeatConfig` class.

For detailed information about migrating from 1.x to 2.x, see [](../../sdk-implement/va-1x-to-2x/va-1x-to-2x.md). 