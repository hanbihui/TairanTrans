#Swift中用Sprite Kit制作的卡牌游戏机制
![Example card image](http://cdn4.raywenderlich.com/wp-content/uploads/2014/07/card-article-header.png)

*学习如何实现基本的卡牌游戏机制和动画。*

在过去的20年里，人们玩过收集类卡牌游戏(CCGs)。[维基百科](http://en.wikipedia.org/wiki/Collectible_card_game)提供了比较详尽的这些游戏演变的过程，它似乎启发了角色扮演游戏（RPG）像龙与地下城。万智牌是一个现代化的CCG的一个例子。

其核心，CCGs是一组自定义卡片代表了人物，位置，能力，事件等。玩游戏前，玩家首先必须构建他们自己的牌组，然后他们才能使用他们独特的牌组来玩游戏。大多数玩家的牌组都会突出某种派别，生物或能力。

在本教程中，你将在CCG应用程序中使用Sprite Kit操纵图像卡。你会在屏幕上移动卡片，移动它们来看哪些卡片是有效的，翻转它们并放大它们来看卡片上的文字 — 或欣赏艺术作品。

如果你还不熟悉SpriteKit，你可以阅读[初学者的教程](http://www.raywenderlich.com/42699/spritekit-tutorial-for-beginners)或[iOS游戏教程](http://www.raywenderlich.com/store/ios-games-by-tutorials)。如果你还不熟悉Swift，确认你已经了解了[Swift快速入门系列](http://www.raywenderlich.com/74438/swift-tutorial-a-quick-start).

##开始

由于这是一个卡牌游戏，最好的开始就是看实际的卡片。下载[启动项目](https://bitbucket.org/bcbroom/ccg-assets-only-swift/get/master.zip)，它提供了一个iPad上横向模式下的SpriteKit项目，以及所有的图像，字体和声音文件，你需要创建一个功能性的示例游戏。

花一点时间来熟悉项目的文件结构和内容。你应该能看到如下的项目目录：

1. **System**: 包含设置一个SpriteKit项目的基础文件。它包含了**AppDelegate.swift**，**GameViewController.swift**和**Main.storyboard**
2. **Scenes**: 包含管理游戏内容的一个空主场景文件**GameScene.swift**。
3. **Card**: 包含管理卡牌的一个空的**Card.swift**文件。
4. **Supporting Files**: 包含你将在本教程中使用的所有图像、字体和声音文件。

这个游戏没有美工不会显得很酷，所以我想特别感谢[gameartguppy.com](http://www.gameartguppy.com/)的Vicki的美丽的卡牌插图！

##一个优雅的开始

既然我们需要使用卡牌玩卡牌游戏，我们将创建一个卡片类来代表它们。目前**Card.swift**是一个空的Swift文件，所以找到它并添加：

	import Foundation
	import SpriteKit
	 
	class Card : SKSpriteNode {
	 
	  required init(coder aDecoder: NSCoder!) {
	    fatalError("NSCoding not supported")
	  }
	 
	  init(imageNamed: String) {
	    let cardTexture = SKTexture(imageNamed: imageNamed)
	    super.init(texture: cardTexture, color: nil, size: cardTexture.size())
	  }
	}

你声明了Card是**SKSpriteNode**的子类。

要根据图像创建一个简单的精灵，你可以使用**SKSpriteNode(imageNamed:)**。为了保持这种行为，你需要使用继承的初始化函数，它将调用父类的指定的初始化函数**init(texture:color:size:)**。在本游戏中你不支持**NSCoding**。

要把精灵放到屏幕上，打开**GameScene.swift**然后添加下列代码到**didMoveToView()**中：

	let wolf = Card(imageNamed: "card_creature_wolf.png")
	wolf.position = CGPointMake(100,200)
	addChild(wolf)
	 
	let bear = Card(imageNamed: "card_creature_bear.png")
	bear.position = CGPointMake(300, 200)
	addChild(bear)

构建和运行项目，并花一点时间来欣赏儿狼和熊。

![Card Images on iPad Screen](http://cdn2.raywenderlich.com/wp-content/uploads/2014/07/ClassyStartStaticResized.png)

*一个好的开始…*

规则 #1 创建卡牌游戏：先有创意和富有想象力的美工。看下来你的应用程序塑造的不错！

>**注：** 根据屏幕的大小，你可能需要缩放模拟器的窗口，使用Window\Scale\50%来适应屏幕。我也推荐使用iPad 2模拟器。

看这一对卡牌非常有趣，但如果你能真正的移动卡片UI将会更酷。你将在下面做这些！

##我想移动它，移动它…

无论美工的质量如何，卡牌静坐在屏幕上不会让你的应用程序获得任何的好评，因为你需要像你在玩真实的纸质卡牌一样能拖动它们。实现它的最简单的办法是在场景中控制触摸事件。

依旧在**GameScene.swift**中，添加这个新的函数到类中：

	override func touchesMoved(touches: NSSet, withEvent event: UIEvent) {
	  for touch in touches {
	    let location = touch.locationInNode(self)
	    let touchedNode = nodeAtPoint(location)
	    touchedNode.position = location
	  }
	}

构建和运行项目，然后试着拖动这两个卡牌。

![Cards move, but sometimes slide under other cards](http://cdn4.raywenderlich.com/wp-content/uploads/2014/07/MoveCardsClip.gif)

*卡牌现在可以移动了，但有时卡牌会被遮住。继续下面的阅读来解决这个问题。*

你玩过之后，将注意到两个主要的问题：

1. 首先，由于精灵都在相同的**zPosition**，所以它们在被添加到场景中时被布置成相同的顺序。这意味着熊卡牌会在狼的卡牌“上面”。如果你拖动狼，它似乎被熊遮住了。
2. 第二，**nodeAtPoint()**返回这个点上的最上面的精灵。所以当你拖动熊下面的狼时，**nodeAtPoint()**返回熊精灵然后开始改变它的位置，所以你可能会发现你拖动狼时熊会被拖动。

这个效果虽然很神奇，但它并不是你在最终的应用程序中想要的！

为了解决这个问题，你将需要在拖动时修改卡牌的zPosition。你的第一反应可能是在**touchesMoved**中修改精灵的**zPosition**，但如果你想在后面把它改回来，这不是一种好的办法。

使用开始和结束函数是一个更好的策略。依旧在**GameScene.swift**中，添加下列函数：

	override func touchesBegan(touches: NSSet, withEvent event: UIEvent) {
	  for touch in touches {
	    let location = touch.locationInNode(self)
	    let touchedNode = nodeAtPoint(location)
	    touchedNode.zPosition = 15
	  }
	}
	 
	override func touchesEnded(touches: NSSet, withEvent event: UIEvent) {
	  for touch in touches {
	    let location = touch.locationInNode(self)
	    let touchedNode = nodeAtPoint(location)
	    touchedNode.zPosition = 0
	  }
	}

再次构建和运行项目，你将看到卡牌像我们期望的那样滑动。

![Cards now correctly move over each other](http://cdn4.raywenderlich.com/wp-content/uploads/2014/07/MoveCardsNoClip.gif)

*卡片现在正确的移动了，但看起来有些平淡。你将在后面解决这个问题。*

确保你挑选了大于其他卡牌的一个**zPosition**值。在本教程的示例游戏中，有一些重叠的元素的**zPosition**为20。数字19确保了重叠的元素显示在卡牌之上。

现在卡牌已经正确的移动了，但你需要添加一些令人满意的内容 — 一个可视化的指示表明卡牌被举起了。

是时候让你的卡牌跳舞了！

##卡牌动画

依旧在**GameScene.swift**中，添加下面的代码到**touchesBegan()**函数中**for**循环的结尾

	let liftUp = SKAction.scaleTo(1.2, duration: 0.2)
	touchedNode.runAction(liftUp, withKey: "pickup")

在**touchesEnded()**中添加类似的内容

	let dropDown = SKAction.scaleTo(1.0, duration: 0.2)
	touchedNode.runAction(dropDown, withKey: "drop")

在这里当你点击卡牌时，使用了**SKAction**的**scaleTo(scale:duration:)**函数来增加卡牌的宽度和高度为它原来的1.2倍，当你松开时变回原来的大小。

构建和运行项目来查看效果。

![Moving cards with pickup and drop down animation.](http://cdn1.raywenderlich.com/wp-content/uploads/2014/07/MoveCardsPickupDrop.gif)

*这个简单的动画像是把卡牌拿起来和放下去。有时候最简单的动画也是最有效的。*

调整你scale和duration的值来找到最适合你的。如果你设置拿起和放下的durations为不同的值，你可以让它看起来拿起的时候比较慢，放下的时候比较快。

##摆动，摆动，摆动

拖动卡牌现在工作得已经很好了，但你应该做得更好。让卡牌围绕y轴拍翅膀当然是很棒的。

由于SpriteKit是一个纯2D框架，似乎没有办法做到精灵的部分转动效果。然后你可以这么做，修改xScale属性来制造转动的假象。

你将添加代码到**touchesBegan()**和**touchesEnded()**函数中。在**touchesBegan()**中添加如下代码到**for**循环的最后：

	let wiggleIn = SKAction.scaleXTo(1.0, duration: 0.2)
	let wiggleOut = SKAction.scaleXTo(1.2, duration: 0.2)
	let wiggle = SKAction.sequence([wiggleIn, wiggleOut])
	let wiggleRepeat = SKAction.repeatActionForever(wiggle)
	 
	touchedNode.runAction(wiggleRepeat, withKey: "wiggle")

在**touchesEnded()**中添加类似的代码：

	touchedNode.removeActionForKey("wiggle")

这些代码让卡片回来转动 — 只是一点点 — 当它来回移动时。这个效果利用了**reaction(action:, withKey:)**函数添加了一个字符串名称到这个动作，这样你可以在后面取消它。

对于这种做法有一个小小的警告：当你删除动画时，无论它在不在动画的生命周期中，它会离开精灵。

你已经有一个动作使卡牌返回它的初始缩放值1.0。由于缩放同时设置x和y缩放，所以这部分不用担心，但如果你使用其他的属性，记得在**touchesEnded**函数中返回到初始值。

构建和运行项目，你会看到当你拖动他们时，卡牌会拍翅膀。

![Card with scaling animation to fake 3d rotation.](http://cdn4.raywenderlich.com/wp-content/uploads/2014/07/CardWiggleX.gif)

*一个简单的动画来显示这个卡牌是当前活动的。*

>**挑战：**在本教程最后的额外的示例游戏中，你将了解如何使用zRotation来让卡牌来回摇晃。
试着用rotateBy更换scaleXTo动作，以“摇晃”动画取代“摆动”动画。记得使其循环，这意味着它需要在重复之前返回它的出发点。

![Card rotates slightly back and forth.](http://cdn2.raywenderlich.com/wp-content/uploads/2014/07/CardWiggleRotation.gif)

*尝试重现这个摇摆动画的效果。*

###解决方法

在**touchesBegan**中替换下列摆动代码:

	let rotR = SKAction.rotateByAngle(0.15, duration: 0.2)
	let rotL = SKAction.rotateByAngle(-0.15, duration: 0.2)
	let cycle = SKAction.sequence([rotR, rotL, rotL, rotR])
	let wiggle = SKAction.repeatActionForever(cycle)
	touchedNode.runAction(wiggle, withKey: "wiggle")

这给你的卡牌添加了令人满意的小摆动，但是仍然有一个问题。试着拖动卡牌完成半个摆动周期。它的旋转是不是错误的？是的，这就是你下一步需要解决的问题，添加下面这行代码到**touchesEnded**中**for**循环的结尾：

	runAction(SKAction.rotateToAngle(0, duration: 0.2), withKey:"rotate")

现在当你释放卡牌时有了一个正确旋转的漂亮的摇摆动画！

##追踪伤害

在很多卡牌收集游戏中，像这些怪物会有关联的伤害值，可以与其他卡牌进行战斗。

要实现这一点，你需要在卡牌的顶端添加一个标签，这样用户可以追踪每个生物生成的伤害。依旧在**GameScene.swift**中，添加如下新方法：

	func newDamageLabel() -> SKLabelNode {
	  let damageLabel = SKLabelNode(fontNamed: "OpenSans-Bold")
	  damageLabel.name = "damageLabel"
	  damageLabel.fontSize = 12
	  damageLabel.fontColor = UIColor(red: 0.47, green: 0.0, blue: 0.0, alpha: 1.0)
	  damageLabel.text = "0"
	  damageLabel.position = CGPointMake(25, 40)
	 
	  return damageLabel
	}

这个辅助方法创建了一个新的**SKLabelNode**，它将显示每个卡牌造成的伤害。它使用了启动项目中正确的info.plist设置中包含的自定义字体。

>**注：**关于安装自定义字体的更多信息，请查看[iOS游戏教程](http://www.raywenderlich.com/store/ios-games-by-tutorials)中的第7章, “标签”.

你想知道示例中的位置是怎么工作的吗？

由于标签是卡牌精灵的子节点，位置默认是相对于精灵中心的锚点。通常经过一些尝试就可以得到你想要的标签位置。

添加如下代码到**didMoveToView()**结尾，来给每个卡牌添加一个伤害标签：

	wolf.addChild(newDamageLabel())
	bear.addChild(newDamageLabel())

构建和运行项目。你现在应该在每个卡牌中看到一个红色的“0”。

![Card with a label for damage taken.](http://cdn2.raywenderlich.com/wp-content/uploads/2014/07/DamageLabelResized.png)

*卡牌现在有一个显示受到多少伤害的标签了。*

试着拖动卡牌，但是点击该标签拖动时拖动的是标签而不是卡牌本身。注意标签飞到了什么地方 — 也许是一个神奇的国度在那里可以肆无忌惮？

不，其他它不是那么神秘。;]

这里的问题是当你调用**nodeAtPoint**时，它返回**SKNode**最上层的任何类型的节点，在这种情况下是**SKLabelNode**。当你改变节点的位置，移动的是标签而不是卡牌。呃。。。是的，这是合乎逻辑的解释。

![Dragging on top of the damage label causes problems.](http://cdn3.raywenderlich.com/wp-content/uploads/2014/07/LabelOops.png)

*触摸顶部伤害标签的结果。哎呀。（改变背景为白色，使标签更明显）*

##Pros and Cons Of Scene Touch Handling

Before going any further, let’s stop for a moment and muse upon some of the advantages and disadvantages of handling the touches at the scene level.

Touch handling at the scene level is a good place to start working with a project because it’s the simplest, easiest approach. In fact, if your sprites have transparent regions that should be ignored, such as hex grids, this may be the only reasonable solution.

However, it starts to fall apart when you have composite sprites. For example, these could contain multiple images, labels or even a health bar. It can also be unwieldy and complicated if you have different rules for different sprites.

One gotcha that comes into play when you use **nodeAtPoint** is that it always returns a node.

What if you drag outside of one of the card sprites? Because **SKScene** is a subclass of **SKNode**, if the touch location intersects no other node, the scene itself returns as a **SKNode**.

When you changed the position and did animations before, you may not have known but you really should’ve checked to see that **touchedNode** was not the scene itself, but it’s okay because this is a learning experience.

You’ll be happy to know there is a better solution…

##Handle Those Touches! Handle Them!

What can you do instead? Well, you can make the Card class responsible for handling its own touches. The logistics of this approach are fairly straightforward. Open **Card.swift** and add the following to **init(imageNamed:)**:

	userInteractionEnabled = true

This allows the Card class to intercept touches as opposed to passing them through to the scene. SpriteKit will send touch events to the topmost instance with this property set.

Next, you remove the three touch handler functions,**touchesBegan()**, **touchesMoved()**, and **touchesEnded()** from **GameScene.swift** and add them to **Card.swift**.

The original code won’t work exactly as-is, so it needs some changes to work within the node.

As a challenge, see if you can make the appropriate changes without checking the spoiler!

Hint: Since touch events are sent directly to the correct sprite, you don’t have to figure out which sprite to modify.

###Solution Inside

	override func touchesBegan(touches: NSSet, withEvent event: UIEvent) {
	  for touch in touches {
	    // note: removed references to touchedNode
	    // 'self' in most cases is not required in Swift
	    zPosition = 15
	    let liftUp = SKAction.scaleTo(1.2, duration: 0.2)
	    runAction(liftUp, withKey: "pickup")
	 
	    let wiggleIn = SKAction.scaleXTo(1.0, duration: 0.2)
	    let wiggleOut = SKAction.scaleXTo(1.2, duration: 0.2)
	    let wiggle = SKAction.sequence([wiggleIn, wiggleOut])
	    let wiggleRepeat = SKAction.repeatActionForever(wiggle)
	 
	    // again, since this is the touched sprite
	    // run the action on self (implied)
	    runAction(wiggleRepeat, withKey: "wiggle")
	  }
	}
	 
	override func touchesMoved(touches: NSSet, withEvent event: UIEvent) {
	  for touch in touches {
	    let location = touch.locationInNode(scene) // make sure this is scene, not self
	    let touchedNode = nodeAtPoint(location)
	    touchedNode.position = location
	  }
	}
	 
	override func touchesEnded(touches: NSSet, withEvent event: UIEvent) {
	  for touch in touches {
	    zPosition = 0
	    let dropDown = SKAction.scaleTo(1.0, duration: 0.2)
	    runAction(dropDown, withKey: "drop")
	    removeActionForKey("wiggle")
	  }
	}

Essentially, this copies the touch handling functions in their current state to the Card implementation. The major difference is that you no longer have to search the node hierarchy to find which node corresponds to that point.

SpriteKit calls the function on the correct instance, so you change the properties directly.

Build and run the project, and you’ll notice that this fixes the earlier issue of the flying label.

![Card can be moved the same as before](http://cdn4.raywenderlich.com/wp-content/uploads/2014/07/CardWiggleX.gif)

*Card can be moved the same as before.*

##Two Sides of the Story

Now take a moment to observe how the Card nodes initialize. Currently, you’re simply using the string image name to create a texture, and sending that to the superclass initializer.

In order to add attributes like attack and defense values, or mystical spell effects, you need to setup properties and configure them based on the specific card’s data. Instead of using strings to identify cards, which are prone to typos, you can define an enumeration instead. Open **Card.swift** and add the following between the import lines and the class definition:

	enum CardName: Int {
	    case CreatureWolf = 0,
	    CreatureBear,       // 1
	    CreatureDragon,     // 2
	    Energy,             // 3
	    SpellDeathRay,      // 4
	    SpellRabid,         // 5
	    SpellSleep,         // 6
	    SpellStoneskin      // 7
	}

This defines **CardName** as a new type that you can use to identify individual cards. The integer values will be helpful as a reference when working with the cards in a deck.

Next, you need to define some custom properties for the **Card** class. Add this between the class declaration line and **init**

	let frontTexture: SKTexture
	let backTexture: SKTexture
	var largeTexture: SKTexture?
	let largeTextureFilename: String

Replace **init(imageNamed:)** in **Card.swift** with

	init(cardNamed: CardName) {
	 
	  // initialize properties
	  backTexture = SKTexture(imageNamed: "card_back.png")
	 
	  switch cardNamed {
	  case .CreatureWolf:
	    frontTexture = SKTexture(imageNamed: "card_creature_wolf.png")
	    largeTextureFilename = "card_creature_wolf_large.png"
	 
	  case .CreatureBear:
	    frontTexture = SKTexture(imageNamed: "card_creature_bear.png")
	    largeTextureFilename = "Card_creature_bear_large.png"
	 
	  default:
	    frontTexture = SKTexture(imageNamed: "card_back.png")
	    largeTextureFilename = "card_back_large.png"
	  }
	 
	  // call designated initializer on super
	  super.init(texture: frontTexture, color: nil, size: frontTexture.size())
	 
	 
	  // set properties defined in super
	  userInteractionEnabled = true
	}

Finally, open **GameScene.swift** and change **didMoveToView()** to use the new enum instead of the string filename:

	let wolf = Card(cardNamed: .CreatureWolf)
	wolf.position = CGPointMake(100,200)
	addChild(wolf)
	 
	let bear = Card(cardNamed: .CreatureBear)
	bear.position = CGPointMake(300, 200)
	addChild(bear)

There are several changes here:

- First, you add a new type called **CardName**, the advantage of an enum like this is that the compiler knows all the possible values and it will warn you if you mistype one of the names. Additionally, Xcode autocomplete should be able to help as you type the first few characters of the name.
- Next, you create four new properties in **Card.swift** to store the values of each **SKTexture**, which will be utilized based on the state of the card. Each card requires a front image, back image and large front image. **largeTextureFilename** is there to save memory by preventing a large image from loading until it’s actually needed.
- Next you update the init method to take a **CardName** rather than a String and set each of the newly created properties based on the type of **Card**. This makes use of the new-and-improved **switch** statement in Swift. Here, switch cases do not automatically fall through. Additionally, you must either provide a **default** case, or cover all possible values. Once you have custom properties, such as attack and defense, you can assign those values inside the switch statement.

	There is a specific order you have to follow when initializing swift objects.
	- First, make sure all of the properties defined in the class have default values.
	- Second, call a **designated initializer** for the superclass.
	- Third, set any properties defined in the superclass, and call any functions you need to on the object.

- Lastly, you update the code in **GameScene.swift** to use the new init method of **Card**.

Build and run the project, and make sure that everything works just as it did before.

>**Note:** Since you’re just working with seven cards, there’s no need for anything more complicated to initialize cards. This particular strategy probably won’t work very well if you have 10′s or 100′s of cards. In that case, you’ll a system to store all of the card attributes in a configuration file, like a .json file. You’ll also want to design the init system to pull out the relevant part of the configuration as a dictionary and build the card data from that.

>**Challenge:**
Finish **Card** by adding the correct images for the other cards, such as the fierce dragon. You’ll find the images in the **cards** folder inside **Supporting Files**.

>![Image of Dragon creature card](http://cdn3.raywenderlich.com/wp-content/uploads/2014/07/dragon-card.png)
	
>*Dun Dun Dun*

##Flip Flop

Finally, add some card-like actions to make the game more realistic. Since the basic premise is that two players will share an iPad, the cards need to be able to turn face down so the other player cannot see them.

An easy way to do this is to make the card flip over when you double tap it. However, you need a property to keep track of the card state to make this possible.

Open **Card.swift** and add the following property below the other properties:

	var faceUp = true

Next, add a function to swap the textures that will make a card appears flipped:

	func flip() {
	  if faceUp {
	    self.texture = self.backTexture
	    if let damageLabel = self.childNodeWithName("damageLabel") {
	      damageLabel.hidden = true
	    }
	    self.faceUp = false
	  } else {
	    self.texture = self.frontTexture
	    if let damageLabel = self.childNodeWithName("damageLabel") {
	      damageLabel.hidden = false
	    }
	    self.faceUp = true
	  }
	}

Finally, add the following to the beginning of touchesBegan, just inside the for-in loop.

	if touch.tapCount > 1 {
	  flip()
	}

Now you understand why you saved the front and back card images as textures earlier — it makes flipping the cards delightfully easy. You also hide **damageLabel** so the number is not shown when the card is face down.

Build and run the project and flip those cards.

![Card flip](http://cdn5.raywenderlich.com/wp-content/uploads/2014/07/CardFlipNoAnim.gif)

*Simple card flip by swapping out the texture. The little bounce is the pick-up animation triggered by the first touch.*

>**Note:** At this point, it’s ideal for the damage label to be a property that initializes during Card initialization. For the sake of keeping this tutorial simple and straightforward, it is in here as a child node. Try pulling it from GameScene and putting it into Card for a little extra credit.

The effect is ok, but you can do better. One trick is to use the **scaleToX** animation to make it look as though it actually flips.

Replace **flip** with:

	func flip() {
	  let firstHalfFlip = SKAction.scaleXTo(0.0, duration: 0.4)
	  let secondHalfFlip = SKAction.scaleXTo(1.0, duration: 0.4)
	 
	  setScale(1.0)
	 
	  if faceUp {
	    runAction(firstHalfFlip) {
	      self.texture = self.backTexture
	      if let damageLabel = self.childNodeWithName("damageLabel") {
	        damageLabel.hidden = true
	      }
	      self.faceUp = false
	      self.runAction(secondHalfFlip)
	    }
	  } else {
	    runAction(firstHalfFlip) {
	      self.texture = self.frontTexture
	      if let damageLabel = self.childNodeWithName("damageLabel") {
	        damageLabel.hidden = false
	      }
	      self.faceUp = true
	      self.runAction(secondHalfFlip)
	    }
	  }
	}

The **scaleXTo** action shrinks only the horizontal direction and gives it a pretty cool 2D flip animation. The animation splits into two halves so that you can swap the texture halfway. The **setScale** function makes sure the other scale animations don’t get in the way.

Build and run the project to see the new “flip” effect in action.

![Card flip with animation](http://cdn4.raywenderlich.com/wp-content/uploads/2014/07/CardFlipWithAnim.gif)

*Now you have a nice looking flip animation.*

Things are looking great, but you can’t fully appreciate the bear’s goofy grin when the cards are so small. If only you could enlarge a selected card to see its details…

##Big Time

The last effect you’ll work with in this tutorial is modifying the double tap action so that it enlarges the card. Add these two properties to the beginning of **Card.swift** with the other properties:

	var enlarged = false
	var savedPosition = CGPointZero

Add the following method to perform the enlarge action:

	func enlarge() {
	  if enlarged {
	    enlarged = false
	    zPosition = 0
	    position = savedPosition
	    setScale(1.0)
	  } else {
	    enlarged = true
	    savedPosition = position
	    zPosition = 20
	    position = CGPointMake(CGRectGetMidX(parent.frame), CGRectGetMidY(parent.frame))
	    removeAllActions()
	    setScale(5.0)
	  }
	}

Remember to update **touchesBegan()** to call the new function, instead of **flip()**

	if touch.tapCount > 1 {
	  enlarge()
	}
	 
	if enlarged { return }

Finally, make a small update to **touchesMoved()** and **touchesEnded** by adding the following line to each before the **for-in** loop:

	if enlarged { return }

You need to add the extra property savedPosition so the card can be moved back to its original position. This is the point when touch-handling logic becomes a bit tricky, as mentioned earlier.

The tapCount check at the beginning of the function prevents glitches when the card is enlarged and then tapped again. Without the early return, the large image would shrink and start the wiggle animation.

It also doesn’t make sense to move the enlarged image, and there is nothing to do when the touch ends, so both functions return early when the card is enlarged.
Build and run the app to see the card grow and grow to fill the screen.

![Basic card enlarging.](http://cdn3.raywenderlich.com/wp-content/uploads/2014/07/CardEnlargeNoAnim.gif)

*Basic card enlarging. Would look much better with some animation, and the enlarged image is fuzzy.*

But why is it all pixelated? Vicki’s artwork is much too nice to place under such duress. You’re enlarging this way because you’re not using the large versions of the images in the **cards_large** folder inside **Supporting Files**.

Because loading the large images for all the cards at the beginning can waste memory, it’s best to make it so they don’t load until the user needs them.

The final version of the enlarge function is as follows:

	func enlarge() {
	  if enlarged {
	    let slide = SKAction.moveTo(savedPosition, duration:0.3)
	    let scaleDown = SKAction.scaleTo(1.0, duration:0.3)
	    runAction(SKAction.group([slide, scaleDown])) {
	      self.enlarged = false
	      self.zPosition = 0
	    }
	  } else {
	    enlarged = true
	    savedPosition = position
	 
	    if largeTexture != nil {
	      texture = largeTexture
	    } else {
	      largeTexture = SKTexture(imageNamed: largeTextureFilename)
	      texture = largeTexture
	    }
	 
	    zPosition = 20
	 
	    let newPosition = CGPointMake(CGRectGetMidX(parent.frame), CGRectGetMidY(parent.frame))
	    removeAllActions()
	 
	    let slide = SKAction.moveTo(newPosition, duration:0.3)
	    let scaleUp = SKAction.scaleTo(5.0, duration:0.3)
	    runAction(SKAction.group([slide, scaleUp]))
	  }
	}

The animations are fairly straightforward at this point.

The card’s position saves before running an animation, so it returns to its original position. To prevent the pickup and drop animations from interfering with the animation as it scales up, you add the **removeAllActions()** function.

When the scale down animations run, the enlarged and zPosition properties don’t set until the animation completes. If these values change earlier, an enlarged card sitting behind another card will appear to slide underneath as it returns to its previous position.

Since **largeTexture** is defined as an **optional**, it can have a value of nil, or “no value”. The if statement tests to see if it has a value, and loads the texture if it doesn’t.

>**Note:** Optionals are a core part of learning Swift, especially since it works differently than nil values in Objective-C.

Build and run the app once again. You should now see a nice, smooth animation from the card’s initial position to the final enlarged position. You’ll also see the cards in full, clean, unpixelated splendor.

![Card enlargement with animation.](http://cdn4.raywenderlich.com/wp-content/uploads/2014/07/CardEnlargeAnim.gif)

*Animating the card enlargement, and swapping to the large image make this look much nicer.*

>**Final Challenge:** Sound effects are an important part of any game, and there are some sound files included in the starter project. See if you can use **SKAction.playSoundFileNamed(soundFile:, waitForCompletion:)** to add a sound effect to the card flip, and the enlarge action.

##Where to Go from Here?

The final project for this tutorial can be found [here](https://bitbucket.org/ecerney/ccg-walkthrough-final/get/master.zip).

At this point, you understand the basic — and some not so basic — card mechanics that you can put to use in your own card game.

This sample project has many subtle animations that you can tweak, so make sure you play around with the different values to find what you like and what works for you.

Once you’re happy with the animations, there are board regions, decks, attacks and many other features that are simply too much to address in a single article like this. Take a look at the completed example game in [Objective-C](https://bitbucket.org/bcbroom/ccg/get/master.zip) and [Swift](https://bitbucket.org/bcbroom/ccg-swift/get/master.zip) to learn more about the other elements that go into game development.

Use the forum below to comment, ask questions or share your ideas for animating cards with Swift. Thanks for taking the time to work through this tutorial!
