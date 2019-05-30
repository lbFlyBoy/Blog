**Widget 概念**

首先我们需要了解下 widget 的概念，google 翻译过来叫小部件。将 widget 想象为一个可视化组件或与应用可视化方面交互的组件，同 view 可视化控件不同的是，widget 不是一个控件，而是对控件的描述，其实我们也不必非要纠结概念这个东西，当你用的多了，也就领悟这个概念了。

在 flutter 中，所有的东西都是 widget ，下面代码中可以看到 Text 、AppBar、 AppDemo、Scaffold 都是widget 。

```Dart
class AppDemo extends StatelessWidget {
	@override
	Widget build(BuildContext context) {
    return new Scaffold(
      appBar: new AppBar(
        title: new Text('flutter'),
      ),
      body: new Center(
        child:new Text('content'),
      ),
		);
  }
}
```

flutter 的核心思想就是用 widget 构建你的 UI 。并且它提供了一套丰富、强大的基础 widget ，可参考 ：<https://flutterchina.club/widgets/>

**Widget 状态**

在 flutter 中，widget 是不可变的，你需要去操纵 widget 的 state。widget 有状态跟无状态的区分。

*无状态 StatelessWidget*

StatelessWidget ( 无状态的 widget ) 在你构建初始化后不再进行改变。

例如上面例子，代码中的 `child: new Text('content')` 这个内容 content 不会变。那么我们就可以用 StatelessWidget 的 Text 。假如我们点击按钮后希望内容改变，我们就需要了解下有状态的 widget 。

*有状态 StatefulWidget*

StatefulWidget ( 有状态的 widget ) 拥有一个 state 对象来存储它的状态数据，并在 widget 树重建时携带着它，因此状态不会丢失。一个 StatefulWidget 类对应一个 state 类，state 是与对应 StatefulWidget 维护的状态，

当 state 被改变时，调用 ` setState()` 方法通知 flutter framework 状态发生改变，重新调用 `build` 方法构建 widget 树，来进行更新操作，看下面实例。

```Dart
class AppDemo extends StatefulWidget {
	@override
	createState() => _AppDemoState();
}

class _AppDemoState extends State<AppDemo> {
	String _content = "content";
	void _updateContent() {
		setState((){
			_content = "flutter content";
		});
	}
	@override
	Widget build(BuildContext context) {
    return new Scaffold(
		appBar: new AppBar(
			title: TextPage(title: 'flutter',),
		),
		body: new Center(
			child:new Text(_content),
		),
		floatingActionButton: FloatingActionButton(
        onPressed: _updateContent,
        child: Icon(Icons.mode_edit),
      ),
		);
  }
}
```

当点击按钮后，显示的内容就会更改为 flutter content 

**Widget 生命周期**

对于一个前端开发，我们都比较关心生命周期这个话题，同样理解 Widget 生命周期对 flutter 开发也是这样的，例子：通过一个计数器 widget 的例子来理解这个话题。

```Dart
class CounterWidget extends StatefulWidget {
	CounterWidget({this.initCounter:0});
	final int initCounter;
	@override
	createState() => _CounterWidgetState();
}

class _CounterWidgetState extends State<CounterWidget> {
	int _counter;
	@override
	void initState() {
		super.initState();
		_counter = widget.initCounter;
		print('***************initState $_counter***************');
	}
	@override
  Widget build(BuildContext context) {
  	print('***************build***************');
    return Center(
    	child: RaisedButton(
      child: Text('$_counter'),
      onPressed: ()=>setState(()=>++_counter),
    ),
   );
  }
  @override
	void didUpdateWidget(CounterWidget oldWidget) {
    super.didUpdateWidget(oldWidget);
    print('***************didUpdateWidget***************');
  }
	@override
	void deactivate() {
		super.deactivate();
		print("***************deactive***************");
	}
	@override
	void dispose() {
		super.dispose();
		print("***************dispose***************");
	}
	@override
	void reassemble() {
		super.reassemble();
		print("***************reassemble***************");
	}
	@override
	void didChangeDependencies() {
		super.didChangeDependencies();
		print("***************didChangeDependencies***************");
	}
}
```

通过现象得结论：

*debug 启动运行结果：*

```Dart
***************initState 0*************** 
***************didChangeDependencies***************
***************build***************
```

`initState` ：该 widiget 插入到 widget 树结构时被调用，只调用一次，因此，该函数一般用来初始化操作，状态初始化、订阅子树事件通知。

`didChangeDependencies()`：当调用 `initState` 后会立即调用这个方法，这个方法是在 state 对象被创建好了但没有准备好构建 `build` 的时候调用的。

`build` : 调用这个方法来构建 widget 子树。触发的时机较多如：调用 `initState()`、`didChangeDependencies()`、`setState()`、`didUpdateWidget()`等方法后重新 build。

*点击保存按钮执行结果：*

```Dart
***************reassemble***************
***************didUpdateWidget***************
***************build***************
```

`reassemble()`：专门为了开发调试而提供的，在热重载时会被调用。

`didUpdateWidget()`：在 widget 重新构建时，由 flutter framework 判断检测 widget 树种同一位置新旧节点，决定是否更新，调用该方法。

*移除 CounterWidget 执行结果：*

```Dart
***************reassemble***************
***************deactive***************
***************dispose***************	
```

`deactive`：当widget 对象从树中被移除时，会调用此方法。

`dispose `：当 widget 对象从树中被永久移除时调用 ，可以在此方法中释放资源。

*点击按钮执行结果：*

```Dart
***************build***************
```

