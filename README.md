# ObservableUserDefaults

Original: https://stackoverflow.com/a/54889233/5227717

These extensions make `UserDefaults` observable and easy to use.


## What's wrong?

See documentation of `UserDefaults.didChangeNotification`

> ### Summary
> 
> Posted when user defaults are changed within the current process.
> 
> 
> ### Declaration
> 
> ```swift
> public class let didChangeNotification: NSNotification.Name
> ```
> 
> 
> ### Discussion
> 
> This notification is posted on the thread that changes the user defaults. The notification object is the UserDefaults object. The notification **doesn't contain a userInfo dictionary**.
> This notification **isn't posted when changes are made outside the current process**, or when ubiquitous defaults change. You can use key-value observing to register observers for specific keys of interest in order to be notified of all updates, regardless of whether changes are made within or outside the current process.
> 

## Example & Usage


### Define Keys

```swift
// IMPORTANT: DON'T use DOT `.` in key.
// DOT `.` used to define `KeyPath` and this is what we don't need here.
extension UserDefaults.Key {
    static let AppleMomentumScrollSupported  : UserDefaults.Key  = "AppleMomentumScrollSupported"     // Bool
    static let enableNetworkDiscovery        : UserDefaults.Key  = "defaults:enableNetworkDiscovery"  // Bool
    static let usesDetailedToolTips          : UserDefaults.Key  = "defaults:usesDetailedToolTips"    // Bool
    static let screenshotSavingPath          : UserDefaults.Key  = "defaults:screenshotSavingPath"    // String
}
```


### Register Initial Values

```swift
// From Source
var initialValues: [UserDefaults.Key: Any?] = [
    .enableNetworkDiscovery  : true,
    .usesDetailedToolTips    : false,
    .screenshotSavingPath    : FileManager.default
        .urls(for: .picturesDirectory, in: .userDomainMask)
        .first?.appendingPathComponent("JSTColorPicker").path,
    .pixelMatchAAColor       : NSColor.systemYellow,
    .pixelMatchDiffColor     : NSColor.systemRed,
]

// From Property List
(try?
    PropertyListSerialization.propertyList(
        from: Data(contentsOf: Bundle.main.url(forResource: "InitialValues", withExtension: "plist")!),
        options: [],
        format: nil
    )
    as? [String : Any?])?.forEach({ initialValues[UserDefaults.Key(rawValue: $0.key)] = $0.value })

UserDefaults.standard.register(defaults: initialValues)
```


### Access Direct Values

```swift
let discoveryEnabled: Bool = UserDefaults.standard[.enableNetworkDiscovery]

// Access Optional Values
let savingPath: String? = UserDefaults.standard[.screenshotSavingPath]
guard let savingPath: String = UserDefaults.standard[.screenshotSavingPath] else { return }
```


### Modify Direct Values

```swift
@IBAction func discoveryMenuItemTapped(_ sender: NSMenuItem) {
    let enabled = sender.state == .on
    sender.state = !enabled ? .on : .off
    UserDefaults.standard[.enableNetworkDiscovery] = !enabled
}
```


### Observe Changes of Single Value

```swift
// You must keep a reference to this object.
let observable = UserDefaults.standard.observe(key: .AppleMomentumScrollSupported) { (defaults, defaultKey, defaultValue: Bool) in
    let momentumScrollSupported = defaultValue
    // do something
}
```


### Observe Changes of Multiple Values

```swift
class MyClass {
    private var observables           : [Observable]?
    private var usesDetailedToolTips  : Bool = false

    private func setupHandlers() {
        observables = UserDefaults.standard.observe(
            keys: [.AppleMomentumScrollSupported, .usesDetailedToolTips], 
            callback: applyDefaults(_:_:_:)
        )
    }

    private func applyDefaults(_ defaults: UserDefaults, _ defaultKey: UserDefaults.Key, _ defaultValue: Any) {
        if defaultKey == .usesDetailedToolTips, let toValue = defaultValue as? Bool {
            if usesDetailedToolTips != toValue {
                usesDetailedToolTips = toValue
                // do something
            }
        }
    }
}
```

