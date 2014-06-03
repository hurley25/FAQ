---
layout: post
category: note
title: 如何设计应用层协议格式
tagline: by 浅墨
tags: [network, protocol]
---

## 协议包设计原则

**一般而言，应用层协议设计有四种常见方法：**

1. 每个发送的包长度固定
2. 包每行均采取特殊结束标记用以区分（例如HTTP使用的\r\n）
3. 包前添加长度信息（所谓的TLV模式，即type、length、value）
4. 利用包本身的格式解析（如XML、JSON等）

**P.S. 还有诸如google的protobuf等跨语言的协议打包格式。**

## 详细分析解释

### 先介绍一点点背景

TCP作为一个流协议，其本身没有数据包分界的概念。简而言之就是所谓的TCP断包就是个伪命题。OSI模型定义的7层结构网络中，TCP协议所在的传输层和应用层之间还有会话层和表示层，原本协议包分界和加密等等操作是在这两层完成的。TCP/IP协议在设计的时候，并没有会话层和表示层。那如果用户需要这两层提供的服务怎么办？比如包的分界？答案是，用户自行在应用层代码中实现吧。

So，我们需要学习一般的应用层协议的设计和打包拆包的实现。下面依次给出实现的例子：

### 1. 每个发送的包长度固定

很简单的思路，我们可以定义服务端和客户端均采用同一个结构体进行数据传输，这样的话很容易根据结构体大小来进行分隔收到的数据。


### 2. 包每行均采取特殊结束标记用以区分（例如HTTP使用的\r\n）

这个也很简单理解，采用纯ASCII发送信息的时候，完全可以采用这种方式。比如一个包中，每行采用\r\n进行分隔，包结束采用\r\n\r\n进行分隔等等。

如果在包的数据中出现了结束标记怎么办？转义呗～


### 3. 包前添加长度信息（所谓的TLV模式，即type、length、value）

这个理解起来也不困难，以结构体为例，即便是服务端和客户端采用多种结构体进行通信，只需要加上一个类型字段和长度字段，这样不就解决了么。

**不过这里的type、length以及value只是指导思想，大家完全可以自行去实现自己的格式。**

下面给出一个简单的收包和解析的例子。

首先是包的结构体和类型等定义：

    // 包类型
    typedef
    enum {
        control_start,
        control_end,
        heart_data
    } package_t;
    
    // 协议包
    typedef
    struct _protocol_t {
        uint32_t length_;
        package_t type;
        uint8_t data_[DATA_LENGTH];
        uint32_t crc_;
    } __attribute__((packed)) protocol_t;

下面是包的接收过程的代码片段。（简单起见，我们采用linux的阻塞套结字进行处理）

    while (1) {
        uint32_t length = sizeof(uint32_t);
        // 读取 length 值
        if (rio_readn(conn_fd, conn_buff->buff, length) == length) {
            length = *(uint32_t *)conn_buff->buff;
            if (rio_readn(conn_fd, conn_buff->buff+sizeof(uint32_t), length) == length) {
                if (analyse_protocol(conn_buff) < 0) {
                    break;
                }
            } else {
                server_print_info(LOG_INFO, "Read Data Error! Close User Link!");
            }
        } else {
            server_print_info(LOG_INFO, "Read Data Length Error! Close User Link!");
            break;
        }
    }

其中 rio_readn 函数的实现如下：

    size_t rio_readn(int fd, void *usrbuf, size_t n)
    {
        size_t nleft = n;
        size_t nread = 0;
        char   *bufp = usrbuf;
    
        while (nleft > 0) {
            if ((nread = read(fd, bufp, nleft)) == -1) {
                if (errno == EINTR) {     // Interrupted by sig handler return
                    nread = 0;     // and call read() again
                } else {
                    return -1;     // errno set by read()
                }
            } else if (nread == 0) {
                break;         // EOF
            }
            nleft -= nread;
            bufp += nread;
        }
    
        return (n - nleft);     // return >= 0
    }

analyse_protocol 是解析函数，简单的实现如下：

    // 协议解析程序
    static int analyse_protocol(server_buffer_t *buff)
    {
        protocol_t *proto = (protocol_t *)buff->buff;
    
        switch (proto->type) {
        case heart_data:
            if (write(data_fd, buff->buff, DATA_LENGTH) != DATA_LENGTH) {
                perror("write file error");
                exit(EXIT_FAILURE);
            }
            break;
        default:
            server_print_info(LOG_ERR, "未知的包类型，解析错误");
            return -1;
        }
    
        return 0;
    }


