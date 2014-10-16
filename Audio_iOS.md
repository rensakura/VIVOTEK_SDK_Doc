### Two way audio support (draft)
TwoWayAudioController header:
```objective-c
@interface TwoWayAudioController : NSObject

@property (weak, nonatomic) id<TwoWayAudioControllerDelegate> delegate;

- (id)initWithDelegate:(id) delegate;
- (void)setupWithIP:(NSString *)IP Port:(int)port Username:(NSString *)username Password:(NSString *)password;
- (void)start;
- (void)stop;

@end

@protocol TwoWayAudioControllerDelegate<NSObject>

- (void)twoWayAudioDidConnected;
- (void)twoWayAudioDidFail;
- (void)twoWayAudioDidDisconnected;

@end

```
#### How to use with DeviceRollingController
In your ViewController header:
```objective-c
#import <DeviceRolling/DeviceRollingController.h>
#import <TwoWayAudioController.h>

@interface ViewController : UIViewController<DeviceRollingControllerDelegate, TwoWayAudioControllerDelegate>
@property (strong, nonatomic) DeviceRollingController *deviceRollingController;
@property (strong, nonatomic) TwoWayAudioController *twoWayAudioController;
```

In your ViewController implement:

```objective-c
- (void)viewDidLoad
{
    [super viewDidLoad];

    // Init the DeviceRollingController with P2P server and delegate. 
    // A DeviceRollingController instance can be reuse for connecting to different devices, you don't need to create another new instance for each device.
    
    self.deviceRollingController = [[DeviceRollingController alloc] initWithP2PServerIP:YOUR_VIVOTEK_P2P_SERVER_IP port:YOUR_VIVOTEK_P2P_SERVER_PORT delegate:self];

    // As long as the DeviceRollingController has init, you can always call this function to establish P2P connection and get the latest P2P ports for your device
    [self.deviceRollingController getDeviceInfoOfID:YOUR_VIVOTEK_DEVICE_ID
                                                  username:YOUR_VIVOTEK_USERNAME
                                                  password:YOUR_VIVOTEK_PASSWORD];
                                                  
    self.twoWayAudioController = [[TwoWayAudioController alloc] initWithDelegate:self];
}

// Implement DeviceRollingControllerDelegate
- (void)getDeviceInfoDidCompleted:(DeviceInfo *)deviceInfo
                           status:(DeviceRollingStatus)rollingStatus
                             type:(DeviceRollingType)rollingType
                        p2pstatus:(P2PStatusType)p2pStatus
{
    // You will get a DeviceInfo object containing device ID, firmware version, username, password, IPs and ports
    
    if (rollingStatus == StatusOK)
    {
        NSString *bestIP;
        int bestHttpPort;
        int bestHttpsPort;
        int bestRtspPort;
        
        switch (rollingType)
        {
            case PUBLIC:
                bestIP = deviceInfo.publicIP;
                bestHttpPort = deviceInfo.publicHTTPPort;
                bestHttpsPort = deviceInfo.publicHTTPSPort;
                bestRtspPort = deviceInfo.publicRTSPPort;
                break;
            case LAN:
                bestIP = deviceInfo.lanIP;
                bestHttpPort = deviceInfo.lanHTTPPort;
                bestHttpsPort = deviceInfo.lanHTTPSPort;
                bestRtspPort = deviceInfo.lanRTSPPort;
                break;
            case P2P:
                bestIP = deviceInfo.p2pIP;
                bestHttpPort = deviceInfo.p2pHTTPPort;
                bestHttpsPort = deviceInfo.p2pHTTPSPort;
                bestRtspPort = deviceInfo.p2pRTSPPort;
                break;
            case RELAY:
                bestIP = deviceInfo.relayIP;
                bestHttpPort = deviceInfo.relayHTTPPort;
                bestHttpsPort = deviceInfo.relayHTTPSPort;
                bestRtspPort = deviceInfo.relayRTSPPort;
                break;
            default:
                break;
        }

        // Setup TwoWayAudioController with your device's available IP & port
        [self.twoWayAudioController setupWithIP:bestIP Port:bestHttpPPort Username:YOUR_VIVOTEK_USERNAME Password:YOUR_VIVOTEK_PASSWORD];
    }
}

// Control your two way audio by UI
- (void)enableTwoWayAudio
{
    [self.twoWayAudioController start];
}

- (void)disableTwoWayAudio
{
    [self.twoWayAudioController stop];
}

// Implement TwoWayAudioControllerDelegate
- (void)twoWayAudioDidConnected
{
    // Ready to talk
}

- (void)twoWayAudioDidFail
{
    // Fail to connect
}

- (void)twoWayAudioDidDisconnected
{
    // Lost connection
}

```