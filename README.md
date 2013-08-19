ShipLib iOS SDK
=====================

The Sincerely Ship SDK for iOS makes it easy to add photo postcard mailing functionality to your app. By including it in your project, your users will be able to mail a real postcard anywhere in the world at a price you set. It's designed to be turn-key: Simply pass the SDK an image, and it handles addressing & billing for you in a modal.

**Current version:** 1.4 [(download)](https://github.com/sincerely/shiplib-ios-framework/archive/master.zip)

**New in this version:**

1. Fixed crashing bug in iPad when dismissing keyboard.

2. Reduced compiled library size by 30%

## Installation

Installing Sincerely.framework is a quick and easy process.  This may be done manually or via Cocoapods.

## Installing via Cocoapods

1. Add the following to your Podfile: 

pod 'ShipLib', :git => 'https://github.com/sincerely/shiplib-ios-framework.git'

2. Select the "Build Phases" tab from your main target settings. Expand the "Copy Bundle Resources" section

3. In the Pods project, expand the Pods/ShipLib/ directory. Drag "Sincerely.framework" to the "Copy bundle Resources" section

## Manual Installation

1. Drag and drop the Sincerely.framework folder into your project folder.

2. Open the "Build Phases" tab of the iOS target you would like to include the library in.

    ![Getting to Build Phases](https://s3.amazonaws.com/sincerely-assets/shiplib-docs/install_1.png)

3. Under the "Link Binary With Libraries" section, hit the plus button. Then select "Add Other…", navigate to where you moved the Sincerely.framework folder and select the the folder named "Sincerely.framework". It should now appear in that list.

    ![Adding frameworks](https://s3.amazonaws.com/sincerely-assets/shiplib-docs/install_2.png)

4. The Sincerely.framework folder should now be in the Xcode file list on the left side. Drag and drop Sincerely.framework from the file list in the left panel of Xcode into the "Copy Bundle Resources" section.

    ![Bundle Resources](https://s3.amazonaws.com/sincerely-assets/shiplib-docs/install_3.png)

5. Under the "Link Binary With Libraries" section, hit the plus button. Add: **AddressBook.framework**, **AddressBookUI.framework**, **SystemConfiguration.framework**, and **CoreTelephony.framework**

    ![Other frameworks](https://s3.amazonaws.com/sincerely-assets/shiplib-docs/install_4.png)

6. That's it. You are now ready to integrate!

## Integration

There are three files included with the framework.

| File          | Description   |
| :------------ | :------------ |
|**Sincerely.h**| The main import file i.e. `#import <Sincerely/Sincerely.h>` |
|**SincerelyConstants.h**| Constants used in the framework |
|**SYSincerelyController.h**| The controller you will need to initiate and present |

1. Prepare a place to launch the SYSincerelyController. As an example, a selector for when a UIButton is touched.
2. Create a SYSincerelyController. Check if one was created (init did not return nil) and then present it modally. You will need to pass it an array, a product type, and an object to act as its delegate `(id <SYSincerelyControllerDelegate>)`
    - Image array: An NSArray containing *only* UIImage objects. The number of UIImage objects in the array must be the amount as required by the product type.
    - Product Type: Currently, you can only pass in the product type SYProductTypePostcard. This type requires the images array to contain one and only one UIImage.
    - Application Key: An application key generated from http://dev.sincerely.com/apps 
    - Delegate: Any object that conforms to the `SYSincerelyControllerDelegate` protocol.

    **Note:** The final image that will be sent to be printed will be 1838 x 1238 (6 inches x 4 inches with margin). The library will automatically scale down images larger than this, and offers cropping functionality. More Information

    ````objective-c
    SYSincerelyController *controller = [[SYSincerelyController alloc] initWithImages:[NSArray arrayWithObject:[UIImage imageNamed:@"demo_image.jpg"]]
                                        product:SYProductTypePostcard
                                        applicationKey:@"YOUR_APP_KEY_HERE"
                                        delegate:self];
                                                                     
    if (controller) {
        [self presentViewController:controller animated:YES completion:NULL];
        [controller release];
    }
    ````

  The biggest takeaway here is to remember that initWithImages:product:applicationKey:delegate: can return nil if the correct inputs are not given. If you then call presentViewController:animated:competion: and pass in nil, your application *will crash*.

3. Implement the SYSincerelyControllerDelegate protocol so that you can receive callbacks. Here are some example implementations:

    ````objective-c
    - (void)sincerelyControllerDidFinish:(SYSincerelyController *)controller {
        /*
        * Here I know that the user made a purchase and I can do something with it
        */
    
        [self dismissViewControllerAnimated:YES completion:NULL];
    }
 
    - (void)sincerelyControllerDidCancel:(SYSincerelyController *)controller {
        /*
         * Here I know that the user hit the cancel button and they want to leave the Sincerely controller
         */
        
        [self dismissViewControllerAnimated:YES completion:NULL];
    }
     
    - (void)sincerelyControllerDidFailInitiationWithError:(NSError *)error {
        /*
         * Here I know that incorrect inputs were given to initWithImages:product:applicationKey:delegate;
         */
        
        NSLog(@"Error: %@", error);
    } 
    ````

4. That's it! Please let us know any thoughts, questions, or feedback that you have.

## Customization

Customizing the experience to fit your application is something you should place close attention to as you integrate the Sincerely Ship Library. The `SYSincerelyController` supports a list of properties, which you can modify to tailor the experience to your application. The properties are as follows:

### shouldSkipCrop
A boolean value that indicates whether the crop screen should be skipped. Default value: NO

`@property (nonatomic, assign) BOOL shouldSkipCrop`

**Discussion**

If you are including the ship library in an application with photo editing, you may find it advantageous to skip the crop screen. By setting this property to YES, the user will never see the crop screen as part of the order process. If this property is set to YES, the image you submitted in `initWithImages:product:applicationKey:delegate:` will be center cropped (using `UIViewContentModeScaleAspectFill`) and resized to 1838 x 1238. Due to the nature of this behavior, you should strive to submit an image as close to 1838 x 1238 as possible.

Declared In: SYSincerelyController.h

### shouldSkipPersonalize

A boolean value that indicates whether the personalize screen should be skipped. Default value: NO

`@property (nonatomic, assign) BOOL shouldSkipPersonalize`

**Discussion**

By setting this property to YES, the personalize screen will not show up during the order process. This means that the user will not be able to input a message or a profile picture. However, you can still specify a message or profile photo for them by using one of the properties below. If YES, any profile photo selections will be cleared, and a profile photo will not appear on the card unless specified in profilePhoto below.

Declared In: SYSincerelyController.h

### message

A string value that will pre-populate the message for the postcard.

`@property (nonatomic, copy) NSString *message`

**Discussion**

Setting this property will pre-populate the message for the back of the postcard. The user will still be able to edit the message on the personalize screen, unless shouldSkipPersonalize is set to YES.

Declared In: SYSincerelyController.h

### profilePhoto

An image that will pre-populate the profile photo for the postcard.

`@property (nonatomic, retain) UIImage *profilePhoto`

**Discussion**

Setting this property will pre-populate the profile photo for the back of the postcard. The user will still be able to edit the profile photo on the personalize screen, unless shouldSkipPersonalize is set to YES. If you do set shouldSkipPersonalize to YES, you should always set the profile photo last. This is because setting shouldSkipPersonalize to YES will clear any profile photo selections that are currently made.

Declared In: SYSincerelyController.h

### recipients

An array of dictionaries that will pre-select recipients for the postcard.

`@property (nonatomic, retain) NSArray *recipients`

**Discussion**

Setting this value will pre-populate the recipients with the contents of this array. The contents of this array must adhere to a specific format. Each object in the array must be an NSDictionary. Each NSDictionary must only contain NSString's as its keys and values. The possible keys are as follows:

* name
* street1
* street2
* city
* state
* zipcode
* country

You must include the 4 required fields: name, street1, city, country in order for your recipient to appear in the list.
Alternatively, for US based addresses, you can submit just name, city, and zipcode and we will fill out the remaining fields for you.

Declared In: SYSincerelyController.h

**Example**

````objective-c
NSDictionary *address1 = [NSDictionary dictionaryWithObjectsAndKeys:@"Rick Harrison", @"name", 
                          @"800 Market St. Floor 6", @"street1", 
                          @"94102", @"zipcode", nil];
NSDictionary *address2 = [NSDictionary dictionaryWithObjectsAndKeys:@"Matt Brezina", @"name", 
                          @"800 Market Street Floor 6", @"street1", 
                          @"San Francisco", @"city", 
                          @"CA", @"state", 
                          @"United States", @"country", 
                          @"94102", @"zipcode", nil];

controller.recipients = [NSArray arrayWithObjects:address1, address2, nil];
````

## Supporting iOS version 3.x

The Sincerely Ship library will only run on devices running iOS 4.0 or later. If your application currently supports 3.x, there are a few steps you must follow in order to ensure that your 3.x support stays intact.

1. Weak link against UIKit:
    - Open the "Build Phases" tab of the iOS target running the library.
    - Expand the "Link Binary With Libraries" section to find the row containing UIKit.
    - Make sure the drop down on the right side of the UIKit row is set to optional.
2. Weak link against libSystem.B.dylib (Objective-C Blocks)
    - Open the "Build Settings" tab of the iOS target running the library.
    - Find the row for "Other Linker Flags"
    - Under all configurations add "-weak_library /usr/lib/libSystem.B.dylib"
3. Make sure that your entrance point to the Ship library does not run on 3.x devices. Here is some example code that only adds a button to the view if the device is running 4.0 or later.
    ````objective-c
    NSString *version = [[UIDevice currentDevice] systemVersion];
    
    if ([version compare:@"4.0" options:NSNumericSearch] != NSOrderedAscending) {
        UIButton *button = [UIButton buttonWithType:UIButtonTypeRoundedRect];
        
        // Configure button for title, action, position, etc.
        
        [self.view addSubview:button];
    }
    ````

## Troubleshooting

1. **Sincerely/Sincerely.h file not found**:
    - If you've recently upgraded and the path to Sincrely.framework changed, make sure that you completely removed the old framework. Sometimes Xcode will keep around directories inside the search paths of your project, which can cause this error.
    - Make sure your `Framework Search Paths` includes the directory root where you placed the Sincerely.framework bundle.
2. **Borders of my cards sometimes get cut off when printed.**: 
    Postcards are produced in large sheets and then cut to size. This process is computer-controlled and very accurate, but working in the physical realm requires planning for tolerance. As such, it's important to include some extra space ("bleed area") in your images, to ensure that all postcards look good when produced. That means making your borders slightly larger (1/10") than they'd appear on screen and not putting any important elements (text, etc) very near the edges.
3. **Error: Could not load NIB in bundle**:
    Make sure your "Build Phases" tab on your project's target shows Sincerely.framework under "Copy Bundle Resources" 

## Pricing

All payments are handled from within the ui. You won't need to collect payment from your users or set up SSL certificates, and we will pay you a percentage of the revenues based on what you charge your users. The base price is $0.99 for US-bound prints, and $1.99 for prints sent anywhere else. You will get 70% of the revenues of anything you charge above that; so if you choose to charge $2.99 us and $3.99 international, you'll make $1.40 per us card ($2.99 - $0.99 = $2.00 * 0.70). We pay by check or PayPal once your revenue hits $25.

## Support

If you have any problems at all getting this up and running, contact us at [devsupport@sincerely.com](mailto:devsupport@sincerely.com).

## Reporting bugs

You may report any issues with the framework or documentation [here](https://github.com/sincerely/shiplib-ios-framework/issues).





