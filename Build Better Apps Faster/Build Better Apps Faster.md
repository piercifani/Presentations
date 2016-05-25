Theme: Ostrich

# Build<br/>Apps 

^ Recall how they wanted you to make an app with calendar, a photo gallery, a chat for nickles. Or how "a simple App like Instagram"

---
# Build **Better**<br/>Apps **Faster**
### Pierluigi Cifani

---

## Software has always been about building **abstractions**
---

### Bit -> Byte

### Byte -> Assembly

### Assembly -> C

### C -> Objective C

^ We went from the bit, which is the software equivalent of a 5V/0V signal, to the byte, because it was easier to represent words and numbers with it. Then assembly helped us make operations with words with bytes easier, Then C came along, a lot easier because we didn't have to learn about memory registers, and allowed us to split our code in different functions. The Smalltalk branch of Software Engineering created ObjC in order to compose problems as the interaction of objects, passing messages to each others to complete the problem 

---
## Objective C -> **Swift**

---

### **Swift** enables us to build **abstractions** that are closer to our **problem domain**


^ And this is one of the reasons for Swift: solve the required business problems using composability of different types 

---

# What will we build?

1. Generic `UICollectionViewDataSource` 

2. Easy to reason networking

3. `¯\_(ツ)_/¯`

---

### **1.** Generic `UICollectionViewDataSource`

---

