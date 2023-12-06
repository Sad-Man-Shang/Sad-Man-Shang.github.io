![image-20231027142546877](/Users/shengquan/Library/Application Support/typora-user-images/image-20231027142546877.png)





## 用户登陆过程需要做哪些分析？

1. 功能测试
   - 正常流程（正确账号密码，点击提交，验证能否正确登陆）
   - 异常流程（错误的账号密码，点击提交，验证登陆失败，并提示相应错误信息）
   - 登陆成功后能否正确跳转
   - 用户名和密码，太短或太长的处理（边界值法）
   - 用户名和密码，有特殊字符（比如空格）及其他非英文的情况
   - 记住用户名，记住密码
   - 登陆失败后，不记录密码
   - 用户名和密码前后有空格的处理
   - 密码是否是密文显示，使用“*”号或圆点等符号代替
   - 验证码的辨认难度，考虑颜色（色盲使用者），刷新或换一个按钮是否好用
   - 输入密码时，大写键盘开启时是否有提示信息
   - 什么都不输入，点击提交按钮，检查提示信息
   - 登陆token测试
2. 界面测试
   - 布局是否合理，按钮和表单是否整齐
   - 按钮和表单高度和长度是否符合要求
   - 界面风格是否符合UI设计稿
   - 文字有无错别字
3. 性能测试
   - 打开登陆页面，需要的时间是否在需求要求的时间内
   - 输入正确的账号密码，点击登陆，是否在需求时间内跳转成功
   - 模拟大量用户同时登陆，检查一定压力下能否正常跳转
4. 安全性测试
   - 用户名或密码是否是通过加密方式，发送给后端服务器
   - 用户名和密码应该在前端和后端做双重验证
   - 防止CSRF攻击，是否存在token
   - 用户名和密码的输入框，应该屏蔽SQL注入攻击
   - 用户名和密码的输入框，应该禁止输入脚本（防止xss攻击）
   - 防止暴力破解，检测是否有错误登陆的次数限制
   - 是否支持多用户在同一机器上登陆
   - 同一用户能否在多台机器上登陆
5. 可用性测试
   - 是否可以用全键盘操作，是否有快捷键
   - 输入用户名，密码后按回车，是否可以登陆
   - 输入框是否可以Tab切换
6. 兼容性测试
   - 不同浏览器下能否显示正常，且功能正常（IE,6,7,8,9,10,11,Firefox,Chrome,Safari）等
   - 同种浏览器下不同版本能否显示正常且功能正常
   - 不同的操作系统是否能正常工作（Windows, Mac）
   - 移动设备上是否正常工作（iOS，Android）
7. 本地化测试
   - 不同语言环境下，页面的显示是否正确





120

600/m

9.30-6.30    2h

9.30， free宵夜+打车

200





# 测试用例设计

### 测试用例的格式

普遍是 Excel 或 Xmind。

Excel 优势是比较细化，可以突出更多的测试要素，适用于等价划分类等黑盒测试设计思路，也适用于输入输出的场景；缺点是结构化不直观，不好体现功能需求，用例数过于臃肿。

Xmind 优势是大部分只需要列出测试点，更加注重探索性测试，能够更好的去描述功能需求，结构化展示比较直观，比较契合产品PRD；缺点是不太适用于输入输出的场景，测试细节不好表达。

------

### 测试用例优先级

- 高（优先执行）：产品基本的功能验证，即关键路径的测试用例，包括最常执行的功能、基本流程的输入（正向流程+正向数据）。
- 中（次级执行）：包括界面数据有效性校验、默认值、边界值。
- 低（最后执行）：建议执行的测试用例，包括不常执行的功能、异常流程的输入以及异常数据的输入。

------

### 测试用例要素

1. 用例标识（id）：唯一的标识号，用以区别其他测试用例
2. 用例标题：表达测试用例的用途
3. 优先级：高中低
4. 前置条件：软硬件
5. 输入
6. 操作步骤
7. 预期结果
8. 作者（选填）

------

### 测试方案设计应从哪些方面去考虑？

无非是从以下方面去考虑，其中，功能测试我认为是最重要的一个环节。

- 功能测试（最重要）
- 文档测试
- UI测试
- 接口测试
- 性能测试（压力、负载）
- 安全测试
- 稳定性测试（Monkey、遍历测试等）
- 异常测试（断网/弱网）
- 兼容性测试（安卓、IOS系统版本以及APP新老版本）
- 易用性测试
- 可用性测试
- 配置测试









# 用例设计

