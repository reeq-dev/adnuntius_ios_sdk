# Adnuntius iOS SDK

Adnuntius iOS SDK is an ios sdk which allows business partners to embed Adnuntius ads in their native ios applications.

## Upgrading from 1.1.4 and 1.1.5

Unfortunately 1.2.0 is not API compatible with 1.1.4 and 1.1.5.  Version 1.2.0 was released with fairly significant upgrades to allow it to work with Objective-C and to enable applications to configure more than one ad configuration in their application.  Before because the configuration was static this was pretty much impossible.

Unfortunately this does mean you will need to make changes to your app to use the new version.  Please refer to the Samples project to figure out what needs to be changed.  Please open an issue if you are still struggling and we will try and create a more detailed migration guide.

If you want to keep compiling your application with the earlier version of the SDK (1.1.4 or 1.1.5) you should adjust your cartfile as follows:

github "Adnuntius/ios_sdk" == 1.1.5

## Building

Use Carthage cli to build the AdnuntiusSDK.framework and import into your project.   Create or modify your Cartfile to include:

github "Adnuntius/ios_sdk" == 1.2.0

Run carthage update 

The framework should be added to your project as a linked framework.  Drag and drop the Carthage/Build/iOS/AdnuntiusSDK.framework onto your project.

![Linked Framework](https://i.imgsafe.org/fd/fd36067938.png)

Add a Run Script Build Phase to your project, make sure you fill in the Input File section too:

![Build Phases Run Script](https://i.imgsafe.org/fd/fd1ea7b820.png)

For more information about Carthage, refer to [If you're building for iOS, tvOS, or watchOS](https://github.com/Carthage/Carthage#if-youre-building-for-ios-tvos-or-watchos)

### Objective C Only

![Always Embed Swift Standard Libraries](https://i.imgsafe.org/ea/ea85b8846b.png)

Because the SDK is Swift based, if you are including it as a framework into your objective c application, the Swift libraries must also be included, they are not by default.

## Integrating

### Swift

- Add UIWebView to your storyboard and create outlet
- Configure each AdnuntiusAdWebView
- Load the ad into the view via the loadFromScript, loadFromConfig or loadFromApi
- Implement the completionHandler to react to a missing ad


- In your `ViewController` file add header and implement the viewDidLoad method:

```swift
import AdnuntiusSDK
```
```swift
    override func viewDidLoad() {
        super.viewDidLoad() 
        
        adView.loadFromScript("""
        <html>
        <head>
            <script type="text/javascript" src="https://cdn.adnuntius.com/adn.js" async></script>
        </head>
        <body>
        <div id="adn-000000000006f450" style="display:none"></div>
        <script type="text/javascript">
            window.adn = window.adn || {}; adn.calls = adn.calls || [];
              adn.calls.push(function() {
                adn.request({ adUnits: [
                    {auId: '000000000006f450', auW: 300, auH: 200, kv: [{'version':'X'}] }
                ]});
            });
        </script>
        </body>
        </html>
        """, completionHandler: self)
    }
    
    func onComplete(_ view: AdnuntiusAdWebView, _ adCount: Int) {
        print("Completed: " + String(adCount))
    }
    
    func onFailure(_ view: AdnuntiusAdWebView, _ message: String) {
        view.loadHTMLString("<h1>Error is: " + message + "</h1>",
        baseURL: nil)
    }
```
- Integrate it with your view for example:
```swift
// Adnuntius injector
    if (indexPath.row % 4 == 0) {
        if let preCell = adCells?[indexPath.row] {
            debugPrint("preCell")
            return preCell
        }
        let cell = UITableViewCell()
        let webView = AdnuntiusAdWebView(frame: CGRect(x: 0, y: 10, width: tableView.frame.width, height: 100))
        adView1.loadFromApi([
               "adUnits": [
                    ["auId": "000000000006f450", "kv": [{"version": "6s"}]
               ]
            ]
        ], completionHandler: self)
        cell.contentView.addSubview(webView)
        cell.contentView.sizeToFit()
        adCells?[indexPath.row] = cell
        return cell
    }
    
    func onComplete(_ view: AdnuntiusAdWebView, _ adCount: Int) {
        print("Completed: " + String(adCount))
    }
    
    func onFailure(_ view: AdnuntiusAdWebView, _ message: String) {
        view.loadHTMLString("<h1>Error is: " + message + "</h1>",
        baseURL: nil)
    }
```

The onComplete / onFailure AdWebViewStateDelegate protocol methods are where you can add logic to react to various outcomes of trying to load an an ad.  For instance if there are no matched ads, the onComplete will return an adCount of 0, and you could hide the ad view for instance.

### Objective C

- Add UIWebView to your storyboard and create outlet
- Declare a @property referencing the AdnuntiusAdWebView declared in the story board
- Load the ad into the view via the loadFromScript, loadFromConfig or loadFromApi
- Implement the completionHandler to react to a missing ad

In the ViewController header file import the AdnuntiusSDK swift header:

```swift
#import <AdnuntiusSDK/AdnuntiusSDK-Swift.h>

@property (weak, nonatomic) IBOutlet AdnuntiusAdWebView *adView;
```

In the ViewController m file, implement the viewDidLoad method:

```swift
[super viewDidLoad];

NSString *adScript = @" \
<html> \
    <head > \
        <script type="text/javascript" src="https://cdn.adnuntius.com/adn.js" async></script> \
    </head> \
    <body> \
        <div id=\"adn-0000000000067082\" style=\"display:none\"></div> \
        <script type="text/javascript"> \
            window.adn = window.adn || {}; adn.calls = adn.calls || []; \
              adn.calls.push(function() { \
                adn.request({ adUnits: [ \
                    {auId: '000000000006f450', auW: 300, auH: 200, kv: [{'version':'X'}] } \
                ]}); \
            }); \
        </script> \  
    </body> \
</html>";

[self.adView loadFromScript:adScriptcompletionHandler:self];
```

- Change Info.plist

```xml
<key>NSAppTransportSecurity</key>
<dict>
  <key>NSAllowsArbitraryLoads</key>
  <true/>
</dict>
```

## Examples

Some examples of using the SDK are available from https://github.com/Adnuntius/ios_sdk_examples
