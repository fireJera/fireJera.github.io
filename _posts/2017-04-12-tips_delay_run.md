---
title: delay run
description: Four way to run code after delay
header:How to delay running
---
Genius often betrays itself into great errors.

There is four ways to delay running.

1. performSelector
2. NSTimer
3. sleep(NSThread)
4. GCD

		func delayMethod() {
			print("delayMethod")
		}

* Method1:

		self.perform(#selector(delayMethod), with: nil, afterDelay: 2.0)
	
It won't block current thread, and it can't stoped

* Method2:

		let timer = Timer.scheduledTimer(timeInterval: 2.0, target: self, selector: #selector(delayMethod), userInfo: nil, repeats: false)
	
It also won't block current thread, using invalidate to stop.

* Method3:

		Thread.sleep(forTimeInterval: 2.0)
	
It will block thread(so we can put it in sub thread), no way to stop.

* Method4:

		let delayTime = DispatchTime.now() + 0.2//.seconds(1)
		DispatchQueue.main.asyncAfter(deadline: delayTime) {
			self.tableView.reloadData()
		}
		
It won't block current thread too, can't stopped.