下载/download：https://drive.google.com/file/d/1sUHNoPTbgYSglzgQCDCHoQ0SsXwWFf4v/view?usp=drive_link

### 功能介绍

这款 Python 编写的工具主要用于碰撞生成比特币隔离见证（SegWit）钱包地址及其对应的助记词。它具备一个图形用户界面（GUI），方便用户进行操作，包括登录验证、开始和停止碰撞过程等。以下是该工具的主要功能概述：

1. **登录验证**：用户需要输入激活码进行验证，验证通过后才能进入主界面。
2. **碰撞功能**：在主界面中，用户可以点击“Start Hunting”按钮开始碰撞过程，工具会不断生成随机的助记词，并根据助记词生成对应的比特币 SegWit 地址。
3. **输出显示**：在碰撞过程中，工具会将生成的助记词、地址以及扫描次数显示在输出区域。
4. **停止功能**：用户可以点击“Stop”按钮停止碰撞过程。
5. **联系我们**：用户可以点击“Contact Us”按钮查看联系方式。

### 碰撞功能代码片段及解析

#### 生成助记词

```python
def generate_mnemonic():
    """生成助记词"""
    mnemo = Mnemonic("english")
    return mnemo.generate(strength=128)
```

**解析**：
- `Mnemonic("english")`：创建一个 `Mnemonic` 对象，指定使用英文单词表。
- `mnemo.generate(strength=128)`：生成一个强度为 128 位的助记词，通常为 12 个英文单词。

#### 生成比特币 SegWit 地址

```python
def generate_btc_segwit_address(mnemonic):
    """生成BTC SegWit地址"""
    # 从助记词生成种子
    seed = Mnemonic.to_seed(mnemonic)
    
    # 从种子生成主私钥
    master_key = BIP32Key.fromEntropy(seed)
    
    # 派生路径 m/49'/0'/0'/0/0
    child_key = master_key.ChildKey(49 | 0x80000000)  # purpose
    child_key = child_key.ChildKey(0 | 0x80000000)    # coin type
    child_key = child_key.ChildKey(0 | 0x80000000)    # account
    child_key = child_key.ChildKey(0)                 # change
    child_key = child_key.ChildKey(0)                 # address index
    
    # 从私钥生成公钥
    public_key = child_key.PublicKey()
    
    # 生成地址
    # 对公钥进行SHA256和RIPEMD160哈希
    sha256_hash = sha256(public_key).digest()
    ripemd160_hash = hashlib.new('ripemd160', sha256_hash).digest()
    
    # 添加版本号(0x05 for SegWit)
    version = b'\x05'
    vh160 = version + ripemd160_hash
    
    # 计算校验和
    checksum = sha256(sha256(vh160).digest()).digest()[:4]
    
    # 组合并Base58编码
    binary = vh160 + checksum
    address = base58.b58encode(binary).decode('utf-8')
    
    return address
```

**解析**：
1. **生成种子**：使用 `Mnemonic.to_seed(mnemonic)` 从助记词生成种子。
2. **生成主私钥**：使用 `BIP32Key.fromEntropy(seed)` 从种子生成主私钥。
3. **派生路径**：按照 BIP49 标准，通过 `ChildKey` 方法派生路径 `m/49'/0'/0'/0/0`，得到子私钥。
4. **生成公钥**：使用 `child_key.PublicKey()` 从子私钥生成公钥。
5. **哈希计算**：对公钥进行 SHA256 和 RIPEMD160 哈希计算。
6. **添加版本号**：添加 SegWit 地址的版本号 `0x05`。
7. **计算校验和**：对版本号和哈希结果进行两次 SHA256 哈希计算，取前 4 个字节作为校验和。
8. **Base58 编码**：将版本号、哈希结果和校验和组合后进行 Base58 编码，得到最终的比特币 SegWit 地址。

#### 碰撞线程

```python
class WalletWorker(QThread):
    update_signal = pyqtSignal(str, str, int)
    
    def __init__(self):
        super().__init__()
        self.is_running = True
        
    def run(self):
        count = 0
        while self.is_running:
            mnemonic = generate_mnemonic()
            addr = generate_btc_segwit_address(mnemonic)
            
            if addr:
                count += 1
                self.update_signal.emit(mnemonic, addr, count)
                
    def stop(self):
        self.is_running = False
```

**解析**：
- `WalletWorker` 继承自 `QThread`，用于在后台线程中进行碰撞操作。
- `update_signal` 是一个自定义信号，用于将生成的助记词、地址和扫描次数发送到主界面进行显示。
- `run` 方法是线程的主要执行方法，在循环中不断生成助记词和地址，并通过 `update_signal` 发送信号。
- `stop` 方法用于停止线程的运行。

-------------------

### Function Introduction

