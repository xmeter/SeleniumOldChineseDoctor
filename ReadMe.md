Selenium老中医
==============
最近学习selenium的测试同仁是越来越多，但是很多人由于是初学，感觉是一头雾水，碰到很多问题也不知道该如何去解决，所以挑选出一些比较具有代表性的问题来做解答。
怎样使用selenium ide
--------------------
对于很多人来说，学习自动化工具的第一步就是学会使用各种IDE，那么我们就从IDE的安装开始：

1. Selenium IDE其实是一系列的firefox插件的组合，如果从firefox内去搜索的话，会发现有很多的结果，而selenium官方其实是提供了整个工具的组合，访问http://docs.seleniumhq.org/download/ 就能够获取。
2. 下载完成后，直接将xpi文件拖到Firefox浏览器内就能成功安装，然后在Firefox的菜单下面的开发者选项当中就能看到。
3. 在成功安装以后，我们还需要做一些设置，打开selenium IDE，options -> options ->General ->Enable experimental features，然后在ide输入你需要录制的页面的url，同时在浏览器地址栏输入你要录制页面的url，就可以开始录制了。注意：Selenium IDE和QTP是不一样的，不是你在工具内输入url，就会调用浏览器开始录制。


如何判断使用的XPath或者css selector是否正确
----------------------------------------
很多人都想在写代码之前先验证自己所写的selector是否正确，Firefox和Chrome就提供了这样的功能。Firefox下你可以使用XPath Checker或者Firepath这两种插件，而在Chrome里面你则可以使用在开发者工具的控制台中输入以$s开头来验证css selector或者以$x来验证XPath。


处理NotConnectedException
-------------------------
很多人都会感到很疑惑，怎么我的脚本丢了一段时间就不能跑了呢，跑的时候就报这个错误：

    org.openqa.selenium.firefox.NotConnectedException: Unable to connect to host 127.0.0.1 on port 7055 after 45000 ms. Firefox console output:
    h failed when adding add-ons to cache
    *** WARN addons.repository: Search failed when adding add-ons to cache

