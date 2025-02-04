---
title: Learn How To Track Core Playback on Roku
description: Learn how to implement core tracking using the Media SDK on Roku.
uuid: a8aa7b3c-2d39-44d7-8ebc-b101d130101f
exl-id: 5272c0ce-4e3d-48c6-bfa6-94066ccbf9ac
feature: Media Analytics
role: User, Admin, Data Engineer
---
# Track core playback on Roku{#track-core-playback-on-roku}

This documentation covers tracking in version 2.x of the SDK.

>[!IMPORTANT]
>If you are implementing a 1.x version of the SDK, you can download 1.x Developers Guides here: [Download SDKs](/help/sdk-implement/download-sdks.md)

1. **Initial tracking setup**

    Identify when the user triggers the intention of playback (the user clicks play and/or autoplay is on) and create a `MediaObject` instance.

    **`MediaObject` reference:**

    |  Variable Name  | Description  | Required  |
    | --- | --- | :---: |
    | `name`  | Video name  | Yes  |
    | `mediaid`  | Video unique identifier  | Yes  |
    | `length`  | Video length  | Yes  |
    | `streamType`  |Stream type (see _StreamType constants_ below)  | Yes  |
    |  `mediaType`  | Media type (see _MediaType constants_ below)  | Yes  |

    **`StreamType` constants:**

    |  Constant Name  | Description&nbsp;&nbsp;  |
    |---|---|
    | `MEDIA_STREAM_TYPE_VOD`  | Stream type for Video on Demand.  |
    | `MEDIA_STREAM_TYPE_LIVE`  | Stream type for LIVE content.  |
    | `MEDIA_STREAM_TYPE_LINEAR`  | Stream type for LINEAR content.  |
    | `MEDIA_STREAM_TYPE_AOD`  | Stream type for Audio On Demand  |
    | `MEDIA_STREAM_TYPE_AUDIOBOOK`  | Stream type for Audio Book  |
    | `MEDIA_STREAM_TYPE_PODCAST`  | Stream type for Podcast  |

    **`MediaType` constants:**

    |  Constant Name  | Description  |
    |---|---|
    |  `MEDIA_STREAM_TYPE_AUDIO`  | Media type for Audio streams.  |
    |  `MEDIA_STREAM_TYPE_VIDEO`  | Media type for Video streams.  |

    **Create a media info object for video with VOD content:**

    ```
     mediaInfo = adb_media_init_mediainfo(
      "<MEDIA_NAME>",
      "<MEDIA_ID>",
      600,
      ADBMobile().MEDIA_STREAM_TYPE_VOD,
      ADBMobile().MEDIA_TYPE_VIDEO
    )
    ```

    or

    ```
    mediaInfo = adb_media_init_mediainfo()
    mediaInfo.name = "<MEDIA_NAME>"
    mediaInfo.id = "<MEDIA_ID>"
    mediaInfo.length = 600
    mediaInfo.streamType = ADBMobile().MEDIA_STREAM_TYPE_VOD
    mediaInfo.mediaType = ADBMobile().MEDIA_TYPE_VIDEO
    ```

    **Create a media info object for video with AOD content:**

    ```
    mediaInfo = adb_media_init_mediainfo(
     "<MEDIA_NAME>",
     "<MEDIA_ID>",
     600,
     ADBMobile().MEDIA_STREAM_TYPE_AOD,
     ADBMobile().MEDIA_TYPE_AUDIO
    )
    ```

    or

    ```
    mediaInfo = adb_media_init_mediainfo()
    mediaInfo.name = "<MEDIA_NAME>"
    mediaInfo.id = "<MEDIA_ID>"
    mediaInfo.length = 600
    mediaInfo.streamType = ADBMobile().MEDIA_STREAM_TYPE_AOD
    mediaInfo.mediaType = ADBMobile().MEDIA_TYPE_AUDIO
    ```

