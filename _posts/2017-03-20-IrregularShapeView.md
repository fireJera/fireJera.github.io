---
title: Irregular shape view(Button)
description: Create Irregualr-shape-button.
header: Irregualr shape View
---

A goal is a dream with a deadline.
目标就是给梦想一个权限
Sometimes ever, sometimes never.

	class CustomButton: UIView {
	    var shapLayer: CAShapeLayer?
    
	    override init(frame: CGRect) {
	        super.init(frame: frame)
	        setUp()
	    }
    
	    required init?(coder aDecoder: NSCoder) {
	        super.init(coder: aDecoder)
	        setUp()
	    }
    
	    func setUp() {
	        self.shapLayer = CAShapeLayer()
	        let path = CGMutablePath()
	        path.move(to: CGPoint(x: 100, y: 0))
	        path.addLine(to: CGPoint(x: 100, y: 100))
	        path.addLine(to: CGPoint(x: 0, y: 100))
	        self.shapLayer?.path = path
	        self.layer.mask = self.shapLayer
	        self.layer.masksToBounds = true
	        self.backgroundColor = UIColor.lightGray
	    }
    
	    override func point(inside point: CGPoint, with event: UIEvent?) -> Bool {
	        if (self.shapLayer?.path?.contains(point))! {
	            return super.point(inside: point, with: event)
	        } else {
	            return false
	        }
	    }
    
	    override func touchesBegan(_ touches: Set<UITouch>, with event: UIEvent?) {
	        print("touchesBegan")
	    }
    }

这是比较简单的算是一个规则的view都是直线的，如果有曲线就要用到一个类UIBezierPath：

	//根据一个矩形画曲线
	+ (UIBezierPath *)bezierPathWithRect:(CGRect)rect
	
	//根据矩形框的内切圆画曲线
	+ (UIBezierPath *)bezierPathWithOvalInRect:(CGRect)rect
	
	//根据矩形画带圆角的曲线
	+ (UIBezierPath *)bezierPathWithRoundedRect:(CGRect)rect cornerRadius:(CGFloat)cornerRadius
	
	//在矩形中，可以针对四角中的某个角加圆角
	+ (UIBezierPath *)bezierPathWithRoundedRect:(CGRect)rect byRoundingCorners:(UIRectCorner)corners cornerRadii:(CGSize)cornerRadii
	
	参数：
	corners:枚举值，可以选择某个角
	cornerRadii:圆角的大小
	//以某个中心点画弧线  + (UIBezierPath *)bezierPathWithArcCenter:(CGPoint)center radius:(CGFloat)radius startAngle:(CGFloat)startAngle endAngle:(CGFloat)endAngle clockwise:(BOOL)clockwise;
	参数：
	center:弧线中心点的坐标
	radius:弧线所在圆的半径
	startAngle:弧线开始的角度值
	endAngle:弧线结束的角度值
	clockwise:是否顺时针画弧线
	
	
	//画二元曲线，一般和moveToPoint配合使用
	- (void)addQuadCurveToPoint:(CGPoint)endPoint controlPoint:(CGPoint)controlPoint
	参数：
	endPoint:曲线的终点
	controlPoint:画曲线的基准点
	
	//以三个点画一段曲线，一般和moveToPoint配合使用
	- (void)addCurveToPoint:(CGPoint)endPoint controlPoint1:(CGPoint)controlPoint1 controlPoint2:(CGPoint)controlPoint2
	参数：
	endPoint:曲线的终点
	controlPoint1:画曲线的第一个基准点
	controlPoint2:画曲线的第二个基准点

test:
	
	override func viewDidLoad() {
        super.viewDidLoad()
        layer = CAShapeLayer()
        
        let startPoint = CGPoint(x: 50, y: 300)
        let endPoint = CGPoint(x: 300, y: 300)
        let controlPoint1 = CGPoint(x: 140, y: 200)
        let controlPoint2 = CGPoint(x: 200, y: 400)
        let path = UIBezierPath()
        path.move(to: startPoint)
        path.addCurve(to: endPoint, controlPoint1: controlPoint1, controlPoint2: controlPoint2)
        
	//        let path = UIBezierPath(roundedRect: CGRect(x: 110, y: 100, width: 150, height: 100), cornerRadius: 20)
	        layer?.path = path.cgPath
	        layer?.fillColor = UIColor.clear.cgColor
	        layer?.strokeColor = UIColor.black.cgColor
        
        animation1()
        animation3()

	//        layer.frame = CGRect(x: 50, y: 50, width: 100, height: 100)
	        self.view.layer.addSublayer(layer!)
	        // Do any additional setup after loading the view, typically from a nib.
	    }
    private func animation1() {
        let animation = CABasicAnimation(keyPath: "strokeEnd")
        animation.fromValue = 0
        animation.toValue = 1
        animation.duration = 2
        layer?.add(animation, forKey: "")
    }
    
    private func animation2() {
        layer?.strokeStart = 0.5
        layer?.strokeEnd = 0.5
        
        let animation = CABasicAnimation(keyPath: "strokeStart")
        animation.fromValue = 0.5
        animation.toValue = 0
        animation.duration = 2
        
        let animation2 = CABasicAnimation(keyPath: "strokeEnd")
        animation2.fromValue = 0.5
        animation2.toValue = 1
        animation2.duration = 2
        
        layer?.add(animation, forKey: "")
        layer?.add(animation2, forKey: "")
    }
    
    private func animation3() {
        let animation = CABasicAnimation(keyPath: "lineWidth")
        animation.fromValue = 1
        animation.toValue = 10
        animation.duration = 2
        layer?.add(animation, forKey: "lineWidth")
    }
   
仿时光网圆弧：

	let finalSize = CGSize(width: self.view.frame.width, height: 400)
	        let layerHeight = finalSize.height * 0.2
	        layer = CAShapeLayer()
	        let bezier = UIBezierPath()
	        bezier.move(to: CGPoint(x: 0, y: finalSize.height - layerHeight))
	        bezier.addLine(to: CGPoint(x: 0, y: finalSize.height - 1))
	        bezier.addLine(to: CGPoint(x: finalSize.width, y: finalSize.height - 1))
	        bezier.addLine(to: CGPoint(x: finalSize.width, y: finalSize.height - layerHeight))
	        bezier.addQuadCurve(to: CGPoint(x: 0, y: finalSize.height - layerHeight), controlPoint: CGPoint(x: finalSize.width / 2, y: (finalSize.height - layerHeight) - 40))
	        layer?.path = bezier.cgPath
	        layer?.fillColor = UIColor.black.cgColor