This tool written in Python is mainly used to collide and generate Bitcoin Segregated Witness (SegWit) wallet addresses and their corresponding mnemonic phrases. It has a graphical user interface (GUI) that facilitates user operations, including login verification, starting and stopping the collision process, etc. The following is an overview of the main functions of this tool:

1. **Login Verification**: Users need to enter an activation code for verification. Only after successful verification can they enter the main interface.
2. **Collision Function**: On the main interface, users can click the "Start Hunting" button to initiate the collision process. The tool will continuously generate random mnemonic phrases and create corresponding Bitcoin SegWit addresses based on these phrases.
3. **Output Display**: During the collision process, the tool will display the generated mnemonic phrases, addresses, and the number of scans in the output area.
4. **Stop Function**: Users can click the "Stop" button to halt the collision process.
5. **Contact Us**: Users can click the "Contact Us" button to view contact information.

### Code Snippet and Explanation of the Collision Function

#### Generate Mnemonic Phrase

```python
def generate_mnemonic():
    """Generate a mnemonic phrase"""
    mnemo = Mnemonic("english")
    return mnemo.generate(strength=128)
```

**Explanation**:
- `Mnemonic("english")`: Creates a `Mnemonic` object, specifying the use of the English word list.
- `mnemo.generate(strength=128)`: Generates a mnemonic phrase with a strength of 128 bits, usually consisting of 12 English words.

#### Generate Bitcoin SegWit Address

```python
def generate_btc_segwit_address(mnemonic):
    """Generate a BTC SegWit address"""
    # Generate a seed from the mnemonic phrase
    seed = Mnemonic.to_seed(mnemonic)
    
    # Generate the master private key from the seed
    master_key = BIP32Key.fromEntropy(seed)
    
    # Derivation path m/49'/0'/0'/0/0
    child_key = master_key.ChildKey(49 | 0x80000000)  # purpose
    child_key = child_key.ChildKey(0 | 0x80000000)    # coin type
    child_key = child_key.ChildKey(0 | 0x80000000)    # account
    child_key = child_key.ChildKey(0)                 # change
    child_key = child_key.ChildKey(0)                 # address index
    
    # Generate the public key from the private key
    public_key = child_key.PublicKey()
    
    # Generate the address
    # Perform SHA256 and RIPEMD160 hashing on the public key
    sha256_hash = sha256(public_key).digest()
    ripemd160_hash = hashlib.new('ripemd160', sha256_hash).digest()
    
    # Add the version number (0x05 for SegWit)
    version = b'\x05'
    vh160 = version + ripemd160_hash
    
    # Calculate the checksum
    checksum = sha256(sha256(vh160).digest()).digest()[:4]
    
    # Combine and Base58 encode
    binary = vh160 + checksum
    address = base58.b58encode(binary).decode('utf-8')
    
    return address
```

**Explanation**:
1. **Generate Seed**: Uses `Mnemonic.to_seed(mnemonic)` to generate a seed from the mnemonic phrase.
2. **Generate Master Private Key**: Uses `BIP32Key.fromEntropy(seed)` to generate the master private key from the seed.
3. **Derivation Path**: According to the BIP49 standard, derives the path `m/49'/0'/0'/0/0` through the `ChildKey` method to obtain the child private key.
4. **Generate Public Key**: Uses `child_key.PublicKey()` to generate the public key from the child private key.
5. **Hash Calculation**: Performs SHA256 and RIPEMD160 hash calculations on the public key.
6. **Add Version Number**: Adds the version number `0x05` for the SegWit address.
7. **Calculate Checksum**: Performs two rounds of SHA256 hash calculations on the version number and the hash result, and takes the first 4 bytes as the checksum.
8. **Base58 Encoding**: Combines the version number, hash result, and checksum, and then performs Base58 encoding to obtain the final Bitcoin SegWit address.

#### Collision Thread

```python
class WalletWorker(QThread):
    update_signal = pyqtSignal(str, str, int)
    
    def __init__(self):
        super().__init__()
        self.is_running = True
        
    def run(self):
        count = 0
        while self.is_running:
            mnemonic = generate_mnemonic()
            addr = generate_btc_segwit_address(mnemonic)
            
            if addr:
                count += 1
                self.update_signal.emit(mnemonic, addr, count)
                
    def stop(self):
        self.is_running = False
```

**Explanation**:
- `WalletWorker` inherits from `QThread` and is used to perform the collision operation in a background thread.
- `update_signal` is a custom signal used to send the generated mnemonic phrase, address, and the number of scans to the main interface for display.
- The `run` method is the main execution method of the thread. It continuously generates mnemonic phrases and addresses in a loop and sends a signal through `update_signal`.
- The `stop` method is used to stop the thread's operation.
