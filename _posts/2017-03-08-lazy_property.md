===
Go as far as you can see; when you get there you'll be able to see farther.
===

---
title: Today stack overflow harvest 
description: a question commit in the stack overflow today with no answer.
header: 今天stack overflow遇到的一个问题
---

lazy property with UIView subclass

	import UIKit
	
	class MyView: UIView {}
	
	class Controller: UIViewController {
	
	    lazy var myView = MyView()
	}

But I have an error:

> Cannot convert values type 'UIView' to specified type 'MyView' I can fix the error with type of property:

	lazy var myView: MyView = MyView()

or change initialization to:

	let myView = MyView()

but why Swift cannot inference the type?

my answer:

	class MyView: UIView {
	init() {
	    super.init(frame: .zero)
	}
	
	required init?(coder aDecoder: NSCoder) {
	    fatalError("init(coder:) has not been implemented")
	}
	}

[question link](http://stackoverflow.com/questions/42639651/lazy-property-with-uiview-subclass/42643235#42643235)