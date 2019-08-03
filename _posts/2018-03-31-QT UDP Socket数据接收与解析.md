---
layout:     post
title:      "QT UDP Socket数据接收与解析"
date:       2018-03-31 12:00:00
author:     "邵正"
header-img: "img/home-bg-o.jpg"
tags:
    - QT
    - Socket
---

| 主题     | 概要                                   |
| -------- | -------------------------------------- |
| QT       | UDP Socket                             |
| -------- | ---                                    |
| **编辑** | **时间**                               |
| 新建     | 20180331                               |
| -------- | ---                                    |
| **序号** | **参考资料**                           |
| 1        | https://doc.qt.io/qt-5/qudpsocket.html |

做直升机航电系统仿真，类似GPS导航接收机的按钮很多，显示的仪表也很多。
![这里写图片描述](https://img-blog.csdn.net/20180331123639997?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NoYW96aGVuZ2Vk/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
按钮的状态是通过底层程序通过UDP发送给控制层的，层次逻辑如下图所示：
![这里写图片描述](https://img-blog.csdn.net/20180331123708230?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NoYW96aGVuZ2Vk/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

现在要实现的功能是用QT自带的QUdpSocket，实现端口的监听，收到数据后，解析并把数据派发到具体的模块。

现在收发的数据结构（类）是：InstrumentBtn，基础设施的按钮信息。这里忽略具体的内容，只需知道字段是全值类型，不需要实现深层拷贝， 复用下面的代码时，只需要按需要替换为自己的数据结构（类）即可。

## 类定义 ##
定义一个InstrumentBtnWorker，表示一个解析基础设施按钮的工人。

```
#ifndef ASM_INSTRUMENT_BTN_WORKER_H
#define ASM_INSTRUMENT_BTN_WORKER_H

#include <QUdpSocket>
#include <QReadWriteLock>
#include "net_fdm.h"

class InstrumentBtnWorker:public QObject
{
    Q_OBJECT
public:
    static InstrumentBtnWorker * instance();
    void destory();
signals:
    void signalBtnValueChanged(InstrumentBtn btnValue);

private slots:
    void readPendingDatagrams();

private:
    InstrumentBtnWorker();
    virtual ~InstrumentBtnWorker();

    void initSocket();
    void processDatagram(char * datagram,int len);
    void dispatchBtnValue(InstrumentBtn btnValue);

    static InstrumentBtnWorker * m_instance;
    QUdpSocket udpSocket;
};



#endif // ASM_INSTRUMENT_BTN_WORKER_H

```
net_fdm.h头文件里面定义了InstrumentBtn结构。
该类采用单例模式。

## 类实现 ##
初始化socket：

```
void InstrumentBtnWorker::initSocket()
{
    udpSocket.bind(GPSGLConfige::INSTRUMENT_BTN_UDP_PORT);
    connect(&udpSocket, SIGNAL(readyRead()),this, SLOT(readPendingDatagrams()));
}

```

绑定端口号，连接readyRead信号与处理槽。

缓存数据，并把数据传递给processDatagram处理。

```
void InstrumentBtnWorker::readPendingDatagrams()
{
    QByteArray datagram;
    while (udpSocket.hasPendingDatagrams())
    {
        int sizeLen=udpSocket.pendingDatagramSize();
        datagram.resize(sizeLen);
        udpSocket.readDatagram(datagram.data(),datagram.size());
        processDatagram(datagram.data(),sizeLen);
    }
}

```
数据处理：

```
void InstrumentBtnWorker::processDatagram(char * datagram,int len)
{
    InstrumentBtn btnValue;
    if(len>0)
    {
        btn_input(datagram,&btnValue);
        dispatchBtnValue(btnValue);
    }
}

```

其中的btn_input直接用memcpy强制转换数据：

```
void btn_input(char* buf,InstrumentBtn* btn)
{
    if(buf==NULL || btn==NULL)
    {
        return;
    }


    memset(btn,0,sizeof(InstrumentBtn));
    memcpy(btn,buf,sizeof(InstrumentBtn));
}

```

这种偷懒的做法不知道对不对，比较担心代码的移植性，结构没有1字节对齐，也没有对传输对象序列化与反序列化。但现在来看，还是能正常工作。

dispatchBtnValue，派发数据，实际上只是发送自定义的signalBtnValueChanged信号，利用QT自带的信号-槽机制实现数据传输。

```
/*
Note:需确保InstrumentBtn全为值类型，如果有引用类型，需实现深层拷贝
*/
void InstrumentBtnWorker::dispatchBtnValue(InstrumentBtn btnValue)
{
    emit signalBtnValueChanged(btnValue);
}

```
私有构造函数、析构函数：

```
InstrumentBtnWorker * InstrumentBtnWorker::m_instance=NULL;
InstrumentBtnWorker * InstrumentBtnWorker::instance()
{
    QReadWriteLock lock;
    if(m_instance==NULL)
    {
        lock.lockForWrite();
        if(m_instance==NULL)
        {
            m_instance=new InstrumentBtnWorker();
        }
        lock.unlock();
    }

    return m_instance;
}

InstrumentBtnWorker::InstrumentBtnWorker()
{
    initSocket();
}

InstrumentBtnWorker::~InstrumentBtnWorker()
{
}

```

## 类使用 ##
在需要接收数据的地方，实现自己的接收槽函数，并连接到InstrumentBtnWorker的signalBtnValueChanged信号：

```
InstrumentBtnWorker * btnWorker=InstrumentBtnWorker::instance();

    btnWorker->connect(btnWorker,SIGNAL(signalBtnValueChanged(InstrumentBtn)),this,SLOT(slotOnBtnChanged(InstrumentBtn)));

```
如果没有信号和槽机制，C++中又没有C#中的委托机制，还需要自己去实现观察者模式，维护一个观察者列表。
