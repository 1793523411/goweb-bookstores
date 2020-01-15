# goweb-书城项目

## 设置处理静态资源，如 css 和 js 文件

`http.Handle("/static/", http.StripPrefix("/static/", http.FileServer(http.Dir("views/static"))))`

这里统一处理了静态资源，改变原前端页面中相对路径，变为以项目根目录开始 view 开始，使客户端可以访问到服务端的资源

**直接去 html 页面**

`http.Handle("/pages/", http.StripPrefix("/pages/", http.FileServer(http.Dir("views/pages"))))`

同上，处理了页面的路径跳转

**去首页**

`http.HandleFunc("/main", controller.GetPageBooksByPrice)`

主页面的路由

## userhander.go

### 去登录

`http.HandleFunc("/login", controller.Login)`
实现了登录功能，获取前端页面的输入，对输入的用户名和密码进行查询，数据库中是否存在，并是否正确，然后返回结果，成功返回登陆成功的界面，失败提示用户名或密码错误，模板渲染用户的名字，登录成功后创建 session，并将信息通过 cookie 存储到 session，cookie 再发送给浏览器

### 去注销

`http.HandleFunc("/logout", controller.Logout)`
获取客户端的 cookie，查询 session 并从数据库中删除 session，让 cookie 失效，返回给浏览器 cookie，并回到首页

### 去注册

`http.HandleFunc("/regist", controller.Regist)`
获取用户输入，在数据库中查询用户名是否存在，并给与提示，注册成功的话返回注册成功的页面，然后就可以进行登陆了

### 通过 Ajax 请求验证用户名是否可用

`http.HandleFunc("/checkUserName", controller.CheckUserName)`
这个函数实现 ajax 提示用户名已存在，与上面的结合使用，前端请求该函数，这样输入了重复的用户名，不会导致之前填写的都消失，也就是页面不会刷新，提高用户体验

## bookhandler.go

### 获取带分页的图书信息

`http.HandleFunc("/getPageBooks", controller.GetPageBooks)`
该处理器获取页码，调用分页的函数，用逻辑实现，上一页和下一页何时消失，解析到模板，到图书馆管理界面，分页函数查询数据库获取总页数，并将所有图书写入结构体返回，渲染的时候，书那一块渲染书的结构体，页数那一块还要调用页数的处理函数进行页数的跳转，在函数中渲染，这些函数（判断是否有首页，是否有尾页，上一页，下一页）和定义页数结构体处于同一个文件，并且输入页数前端获取指定的页数值，还是调用了处理器，只不过，处理器会判断有没有页数，有的话到指定页数，没有默认就是第一页即首页

渲染的页面有删除功能和修改功能和增加功能，每个 herf 会回到处理器函数继续处理，删除功能调用查询数据库并删除，然后执行获取分页图书的处理器函数，增加功能和修改功能都会转到同一个页面，用动作来判断渲染哪一部分，修改会通过书的 id(隐藏的 input 获取 id)来连接前后端，转到修改页面，渲染适当的内容，修改过后提交，提交执行`UpdateOrAddBook`函数，获取数据，向数据库修改数据，实现对数据进行刷新，增加与修改类似，函数末尾跟删除一样执行获取页码的处理器函数，实现更新过的数据显示

### 删除图书

`http.HandleFunc("/deleteBook", controller.DeleteBook)`

在上面已经介绍过了

### 去更新图书的页面

`http.HandleFunc("/toUpdateBookPage", controller.ToUpdateBookPage)`
在上面已经介绍过了

### 更新或添加图书

`http.HandleFunc("/updateOraddBook", controller.UpdateOrAddBook)`
在上面已经介绍过了

### 获取带分页和价格范围的图书

`http.HandleFunc("/getPageBooksByPrice", controller.GetPageBooksByPrice)`

该处理器实现了在首页根据输入价格的最大和最小查询符合条件的图书，因为是给用户看的，所以需要验证是否处于登录状态，通过数据库查询 session 来验证用户是否登录，查询符合条件的图书后，判断 session，并更新渲染的信息，是否渲染请登录，信息为怕个结构体，在此处复用一下上面写的页数代码就 OK 了

## carthandler.go

### 添加图书到购物车中

`http.HandleFunc("/addBook2Cart", controller.AddBook2Cart)`

此处理器先判断用户是否登录，没登录 ajax 提示登录，没登录登录的话，登录的话，根据图书获取图书信息，通过 session 获取用户 id，判断该用户有没有购物车，没有购物车，创建购物车，并将选中的图书加入新创建的购物项，购物项保存到切片中，完成这一步操作后，提示用户将图书加入了购物项，如果购物车已存在，去数据库中先找到该用户的购物车，添价商品先判断购物车的购物项中是否有改购物项，没有则创建购物项，加入到切片上，有则在原来的购物项的 count 上+1，并更新数据库中购物项数据，购物车和购物项在数据库中与用户相关联，购物车中有购物项的结构体，session 的 id 有用户登录时与用户的 id 相关联，这有一个容易犯的小错误，在获取购物项时。一定要获取图书，不然会导致购物项中图书加入到数据库中看不到想要的结果，另外，购物车结构体所在的文件有计算总数量和总价钱的方法，购物项结构体所在文件有计算金额小计的方法

### 获取购物车信息

`http.HandleFunc("/getCartInfo", controller.GetCartInfo)`