![image-20231027141836927](/Users/shengquan/Library/Application Support/typora-user-images/image-20231027141836927.png)









# 测试理论基础

### 什么是软件测试？

答：软件测试是在规定的条件下对程序进行操作，以发现错误，对软件质量进行评估。

------

### 软件测试的目的是什么？

答：软件测试的目的在于（1）**发现软件的缺陷和错误**（2）**保证软件的质量**，**确保能够满足用户以及产品的需求**。

（标重点）**软件测试的目的是为了找bug，并不是验证软件没有bug**。

------

### 黑盒与白盒的定义

- **白盒：**

白盒测试是测试人员要了解**程序结构和处理过程**,按照**程序内部逻辑**测试程序,检查程序中的每条通路是否按照预定要求正确工作.它主要的针对被测程序的**源代码**。

不太需要关注程序功能

- **黑盒：**

黑盒测试又称为功能测试或数据驱动测试）是把测试对象看作一个黑盒子。利用黑盒测试法进行动态测试时，需要**测试软件产品的功能**，不需测试软件产品的**内部结构和处理过程**。

------

### 白盒测试用例设计常用方法

答：

静态测试：不用运行程序的测试，如文档测试、代码检查等

动态测试：需要执行代码，接口测试、覆盖率分析、性能分析、内存分析等。

**逻辑覆盖法**：主要包括**语句覆盖，判断覆盖，条件覆盖，判断/条件覆盖，条件组合覆盖，路径覆盖**等。

六种覆盖标准发现错误的能力由弱到强的变化：

1. **语句覆盖**，每条语句至少执行一次。
2. 判断覆盖，每个判断的每个分支至少执行一次。
3. 条件覆盖，每个判段的每个条件应取到的各种可能的值。
4. 判断/条件覆盖，同时满足判断覆盖条件覆盖。
5. 条件组合覆盖，每个判定中各条件的每一种组合至少出现一次。
6. **路径覆盖**，使程序中每一条可能的路径至少执行一次。

------

### 黑盒测试用例设计常用方法

答：**等价划分类，边界值分析，错误推测法**、**判定表分析法、因果图法、正交试验设计法、场景法、功能图分析法**等。

等价类：等价类是指某个输入域的子集合.在该子集合中,各个输入数据对于揭露程序
中的错误都是等效的.因此,可以把全部输入数据合理划分为若干等价类,在每一个等价类中取一个数据作为测试的输入条件,就可以用少量代表性的测试数据.取得较好的测试结果.
等价类划分可有两种不同的情况:有效等价类和无效等价类.

边界值分析法：边界值分析方法是对等价类划分方法的补充。测试工作经验告诉我,大量的错误是发生在输入或输出范围的边界上,而不是发生在输入输出范围的内部.因此针对各种边界情况设计测试用例,可以查出更多的错误.(内点，上点，离点)

错误猜测法：基于经验和直觉推测程序中所有可能存在的各种错误,从而有针对性的设计测试用例的方法.

判定表分析法：等价类未考虑输入条件之间的联系,相互组合等.无效类每个用例只出现一次，不能覆盖多个无效等价类并存的情况。不同的组合用二进制解决。(条件桩和动作桩)

因果图法：输入之间有相互的组合关系，且输入和输出之间有相互的制约和依赖关系，是判定表的复杂版，最终生成的就是判定表，根本上等价

正交表分析法：有时候，大量的参数的组合而引起测试用例数量上的激增，同时，这些测试用例并没有明显的优先级上的差距，而测试人员又无法完成这么多数量的测试，就可以通过正交表来进行缩减一些用例，从而达到尽量少的用例覆盖尽量大的范围的可能性。(打印机)

场景法：指根据用户场景来模拟用户的操作步骤，一般借助流程图来确定基本流和备选流(淘宝下单流程)。

状态图法：针对流程分析法不跨多个界面，状态图法保证每一个功能/状态的可达项都被覆盖，但对无效的路径无法覆盖(MP3).

------

### 什么是灰盒测试？

