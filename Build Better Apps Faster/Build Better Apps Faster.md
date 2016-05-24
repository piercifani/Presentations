Theme: Ostrich

# Build<br/>Apps 

^ Recall how they wanted you to make an app with calendar, a photo gallery, a chat for nickles. Or how "a simple App like Instagram"

---
# Build **Better**<br/>Apps **Faster**
### Pierluigi Cifani

---

## Software has always been about building **abstractions**
---

## Bit -> Byte

^ We went from the bit, which is the software equivalent of a 5V/0V signal, to the byte, because it was easier to represent words and numbers with it

---
## Byte -> Assembly

^ Then assembly helped us make operations with words with bytes easier

---
## Assembly -> C

^ Then C came along, a lot easier because we didn't have to learn about memory registers, and allowed us to split our code in different functions 

---
## C -> Objective C

^ The Smalltalk branch of Software Engineering created ObjC in order to compose problems as the interaction of objects, passing messages to each others to complete the problem 

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

### Generic
### `UICollectionViewDataSource`

---

![](https://i.imgur.com/gUyyiTs.png) 
![](https://i.imgur.com/k336ijZ.png) 
![](https://i.imgur.com/VwDrXRW.png)

---

![](https://i.imgur.com/WzjI2He.png) 
![](https://i.imgur.com/5ATQLlf.png) 
![](https://i.imgur.com/LpKpDMB.png)

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
    
    list.state = .Loading
    
    searchUserInteractor.fetchLocationAndUsers() { [weak self] result in
        
        guard let strongSelf = self else { return }
        
        switch result {
        case .Failure(let error):
            strongSelf.list.state = .Failure(error: error)
        case .Success(let users):
            strongSelf.list.state = .Loaded(data: users)
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
        return NSStringFromClass(self).componentsSeparatedByString(".").last!
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

# Thank you!
### @piercifani
### pcifani@blurredsoftware.com
