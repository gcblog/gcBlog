---
title: Swift小结
date: 2016-11-29 23:30:40
categories:
  - Swfit
tags:
	- Swift
	- 单例
	- 懒加载
	- extension
	- 网络请求
---

### 懒加载
- 格式 `lazy var 变量: 类型 = { 创建变量代码 }()`
- 以 `lazy var` 开头，闭包末尾跟一个'()'
- 	懒加载的写法本质上是定义并执行一个闭包
-  好处：没有解包的麻烦，并且 延迟创建
-  与OC懒加载的区别：这里的懒加载只会执行一次，如果中途被设置成nil，也不会再次执行
懒加载完整的写法：
eg:懒加载一个数组

```
lazy var dataList: [String] = { () -> [String] in
        return ["zero","one", "two"]
}()
```

in 和 前面的代码块可以省略，写成这样

```
lazy var dataList: [String] = {
        return ["zero","one", "two"]
}()
```

后面的闭包和括号也能省略,这种写法是最为简洁的懒加载形式，一般这样写就可以了

`lazy var dataList: [String] = ["zero","one", "two"]`

比如懒加载一个label

`lazy var label: UILabel = UIlabel()`

如果对label需要添加其他属性，就可以写成带（）的

```
lazy var label :UILabel = {
       let label = UILabel()
        label.font = UIFont.systemFont(ofSize: 15)
        label.textColor = UIColor.red
        return label
}()
```
<!--more-->
### extension
swift中一个类只有一个.m文件，所有代码都会写在.m中，代码多了，难免就会混乱，`extension` 就是用来隔离代码用的。
extension最常用的几个地方：
#### 代理方法
```
extension HomeViewController : UITableViewdelegate,UItableViewDataSource {
}

```
#### 对类的扩展
```
import UIKit

//对UIBarButtonItem UIbarButton 的一个扩展
extension UIBarButtonItem {
    // 便利构造函数
    convenience init(imageName: String, highImageName: String, size: CGSize = CGSize.zero) {
        
        let btn = UIButton();
       
        btn.setImage(UIImage(named: imageName), for: UIControlState())
        if(highImageName != "") {
            btn.setImage(UIImage(named: highImageName), for: .highlighted)
        }
        
        if size == CGSize.zero {
            btn.sizeToFit()
        }else {
            btn.frame = CGRect(origin: CGPoint.zero, size: size)
        }
        self.init(customView: btn)
    }
}
```
#### 本类中的私有方法
extension 外如果想调用extension 内的私有方法，需要加上`fileprivate`
```
// MARK: - 设置UI界面
extension HomeController {
    fileprivate func setupUI() {
        setupNavgationbar()
    }
    private func setupNavgationbar() {
   self.navigationItem.leftBarButtonItem = UIBarButtonItem(imageName:"logo")
           
}

```

### Swift宏定义
Swift中没有宏的存在，但是有其他实现的方法。而且Swift共享整个命名空间，不在需要.pch文件。新建一个Swfit file文件，最好导入UIKit框架，没有参数的宏直接使用常量定义即可，有参数的宏使用函数代替

#### 无参数的宏
```
//oc中的宏定义
#define kIOS7   [UIDevice currentDevice].systemVersion.doubleValue>=7.0 ? 1 :0
#define kIOS8   [UIDevice currentDevice].systemVersion.doubleValue>=8.0 ? 1 :0
#define kScreenHeight     [UIScreen mainScreen].bounds.size.height
#define kScreenWidth      [UIScreen mainScreen].bounds.size.width
//转换成Swift的写法
let kIOS7 = Double(UIDevice().systemVersion)>=7.0 ? 1 :0
let kIOS8 = Double(UIDevice().systemVersion)>=8.0 ? 1 :0

let kScreenHeight = UIScreen.mainScreen().bounds.size.height
let kScreenWidth = UIScreen.mainScreen().bounds.size.width

```
#### 有参数的宏

```
//oc写法
#define RGBCOLOR(r,g,b) [UIColor colorWithRed:(r)/255.0 green:(g)/255.0 blue:(b)/255.0 alpha:1]
//Swift中的写法
func RGBCOLOR(r:CGFloat,_ g:CGFloat,_ b:CGFloat) -> UIColor
{
    return UIColor(red: (r)/255.0, green: (g)/255.0, blue: (b)/255.0, alpha: 1.0)
}

```

### Swift中weakSelf 的 写法

#### 第一种 类似OC的写法
这里只能用 `var` 修饰
```
weak var weakSelf = self
loadData { (result) in
    print(weakSelf?.view)
}
```

#### 第二种 Swift推荐的方式
```
loadData { [weak self] (result) in
     print(self?.view)
}
```
#### 第三种 不推荐使用
[unowned self] 表示{} 中的 所有self 都是 assgined， 不会强引用，但是对象释放，指针地址不会变化 
```
//相当于 OC  __unsafe_unretain 
// 如果对象被释放，继续调用，就会出现野指针,有极大的安全隐患
loadData { [unowned self] (result) in
		print(self.view) 
}
```

### 单例
Swift中单例写法较为简单，并且线程安全
```
static let shared: XCRequest = {
     // 实例化对象
     let instance = XCRequest()
     // 返回对象
     return instance
 }()
```
还有另外一种写法
```
static let instance: XCRequest = XCRequest()
    
      class func shareManager() ->XCRequest {
        
      return instance  
}
```

### getter&setter