这个处理器跳转到购物车信息页面，首先根据 session 的 id 获取对应的购物车，没有则显示购物车空空如也，有购物车则显示有购物车，渲染到界面上，购物车存在 session 结构体上，所以可以直接传 session。在其前端界面用 ajax 显示了金额和数量。

### 清空购物车

`http.HandleFunc("/deleteCart", controller.DeleteCart)`
调用函数删除购物车，因为购物车与购物项关联，所以删除购物车所有购物项后，才能删除购物该购物车，前端界面实现挽留：是否要删除

### 删除购物项

`http.HandleFunc("/deleteCartItem", controller.DeleteCartItem)`
获取图书信息，在对应数据库中删除购物项

### 更新购物项

`http.HandleFunc("/updateCartItem", controller.UpdateCartItem)`
该处理器实现更新更改购物项后的数据，在前端事件监听那监听到购物项发生变化就会调用该处理器，从而用 ajax 实现局部更新

### 去结账

`http.HandleFunc("/checkout", controller.Checkout)`

该处理器在页面实现跳转，获取购物车，生成订单时间，并创建订单，保存到数据库，遍历数据项，保存到订单中，一气合成，在首页处更新书的数量：售出多少，还剩多少，清空该购物车，设置单号到 session，解析模板，页面显示单号，结账完毕

## orderhandler.go

### 获取所有订单

`http.HandleFunc("/getOrders", controller.GetOrders)`
获取所有订单，在页面所有订单信息，会显示发货，收获等，管理端和客户端不一样，为不同的单号设置不同的状态，获取状态为订单结构体的方法

### 获取订单详情，即订单所对应的所有的订单项

`http.HandleFunc("/getOrderInfo", controller.GetOrderInfo)`

订单页面点击订单详情，会跳转到订单详情页面，显示该订单中的所有图书购买状况，

### 获取我的订单

`http.HandleFunc("/getMyOrder", controller.GetMyOrders)`

针对客户端，用户登录，根据 session 的 id 获得对应的订单并渲染模板

### 发货

`http.HandleFunc("/sendOrder", controller.SendOrder)`

点击调用函数改变订单的状态，管理端使用

### 确认收货

`http.HandleFunc("/takeOrder", controller.TakeOrder)`

点击调用函数改变订单的状态，客户端使用

## 最后

前端的 herf 实现不同的处理器的功能的实现，很灵活，一个处理器可以多处去链接，多个链接可以使用一个处理器，入口处静态资源处理一下，这就是入口函数要干的事，控制出全部写处理器，处理器要用到的函数全写到 dao 的文件夹里去，数据的定义写到 model 中区，utils 写了数据库连接和生成随机数值（订单号，session 的 id），view 里面写静态页面和静态资源，其实标准的应该是 static 里写静态资源，view 写页面，这是一个纯后端项目所以这每份这么细，

最后列一下处理器清单
从 main 入口可以知道所有的功能：

```go

	//设置处理静态资源，如css和js文件
	http.Handle("/static/", http.StripPrefix("/static/", http.FileServer(http.Dir("views/static"))))
	//直接去html页面
	http.Handle("/pages/", http.StripPrefix("/pages/", http.FileServer(http.Dir("views/pages"))))
	//去首页
	http.HandleFunc("/main", controller.GetPageBooksByPrice)
	//去登录
	http.HandleFunc("/login", controller.Login)
	//去注销
	http.HandleFunc("/logout", controller.Logout)
	//去注册
	http.HandleFunc("/regist", controller.Regist)
	//通过Ajax请求验证用户名是否可用
	http.HandleFunc("/checkUserName", controller.CheckUserName)
	//获取所有图书
	// http.HandleFunc("/getBooks", controller.GetBooks)
	//获取带分页的图书信息
	http.HandleFunc("/getPageBooks", controller.GetPageBooks)
	http.HandleFunc("/getPageBooksByPrice", controller.GetPageBooksByPrice)
	//添加图书
	// http.HandleFunc("/addBook", controller.AddBook)
	//删除图书
	http.HandleFunc("/deleteBook", controller.DeleteBook)
	//去更新图书的页面
	http.HandleFunc("/toUpdateBookPage", controller.ToUpdateBookPage)
	//更新或添加图书
	http.HandleFunc("/updateOraddBook", controller.UpdateOrAddBook)
	//添加图书到购物车中
	http.HandleFunc("/addBook2Cart", controller.AddBook2Cart)
	//获取购物车信息
	http.HandleFunc("/getCartInfo", controller.GetCartInfo)
	//清空购物车
	http.HandleFunc("/deleteCart", controller.DeleteCart)
	//删除购物项
	http.HandleFunc("/deleteCartItem", controller.DeleteCartItem)
	//更新购物项
	http.HandleFunc("/updateCartItem", controller.UpdateCartItem)
	//去结账
	http.HandleFunc("/checkout", controller.Checkout)
	//获取所有订单
	http.HandleFunc("/getOrders", controller.GetOrders)
	//获取订单详情，即订单所对应的所有的订单项
	http.HandleFunc("/getOrderInfo", controller.GetOrderInfo)
	//获取我的订单
	http.HandleFunc("/getMyOrder", controller.GetMyOrders)
	//发货
	http.HandleFunc("/sendOrder", controller.SendOrder)
	//确认收货
	http.HandleFunc("/takeOrder", controller.TakeOrder)

	http.ListenAndServe(":8080", nil)

```
