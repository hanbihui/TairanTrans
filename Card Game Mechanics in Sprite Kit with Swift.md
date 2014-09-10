#在Swift中用Sprite Kit制作的卡牌游戏机制
![Example card image](http://cdn4.raywenderlich.com/wp-content/uploads/2014/07/card-article-header.png)

*学习如何实现基本的卡牌游戏机制和动画。*

在过去的20年里，人们玩过收集类卡牌游戏(CCGs)。[维基百科](http://en.wikipedia.org/wiki/Collectible_card_game)提供了比较详尽的这些游戏演变的过程，它似乎启发了角色扮演游戏（RPG）像龙与地下城。万智牌是一个现代化的CCG的一个例子。

其核心，CCGs是一组自定义卡片代表了人物，位置，能力，事件等。玩游戏前，玩家首先必须构建他们自己的牌组，然后他们才能使用他们独特的牌组来玩游戏。大多数玩家的牌组都会突出某种派别，生物或能力。

在本教程中，你将在CCG应用程序中使用Sprite Kit操纵图片卡。你会在屏幕上移动卡牌，移动它们来看哪些卡牌是有效的，翻转它们并放大它们来看卡牌上的文字 — 或欣赏艺术作品。

如果你还不熟悉SpriteKit，你可以阅读[初学者的教程](http://www.raywenderlich.com/42699/spritekit-tutorial-for-beginners)或[iOS游戏教程](http://www.raywenderlich.com/store/ios-games-by-tutorials)。如果你还不熟悉Swift，确认你已经了解了[Swift快速入门系列](http://www.raywenderlich.com/74438/swift-tutorial-a-quick-start).

##开始

由于这是一个卡牌游戏，最好的开始就是看实际的卡牌。下载[启动项目](https://bitbucket.org/bcbroom/ccg-assets-only-swift/get/master.zip)，它提供了一个iPad上横向模式下的SpriteKit项目，以及所有的图像，字体和声音文件，和你需要创建的一个功能性的示例游戏。

花一点时间来熟悉项目的文件结构和内容。你应该能看到如下的项目目录：

1. **System**: 包含设置一个SpriteKit项目的基础文件。它包含了**AppDelegate.swift**，**GameViewController.swift**和**Main.storyboard**
2. **Scenes**: 包含管理游戏内容的一个空主场景文件**GameScene.swift**。
3. **Card**: 包含管理卡牌的一个空的**Card.swift**文件。
4. **Supporting Files**: 包含你将在本教程中使用的所有图像、字体和声音文件。

这个游戏没有美工不会显得很酷，所以我想特别感谢[gameartguppy.com](http://www.gameartguppy.com/)的Vicki的美丽的卡牌插图！

##一个优雅的开始

既然我们需要使用卡牌玩卡牌游戏，我们将创建一个卡牌类来代表它们。目前**Card.swift**是一个空的Swift文件，所以找到它并添加：

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

你声明了**Card**是**SKSpriteNode**的子类。

要根据图片创建一个简单的精灵，你可以使用**SKSpriteNode(imageNamed:)**。为了保持这种行为，你需要使用继承的初始化函数，它将调用父类的指定的初始化函数**init(texture:color:size:)**。在本游戏中你不支持**NSCoding**。

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

规则 #1 创建卡牌游戏：先有创意和富有想象力的美工。看起来你的应用程序塑造的不错！

>**注：** 根据屏幕的大小，你可能需要缩放模拟器的窗口，使用Window\Scale\50%来适应屏幕。我也推荐使用iPad 2模拟器。

看这一对卡牌非常有趣，但如果你能真正的移动卡片UI将会更酷。你将在下面做到这些！

##我想移动它，移动它…

无论美工的质量如何，卡牌静坐在屏幕上不会让你的应用程序获得任何的好评，因为你需要像你在玩真实的纸质卡牌一样能拖动它们。实现它的最简单的办法是在场景中处理触摸事件。

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

为了解决这个问题，你将需要在拖动时修改卡牌的**zPosition**。你的第一反应可能是在**touchesMoved**中修改精灵的**zPosition**，但如果你想在后面把它改回来，这不是一种好的办法。

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

调整你scale和duration的值来找到最适合你的。如果你设置拿起和放下的durations为不同的值，你可以让它看起来像拿起的时候比较慢，放下的时候比较快。

##摆动，摆动，摆动

拖动卡牌现在工作得已经很好了，但你应该做得更好。让卡牌围绕y轴拍翅膀当然是很棒的。

由于SpriteKit是一个纯2D框架，似乎没有办法做到精灵的部分转动效果。然而你可以这么做，修改xScale属性来制造转动的假象。

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

要实现这一点，你需要在卡牌的顶端添加一个标签，这样用户可以追踪每个生物造成的伤害。依旧在**GameScene.swift**中，添加如下新方法：

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

试着拖动卡牌，但是点击该标签拖动时拖动的是标签而不是卡牌本身。注意标签飞到了什么地方 — 也许是一个神奇的国度在那里可以肆无忌惮的移动？

不，其他它不是那么神秘。;]

这里的问题是当你调用**nodeAtPoint**时，它返回**SKNode**最上层的任何类型的节点，在这种情况下是**SKLabelNode**。当你改变节点的位置，移动的是标签而不是卡牌。呃。。。是的，这是合乎逻辑的解释。

![Dragging on top of the damage label causes problems.](http://cdn3.raywenderlich.com/wp-content/uploads/2014/07/LabelOops.png)

*触摸顶部伤害标签的结果。哎呀。（改变背景为白色，使标签更明显）*

##场景触摸处理的利弊

在继续之前，让我们考虑考虑场景级别的触摸处理的优点和缺点。

在项目中，场景级别的触摸处理是一个好的出发点，因为它是最简单，最直接的办法。事实上，如果你的精灵有透明区域需要被忽略，比如六角格，这可能是唯一合理的解决方案。

但是，当你有复合的精灵时它就开始显出缺点了。例如，这些可能包含多个图像、标签或甚至血条。如果你有不同的规则对应不同的精灵，它可以是很笨重和复杂的。

一个好的办法是你使用**nodeAtPoint**，它总是返回一个节点。

如果你拖动到一个卡牌精灵之外会发生什么？因为**SKScene**是**SKNode**的一个子类，如果触摸位置相交没有其他节点，则场景本身会返回一个**SKNode**。

当你修改了位置并在之前做了动画，你可能不知道但你实际上应该检查**touchedNode**是不是场景本身，但因为现在是一个学习阶段，所以也没有关系。

你会很高兴的知道这里有一个更好的解决方案…

##处理那些触摸！处理它们！

你能做什么替代它呢？好吧，你可以使卡牌类负责处理它自己的触摸事件。这种方法的逻辑是相当明确的。打开**Card.swift**然后添加下列内容到**init(imageNamed:)**中：

	userInteractionEnabled = true

这使得卡牌类拦截了触摸事件而不是传递它们到场景里。SpriteKit将根据这个属性设置发送触摸事件到最上层的实例。

接下来，你需要删除这三个触摸处理函数，**GameScene.swift**中的**touchesBegan()**，**touchesMoved()**和**touchesEnded()**然后把它们添加到**Card.swift**。

原来的代码不能像拿过来就直接用，所以它需要一些变化来与节点工作。

作为一个挑战，让我们看看你能不能不看答案做出适当的改变！

提示：由于触摸事件直接发送到了正确的精灵，所以你不需要指出需要修改的精灵。

###解决方法

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

从本质上讲，这个复制了在他们当前的状态的触摸的处理函数到卡牌的实现里。主要的区别是你不再需要搜索节点树来找对应的节点。

SpriteKit调用了正确实例的函数，所以你只需要直接修改属性。

构建和运行项目，你会注意到它解决了之前乱飞的标签的问题。

![Card can be moved the same as before](http://cdn4.raywenderlich.com/wp-content/uploads/2014/07/CardWiggleX.gif)

*卡牌可以像以前一样移动。*

##两面性

现在花一点时间来研究卡牌节点是如何初始化的。目前，我们简单的使用了字符串名称来创建纹理，然后发送它到父类的初始化函数。

为了添加属性，如攻击和防御值，或神秘的魔法效果，你需要基于指定的卡牌数据设置属性和配置它们。你应该用枚举代替使用字符串来定义卡牌，因为字符串定义卡牌容易出现错别字。打开**Card.swift** 然后添加如下内容到import行和类定义中间：

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

这定义了**CardName**为一个新的类型，你可以使用它来识别卡牌。整数值作为引用在一套牌里将会很有帮助。

接下来，你需要为去**Card**类定义一些自定义属性。添加下列代码到类声明和**init**中间：

	let frontTexture: SKTexture
	let backTexture: SKTexture
	var largeTexture: SKTexture?
	let largeTextureFilename: String

替换**Card.swift**中的**init(imageNamed:)**为

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

最后，打开**GameScene.swift**然后修改**didMoveToView()**来使用新的枚举替换字符串文件名称：

	let wolf = Card(cardNamed: .CreatureWolf)
	wolf.position = CGPointMake(100,200)
	addChild(wolf)
	 
	let bear = Card(cardNamed: .CreatureBear)
	bear.position = CGPointMake(300, 200)
	addChild(bear)

下面是修改的内容：

- 首先，你添加了一个新的叫做**CardName**的类型，这种枚举类型的优点是编译器知道所有可能的值并会在你输入错误的时候警告你。另外，Xcode的自动完成功能可以在你输入名称的开始几个字符时提示你。
- 接下来，你在**Card.swift**中创建了四个新的属性来存储每个**SKTexture**的值，它将基于卡牌的状态使用。每个卡牌需要一个字体图片，背景图片和大的前面图片。**largeTextureFilename**使图片需要使用时再加载以节省内存，防止载入大图片时内存溢出。
- 接下来你更新了init方法来接受一个**CardName**而不是一个字符串，然后根据**Card**的类型设置了每个新创建的属性。这利用了Swift新的**switch**语句。这里switch的cases不会自动失败。另外，你需要提供一个**default** case，或者覆盖所有可能的值。一旦你有自定义的属性，比如攻击或防御，你可以分配这些值到switch语句中。

	当初始化swift对象时有一个特定的顺序你必须遵循。
	- 首先，确保类定义的所有属性有默认的值。
	- 其次，调用父类的**designated initializer**。
	- 第三，设置所有父类中定义的属性，然后调用你需要的对象的任意函数。

- 最后，你更新了**GameScene.swift**中的代码来使用新的**Card**的init函数。

构建和运行项目，确保一切像之前一样工作。

>**注：**因为你只操作了七个卡牌，所以不需要复杂的初始化卡片。当你有几十或几百个卡牌时这种特殊的策略可能不太好用。到那时，你需要系统的存储所有卡牌的属性到一个配置文件里，比如一个.json文件。你还需要设置初始化系统从配置文件中抽出数据做为字典来构建卡牌。

>**挑战：**
通过为其他卡牌添加正确的图片来完成**Card**，比如凶猛的龙。你将会在**Supporting Files**中的**cards**文件夹中找到图片。

>![Image of Dragon creature card](http://cdn3.raywenderlich.com/wp-content/uploads/2014/07/dragon-card.png)
	
>*Dun Dun Dun*

##触发翻转

最后，添加一些卡牌类动作来使游戏更逼真。由于基本前提是两个玩家共享一个iPad，所以卡牌需要正面朝下来使其他玩家无法看到它们。

一种实现的简单方法是当双击它时翻转卡牌。但是，你需要一个属性来追踪卡牌的状态。

打开**Card.swift**然后添加下列属性到其他属性的下面：

	var faceUp = true

接下来，添加一个交换的纹理使卡牌出现翻转效果：

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

最后，添加如下代码到touchesBegan的开始，在for-in循环中：

	if touch.tapCount > 1 {
	  flip()
	}

现在你应该明白了为什么之前我们保存了卡牌的正面和背面的图片为纹理 — 它让翻转卡牌如此简单。你还需要隐藏**damageLabel**，这样卡牌朝下时数字才不会显示。

构建和运行项目然后翻转那些卡牌。

![Card flip](http://cdn5.raywenderlich.com/wp-content/uploads/2014/07/CardFlipNoAnim.gif)

*通过交换纹理来实现卡牌的翻转。第一次点击卡牌时会触发一个小的弹跳效果。*

>**注：**在这里，伤害标签在卡牌初始化时作为属性初始化是完美的。作为保持本教程简洁的目的，它在这里还是一个子节点。试着把它从GameScene放到Card中。

效果还不错，但你还可以做得更好。一个技巧是使用**scaleToX**动画来使它看下来像真的翻转了。

用如下代码替换**flip**：

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

**scaleXTo**只收缩了水平方向，制造了一个非常酷的2D翻转动画。动画分成两半，这样你可以分别互换它们的纹理。**setScale**函数确保其他缩放动画不会影响到翻转。

构建和运行项目来看看新的“翻转”效果。

![Card flip with animation](http://cdn4.raywenderlich.com/wp-content/uploads/2014/07/CardFlipWithAnim.gif)

*现在你有了看起来很不错的翻转动画。*

看起来还不错，但卡牌这么小，你还不能完全理解熊的傻笑。所以你需要能放大选中的卡牌来看它的细节…

##放大时刻

你将在本教程学习的最后一个效果是修改双击动作来放大卡牌。添加这两个属性到**Card.swift**的最开始的其他属性的前面：

	var enlarged = false
	var savedPosition = CGPointZero

添加如下方法来执行放大动作：

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

记得更新**touchesBegan()**来调用新的函数替换**flip()**

	if touch.tapCount > 1 {
	  enlarge()
	}
	 
	if enlarged { return }

最后，给**touchesMoved()**和**touchesEnded**做一点小修改，添加如下代码到每个**for-in**循环前面：

	if enlarged { return }

你需要添加额外的属性savedPosition，这样卡牌才能移回它原来的位置。正如前面提到的，这是触摸处理逻辑比较棘手的问题。

在函数开始处的tapCount检查防止了卡牌已经被放大了再次双击的错误。没有提前退出，大图片会收缩并开始摆动动画。

移动放大的图片也是没有意义的，所以触摸结束后没有什么可做的，所以当卡牌放大时所有函数都提前退出了。

构建和运行应用程序，查看卡牌放大到填充屏幕。

![Basic card enlarging.](http://cdn3.raywenderlich.com/wp-content/uploads/2014/07/CardEnlargeNoAnim.gif)

*基础卡牌放大效果。动画看起来好多了，放大的图片是模糊的。*

但是为什么它是像素化的？Vicki的美工在这种情况下可应该是很漂亮的。你放大成这样是因为你没有使用**Supporting Files**里**cards_large**文件夹里的放大版本的图片。

因为一开始就加载所有卡牌的大图片会浪费内存，所以最好在用户不需要它们的时候不加载它们。

放大函数的最终版本如下：

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

动画现在非常明确了。

卡牌的位置在运行动画之前保存了，所以它返回了它原来的位置。为了防止它放大时被拿起和放下动画打断，你添加了**removeAllActions()**函数。

当缩小动画运行时，enlarged和zPosition属性没有设置直到动画完成。如果这些值在完成前被修改了，被放大的卡牌后面的卡牌将会显示出来，它也会返回它之前的位置。

由于**largeTexture**被定义为**optional**，它的值可以为nill，或“没有值”。if语句测试了它，来看它是否有值，如果它没有则加载纹理。

>**注：** Optional是学习Swift的一个核心部分。特别是它不同于Objective-C中的**nil**值。

再次构建和运行应用程序。你现在应该看到一个从初始位置到最终的放大的位置漂亮的，平滑的动画。你也会看到卡牌是非像素化的、干净的、填充的。

![Card enlargement with animation.](http://cdn4.raywenderlich.com/wp-content/uploads/2014/07/CardEnlargeAnim.gif)

*把卡牌变大，然后换成大图片让它看起来更好。*

>**最终挑战：**音效是任何游戏的一个重要部分，启动项目中包含了很多声音文件。看看你能不能使用**SKAction.playSoundFileNamed(soundFile:, waitForCompletion:)**来添加音效到卡牌翻转中和放大的动作中。

##下一步？

你可以在[这里](https://bitbucket.org/ecerney/ccg-walkthrough-final/get/master.zip)找到本教程的最终项目。

到了这里，你已经理解了一些可以在你自己的卡牌游戏中使用的基础的 — 和一些不那么基础的 — 卡牌机制。

这个示例项目中有很多微秒的动画你可以调整，所以确保你已经试过不同的值来找到你喜欢的和适合你的。

一旦你对动画满意了，这里还有板区域，牌组，攻击和很多其他特性，内容很多所以不能简单像这篇一样的一篇文章中全部讲完。你可以查看用[Objective-C](https://bitbucket.org/bcbroom/ccg/get/master.zip)和[Swift](https://bitbucket.org/bcbroom/ccg-swift/get/master.zip)完成的示例游戏，了解更多有关游戏开发的其他元素。

请使用论坛在下面评论、问问题或分享你对Swift的卡牌动画的想法。感谢你抽出宝贵的时间学习本教程！
