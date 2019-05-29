
# The Trusted Execution Environment(TEE)

TEE与Rich OS并行的独立运行环境，为Rich OS提供安全保护。TEE包含了一个执行空间来提供比Rich OS更高级别的安全保护。尽管达不到SE能提供的安全程度，但对大多数应用来说，TEE能满足安全所需。


## 体系
![](pic/tee.png)


## TEE vs SE
TEE提供了一个比Rich OS更安全的执行空间，尽管安全级别不到SE的程度，但对于大多数应用足够。而且TEE提供了比SE更快的处理速度和更强大的内存访问能力。

TEE提供了比SE更多的用户接口和外部连接能力，可以在TEE上开发安全应用，这些安全应用可以给用户提供Rich OS一样的用户体验，同时，由于TEE是独立于Rich OS的执行环境，TEE还保障了足够的安全，可以抵挡Rich OS的软件攻击，如：获取OS的root、越狱、恶意软件。

SE提供了健壮的物理特性，可以抵抗高级别侧信道攻击，SE具有最高级别的安全认证（等同于智能卡的EAL4+及以上）。SE具有可移动性，支持安全和数据的可移植（就像UICC和MicroSD），可以在不同设备上移动。具有NFC功能的SE，还可以在设备低电量或关机下使用。

TEE对SE并是不排斥的，TEE结合SE的特性，可以保证各SE中外部进程间的无缝交互和安全。

![](pic/tee-1.png)

