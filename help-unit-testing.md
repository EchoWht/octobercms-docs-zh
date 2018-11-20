# 单元测试

- [测试插件](#testing-plugins)
- [系统测试](#testing-system)

<a name="testing-plugins"></a>
## 测试插件

可以通过在基本插件目录中运行`phpunit`来执行插件单元测试。

### 系统测试

可以通过在基本目录中创建一个名为`phpunit.xml`的文件来测试插件，其中包含以下内容，例如，在文件**/plugins/acme/blog/phpunit.xml**中：

    <?xml version="1.0" encoding="UTF-8"?>
    <phpunit backupGlobals="false"
             backupStaticAttributes="false"
             bootstrap="../../../tests/bootstrap.php"
             colors="true"
             convertErrorsToExceptions="true"
             convertNoticesToExceptions="true"
             convertWarningsToExceptions="true"
             processIsolation="false"
             stopOnFailure="false"
             syntaxCheck="false"
    >
        <testsuites>
            <testsuite name="Plugin Unit Test Suite">
                <directory>./tests</directory>
            </testsuite>
        </testsuites>
        <php>
            <env name="APP_ENV" value="testing"/>
            <env name="CACHE_DRIVER" value="array"/>
            <env name="SESSION_DRIVER" value="array"/>
        </php>
    </phpunit>

然后可以创建 **tests/** 目录以包含测试类。 文件结构应该使用具有`Test`后缀的类来模仿基目录。 还建议为该类使用命名空间。

    <?php namespace Acme\Blog\Tests\Models;

    use Acme\Blog\Models\Post;
    use PluginTestCase;

    class PostTest extends PluginTestCase
    {
        public function testCreateFirstPost()
        {
            $post = Post::create(['title' => 'Hi!']);
            $this->assertEquals(1, $post->id);
        }
    }

测试类应该扩展基类`PluginTestCase`，这是一个特殊的类，它将设置存储在内存中的October数据库，作为`setUp`方法的一部分。 它还将刷新正在测试的插件以及插件注册文件中的任何已定义的依赖项。 这相当于在每次测试之前运行以下内容：

    php artisan october:up
    php artisan plugin:refresh Acme.Blog
    [php artisan plugin:refresh <dependency>, ...]
    
> **注意:** 如果你的插件使用[配置文件](../plugin/settings#file-configuration)，那么你需要在`setUp中运行`System\Classes\PluginManager::instance()->registerAll(true);`测试方法。 下面是一个基本测试用例类的示例，如果您需要测试插件使用其他插件而不是单独使用，则应使用该示例。

    use System\Classes\PluginManager;
        
    class BaseTestCase extends PluginTestCase
    {
        public function setUp()
        {
            parent::setUp();

            // 获取插件管理器
            $pluginManager = PluginManager::instance();
            
            // 注册插件以使文件配置等功能可用
            $pluginManager->registerAll(true);

            // 引导所有插件以使用此插件的依赖项进行测试
            $pluginManager->bootAll(true);
        }

        public function tearDown()
        {
            parent::tearDown();

            // 获取插件管理器
            $pluginManager = PluginManager::instance();
            
            // 确保再次注册插件以进行下一次测试
            $pluginManager->unregisterAll();
        }
    }

<a name="testing-system"></a>
## 系统测试

要对October核心文件执行单元测试，您应该使用composer下载开发副本或克隆git repo。 这将确保您拥有`tests/`目录。

### 单元测试

可以通过在根目录或`/tests/unit`中运行`phpunit`来执行单元测试。

### 功能测试

可以通过在`/ tests / functional`目录中运行`phpunit`来执行功能测试。 确保满足以下配置：

- 正在使用的主题是 `demo`
- 语言是`en`

#### 安装Selenium

1. 从http://java.sun.com/下载最新的Java SE并安装
1. 下载[Selenium Server](http://seleniumhq.org/download/)的分发档案。
1. 解压缩分发存档，并将selenium-server-standalone-2.42.2.jar（检查版本后缀）复制到/usr/local/bin。
1. 运行`java -jar /usr/local/bin/selenium-server-standalone-2.42.2.jar`启动Selenium Server服务器。

#### Selenium配置

在根目录中创建一个新文件`selenium.php`，添加以下内容：

    <?php

    // Selenium server details
    define('TEST_SELENIUM_HOST', '127.0.0.1');
    define('TEST_SELENIUM_PORT', 4444);
    define('TEST_SELENIUM_BROWSER', '*firefox');

    // Back-end URL
    define('TEST_SELENIUM_URL', 'http://localhost/backend/');

    // Active Theme
    define('TEST_SELENIUM_THEME', 'demo');

    // Back-end credentials
    define('TEST_SELENIUM_USER', 'admin');
    define('TEST_SELENIUM_PASS', 'admin');