模仿OC的写法，事实上Swift不会这么写
```
var _name: String?

var name: String? {
    get {
        return _name
    }
    set {
        _name = newValue
    }
}

      var age:Int{
        // 如果只重写了get,没有set. 那么属性是一个"计算型"属性
        // 计算型属性不占用存储空间, 相当于OC中的readOnly
        get{
            return 30
        }
    }
   
    // 如果只有get可以简写为
    var gender:String{
        return "lnj" 
    }

```
#### 计算型属性
只有getter，没有setter的属性被称为计算型属性
```
var title: String { 
		get { 
			return "Mr " + (name ?? "") 
		} 
}
```

- 只实现 getter 方法的属性被称为计算型属性，等同于 OC 中的 ReadOnly 属性
- 计算型属性本身不占用内存空间
- 不可以给计算型属性设置数值

计算型属性可以使用以下代码简写:
```
var title: String { 
	return "Mr " + (name ?? "") 
}
```

#### didSet
OC中最常见的就是重写setter方法，然后同时做一些额外的事情，那么Swift中就用didSet来实现

```
var room_list : [[String : NSObject]]? {
        didSet {
            guard let room_list = room_list else { return }
            for dict in room_list {
                anchors.append(AnchorModel(dict: dict))
            }
        }
    }
```
### 构造函数，析构函数
先了解两个概念
#### 方法重载：
	•	函数名相同，参数名／参数类型／参数个数不同
	•	重载函数并不仅仅局限于构造函数
	•	函数重载是面相对象程序设计语言的重要标志
	•	OC 不支持函数重载，OC 的替代方式是 withXXX…
#### 方法重写：
	•	也叫覆盖，指在子类中定义一个与父类中方法同名同参数列表的方法。
	•	重写父类方法需要加override
	•	重写是子类的方法覆盖父类的方法，要求方法名和参数都相同
	•	因为子类会继承父类的方法，而重写就是将从父类继承过来的方法重新定义一次，重新填写方法中的代码。
	•	重写必须继承，重载不用
	
#### 构造函数

```
class Person: NSObject {

    var name: String
    
    ///最简单的必选属性的构造函数
    ///构造函数的目的，给自己分配空间并设置初始值
    ///属性的初始化放在super.init前面
    /// 重写父类方法需要加override
    override init () {
        
        name = "default name:Tom"
        
        super.init()
    }
    
    /// 如果实现了 构造函数的重载，并且没有重写构造函数，那么系统不再提供原始的init()函数
    /// 方法重载（类似OC自定义初始化方法）
    init(name: String) {
        self.name = name
        super.init()
    }
    
}
```
#### 析构函数
相当于OC中的dealloc方法
```
deinit {
    print("被释放了")
}
```

### guard let & if let 
相当于OC中用if来判断某个值是不是为空
我认为这个语法最大的好处是避免了写大量的 ？！，
用来判断的这个属性必须是可选的
guard 与 if 的区别是 guard只有在条件不满足的时候才会执行这段代码
```
guard let _:String = pe.name else {
		return
}
```

如果要判断多个参数,一直在后面加
```
guard let _ = pe.name, let _ = pe.title else {
            return
}
```

if let 可以在条件成立或者不成立的情况下，在{}中分别处理
```
if let name = pe.name {
            print(name)
     } else {
            return
}
```

### as
as 的三种情况

#### as？ 
	1.	前面的返回值是可选的
	2.	guard let / if let 一定用 as?
#### as! 
	1.	前面的返回值一定有值
#### as 
	1.	NSString -> String
	2.	NSArray ->[ ]
	3.	NSDictionary -> [ ]
	4.	这种情况是因为底层做了结构体和OC对象的桥接

### try 处理错误异常

```
let jsonSTring = "{\"name\": \"zhang\"]"
let data = jsonSTring.data(using: .utf8)
//方法1. 推荐，如果解析成功就有值，否则 为nil
let json1 = try? JSONSerialization.jsonObject(with: data!, options: [])
print(json1 ?? "json1 为 nil")

//方法2. 墙裂不推荐 如果解析成功就有值，否则crash
//let json2 = try! JSONSerialization.jsonObject(with: data!, options: [])

//方法3. 异常处理,能够接受错误，可以在错误的情况下另行处理
do {
    let json3 = try JSONSerialization.jsonObject(with: data!, options: [])
    print(json3)
} catch {
    print(error);
}

```

### 网络请求
#### GET

```
func getWithPath(path: String,param: Dictionary<String,Any>?,completion: @escaping ((_ result: Any?, _ success:Bool) -> ())) {
        
        let url = URL(string: path.addingPercentEncoding(withAllowedCharacters: CharacterSet.urlQueryAllowed)!)
        if let para = param {
            //对参数进行处理
            print(para)
        } else {
            
        }
        let session = URLSession.shared
        
        let dataTask = session.dataTask(with: url!) { (data, respond, error) in
            
            if let data = data {
                
                if let result = try? JSONSerialization.jsonObject(with: data, options: .allowFragments){
                    
                    completion(result,true)
                }
            }else {
                
                completion(error,false)
            }
        }
        dataTask.resume()
    }
```

#### POST 

```
func postWithPath(path: String,paras: Dictionary<String,Any>?,success: @escaping ((_ result: Any) -> ()),failure: @escaping ((_ error: Error) -> ())) {
        
        let url = URL(string: path)
        var request = URLRequest.init(url: url!)
        request.httpMethod = "POST"
        print(path)
        request.httpBody = path.data(using: .utf8)
        let session = URLSession.shared
        let dataTask = session.dataTask(with: request) { (data, respond, error) in
            
            if let data = data {
                
                if let result = try? JSONSerialization.jsonObject(with: data, options: .allowFragments) {
                    
                    success(result)
                }
                
            }else {
                failure(error!)
            }
        }
        dataTask.resume()
    }
```