答：灰盒测试，是介于[白盒测试](https://baike.baidu.com/item/白盒测试/934440)与[黑盒测试](https://baike.baidu.com/item/黑盒测试/934030)之间的一种测试，灰盒测试多用于[集成测试](https://baike.baidu.com/item/集成测试/1924552)阶段。目前互联网的测试大多数都是灰盒测试。

------

### 列举出你所了解的软件测试方式

答：

按照软件的**生命周期**划分：**单元测试、集成测试、系统测试、回归测试、验收测试**。

按照**测试关注点**划分：**功能测试、性能测试、稳定性测试、易用性测试、安全性测试**。

按照**测试实施者**划分：**开发方测试（α测试）、用户测试（β测试）、第三方测试**。

按照**技术/测试用例设计**划分：**白盒测试、黑盒测试、灰盒测试**。

按照**分析方法**划分：**静态测试、动态测试**。

按照**测试执行方式**划分：**手工测试、自动化测试**。

按照**测试对象**划分：**程序测试、文档测试**。

------

### 什么是单元测试

答：完成最小的软件设计单元（模块）的验证工作，确保模块被正确编码。通常情况下是白盒的，对代码风格和规则、程序设计和结构、业务逻辑等进行静态测试，及早发现和解决不易显现的错误。

------

### 单元测试、集成测试、系统测试、验收测试、回归测试这几步最重要的是哪一步？

答：这些测试步骤分别在软件开发的不同阶段对软件进行测试，我认为**对软件完整功能进行测试的系统测试**很重要，因为此时单元测试和集成测试已完成，**系统测试能够对软件所有功能进行功能测试，能够覆盖系统所有联合的部件，是针对整个产品系统进行的测试**，能够验证系统是否满足需求规格的定义，因此，我认为系统测试很重要。

------

### 集成测试和系统测试的区别，以及应用场景分别是什么？

答：

区别：

- 执行顺序：先执行集成测试，待集成测试问题修复后，再做系统测试。
- 用例粒度：集成测试比系统测试用例更详细，集成测试对于接口部分也要重点写，而系统测试的用例更接近用户接受的测试用例。

应用场景：

- 集成测试：一般包含接口测试，对程序的提测部分进行测试。测试方法一般选用黑盒测试和白盒测试相结合。
- 系统测试：针对整个产品的全面测试，既包含各模块的验证性测试和功能性测试，又包含对整个产品的健壮性、安全性、可维护性及各种性能参数的测试。测试方法一般采用黑盒测试法。

------

### 测试开发需要哪些知识？具备哪些能力？

答：

需要的知识：

**软件测试基础理论知识**，如黑盒测试、白盒测试等；

**编程语言基础**，如C/C++、java、python等；

**自动化测试工具**，如Selenium、Appium等；

**计算机基础知识**，如数据库、Linux、计算机网络等；

**测试卡框架**，如JUnit、Pytest、Unittest等。

具备的能力：

业务分析能力、缺陷洞察能力、团队协作能力、专业技术能力、逻辑思考能力、问题解决能力、沟通表达能力和宏观把控能力。

------

### 请说一下手动测试与自动化测试的优缺点

答：

手工测试缺点：

1. 重复的手工回归测试，代价昂贵、容易出错。
2. 依赖于软件测试人员的能力。

手工测试的优点：

1. 测试人员具有经验和对错误的猜测能力。
2. 测试人员具有审美能力和心理体验。
3. 测试人员具有是非判断和逻辑推理能力。

自动化测试的缺点：

1. 不能取代手工测试。
2. 无法运用在测试复杂的场景
3. 手工测试比自动化测试发现的缺陷更多。
4. 对测试质量的依赖性极大。
5. 自动化测试不能提高有效性。
6. 比手动测试脆弱，需要维护成本。
7. 工具本身并无想象力。

自动化测试的优点：

1. 对程序的回归测试更方便。
2. 可以运行更多更繁琐的测试。
3. 可以执行一些手工测试困难或不可能进行的测试。
4. 更好地利用资源。
5. 测试具有一致性和可重复性。
6. 测试的复用性。
7. 增加软件的信任度。

------

### 自动化测试的定义 前提条件 适用的测试类型

答：

**定义：**
自动化测试是把以人为驱动的测试行为转化为机器执行的一种过程。通常，在设计了测试用例并通过评审之后，由测试人员根据测试用例中描述的规程一步步执行测试，得到实际结果与期望结果的比较。在此过程中，为了节省人力、时间或硬件资源，提高测试效率，便引入了自动化测试的概念。

**前提条件：**
实施自动化测试之前需要对软件开发过程进行分析，以观察其是否适合使用自动化测试。通常需要同时满足以下条件：

![image](https://note.youdao.com/yws/public/resource/3c02a6da189621dc689a5c2da5d765d7/BB9450F3E5214D16B217A725BC0D70AF/6F3457C7D18A41BB8D076732DC822D78?ynotemdtimestamp=1698387538831)

**适用的测试类型：**

- 回答一：

![image](https://note.youdao.com/yws/public/resource/3c02a6da189621dc689a5c2da5d765d7/BB9450F3E5214D16B217A725BC0D70AF/A396BF9B83D442F4B90E152C48533DDF?ynotemdtimestamp=1698387538831)

- 回答二：

![image](https://note.youdao.com/yws/public/resource/3c02a6da189621dc689a5c2da5d765d7/BB9450F3E5214D16B217A725BC0D70AF/AD0BE4633DF14F06B14354596507D1AC?ynotemdtimestamp=1698387538831)
![image](https://note.youdao.com/yws/public/resource/3c02a6da189621dc689a5c2da5d765d7/BB9450F3E5214D16B217A725BC0D70AF/35771DBD9C334B15BA8D844641A7F064?ynotemdtimestamp=1698387538831)

------

### 自动化测试的运用场景举例

答：

1. 线上巡检（UI+接口）
2. 简单场景监控
3. 稳定性测试（monkey+遍历测试）

------

### 软件测试的核心竞争力是什么？

答：**早发现问题**和**发现别人无法发现的问题**。

------

### 测试和开发要怎么结合才能使软件的质量得到更好的保障

答：测试和开发可以按照V模型或W模型的方式进行结合。但应该按照W模型的方式进行结合比较合理。

V模型：

![v.jpg](http://ww1.sinaimg.cn/mw690/00788Gqbgy1g6ng9otngfj30ao0650th.jpg?ynotemdtimestamp=1698387538831)

测试过程加在开发过程的后半段，比较被动。

W模型：

![w.jpg](http://ww1.sinaimg.cn/large/00788Gqbgy1g6ngb9lyukj30co092wfe.jpg?ynotemdtimestamp=1698387538831)

测试提前，甚至和开发是同步进行，测试不仅是程序，还包括需求和设计。W模型有利于尽早地全面的发现问题，降低软件开发的成本，风险前置。

------

### 怎么实施自动化测试（过程）

答：

1. 首先判断项目适不适合进行自动化测试。
2. 对项目做需求分析。
3. 制定测试计划和测试方案。
4. 搭建自动化测试框架。
5. 设计或编写测试用例。
6. 执行自动化测试。
7. 评估。

------

### 测试的相关流程

答：

按W模型：

需求测试 -> 概要设计测试 -> 详细设计测试 -> 单元测试 -> 集成测试 -> 系统测试 -> 验收测试

我工作中实际测试流程：

需求评审 -> 技术评审 -> case评审 -> 开发自测以及冒烟测试 -> 整体提测（集成测试） -> 回归测试 -> 系统测试 -> 验收测试

测试流程：了解用户需求-->测试计划（测试范围，人力物力时间进度的安排）-->编写测试用例-->评审用例-->搭建环境-->测试包安排预测（冒烟测试）-正式测试-bug管理-测试结束出报告，决定是否上线-->版本上线-->面向用户

------

### 测试项目具体工作是什么

答：

1. 搭建测试环境
2. 撰写测试用例
3. 执行测试用例
4. 写测试计划、测试报告
5. 测试并提交BUG
6. 跟踪BUG修改情况
7. 自动化测试
8. 性能测试、压力测试、安全测试等其他测试

------

### BUG分级

答：两个维度去划分

1. 按BUG严重程度划分等级：
   - blocker：系统无法执行，崩溃，或严重资源不足，应用模块无法启动或异常退出，无法测试，系统不稳定。常见的：严重花屏、内存泄漏、用户数据丢失或破坏、系统崩溃/死机/冻结、模块无法启动或异常退出、严重的数值计算错误、功能设计与需求严重不符、服务器500等。
   - critical：影响系统功能或操作，主要功能存在严重缺陷，但不会影响到系统的稳定性。常见的有：功能未实现，功能错误、系统刷新错误、数据通讯错误、轻微的数值计算错误、影响功能及洁面的错别字或拼写错误。
   - major：界面、性能缺陷、兼容性。常见的有：操作界面错误、边界条件错误、提示信息错误，长时间操作无法提示、系统未优化、兼容性问题。
   - minor/trivial：易用性及建议性的问题。
2. 按BUG处理优先级划分：
   - immediate：马上解决
   - urgent：急需解决
   - high：高度重视，有时间马上解决
   - low：在系统发布前解决或确认可以不用解决

------

### 性能指标有哪些？

![image](https://note.youdao.com/yws/public/resource/3c02a6da189621dc689a5c2da5d765d7/BB9450F3E5214D16B217A725BC0D70AF/73C79D59355B4713BAC68CBD051F5911?ynotemdtimestamp=1698387538831)

![image](https://note.youdao.com/yws/public/resource/3c02a6da189621dc689a5c2da5d765d7/BB9450F3E5214D16B217A725BC0D70AF/53A13A700A6240BFA74FF3EDC0DB296C?ynotemdtimestamp=1698387538831)

![image](https://note.youdao.com/yws/public/resource/3c02a6da189621dc689a5c2da5d765d7/BB9450F3E5214D16B217A725BC0D70AF/0733C86A23E94D91B0817FA06BAD9517?ynotemdtimestamp=1698387538831)

------

### APP性能指标有哪些？

答：内存、CPU、流量、电量、启动速度、滑动速度、界面切换速度、与服务器交互的网络速度。

------

### APP测试工具有哪些？

接口测试：postman

性能测试：jmeter

抓包工具：chales、fiddler

UI自动化：uiautomator2、appium、atx

稳定性测试：monkey、maxim、uicrawler、appcrawler

兼容性测试：wetest、testin

内存、cpu、电量测试：GT、soloPi

弱网测试：chales

------

### BUG的生命周期

答：

**复杂版**：

1. New（新的）
2. Assigned（已指派）
3. Open（打开的）
4. Fixed（已修复）
5. Pending Reset（待测试）
6. Reset（再测试）
7. Closed（已关闭）
8. Reopen（再次打开）
9. Pending Reject（拒绝中）
10. Rejected（被拒绝的）
11. Postponed（延期）

**简单版**：

1. 创建bug
2. 分配bug
3. 修复完待测试
4. 关闭
5. 重新开启
6. 无效

------

### 什么是α测试和β 测试？

答：

**α测试**：在受控的环境下进行，由用户在开发者的场所进行，开发者指导用户测试，开发者负责记录发现的错误和使用中遇到的问题。

**β 测试**：在开发者不可控的环境下进行，由软件最终用户在一个或多个客户场所下进行，用户记录测试中遇到的问题，并定期上报给开发者。

------

### 谈谈对敏捷的理解

- **简单谈四种开发模型**：

传统的**瀑布式开发**，也就是从制定计划到需求分析到系统设计，到编码，到测试，到提交到运维大概这样的流程，要求每一个开发阶段都要做到最好。

**迭代式开发**，不要求每一个阶段的任务做的都是最完美的，而是明明知道还有很多不足的地方，却偏偏不去完善它，而是把主要功能先搭建起来为目的，以最短的时间，最少的损失先完成一个“不完美的成果物”直至提交。然后再通过客户或用户的反馈信息，在这个“不完美的成果物”上逐步进行完善。

**螺旋开发**，很大程度上是一种风险驱动的方法体系，因为在每个阶段之前及经常发生的循环之前，都必须首先进行风险评估。

**敏捷开发**，相比迭代式开发两者都强调在较短的开发周期提交软件，但是，敏捷开发的周期可能更短，并且更加强调队伍中的高度协作。

- **谈敏捷测试**：

敏捷开发也对应着有敏捷测试，测试环节贯穿整个迭代周期，从需求评审到发布上线，都离不开测试快速跟进。

测试左移：需求评审、用例设计、自测工具、静态代码扫描等；

测试中：业务测试，接口测试，性能测试等；

测试右移：稳定性测试，回归测试，灰度测试等

------

### 什么是压力测试？压力测试需要考虑哪些因素？

答：

压力测试是在高负载情况下，对系统稳定性进行测试。在高负载的情况下，系统出现异常的概率要比正常负载时要高。高负载包含**长时间运行、大数据、高并发**等情况。

在做压力测试时，一般要考虑**环境因素、性能指标、运行时间**等要素。

1. 压测环境最好和生产环境一致，假如要在生产环境进行压测，需要在凌晨等在线用户量极少的情况下进行。在生产环境测试时要做好数据隔离，生产环境需提供虚拟数据，采用虚拟账号，避免对真实线上用户造成影响。
2. 性能指标包括，内存、CPU、吞吐量、QPS、网络流量、错误统计等，这些指标需要监控。
3. 压测一般需要运行长时间，最好能够通过长时间的压测，绘制出曲线图，这样更容易观察到性能瓶颈。

------

### web测试和APP测试的不同？

![image](https://note.youdao.com/yws/public/resource/3c02a6da189621dc689a5c2da5d765d7/BB9450F3E5214D16B217A725BC0D70AF/2B70C8B69E594A218D6E0975A99F4ADE?ynotemdtimestamp=1698387538831)