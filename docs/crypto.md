# crypto 模块

## 哈希算法

哈希算法是从任意文件生成短数字的算法，对于相同文件（数据）始终返回相同的值

严格来说，哈希算法并不属于对数据的加密，而属于对数据的转换

常用的哈希算法有md5 sha1 sha-256 等

### crypto.getHashes() 返回支持的加密算法数组

```javascript
crypto.getHashes()
// [
// 'RSA-MD5',
//     'RSA-RIPEMD160',
//     'RSA-SHA1',
//     'RSA-SHA1-2',
//     'RSA-SHA224',
//     'RSA-SHA256',
//     'RSA-SHA3-224',
//     'RSA-SHA3-256',
//     'RSA-SHA3-384',
//     'RSA-SHA3-512',
//     'RSA-SHA384',
//     'RSA-SHA512',
//     'RSA-SHA512/224',
//     'RSA-SHA512/256',
//     'RSA-SM3',
//     'blake2b512',
//     'blake2s256',
//     'id-rsassa-pkcs1-v1_5-with-sha3-224',
//     'id-rsassa-pkcs1-v1_5-with-sha3-256',
//     'id-rsassa-pkcs1-v1_5-with-sha3-384',
//     'id-rsassa-pkcs1-v1_5-with-sha3-512',
//     'md5',
//     'md5-sha1',
//     'md5WithRSAEncryption',
//     'ripemd',
//     'ripemd160',
//     'ripemd160WithRSA',
//     'rmd160',
//     'sha1',
//     'sha1WithRSAEncryption',
//     'sha224',
//     'sha224WithRSAEncryption',
//     'sha256',
//     'sha256WithRSAEncryption',
//     'sha3-224',
//     'sha3-256',
//     'sha3-384',
//     'sha3-512',
//     'sha384',
//     'sha384WithRSAEncryption',
//     'sha512',
//     'sha512-224',
//     'sha512-224WithRSAEncryption',
//     'sha512-256',
//     'sha512-256WithRSAEncryption',
//     'sha512WithRSAEncryption',
//     'shake128',
//     'shake256',
//     'sm3',
//     'sm3WithRSAEncryption',
//     'ssl3-md5',
//     'ssl3-sha1'
// ]

```

### crypto.createHash() 创建哈希算法加密

```javascript
// 创建MD5加密
const md5 = crypto.createHash('md5')
// 创建SHA1加密
const sha1 = crypto.createHash('sha1')
// 创建SHA-256加密
const sha256 = crypto.createHash('sha-256')
```

### update() 对参数进行加密

update的返回值为当前实例，因此可进行链式调用

update函数可以多次调用对数据进行分段加密

```javascript
const md5_1 = crypto.createHash('md5')
const md5_2 = crypto.createHash('md5')
let result1 = md5_1.update('123456789').digest('hex')
let result2 = md5_2.update('12345').update('6789').digest('hex')

console.log(result1 === result2) // true
```

```javascript
// 通过MD5算法对参数进行加密
const md5Res = md5.update('123456789')
// 通过SHA1算法对参数进行加密
const sha1Res = sha1.update('123456789')
// 通过SHA-256算法对参数进行加密
const sha256Res = sha256.update('123456789')
```

### digest() 返回加密结果，默认返回Buffer格式

```javascript
// 返回Buffer格式加密结果
md5Res.digest()
md5Res.digest('binary')
// 返回base64格式加密结果
md5Res.digest('base64')
// 返回16进制加密结果
md5Res.digest('hex')
// 返回base64url格式加密结果
md5Res.digest('base64url')
```

## 对称加密

对称加密是指加密与解密使用的是相同的密钥

即使用同一个密钥可以将明文进行加密，也可以将加密数据转换为明文

### crypto.createCipheriv('算法','密码','初始化向量') 创建加密算法

ecb模式的算法不需要初始向量，相同数据产生相同结果，安全性低，简单

cbc模式的算法需要初始化向量，对于相同数据因为初始化向量不同，将产生不同的结果，安全性更高

初始化向量长度与密钥一致，一般使用随机生成，crypto提供随机方法
```javascript
const iv = crypto.randomBytes(16);  // 生成一个16字节的随机IV
```

初始化向量是为了防止相同的明文在不同的加密过程中生成相同的密文，一般不需要保密

```javascript
// aes-128加密的数据只能为16字节的倍数，如数据不满足，会自动进行填充
// Node.js 使用 PKCS7Padding填充，实际使用中需注意填充规则
let AES = crypto.createCipheriv('aes-128-ecb','1234567812345678','')
let encrypted = AES.update('hello world','utf8','hex')
// 调用final表示加密结束
encrypted += AES.final('hex');
console.log(encrypted)
// 6758c061760e90a69e5e4dfc0c61c102
```

### crypto.createCipheriv('算法','密码','初始化向量') 创建解密算法
```javascript
// 解密 密码与初始化向量要与加密时使用的保持一致，否则会抛出异常，解密失败
let AES1 = crypto.createDecipheriv('aes-128-ecb', '1234567812345678', '')
let decrypted = AES1.update(encrypted, 'hex', 'utf8')
decrypted += AES1.final('utf8');
console.log(decrypted) // hello world
```
## 非对称加密

非对称加密是指数据的加密与解密使用的是不同的密钥

非对称加密一般具有公钥与私钥

公钥对所有人公开，私钥自己保存

任何人都可以使用公钥对数据进行加密，但无法对数据进行解密，只有使用私钥才可以对数据进行解密

无法通过公钥推导出私钥

非对称加密安全的基础是对两个大素数的乘积进行因式分解是困难的

### crypto.generateKeyPairSync('类型','配置') 生成公钥和私钥
```javascript
// 生成公钥私钥
const {publicKey, privateKey} = crypto.generateKeyPairSync('rsa', {
    // 密钥长度
    modulusLength: 2048
})

let text = '这是一段需要加密的数据'

```

### crypto.publicEncrypt(公钥,数据) 使用公钥加密
```javascript
// 公钥加密
let publicEncryptResult = crypto.publicEncrypt(publicKey, Buffer.from(text))
console.log(publicEncryptResult.toString('hex'))
```

### crypto.privateDecrypt(私钥,数据) 使用私钥解密
```javascript
// 解密数据
let privateDecryptResult = crypto.privateDecrypt(privateKey, publicEncryptResult)
console.log(privateDecryptResult.toString('utf8')) // 这是一段需要加密的数据
```