### 4. 利用包本身的格式解析（如Xml、Json等）

这种方式也不难，我们可以通过Xml，Json本身的格式来匹配每一个具体的包。下面给出一个简单的XML接收和解析的简单例子和Qt下使用QtXml的解析方法。

接收过程如下：

    void MainWindow::clientDataReceived()
    {
        while (clientSocket->bytesAvailable()) {
            QByteArray recvMsg = clientSocket->readAll();
            recvBuffer.append(recvMsg);
    
            QString strProtoTag("</wiidroid>");
            int tagLen = strProtoTag.size();
    
            int pos = recvBuffer.indexOf(strProtoTag);
            while ((pos = recvBuffer.indexOf(strProtoTag)) != -1) {
                QByteArray recvPacket(recvBuffer.data(), pos+tagLen);
                recvBuffer.remove(0, recvPacket.size());
                if (recvBuffer.at(0) == '\n') {
                    recvBuffer.remove(0, 1);
                } else if (recvBuffer.at(0) == '\r' && recvBuffer.at(1) == '\n') {
                    recvBuffer.remove(0, 2);
                }
                parseProtoPackage(recvPacket);
            }
        }
    }

**这里是非阻塞的套结字，所以有类的成员变量recvBuffer来存储临时的包，直到收到一个完整的包再进行解析。**

解析Xml的过程：

    void MainWindow::parseProtoPackage(QByteArray &recvPacket)
    {
        KeyPressInfo keyPressInfo;
        ASpeedInfo aSpeedInfo;
        GyroscopeInfo gyroscopeInfo;
    
        switch (ProtocolXml::getXmlInfoType(recvPacket)) {
            // 客户端控制（键盘消息）
            case PROTO_CONTROL_KEY:
                ProtocolXml::parseKeyInfo(recvPacket, keyPressInfo);
                qDebug() << "客户端控制（键盘消息）: \n Key: " << keyPressInfo.key
                    << " isPress: " << keyPressInfo.isPress << "\n";
                break;
                // 客户端控制（加速度传感器）
            case PROTO_CONTROL_ASPEED:
                ProtocolXml::parseASpeedInfo(recvPacket, aSpeedInfo);
                qDebug() << "客户端控制（加速度传感器）\n x: " << aSpeedInfo.x
                    << " y: " << aSpeedInfo.y << " z: " << aSpeedInfo.z << "\n";
                break;
                // 客户端控制（陀螺仪）
            case PROTO_CONTROL_GYROSCOPE:
                ProtocolXml::parseGyroscopeInfo(recvPacket, gyroscopeInfo);
                qDebug() << "客户端控制（陀螺仪）\n x: " << gyroscopeInfo.x
                    << " y: " << gyroscopeInfo.y << " z: " << gyroscopeInfo.z << "\n";
                break;
            case -1:
                qDebug() << "Error Package Format!\n";
        }
    }

具体的一个解析函数：

    /*
    <?xml version="1.0" encoding="utf-8" ?>
        <wiidroid type="11">           // 消息类型 11
            <coord-x>x</coord-x>       // x, y, z 三轴数据
            <coord-y>y</coord-y>
            <coord-z>z</coord-z>
         </wiidroid>
     */
    void ProtocolXml::parseASpeedInfo(QByteArray &recvPacket, ASpeedInfo &aSpeedInfo)
    {
        QXmlStreamReader reader(recvPacket);
    
        while (!reader.atEnd()) {
            QXmlStreamReader::TokenType type = reader.readNext();
    
            if (type == QXmlStreamReader::StartElement) {
                if (reader.name() == "coord-x") {
                    aSpeedInfo.x = reader.readElementText(QXmlStreamReader::SkipChildElements).toDouble();
                } else if (reader.name() == "coord-y") {
                    aSpeedInfo.y = reader.readElementText(QXmlStreamReader::SkipChildElements).toDouble();
                } else if (reader.name() == "coord-z") {
                    aSpeedInfo.z = reader.readElementText(QXmlStreamReader::SkipChildElements).toDouble();
                }
            }
        }
    
        if (reader.hasError()) {
            qDebug() << "XML Format Error:" << reader.errorString() << "\r\n";
        }
    }


