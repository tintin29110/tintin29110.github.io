---
layout: post
current: post
cover:  assets/images/custom_components_cover.jpeg
navigation: True
title: Customized Components
date: 2020-08-18 10:47:00
tags: devlog
class: post-template
subclass: 'post'
author: seyoon
---

#### 체크박스

> 이미지만 바꾸어서 라디오버튼으로도 사용 가능하다. 물론 그룹 중 하나만 선택되는 로직은 추가 구현 필요.

{% highlight swift %}
import UIKit

class Checkbox: UIButton {
    var isChecked: Bool = false {
        didSet {
            setImage(isChecked ? self.selectedImg : self.defaultImg, for: .normal)
        }
    }

    var defaultImgName: String = "icCheckDefault" {
        didSet {
            self.defaultImg = UIImage(named: defaultImgName)
            if !isChecked {
                setImage(self.defaultImg, for: .normal)
            }
        }
    }
    var selectedImgName: String = "icCheckSelect" {
        didSet {
            self.selectedImg = UIImage(named: selectedImgName)
            if isChecked {
                setImage(self.selectedImg, for: .normal)
            }
        }
    }

    var defaultImg: UIImage?
    var selectedImg: UIImage?

    override init(frame: CGRect) {
        super.init(frame: frame)
        initCheckboxWithImg()
    }

    required init?(coder: NSCoder) {
        super.init(coder: coder)
        initCheckboxWithImg()
    }

    override func touchesEnded(_ touches: Set<UITouch>, with event: UIEvent?) {
        super.touchesEnded(touches, with: event)
        self.isChecked = !isChecked
        self.sendActions(for: .valueChanged)
    }

    private func initCheckboxWithImg() {
        self.defaultImg = UIImage(named: self.defaultImgName)
        self.selectedImg = UIImage(named: self.selectedImgName)

        setImage(isChecked ? self.selectedImg : self.defaultImg, for: .normal)
    }
}
{% endhighlight %}
---
#### 스위치

> thumb의 색상, 크기, on/off 상태의 색상들을 간편하게 수정할 수 있는 스위치.

{% highlight swift %}
import Foundation
import UIKit

class CustomSwitch: UIControl {
    public var padding: CGFloat = 1 {
        didSet {
            self.layoutSubviews()
        }
    }
    public var onTintColor = UIColor(red: 144/255, green: 202/255, blue: 119/255, alpha: 1) {
        didSet {
            self.setupUI()
        }
    }
    public var offTintColor = UIColor.lightGray {
        didSet {
            self.setupUI()
        }
    }
    public var cornerRadius: CGFloat = 0.5 {
        didSet {
            self.layoutSubviews()
        }
    }
    public var thumbTintColor = UIColor.white {
        didSet {
            self.thumbView.backgroundColor = self.thumbTintColor
        }
    }
    public var thumbCornerRadius: CGFloat = 0.5 {
        didSet {
            self.layoutSubviews()
        }
    }
    public var thumbSize = CGSize.zero {
        didSet {
            self.layoutSubviews()
        }
    }
    public var thumbShawdow = false {
        didSet {
            self.setupUI()
        }
    }

    public var isOn = true {
        didSet {
            self.thumbView.frame.origin.x = self.isOn ? self.onPoint.x : self.offPoint.x
            self.backgroundColor = self.isOn ? self.onTintColor : self.offTintColor
        }
    }

    public var animationDuration: Double = 0.5

    fileprivate var thumbView = UIView(frame: CGRect.zero)

    fileprivate var onPoint = CGPoint.zero

    fileprivate var offPoint = CGPoint.zero

    fileprivate var isAnimating = false

    private func clear() {
        for view in self.subviews {
            view.removeFromSuperview()
        }
    }

    func setupUI() {
        self.clear()
        self.clipsToBounds = false
        self.thumbView.backgroundColor = self.thumbTintColor
        self.thumbView.isUserInteractionEnabled = false
        self.addSubview(self.thumbView)

        if self.thumbShawdow {
            self.thumbView.layer.shadowColor = UIColor.black.cgColor
            self.thumbView.layer.shadowRadius = 1.5
            self.thumbView.layer.shadowOpacity = 0.4
            self.thumbView.layer.shadowOffset = CGSize(width: 0.75, height: 2)
        }
    }

    public override func layoutSubviews() {
        super.layoutSubviews()
        if !self.isAnimating {
            self.layer.cornerRadius = self.cornerRadius
            self.backgroundColor = self.isOn ? self.onTintColor : self.offTintColor

            // thumb managment
            let thumbSize = self.thumbSize != CGSize.zero ? self.thumbSize : CGSize(width: self.bounds.size.height - 2, height: self.bounds.height - 2)
            let yPostition = (self.bounds.size.height - thumbSize.height) / 2

            self.onPoint = CGPoint(x: self.bounds.size.width - thumbSize.width - self.padding, y: yPostition)
            self.offPoint = CGPoint(x: self.padding, y: yPostition)

            self.thumbView.frame = CGRect(origin: self.isOn ? self.onPoint : self.offPoint, size: thumbSize)

            self.thumbView.layer.cornerRadius = self.thumbCornerRadius

        }
    }

    private func animate() {
        self.isOn = !self.isOn
        self.sendActions(for: UIControl.Event.valueChanged)
        self.isAnimating = true
        UIView.animate(withDuration: self.animationDuration, delay: 0, usingSpringWithDamping: 0.7, initialSpringVelocity: 0.5, options: [UIView.AnimationOptions.curveEaseOut, UIView.AnimationOptions.beginFromCurrentState], animations: {
            self.thumbView.frame.origin.x = self.isOn ? self.onPoint.x : self.offPoint.x
            self.backgroundColor = self.isOn ? self.onTintColor : self.offTintColor
        }, completion: { _ in
            self.isAnimating = false
        })
    }

    override func beginTracking(_ touch: UITouch, with event: UIEvent?) -> Bool {
        super.beginTracking(touch, with: event)
        self.animate()
        return true
    }
}
{% endhighlight %}


> How to use

{% highlight swift %}
let cardViewVideoSwitch: CustomSwitch =  {
    let customSwitch = CustomSwitch()
    customSwitch.thumbSize = CGSize(width: 14, height: 14)
    customSwitch.thumbCornerRadius = 7
    customSwitch.offTintColor = UIColor(red: 198 / 255, green: 191 / 255, blue: 185 / 255, alpha: 1)
    customSwitch.onTintColor = CommonUtils.shared.color_text_black
    customSwitch.cornerRadius = 10
    customSwitch.padding = 3

    return customSwitch
}()

@objc func videoSwitchChanged(_ sender: CustomSwitch) {
    self.selectedButton?.type = sender.isOn ? .Video : .General
    self.cardViewCreateLinkBtn.isHidden = !sender.isOn
    if sender.isOn {
        self.buildingTextBtn.setTitle("화상회의", for: .normal)
        cardViewLocationLblLeadingConstraint.constant = 125
    } else if selectedBuilding >= 0 {
        self.buildingTextBtn.setTitle(buildings[selectedBuilding], for: .normal)
        cardViewLocationLblLeadingConstraint.constant = 26
    }
}
{% endhighlight %}

[notion test](https://www.notion.so/seyoon37/1-Meet-async-await-in-Swift-d5b31f54404a44d79696b5fee0200741)