![fit](https://i.imgur.com/gUyyiTs.png) 
![fit](https://i.imgur.com/k336ijZ.png) 
![fit](https://i.imgur.com/VwDrXRW.png)

---

![fit](https://i.imgur.com/WzjI2He.png) 
![fit](https://i.imgur.com/5ATQLlf.png) 
![fit](https://i.imgur.com/LpKpDMB.png)

^ If we only try to build a generic CollectionViewDataSource, we'll be falling short 

---

### Generic **stateful**
### `UICollectionViewDataSource`

---

## Swift == stateless


## 🤔

---

### In Swift you can be more **explicit** about the state

---

## In any `UITableViewDataSource`

``` objectivec

- (void)requestWithCompletionBlock:(AAPLCompletionBlock)block
{
    self.executingRequest = YES;
    
    [self showLoadingView];
    
    [self requestUsersWithPage:self.currentPagination completionBlock:^(NSArray *array, NSError *error) {
    	if (error) {
    		[self showError];
    	} else {
    		self.array = array;
    		[self.tableView reloadData];
    	}
    
  	    self.executingRequest = NO;

        block(array, error);
    }];
}
```
---

```swift

public enum ListState<T> {
    case Loading
    case Loaded(data: [T])
    case Failure(error: ErrorType)
}
```

---

```swift

private func fetchData() {
    
    collectionView.state = .Loading
    
    searchUserInteractor.fetchLocationAndUsers() { [weak self] result in
        
        guard let strongSelf = self else { return }
        
        switch result {
        case .Failure(let error):
            strongSelf.collectionView.state = .Failure(error: error)
        case .Success(let users):
            strongSelf.collectionView.state = .Loaded(data: users)
        }
    }
}
```

---

However, last time I tried to do 

`collectionView.state = .Loading`

this happened:

`error: value of type 'UICollectionView' has no member 'state'
        collectionView.state = .Loading
        ^~~~~~~~~~~~~~ ~~~~~`

# 😭😭😭😭😭😭

---

# Apple provides 3 points of customization:

1. Subclassing
2. `UICollectionViewFlowLayout`
3. `UICollectionViewDataSource`

---

# Apple provides 3 points of customization:

1. ~~Subclassing~~
2. ~~`UICollectionViewFlowLayout`~~
3. `UICollectionViewDataSource` ✅

---

## Objectives:

1. Enforce MVVM[^1]
2. Map one model object to one cell kind **at compile time**
3. Support for `ListState`
4. Customization for the `.Loading` and `.Error` state
5. Independence from any Layout

[^1]: The one good thing that came out of [Microsoft](https://msdn.microsoft.com/en-us/library/hh848246.aspx).

---

# Enforce MVVM

```swift

public protocol ViewModelConfigurable {
    associatedtype VM
    func configureFor(viewModel viewModel: VM) -> Void
}

public protocol ViewModelReusable: ViewModelConfigurable {
    static var reuseType: ReuseType { get }
    static var reuseIdentifier: String { get }
}

public enum ReuseType {
    case NIB(UINib)
    case ClassReference(AnyClass)
}

```
---

```swift
//MARK:- Extensions

extension ViewModelReusable where Self: UICollectionViewCell {
    public static var reuseIdentifier: String {
        return NSStringFromClass(self)
    }
    
    public static var reuseType: ReuseType {
        return .ClassReference(Self)
    }
}
```

---
## Map one model object to one cell kind **at compile time**

---

```swift

class CollectionViewStatefulDataSource <Model, Cell:ViewModelReusable
				where Cell:UICollectionViewCell	> {

	typealias ModelMapper = (Model) -> (Cell.VM)

    weak var collectionView: UICollectionView?
    let mapper: ModelMapper	
    var state: ListState<Model> {
        didSet {
            collectionView?.reloadData()
        }
    }			
}
```

---

![](http://i.giphy.com/13HgwGsXF0aiGY.gif)

---

# Possible improvements:
- Pull to refresh
- Inifite scrolling
- Support for `n` sections

---

### **2.** Easy to reason networking

---

![fit](https://i.imgur.com/iThwz18.png)

---

![fit](https://i.imgur.com/ganJa2m.png)

---

```objectivec
	ConfigurationLogic *logic = [ConfigurationLogic currentConfiguration];
	
    [NetworkFetcher callService:@"/users"
                           host:[logic baseURL]
                         method:GET
                       postData:nil
                     bodyInJSON:NO
                       hasCache:YES
                timeoutInterval:[logic servicesTimeOut]
                 successHandler:successHandler
                 failureHandler:failureHandler];
```

---

```swift
    func lookForStockForSymbols (symbol : String, handler : FetchSymbolHandler?) -> StockFetcherOperation {

        let parameters = [
            "input" : symbol
        ]
        
        let request = networkFetcher.request(
            .GET,
            "http://dev.markitondemand.com/Api/v2/Lookup/json",
            parameters:parameters ,
            encoding:.URL
        )
        
        request.responseJSON { (_, _, operationResult) -> Void in
            /**
             Some code
             */
        }
        
        return StockFetcherOperation(request: request)
    }
    
```

---

## Objectives:

1. Composability
2. Sensitive default values
3. Readability
4. Self documented

---

![right](http://i.giphy.com/ujUdrdpX7Ok5W.gif)

# More protocols

---

```swift
/**
 Protocol used to describe what is needed
 in order to send REST API requests.
*/
public protocol Endpoint {
    
    /// The path for the request
    var path: String { get }
    
    /// The HTTPMethod for the request
    var method: HTTPMethod { get }
    
    /// Optional parameters for the request
    var parameters: [String : AnyObject]? { get }
    
    /// How the parameters should be encoded
    var parameterEncoding: HTTPParameterEncoding { get }
    
    /// The HTTP headers to be sent
    var httpHeaderFields: [String : String]? { get }
}
```

---
```swift

//  This is the default implementation for Endpoint 
extension Endpoint {
    public var method: HTTPMethod {
        return .GET
    }
    
    var parameters: [String : AnyObject]? {
        return nil
    }
    
    public var parameterEncoding: HTTPParameterEncoding {
        return .URL
    }
    
    public var httpHeaderFields: [String : String]? {
        return nil
    }
}

```

---

```swift
enum AuthenticationAPI: Endpoint {
    case LoginFacebook(token: String)
}
```
    
---

```swift
extension AuthenticationAPI: Endpoint {
    var path: String {
        switch self {
        case .LoginFacebook(_):
            return "/authentication"
        }
    }
    var method: HTTPMethod {
        switch self {
        case .LoginFacebook(_):
            return .POST
        }
    }
    var parameters: [String : AnyObject]? {
        switch self {
        case .LoginFacebook(let token):
            return [
                "token" : token
            ]
        }
    }
}
```

---

```swift
enum UsersAPI {
    case Users(SearchFilter)
    case UserDetails(UUID)
    case UserPictures(UUID)
}
```

---

```swift
extension UsersAPI: Endpoint {
    public var path: String {
        switch self {
        case .Users(_):
            return "/users"
        case .UserDetails(let uuid):
            return "/users/\(uuid)"
        case .UserPictures(let uuid):
            return "/users/\(uuid)/pictures"
        }
    }

    public var parameters: [String : AnyObject]? {
        switch self {
        case .Users(let filter):
            var params = [String : AnyObject]()
            params["min_age"] = filter.minAge
            params["max_age"] = filter.maxAge
            params["min_dist"] = filter.minDistance
            params["max_dist"] = filter.maxDistance
            params["lat"] = filter.latitude
            params["long"] = filter.longitude
            return params
        default:
            return nil
        }
    }
}
```

---
> "Applications interacting with web services in a significant manner are encouraged to have custom types conform to `URLRequestConvertible` as a way to ensure consistency of requested endpoints."

-- @mattt[^2]

[^2]: Alamofire's [Readme](https://github.com/Alamofire/Alamofire).

---

```swift
/**
    Types adopting the `URLRequestConvertible` protocol can be used to construct URL requests.
*/
public protocol URLRequestConvertible {
    /// The URL request.
    var URLRequest: NSMutableURLRequest { get }
}

protocol Endpoint: URLRequestConvertible {
	
	/* The other stuff */
	
    var URLRequest: NSMutableURLRequest {
        let request = NSMutableURLRequest(URL: NSURL(string: path)!)
        request.HTTPMethod = method.rawValue
        request.allHTTPHeaderFields = httpHeaderFields
        let requestTuple = parameterEncoding.encode(request, parameters: parameters)
        return requestTuple.0
    }
}

```

---

# Possible improvements:
- Multiple environments
- Authentication tokens
- Response parsing


---

## Build abstractions **on top** of the problems we're solving **over and over**
---

![fit](https://i.imgur.com/SvletJq.png)

---

# Available for:
- Consulting
- Auditing
- Training

---

# Thank you!
### @piercifani
### pcifani@blurredsoftware.com
