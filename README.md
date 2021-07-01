# Kaltura Minimal Editor
A proof of concept that demonstrates the minimal amount of code to run the The Kaltura [Editor](https://github.com/kaltura-vpaas/kaltura-editor-app-embed). First a  [Kaltura Express Recorder]( https://github.com/kaltura-vpaas/express-recorder) is used to record a video, then the editor is implemented to allow recordings to be trimmed, finally the Kaltura javascript player is used to playback the recording. This example helps you understand the flow of a recording through the Kaltura API and provides a pattern for the kind of user experience you should consider implementing when integrating the editor.

| [Live Demo](https://kaltura-minimal-editor.herokuapp.com/) | [Video Guide](http://www.kaltura.com/tiny/ksa4z) |
| :--------------------------------------------------------: | :----------------------------------------------: |

## Requirements:

1. [Nodejs](https://nodejs.org/en/)
2. [Kaltura VPaaS account](https://corp.kaltura.com/video-paas/registration?utm_campaign=Meetabout&utm_medium=affiliates&utm_source=GitHub)

## Getting Started:

1. Copy [env.template](https://github.com/kaltura-vpaas/kaltura-minimal-editor/blob/main/env.template) to .env and fill in your information
2. Run: npm install
3. For developement run: npm run dev   
4. Or, for production run: npm start

## Underneath the hood

This sample application consists of three screens. The first screen uses the [Kaltura Express Recorder](https://github.com/kaltura-vpaas/express-recorder ) to record a video https://github.com/kaltura-vpaas/kaltura-minimal-editor/blob/main/views/index.ejs

In the third screen, The Kaltura Player is used to display the edited video.

### The Editor

The editor is another standalone javascript based API component. And this project a minimal implementation of [Kaltura Editor API component](https://github.com/kaltura-vpaas/kaltura-editor-app-embed) 

From the [previous screen:](https://github.com/kaltura-vpaas/kaltura-minimal-editor/blob/main/views/index.ejs#L28) 

```javascript
    window.location = "edit?entryId="+event.detail.entryId;
```

will route execution to [edit.js](https://github.com/kaltura-vpaas/kaltura-minimal-editor/blob/main/routes/edit.js) which configures a Kaltura API session and routes execution to [edit.ejs](https://github.com/kaltura-vpaas/kaltura-minimal-editor/blob/main/views/edit.ejs) 

The editor javascript component is a sophisticated editor capable of many different editing modes like quizzes, hotspots and more. The teleprompter only uses a single editing mode, which is clipping and trimming of videos. 

The editing mode is specified in `tabs` field of the `data` dict used to configure the editor

```javascript
 'tabs': {
            'edit': {
              name: 'edit',
              permissions: ['clip', 'trim'],
              userPermissions: ['clip', 'trim'],
              showOnlyExpandedView: true,
              showSaveButton: false,
              showSaveAsButton: true,
              preSaveMessage: '',
            }
          },
```

Just like the Express Recorder, the editor has an extensive event listener interface which you can read more about at [Kaltura Editor API component](https://github.com/kaltura-vpaas/kaltura-editor-app-embed) 

In [edit.ejs](https://github.com/kaltura-vpaas/kaltura-minimal-editor/blob/main/views/edit.ejs#L121)

```javascript
      /* received when a clip was created.
      * postMessageData.data: {
      *  originalEntryId,
      *  newEntryId,
      *  newEntryName
      * }
      * should return a message where message.messageType = kea-clip-message,
      * and message.data is the (localized) text to show the user.
      * */
      if (postMessageData.messageType === 'kea-clip-created') {
        // send a message to editor app which will show up after clip has been created:
        var message = 'Thank you for creating a new clip, the new entry ID is: ' + postMessageData.data.newEntryId;
        e.source.postMessage({
          'messageType': 'kea-clip-message',
          'data': message
        }, e.origin);
        window.location = "/share?entryId="+postMessageData.data.newEntryId;
      }
```

The events this application uses is `kea-clip-created` which fires after the clip has been created. As this example uses the Save As button specified above by 

```javascript
showSaveAsButton: true
```

The newEntryId will correspond to the new entry created when the editor is used to clip or trim the original entry.

You could call [media.get](https://developer.kaltura.com/console/service/media/action/get) on the new entry which stores the originalEntry as `rootEntryId` and also stores the timerange of the clip operation:

```json
  "rootEntryId": "1_123456",
  "operationAttributes": [
    {
      "offset": 0,
      "duration": 29784,
      "effectArray": [],
      "objectType": "KalturaClipAttributes"
    },
```

Finally, the `newEntryId` is used to continue the demo app's flow to the final share screen:

```javascript
 window.location = "/share?entryId="+postMessageData.data.newEntryId;
```



### Sharing The Video

The sharing page is based off https://developer.kaltura.com/player. But before the player is shown, the video must be ready for sharing.

In [views/share.ejs](https://github.com/kaltura-vpaas/kaltura-minimal-editor/blob/main/views/share.ejs)

```
        function poll() {
            $.getJSON("/status?entryId=<%=entryId%>", function (data) {
                if (data['ready']) {
                    //show player
                } else {
                    setTimeout(poll, 3000);
                }
            }
        }
        poll();
```

the `/status` method in [routes/index.js](https://github.com/kaltura-vpaas/kaltura-minimal-editor/blob/main/routes/index.js) is polled recursively every 3 seconds until the video is ready.

And when the video is ready, it is displayed:

```
                if (data['ready']) {
                    $("#spinner").hide();
                    try {
                        var player = KalturaPlayer.setup({
                            targetId: "kaltura-player",
                            provider: {
                                partnerId: <%= process.env.KALTURA_PARTNER_ID %>,
                                uiConfId: <%= process.env.KALTURA_REC_PLAYER_ID %>
                            }
                        });
                        //load first entry in player
                        player.loadMedia({ entryId: '<%= entryId%>' });
```

# How you can help (guidelines for contributors) 
Thank you for helping Kaltura grow! If you'd like to contribute please follow these steps:
* Use the repository issues tracker to report bugs or feature requests
* Read [Contributing Code to the Kaltura Platform](https://github.com/kaltura/platform-install-packages/blob/master/doc/Contributing-to-the-Kaltura-Platform.md)
* Sign the [Kaltura Contributor License Agreement](https://agentcontribs.kaltura.org/)

# Where to get help
* Join the [Kaltura Community Forums](https://forum.kaltura.org/) to ask questions or start discussions
* Read the [Code of conduct](https://forum.kaltura.org/faq) and be patient and respectful

# Get in touch
You can learn more about Kaltura and start a free trial at: http://corp.kaltura.com    
Contact us via Twitter [@Kaltura](https://twitter.com/Kaltura) or email: community@kaltura.com  
We'd love to hear from you!

# License and Copyright Information
All code in this project is released under the [AGPLv3 license](http://www.gnu.org/licenses/agpl-3.0.html) unless a different license for a particular library is specified in the applicable library path.   

Copyright © Kaltura Inc. All rights reserved.   
Authors and contributors: See [GitHub contributors list](https://github.com/kaltura/YOURREPONAME/graphs/contributors).  

### Open Source Libraries Used

https://github.com/manifestinteractive/teleprompter  MIT License