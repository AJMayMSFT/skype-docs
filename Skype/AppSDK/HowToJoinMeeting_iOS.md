# Use the SDK to join a meeting with an iOS device

This article shows an iOS developer how to join the **Skype for Business meeting** using a [**meeting URL**](https://msdn.microsoft.com/en-us/skype/appsdk/getmeetingurl) and enable core **Skype for Business App SDK** features like Text chat, Audio/Video chat in your app. Android developers should read
[Use the SDK to join a meeting with an Android device](HowToJoinMeeting_Android.md). 

 No **Skype for Business** credentials are used to join the meeting.

##Prerequisites
 **Objective C**
 - **Import the SDK header file**: import the required header files.

      ```objective-c
             #import <SkypeForBusiness/SkypeForBusiness.h>
            
            //Import SfBConversationHelper classes for Audio/Video Chat
             #import "SfBConversationHelper.h"
      ```

**Swift**

- **Create Swift Bridging - Header file**: Create the bridging-header file and add the following import statement.

      ```swift
        //Add SfBConversationHelper classes for Audio/Video Chat
            #import "SfBConversationHelper.h"
```
 
  >Note: Be sure to read [Getting started with Skype App SDK development](GettingStarted.md) to learn how to configure your iOS project 
for the **Skype for Business** App SDK.  In particular, the following steps assume you have added the _ConversationHelper_ class
 to your source to let you complete the scenario with a minimum of code. 

## How to get started
1. In your code, initialize the **App SDK** application :

 **Objective C**
  ```Objectivec 
   SfBApplication *sfb = SfBApplication.sharedApplication;
   ```
 **Swift**
  ```swift 
   let sfb:SfBApplication? = SfBApplication.sharedApplication()
```
2. You can handle application level Skype configurations like requireWifiForAudio, maxVideoChannels, requireWifiForVideo, setActiveCamera, get available cameras list  and other types of information that can impact the Skype session, for example, by default, video service will be disabled while not on Wi-Fi network. To allow video call on any network connection, we can configure requireWifiForVideo as follow:

 **Objective C**
  ```Objectivec 
sfb.configurationManager.requireWifiForVideo = NO;
```
 **Swift**
  ```swift 
  sfb.configurationManager.requireWifiForVideo = false
  ```
 
 > Note: Please refer SfBApplication, SfBConfigurationManager, SfBVideoService, SfBDevicesManager and other classes in SkypeForBusiness framework to handle application level Skype configurations.

3. Start joining the meeting by calling _Application.joinMeetingAnonymously(String displayName, URI meetingUri)_. This function returns the new conversation instance that represents the meeting.  

    **Objective C**
  ```Objectivec 
   SfBConversation *conversation = [sfb joinMeetingAnonymousWithUri:[NSURL URLWithString:meetingURLString]
                                        displayName:meetingDisplayName 
                                        error:&error];
  ```
    **Swift**
  ```Swift
   let conversation: SfBConversation  = try sfb.joinMeetingAnonymousWithUri(NSURL(string:meetingURLString)!, displayName: meetingDisplayName)
  ```
 > Note: all of the SDK’s interfaces must be used only from the application main thread (main run loop).  Notifications are delivered in the same thread as well.  As a result, no synchronization around the SDK’s interfaces is required.  The SDK, however, may create threads for internal purposes.      

4. Initialize the conversation helper with the conversation instance obtained in the previous step and delegate object that should receive callbacks from this conversation helper.  This will automatically start incoming and outgoing video. The delegate class must conform to _SfBConversationHelperDelegate_ protocol.
  
  **Objective C**
     ```Objectivec 
       if (conversation) {
       _conversationHelper = [[SfBConversationHelper alloc] initWithConversation:conversation
                                                     delegate:self
                                                     devicesManager:sfb.devicesManager
                                                     outgoingVideoView:self.selfVideoView
                                                     incomingVideoLayer:(CAEAGLLayer *) self.participantVideoView.layer
                                                     userInfo:@{DisplayNameInfo:meetingDisplayName}];
                                                     
      }
  ```      
    **Swift**
     ```Swift
   self.conversationHelper = SfBConversationHelper(conversation: conversation,
                                                            delegate: self,
                                                            devicesManager: sfb.devicesManager,
                                                            outgoingVideoView: self.selfVideoView,
                                                            incomingVideoLayer: self.participantVideoView.layer as! CAEAGLLayer,
                                                            userInfo: [DisplayNameInfo:meetingDisplayName])
```
  > Note: as per the license terms, before you start video for the first time after install, you must prompt the user to accept the Microsoft end-user license (also included in the SDK).  Subsequent versions of the SDK preview will include code to assist you in doing so.
      
5. Implement SfBConversationHelperDelegate methods to handle video service state changes.

    **Objective C**
     ```Objectivec 
      #pragma mark - Skype Delegates
    
    // At incoming video, unhide the participant video view
    - (void)conversationHelper:(SfBConversationHelper *)avHelper didSubscribeToVideo:(SfBParticipantVideo *)video {
        self.participantVideoView.hidden = NO; 
    }

    **Objective C**
     ```Objectivec 
    // When it's ready, start the video service and show the outgoing video view.
    - (void)conversationHelper:(SfBConversationHelper *)avHelper videoService:(SfBVideoService *)videoService didChangeCanStart:(BOOL)canStart {
        if (canStart) {
            [videoService start:nil];
            
            if (self.selfVideoView.hidden) {
                self.selfVideoView.hidden = NO;
            }       
        } 
    }

    // When incoming video is ready, show it.
    - (void)conversationHelper:(SfBConversationHelper *)avHelper didSubscribeToVideo:(SfBParticipantVideo *)video {
        self.participantVideoView.hidden = NO; 
    }
    ```   
    
    **Swift**
     ```swift
    // When it's ready, start the video service and show the outgoing video view.
    func conversationHelper(conversationHelper: SfBConversationHelper, videoService: SfBVideoService, didChangeCanStart canStart: Bool) {     
        if (canStart) {
            do {
                try videoService.start()
            }
            catch let error as NSError {
                print(error.localizedDescription)
                                
            }
            if (self.selfVideoView.hidden) {
                self.selfVideoView.hidden = false
            }
        }
    }
    ```   
    
    **Swift**
     ```swift
     
     //MARK - Skype SfBConversationHelperDelegate methods
     
     // At incoming video, unhide the participant video view
    func conversationHelper(conversationHelper: SfBConversationHelper, didSubscribeToVideo video: SfBParticipantVideo?) {
        self.participantVideoView.hidden = false
    }
    
    // When video service is ready to start, unhide self video view and start the service.
    func conversationHelper(conversationHelper: SfBConversationHelper, videoService: SfBVideoService, didChangeCanStart canStart: Bool) {
        
        if (canStart) {
            if (self.selfVideoView.hidden) {
                self.selfVideoView.hidden = false
            }
            do{
                try videoService.start()
            }
            catch let error as NSError {
                print(error.localizedDescription)
                                
            }
        }
    }
     // When the audio status changes, reflect in UI
    func conversationHelper(avHelper: SfBConversationHelper, selfAudio audio: SfBParticipantAudio, didChangeIsMuted isMuted: Bool) {
        if !isMuted {
            self.muteButton.setTitle("Unmute", forState: .Normal)
        }
        else {
            self.muteButton.setTitle("Mute", forState: .Normal)
        }
    }

     ```
     
<<<<<<< HEAD
6. Now the meeting is established, you can use additional APIs to switch camera, mute/hold audio, pause video, send/receive messages, leave the meeting etc.  See the [API reference documentation](http://aka.ms/sfbAppSDKRef_iOS) for more details and the sample apps ([Objective C](https://github.com/OfficeDev/skype-ios-app-sdk-samples/tree/master/BankingAppObjectiveC), [Swift](https://github.com/OfficeDev/skype-ios-app-sdk-samples/tree/master/BankingAppSwift)) for examples of how to do this.  You may also want to adapt SfBConversationHelper for your own needs: its source code is available in the SDK download.
     
     
## Error Handling

SkypeForBusiness SDK API has both _SfBApplication_ and _SfBConversation_ level delegate method for handling possible errors or exceptions. The _SfBApplication_ alertDelegate handles global level concerns, while the _SfBConversation_ alertDelegate handles alerts specific to the conversation instance.

The delegate method _didReceiveAlert_ is called when new alert appears in the context where alertDelegate is attached.

=======
6. To end the video meeting, monitor _canLeave_ property of a conversation to prevent leaving prematurely.

    **Objective C**
     ```Objectivec 
     //Add observer to _canLeave_ property
 if (conversation) {
        [conversation addObserver:self forKeyPath:@"canLeave" options:NSKeyValueObservingOptionInitial | NSKeyValueObservingOptionNew context:nil];
 }
 
    // Monitor canLeave property of a conversation to prevent leaving prematurely
     
      - (void)observeValueForKeyPath:(NSString *)keyPath ofObject:(id)object change:(NSDictionary<NSString *,id> *)change context:(void *)context {
    
       if ([keyPath isEqualToString:@"canLeave"]) {
        self.endCallButton.enabled = _conversationHelper.conversation.canLeave;
     }
    }
    
    // Use SfBConversation class leave function to leave the conversation.
    NSError *error = nil;
    [_conversationHelper.conversation leave:&error];
    
    if (error) {
        [self handleError:error];
    }
    else {
        [_conversationHelper.conversation removeObserver:self forKeyPath:@"canLeave"];
    }
 ```
    **Swift**
     ```swift
     // Add observer to _canLeave_ property
        conversation.addObserver(self, forKeyPath: "canLeave", options: [.Initial, .New] , context: nil)
        
    // Monitor canLeave property of a conversation to prevent leaving prematurely
    
        override func observeValueForKeyPath(keyPath: String?, ofObject object: AnyObject?, change: [String : AnyObject]?, context: UnsafeMutablePointer<Void>) {
        if (keyPath == "canLeave") {
            self.endCallButton.enabled = (self.conversationHelper?.conversation.canLeave)!
         }
    }
    
    // Use SfBConversation class leave function to leave the conversation.
        do{
            try self.conversationHelper?.conversation.leave()
            self.conversationHelper?.conversation.removeObserver(self, forKeyPath: "canLeave")
        }
```

## Text Chat 

 >Note: ChatHandler is the helper class that can be used to integrate Skype text chat feature into your application. It can be integrated in similar manner to SfBConversationHelper. Please refer [iOS sample apps](https://github.com/OfficeDev/skype-ios-app-sdk-samples) for further details.
>>>>>>> master
