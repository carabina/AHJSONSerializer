# AHJSONSerializer

[![CI Status](http://img.shields.io/travis/AlexHmelevskiAG/AHJSONSerializer.svg?style=flat)](https://travis-ci.org/AlexHmelevski/AHJSONSerializer)
[![Version](https://img.shields.io/cocoapods/v/AHJSONSerializer.svg?style=flat)](http://cocoapods.org/pods/AHJSONSerializer)
[![License](https://img.shields.io/cocoapods/l/AHJSONSerializer.svg?style=flat)](http://cocoapods.org/pods/AHJSONSerializer)
[![Platform](https://img.shields.io/cocoapods/p/AHJSONSerializer.svg?style=flat)](http://cocoapods.org/pods/AHJSONSerializer)

AHJSONSerializer is a framework written in Swift that makes it easy for you to convert your model objects (classes and structs) to and from JSON. 

- [Features](#features)
- [The Basics](#the-basics)
- [Mapping Nested Objects](#easy-mapping-of-nested-objects)
- [Subclassing](#subclasses)
- [Generic Objects](#generic-objects)
- [To Do](#to-do)
- [Contributing](#contributing)
- [Installation](#installation)

# Features:
- Mapping JSON to objects (classes,structs)
- Mapping objects (classes,structs)  to JSON
- Nested Objects 
 
# The Basics
To support json decoding, a class or struct just needs to implement the ```JSONDecodable``` protocol:
```swift
public protocol JSONDecodable {
    init(decoder: AHJSONDecoder)
}
```

To support json encoding, a class or a struct need to implement the ```JSONEncodable``` protocol:
```swift
public protocol JSONEncodable {
    func encode(with encoder: AHJSONEncoder)
}
```
AHJSONSerializer uses the ```<~``` and ``~>``` syntaxic sugar operators to define how each member variable maps to and from JSON.

```swift
class User: JSONDecodable,JSONEncodable {
    var username: String?
    var age: Int?
    var weight: Double!
    var array: [AnyObject]?
    var dictionary: [String : AnyObject] = [:]
    var bestFriend: User?                       // Nested User object
    var friends: [User]?                        // Array of Users
    var birthday: Date?

    required init(decoder: AHJSONDecoder) {
        username    <~ decoder["username"]
        age         <~ decoder["age"]
        weight      <~ decoder["weight"]
        array       <~ decoder["arr"]
        dictionary  <~ decoder["dict"]
        bestFriend  <~ decoder["best_friend"]
        friends     <~ decoder["friends"]
        birthday    <~ decoder["birthday"]
    }
    
    func encode(with encoder: AHJSONEncoder) {
        username    ~> encoder["username"]
        age         ~> encoder["age"]
        weight      ~> encoder["weight"]
        array       ~> encoder["arr"]
        dictionary  ~> encoder["dict"]
        bestFriend  ~> encoder["best_friend"]
        friends     ~> encoder["friends"]
        birthday    ~> encoder["birthday"]
    }

}

struct Temperature: JSONDecodable {
    let celsius: Double
    let fahrenheit: Double?
    
    init(decoder: AHJSONDecoder) {
        celsius = decoder["temp_in_c"].value() ?? 0.0
        fahrenheit = decoder["temp_in_f"].value()
    }
    
}
```

Once your class implements `JSONDecodable` and `JSONEncodable`, AHJSONSerializer allows you to easily convert to and from JSON. 

Convert a JSON string to a model object:
```swift
let user = Object(json: dictionary)
```
Convert a model object to a JSON dictionary:
```swift
let json = user.json
```

AHJSONSerializer can map classes composed of the following types:
- `Int`
- `Bool`
- `Double`
- `Float`
- `String`
- `RawRepresentable` (Enums)
- `Array<AnyObject>`
- `Dictionary<String, AnyObject>`
- `Object<T: JSONDecodable>`
- `Array<T: JSONDecodable>`
- `Array<Array<T: JSONDecodable>>`
- `Set<T: JSONDecodable>` 
- `Dictionary<String, T: JSONDecodable>`
- `Dictionary<String, Array<T: JSONDecodable>>`
- Optionals of all the above
- Implicitly Unwrapped Optionals of the above

## ```JSONDecodable``` Protocol

#### ` init(decoder: AHJSONDecoder)` 
This function is where all mapping definitions should go. When parsing JSON, this function is executed during object creation. When generating JSON, it is the only function that is called on the object.

## ```JSONEncodable``` Protocol

#### ` init(decoder: AHJSONEncoder)` 
This function is where all encoding  definitions should go. The function will be used when `Object.json` will be called 

# Easy Mapping of Nested Objects
AHJSONSerializer supports dot notation within keys for easy mapping of nested objects. Given the following JSON String:
```json
"distance" : {
     "text" : "102 ft",
     "value" : 31
}
```
You can access the nested objects as follows:
```swift
func init(decoder: AHJSONDecoder) {
    distance <~ map["distance.value"]
}
```

# Custom Transforms
AHJSONSerializer supports ``map`` function so you can do crazy stuff like:
```swift
struct Car: JSONDecodable, JSONEncodable {
   let model: String 
   init(decoder: AHJSONDecoder) {
      model = decoder["model"].map(transform: transformToInt)
                              .map(transform: increase)
                              .map(transform: transformToString)
                              .value() ?? ""
   }
   
   func encode(with encoder: AHJSONEncoder) {
	  model ~> encoder["firstName"].map{uppercased}
   }
   private func uppercased(str: String) -> String {
      return str.uppercased()
   }
}

```
*map function should be strong typed (no generics)


# Subclasses

Classes that implement the ```JSONDecodable``` and ```JSONEncodable``` protocols can easily be subclassed. When subclassing mappable classes, follow the structure below:

```swift
class Base: JSONDecodable {
	var base: String?
	
	required init(decoder: AHJSONDecoder) {
        base <~ map["base"]
	}
}

class Subclass: Base {
	var sub: String?

	override init(decoder: AHJSONDecoder) {
	    super.init(decoder)
		sub <~ map["sub"]
	}
}
```

Make sure your subclass implemenation calls the right initializers and mapping functions to also apply the mappings from your superclass.

# Generic Objects

AHJSONSerializer can handle classes with generic types as long as the generic type also conforms to `JSONDecodable`. See the following example:
```swift
class Result<T: JSONDecodable>: JSONDecodable {
    var result: T?

  required init(decoder: AHJSONDecoder) {
        result <~ map["result"]
  }
}

```


# To Do
- reduce number of methods for value casting

# Contributing

Contributions are very welcome 👍😃. 

Before submitting any pull request, please ensure you have run the included tests and they have passed. If you are including new functionality, please write test cases for it as well.

# Installation
### Cocoapods
AHJSONSerializer is available through [CocoaPods](http://cocoapods.org). To install
it, simply add the following line to your Podfile:

```ruby
pod "AHJSONSerializer"
```

### Swift Package Manager
To add AHJSONSerializer to a [Swift Package Manager](https://swift.org/package-manager/) based project, add:

```swift
.Package(url: "hhttps://github.com/AlexHmelevski/AHJSONSerializer.git", majorVersion: 2, minor: 2),
```
to your `Package.swift` files `dependencies` array.



## Author

Alexei Hmelevski, alexei.hmelevski@gmail.com

## License

AHJSONSerializer is available under the MIT license. See the LICENSE file for more info.