1. **Attach metadata**

    Optionally attach standard and/or custom metadata objects to the tracking session through context data variables.

    * **Standard metadata**

       [Implement standard metadata on Roku](/help/sdk-implement/track-av-playback/impl-std-metadata/impl-std-metadata-roku.md)

       >[!NOTE]
       >Attaching the standard video metadata object to the media object is optional.

    * **Custom metadata**

       Create a variable object for the custom variables and populate with the data for this video. For example:     

       ```    
       mediaContextData = {}
       mediaContextData["cmk1"] = "cmv1"
       mediaContextData["cmk2"] = "cmv2"
       ```

1. **Track the intention to start playback**

    To begin tracking a media session, call `trackSessionStart` on the Media Heartbeat instance:

    ```
    ADBMobile().mediaTrackSessionStart(mediaInfo,mediaContextData)
    ```

    >[!TIP]
    >The second value is the custom video metadata object name that you created in step 2.

    >[!IMPORTANT]
    >`trackSessionStart` tracks the user intention of playback, not the beginning of the playback. This API is used to load the video data/metadata and to estimate the time-to-start QoS metric (the time duration between `trackSessionStart` and `trackPlay`).

    >[!NOTE]
    >If you are not using custom video metadata, simply send an empty object for the `data` argument in `trackSessionStart`, as shown in the commented out line in the iOS example above.

1. **Track the actual start of playback**

    Identify the event from the video player for the beginning of the video playback, where the first frame of the video is rendered on the screen, and call `trackPlay`:

    ```
    ADBMobile().mediaTrackPlay()
    ```

1. **Update playhead value**

    When media playhead changes notify the SDK by calling `mediaUpdatePlayhead` API. <br /> For video-on-demand (VOD), the value is specified in seconds from the beginning of the media item. <br /> For live streaming, if the player does not provide information about the content duration, the value can be specified as the number of seconds since midnight UTC of that day. <br /> Note: When using progress markers, the content duration is required and the playhead needs to be updated as number of seconds from the beginning of the media item, starting with 0.
  

    ```
    ADBMobile().mediaUpdatePlayhead(position)
    ```

1. **Track the completion of playback**

    Identify the event from the video player for the completion of the video playback, where the user has watched the content until the end, and call `trackComplete`:

    ```
    ADBMobile().mediaTrackComplete()
    ```

1. **Track the end of the session**

    Identify the event from the video player for the unloading/closing of the video playback, where the user closes the video and/or the video is completed and has been unloaded, and call `trackSessionEnd`:

    ```
    ADBMobile().mediaTrackSessionEnd()
    ```

    >[!IMPORTANT]
    >`trackSessionEnd` marks the end of a video tracking session. If the session was successfully watched to completion, where the user watched the content until the end, ensure that `trackComplete` is called before `trackSessionEnd`. Any other `track*` API call is ignored after `trackSessionEnd`, except for `trackSessionStart` for a new video tracking session.

1. **Track all possible pause scenarios**

    Identify the event from the video player for video pause and call `trackPause`:

    ```
    ADBMobile().mediaTrackPause()
    ```

    **Pause Scenarios**

    Identify any scenario in which the Video Player will pause and make sure that `trackPause` is properly called. The following scenarios all require that your app call `trackPause()`:

    * The user explicitly hits pause in the app.
    * The player puts itself into the Pause state.
    * (*Mobile Apps*) - The user puts the application into the background, but you want the app to keep the session open.
    * (*Mobile Apps*) - Any type of system interrupt occurs that causes an application to be backgrounded. For example, the user receives a call, or a pop up from another application occurs, but you want the application to keep the session alive to give the user the opportunity to resume the video from the point of interruption.

1. Identify the event from the player for video play and/or video resume from pause and call `trackPlay`:

    ```
    ADBMobile().mediaTrackPlay()
    ```

    >[!TIP]
    >This may be the same event source that was used in Step 4. Ensure that each `trackPause()` API call is paired with a following `trackPlay()` API call when the video playback resumes.

* Tracking scenarios: [VOD playback with no ads](/help/sdk-implement/tracking-scenarios/vod-no-intrs-details.md)
* Sample player included with the Roku SDK for a complete tracking example.
