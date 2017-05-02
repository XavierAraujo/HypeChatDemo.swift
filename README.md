
![alt tag](https://hypelabs.io/static/img/NQMAnSZ.jpg)
![alt tag](https://hypelabs.io/static/img/logo200x.png)

[Hype](http://hypelabs.io/?r=9) is an SDK for cross-platform peer-to-peer communication with mesh networking. Hype works even without Internet access, connecting devices via other communication channels such as Bluetooth, Wi-Fi direct, and Infrastructural Wi-Fi.

The Hype SDK has been designed by [Hype Labs](http://hypelabs.io/?r=9). It is currently in private Beta for all Apple platforms (iOS, macOS, and tvOS), [Android](https://github.com/Hype-Labs/HypeChatDemo.android), and [Universal Windows Platform](https://github.com/Hype-Labs/HypeChatDemo.uwp). For Apple platforms it's also available for [Objective-C](https://github.com/Hype-Labs/HypeChatDemo.ios).

You can start using Hype today, [join the beta by subscribing on our website](https://hypelabs.io/?r=15).

## What does it do?

This project consists of a chat app sketch written to illustrate how to work with the SDK in Swift. The app displays a list of devices in close proximity, which can be tapped for exchanging text content. The SDK itself allows sending other kinds of media, such as pictures, or video, but the demo is limited for simplicity purposes.

Most of the documentation is inline with the code, and further information can be found on the Hype Labs [official documentation site](https://hypelabs.io/docs/).

## Setup

To run the project you'll need the Hype SDK binary. To access it, subscribe on the Hype Labs [website](https://hypelabs.io/?r=15) to get early access to the SDK. Then all you need is to drag the binary to the root folder and the demo is ready to go.

## Overview

Most of the action takes place in `ContactViewController`. There you'll learn about Hype's lifecycle, setup, and maintenance in around 20 lines of code. The `ChatViewController` shows how to send and receive messages. Refer to the inline documentation for more details.

#### 1. Download the Hype SDK

The first thing you need is the Hype SDK binary. Subscribe for the Beta program at the Hype Labs [website](http://hypelabs.io/downloads/) and follow the instructions from your inbox. You'll need your subscription to be activated before proceeding.

#### 2. Add the SDK to your Xcode project

Hype is really easy to configure! Open the project with Xcode and drag the binary into the project in the Project Navigator. Also see [Apple's documentation page](https://developer.apple.com/library/ios/recipes/xcode_help-structure_navigator/articles/Adding_a_Framework.html) with details and alternative solutions. Some versions of Xcode require adding the framework to Embedded Binaries in the project's General configurations.

#### 3. Register an app

Go to [the apps page](http://hypelabs.io/apps) and create a new app by pressing the _Create new app_ button on the top left. Enter a name for your app and press Submit. The app dialog that appears afterwards yields a 8-digit hexadecimal number, called a _realm_. Keep that number for step 4. Realms are a means of segregating the network, by making sure that different apps do not communicate with each other, even though they are capable of forwarding each other's contents. If your project requires a deeper understanding of how the technology works we recommend reading the [Overview](http://hypelabs.io/docs/ios/overview/) page. There you'll find a more detailed analysis of what realms are and what they do, as well as other topics about the Hype framework.

#### 4. Setup the realm

The realm must be set in the project's Info.plist file or when starting the Hype services. You'll find your Info.plist in Project Navigator under the Supporting Files group. If you are not sure how to edit your Info.plist, we recommend reading Apple's documentation [here](https://developer.apple.com/library/ios/documentation/General/Reference/InfoPlistKeyReference/Articles/AboutInformationPropertyListFiles.html) for that regard. Add a new top level entry on your Info.plist called _com.hypelabs.hype_ and set its type to _Dictionary_. Expand the entry by clicking on the arrow on the left and replace the _New item_ text with _realm_ (all lower case). Set its type to _String_ and the value to the realm identifier that you got from step 3. Alternatively, you can set the `HYPOptionRealmKey` when starting the framework's services with an `NSString` value indicating the realm. The following example illustrates how to do this. The 00000000 realm is reserved for testing purposes and apps should not be deployed with this realm. Also, setting the realm with `start(options:)` takes precedence over the realm read from the Info.plist file.

```objc
    HYP.instance().start(options: [HYPOptionRealmKey:"00000000"])
```

#### 5. Start Hype services

It's time to write some code! Have the view controller of your choice implement the observer protocols (HYPStateObserver, HYPNetworkObserver, and HYPMessageObserver) and registering itself as an observer. After that, it's time to start the Hype services. Like so:

```objc

class ContactViewController:  UITableViewController, HYPStateObserver, HYPNetworkObserver, HYPMessageObserver {

    override func viewDidLoad() {
        super.viewDidLoad()

        // Request Hype to start when the view loads. If this is successful the table
        // should start displaying other peers soon. Notice that it's OK to request the
        // framework to start if it's already running, so this method can be called
        // several times.
        requestHypeToStart()
    }

    func requestHypeToStart() {

        // notifications for lifecycle events being triggered by the Hype framework. These
        // events include starting and stopping, as well as some error handling.
        HYP.instance().add(self as HYPStateObserver)

        // Network observer notifications include other devices entering and leaving the
        // network. When a device is found all observers get a -hype:didFindInstance:
        // notification, and when they leave -hype:didLoseInstance:error: is triggered instead.
        HYP.instance().add(self as HYPNetworkObserver)

        // Message notifications indicate when messages are sent (not available yet) or fail
        // to be sent. Notice that a message being sent does not imply that it has been
        // delivered, only that it has left the device. If considering mesh networking,
        // in which devices will be forwarding content for each other, a message being
        // means that its contents have been flushed out of the output stream, but not
        // that they have reached their destination. This, in turn, is what acknowledgements
        // are used for, but those have not yet available.
        HYP.instance().add(self as HYPMessageObserver)

        // Requesting Hype to start is equivalent to requesting the device to publish
        // itself on the network and start browsing for other devices in proximity. If
        // everything goes well, the -hypeDidStart: delegate method gets called, indicating
        // that the device is actively participating on the network. The 00000000 realm is
        // reserved for test apps, so it's not recommended that apps are shipped with it.
        // For generating a realm go to https://hypelabs.io, login, access the dashboard
        // under the Apps section and click "Create New App". The resulting app should
        // display a realm number. Copy and paste that here.
        HYP.instance().start(options: [HYPOptionRealmKey:"00000000"])
    }

    func hypeDidStart(_ hype: HYP!) {

        // At this point, the device is actively participating on the network. Other devices
        // (instances) can be found at any time and the domestic (this) device can be found
        // by others. When that happens, the two devices should be ready to communicate.
        print("Hype started!")
    }

    func hype(_ hype: HYP!, didFind instance: HYPInstance!) {

        DispatchQueue.main.async {

            // Hype instances that are participating on the network are identified by a full
            // UUID, composed by the vendor's realm followed by a unique identifier generated
            // for each instance.
            print("Found instance: \(instance.stringIdentifier)")

            // Instances should be strongly kept by some data structure. Their identifiers
            // are useful for keeping track of which instances are ready to communicate.
            self.stores.updateValue(Store (instance: instance), forKey: instance.stringIdentifier)

            // Reloading the table reflects the change
            self.tableView.reloadData()
        }
    }
}
```

This code demonstrates how to add an instance as an Hype observer and starting the framework's services. Observers get notifications from the framework indicating when events happen, such as Hype's lifecycle and receiving messages from other devices. Remember, you still need to implement the methods defined by the protocol. That is, your view controller should implement methods such as `hype(_ hype: HYP!, didReceive message: HYPMessage!, from fromInstance: HYPInstance!)`, `hype(_ hype: HYP!, didFind instance: HYPInstance!)`, and so on, otherwise you'll get compiler warnings and no notifications at all. The last line in `viewDidLoad()` requests Hype to start it's services, by publishing the device on the network and browsing for other devices in proximity. At this point, the framework is still not actively participating on the network, though. Only when the observer method `hypeDidStart(_ hype: HYP!)` is called is the device actively participating on the network. After that happens, instances can be found at any time if other devices are in proximity. When that happens, the framework triggers a `hype(_ hype: HYP!, didFind instance: HYPInstance!)` notification, indicating that another peer is ready to communicate. You should keep found instances in a dictionary, as you'll need those later to exchange content. Here's one way how that could be accomplished, while expanding on the previous example:


```objc
class ContactViewController:  UITableViewController, HYPStateObserver, HYPNetworkObserver, HYPMessageObserver {

    func hype(_ hype: HYP!, didFind instance: HYPInstance!) {

        DispatchQueue.main.async {

            // Hype instances that are participating on the network are identified by a full
            // UUID, composed by the vendor's realm followed by a unique identifier generated
            // for each instance.
            print("Found instance: \(instance.stringIdentifier)")

            // Instances should be strongly kept by some data structure. Their identifiers
            // are useful for keeping track of which instances are ready to communicate.
            self.stores.updateValue(Store (instance: instance), forKey: instance.stringIdentifier)

            // Reloading the table reflects the change
            self.tableView.reloadData()
        }
    }

    func hype(_ hype: HYP!, didLose instance: HYPInstance!, error: Error!) {

        DispatchQueue.main.async {

            // An instance being lost means that communicating with it is no longer possible.
            // This usually happens by the link being broken. This can happen if the connection
            // times out or the device goes out of range. Another possibility is the user turning
            // the adapters off, in which case not only are all instances lost but the framework
            // also stops with an error.
            print("Lost instance: \(instance.stringIdentifier) [\(String(describing: error.localizedDescription))]")

            // Cleaning up is always a good idea. It's not possible to communicate with instances
            // that were previously lost.
            self.stores.removeValue(forKey: instance.stringIdentifier)

            // Reloading the table reflects the change
            self.tableView.reloadData()
        }
    }
}
```

### 6. Sending messages

Sending messages is also very simple. All it takes is a previously found instance and a couple lines of code.

```objc
@IBAction func didTapSendButton(_ sender: Any) {

    let text: String = textView.text!

    if (text.characters.count ) > 0 {

        // When sending content there must be some sort of protocol that both parties
        // understand. In this case, we simply send the text encoded in UTF8. The data
        // must be decoded when received, using the same encoding.
        let data: Data? = text.data(using: String.Encoding.utf8)

        let message: HYPMessage? = HYP.instance().send(data, to: store?.instance)

        // Clear the input view
        textView.text = ""

        // Adding the message to the store updates the table view
        store?.add(message!)
    }
}
```

Finally, messages are received by all `HYPMessageObserver` instances actively observing framework events, which, in this case, is the `ContactViewController`.

```objc
func hype(_ hype: HYP!, didReceive message: HYPMessage!, from fromInstance: HYPInstance!) {

    DispatchQueue.main.async {

        print("Got a message from: \(fromInstance.stringIdentifier)")

        let store = self.stores[fromInstance.stringIdentifier]

        // Storing the message triggers a reload update in the chat view controller
        store?.add(message)

        // The data is reloaded so the green circle indicator is shown for contacts that have new
        // messages. Reloading is probably an overkill, but the point is to maintain focus on how
        // the framework works.
        self.tableView.reloadData()
    }
}

func hype(_ hype: HYP!, didFailSendingMessage messageInfo: HYPMessageInfo!, to toInstance: HYPInstance!, error: Error!) {

    // Sending messages can fail for a lot of reasons, such as the adapters
    // (Bluetooth and Wi-Fi) being turned off by the user while the process
    // of sending the data is still ongoing. The error parameter describes
    // the cause for the failure.
    print("Failed to send message: \(UInt(messageInfo.identifier)) [\(error.localizedDescription)]")
}

func hype(_ hype: HYP!, didSendMessage messageInfo: HYPMessageInfo!, to toInstance: HYPInstance!, progress: Float, complete: Bool) {

    // A message being "sent" indicates that it has been written to the output
    // streams. However, the content could still be buffered for output, so it
    // has not necessarily left the device. This is useful to indicate when a
    // message is being processed, but it does not indicate delivery by the
    // destination device.
    print("Message being sent: \(progress)")
}

func hype(_ hype: HYP!, didDeliverMessage messageInfo: HYPMessageInfo!, to toInstance: HYPInstance!, progress: Float, complete: Bool) {

    // A message being delivered indicates that the destination device has
    // acknowledge reception. If the "done" argument is true, then the message
    // has been fully delivered and the content is available on the destination
    // device. This method is useful for implementing progress bars.
    print("Message being delivered: \(progress)")
}
```

In this case messages are just being stored for later processing, which happens in the `ChatViewController` when the messages are being displayed. Notice that the encoding used is the same both when sending and when receiving. The protocol must be the same on both ends of the link.

## License

This project is MIT-licensed.

## Follow us

Follow us on [twitter](http://www.twitter.com/hypelabstech) and [facebook](http://www.facebook.com/hypelabs.io). We promise to keep you up to date!
