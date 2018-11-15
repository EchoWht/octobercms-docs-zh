# 哈希和加密

- [配置](#configuration)
- [哈希](#hashing)
- [加密](#encryption)

<a name="configuration"></a>
## 配置

当您第一次安装October时，应该为您生成一个随机密钥。 您可以通过检查`config / app.php`配置文件的`key`选项来确认这一点。 如果密钥保持不变，则应将其设置为32个字符的随机字符串。 如果未正确设置此值，则所有加密值都将不安全。

<a name="hashing"></a>
## 哈希

`Hash`facade提供安全的Bcrypt哈希来存储用户密码。 Bcrypt是哈希密码的绝佳选择，因为它的“工作因子”是可调整的，这意味着随着硬件功率的增加，生成哈希所需的时间也会增加。

您可以通过调用`Hash`facade上的`make`方法来哈希密码：

    $user = new User;
    $user->password = Hash::make('mypassword');
    $user->save();

或者，模型可以实现[Hashable trait](../database/traits#hashable) 来自动哈希属性。

#### 根据哈希验证密码

`check`方法允许您验证给定的纯文本字符串是否对应于给定的哈希。

    if (Hash::check('plain-text', $hashedPassword)) {
        // The passwords match...
    }

#### 检查是否需要重新验证密码

`needsRehash`函数允许您确定自密码被哈希以来哈希使用的工作因子是否已更改：

    if (Hash::needsRehash($hashed)) {
        $hashed = Hash::make('plain-text');
    }

<a name="encryption"></a>
## 加密

您可以使用`Crypt`facade加密值。 所有加密值都使用OpenSSL和`AES-256-CBC`密码加密。 此外，所有加密值都使用消息验证代码（MAC）进行签名，以检测对加密字符串的任何修改。

例如，我们可以使用`encrypt`方法加密并将其存储在[数据库模型](../database/model)中：

    $user = new User;
    $user->secret = Crypt::encrypt('shhh no telling');
    $user->save();

#### 解密一个值

当然，您可以使用`Crypt`facade上的`decrypt`方法解密值。 如果无法正确解密该值，例如当MAC无效时，将抛出`Illuminate\Contracts\Encryption\DecryptException` 异常：

    use Illuminate\Contracts\Encryption\DecryptException;

    try {
        $decrypted = Crypt::decrypt($encryptedValue);
    }
    catch (DecryptException $ex) {
        //
    }