这其实的根本原因是在于你的Firefox的版本和webdriver的版本不匹配，因为Firefox默认是自动升级的。如果碰到这种情况的话，要么去升级你的webdriver，要么就去降低你的Firefox的[版本](https://ftp.mozilla.org/pub/mozilla.org/firefox/releases/)。***在做自动化之前请关掉Firefox的自动升级。***


处理弹出框
---------
怎样处理这种[弹出框](http://www.w3schools.com/js/tryit.asp?filename=tryjs_alert)呢？
在这种弹出框内，这个button实际上是在一个frame当中的，这时候就需要先切到frame当中去，然后再按照常规操作来操作了。

    WebDriver driver = new FirefoxDriver();
    WebDriverWait wait = new WebDriverWait(driver, 30);
    driver.get("http://www.w3schools.com/js/tryit.asp?filename=tryjs_alert");
    driver.switchTo().frame("iframeResult");
    driver.findElement(By.xpath("//html/body/button")).click();
    wait.until(ExpectedConditions.alertIsPresent());
    driver.switchTo().alert().accept();
    driver.switchTo().defaultContent(); 

Chrome浏览器输入汉字乱码
----------------------
chrome浏览器输入文字乱码的主要原因是由于chrome浏览器的版本与你当前的selenium版本不匹配，因此有两种解决办法：

1. 升级你的selenium版本
2. 降低浏览器版本

但是就现在的情况来讲，chrome版本在30以上就很少会见到这种情况了。

处理类似tomcat登录界面的弹出框的方法
--------------------------------
tomcat登录界面弹出框实际上不是一个dom对象，因此selenium并不能直接操作它。这时候我们就需要借助其他方式。![pop-up](http://i.stack.imgur.com/opWtk.jpg)

方法一：http://username:password@yoururl

方法二：
1. 打开Firefox
2. 在地址栏当中输入 `about:config`
3. 找到 `network.http.phishy-userpass-length`选项，如果没找到，可以手动创建一个 (鼠标右键->新建->数字): 选项`network.http.phishy-userpass-length`，值为 `255` 
完成了这些操作以后，使用以下代码创建一个driver:

    FirefoxProfile profile = new FirefoxProfile();
    profile.SetPreference("network.http.phishy-userpass-length", 255);
    profile.SetPreference("network.automatic-ntlm-auth.trusted-uris", "YOUR HOST ADDRESS HERE");
    _driver = new FirefoxDriver(profile);

怎样操作那种只显示几秒的提示信息
----------------------------
很多前端工程师都比较爱使用那种显示几秒的浮动特效的提示信息，效果虽然很华丽，但是也给自动化工作提升了难度。因此要操作这类提示信息，我们需要使用显示等待，大概的代码如下：

    WebDriverWait wait = new WebDriverWait(driver, 15);
    wait.until(ExpectedConditions.textToBePresentInElementLocated(By.xpath("//div[@id='timeLeft']"), "Time left: 7 seconds")); 
切换frame
---------
通读selenium2.0的API可以发现，我们有3种方法来切换frame：

- frame(index)
- frame(Name of Frame [or] Id of the frame
- frame(WebElement frameElement)

而切换frame的顺序则必须从外到内一级一级递进，不过要想跳出所有frame则只需要执行`driver.switchTo().defaultContent()`。具体可以参见[这儿](http://assertselenium.com/2013/02/22/handling-iframes-using-webdriver/)。

在IE上怎样对多选框进行操作
-----------------------
有的时候，在ie上使用`.click()`对checkbox进行操作不起作用，我们可以借助模拟一个空格键的方式来达到我们勾选这个checkbox的目的：

    String currChkboxValue = checkbox.getAttribute("checked");
    if(currChkboxValue==null || "false".equalsIgnoreCase(currChkboxValue))
    {
    if(checkbox instanceof InternetExplorerElement)
    {
    checkbox.sendKeys(Keys.SPACE);
    }
    else
    checkbox.click();
    
    }


关于打开浏览器的问题的集中解决
--------------------------
从Selenium的官方文档当中，我们看到，不论是什么浏览器，好像是new一个对应类型的driver就可以打开浏览器了，但是实际使用中，我们往往会发现很多问题，那么我们就分浏览器来说说这些问题：

Firefox: 对于firefox来讲，因为selenium最初就是基于firefox的，所以firefox是容易启动的。但是要注意的是，如果你的Firefox是安装在非默认路径下，那么你就需要去设置firefox.exe的所在路径`System.setProperty("webdriver.firefox.bin","你的firefox.exe所在路径");  `,如果是默认路径下，你只需要new一个firefoxdriver就可以了。

Chrome： 对于chrome来讲，我们需要一个[chromedriver](http://chromedriver.storage.googleapis.com/index.html)来驱动浏览器,在下载完成后，设置你的chromedriver所在的路径 `System.setProperty("webdriver.chrome.driver", "你的chromedriver所在路径"); `，然后再去new一个chromedriver。

IE: 对于IE来讲，我们同样需要[iedriver](http://selenium-release.storage.googleapis.com/index.html)来驱动浏览器。但是，单单有iedriver，我们并不一定能成功调用IE.从IE7时代开始，ie着重提升了安全性，这就导致我们的程序有时候并不能成功调用ie。针对此种情况，有两种解决办法：

1. 手动去关掉ie的保护模式，并且调整安全等级为低。
2. 通过代码来设置安全等级：
	
    DesiredCapabilities ieCapabilities = DesiredCapabilities.internetExplorer();
    ieCapabilities.setCapability(InternetExplorerDriver.INTRODUCE_FLAKINESS_BY_IGNORING_SECURITY_DOMAINS,true);
    WebDriver driver = new InternetExplorerDriver(ieCapabilities);

虽然有两种方法，但是方法一不一定次次管用，所以推荐使用方法二。

火狐通行证的处理方法
------------------
很多人使用的Firefox是中国版的Firefox，这个版本在默认的profile里面添加了一些插件，因此，每当你new一个Firefoxdriver的时候，它就会弹出一个火狐通行证，这个很让人烦恼。那么怎么解决呢，方法有二：

1. 新建一个Firefox的profile，在每次new driver的时候，都调用这个profile。
2. 使用国际版的[Firefox](https://ftp.mozilla.org/pub/mozilla.org/firefox/releases/)。