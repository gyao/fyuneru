Fyuneru
=======

**向下拉动，见简体中文版本。**

**Fyuneru** lets you set up a server and a client computer within a virtual
ethernet, whose data frames over the virtual network cable are proxified
parallel via imagnative a lot of protocols. It also provides basical
encryption, which makes analyzing the traffic more difficult.

This software is licensed under GPLv2.



## Principle

Fyuneru ships with 2 parts: a core, and several proxies.

The **core** behaves on the server and client computer in an identical manner.
It first sets up the virtual network interface using Linux's TUN device and
deals with intercepting IP frames sending to and coming from it. When these
frames are leaving our computer, they are encrypted to ensure the 
confidentiality and integrity. It then listens for incoming connections on
several UDP ports.

Once a UDP connection to a port, initiated by a proxy(described below), is
confirmed, this UDP port will be used for emitting the payload: a subset of the
encrypted IP frames. Which IP frame will be sent to a given UDP port, is
choosen randomly.

**Proxies** transmit bidirectional UDP packets. A proxy knocks both UDP ports
on the server and the client. It delivers the UDP packet using its own
mechanism, whatever HTTP/AJAX/WebSocket or XMPP or more. The proxy may also
not serves on the same server(e.g. you may connect a XMPP server, and send your
packet to a fixed account which is run by the proxy server).

Notice that the core on both server and client behaves the same: it chooses for
every IP frame a random proxy, therefore for a given connection or session, the
actual IP frame of request and response travels highly likely over different
routes with different protocols or even different IPs. This schema should
confuse an observer, e.g. a firewall.



## Installation

### System Requirements

1. Operating System: currently tested over Fedora 21 and Ubuntu 14.04;
1. Installed Python2 environment;
1. NodeJS is required by some proxies.

### Dependencies

#### Core

1. `python-pytun`, a python module used for setting up TUN devices. Use
   `sudo pip install python-pytun` to get this.
1. `salsa20`, a python module providing encryption for our program. Use
   `sudo pip install salsa20` to get this.

#### Proxies

For different proxies, you may need proxy-specific dependencies.

Following proxies are written in NodeJS. Install NodeJS, and run `npm install`
in the corresponding path to get dependencies installed.

1. `proxies/websocket`

## Usage

### Configuration

Install Fyuneru on both your server and your client. Before running, create a
file named `config.json` and place it in the same folder as the `run_as.py`.
Both server and client needs such a file, make sure they're **identical**.

An example of `config.json` is below. Do not remove any entry, just modify
their value if desired.

```
{
    "core": {
        "server": {
            "ip": "10.1.0.1",
            "ports": [17100,17101,17102,17103,17104]
        },
        "client": {
            "ip": "10.1.0.2",
            "ports": [7100,7101,7102,7103,7104]
        },
        "key": "<USE A RANDOM AND LONG KEY TO REPLACE THIS>"
    },
    "proxies": {
        "websocket": {
            "server": {
                "ip": "<ADDRESS OF THE SERVER RUNNING PROXY>",
                "webport": 6000,
                "coreports": [17100, 17101]
            },
            "client": {
                "coreports": [7100, 7101]
            }
        }
    }
}
```

Explanations to the configurations for the core.

1. Set `core.server.ip` and `core.client.ip` as the desired virtual IP
   addresses for both computers.
1. `core.server.ports` and `core.client.ports` are arrays containing a series
   of port numbers. They are the ports used by proxies.
1. Remeber to change `core.key` to a long key consists of random characters.
   This is cricital to your security.

To configure using a specific type of proxy, specify parameters in `proxies`
section.

#### Using WebSocket Proxy

Add section `proxies.websocket`.

`proxies.websocket.server` have 3 keys: `ip`, `webport`, `coreports`.

1. `ip` is the address of the server, which the client uses to connect.
2. `webport` is the TCP port the proxy server listens on. Client will connect
   to this port.
3. `coreport` must be a subset of `core.server.ports`. Specify which UDP ports
   of the core will be served by this proxy.

`proxies.websocket.client` have only one key: `coreports`. It has to be a
subset of `core.client.ports`.

Remember to configure the firewall on the server, to let
`proxies.websocket.server.webport` allowed.

### Run

Find the `run_as.py`, run `python run_as.py s` for setting up a server, and
`python run_as.py c` for setting up the client. You need root priviledge to
do this.

Add `--debug` after `run_as.py` will enable the debug mode(currently only for
the core). The IP frames will be dumped to the console.

---

Fyuneru
=======

**Scroll up for English version.**

**Fyuneru**允许您将服务器和客户端计算机用虚拟的以太网连接。
在虚拟的网线上传递的数据帧实际借助可以想象的一系列协议传送。
它同时提供基本的加密，使得分析流量更加困难。

本软件在GPLv2协议下进行许可。



## 原理

Fyuneru有两个部分：核心，和一系列代理。

**核心**在服务器和客户端计算机上运行时行为相同。
它首先使用Linux的TUN设备，建立一个虚拟的网卡。
之后，它就截获发往和来自这一网卡的IP数据帧。
这些数据帧在离开计算机之前会被加密，以确保其机密性和完整性。
之后，核心在多个UDP端口上监听，等待连接。

一旦在一个端口上由代理程序打通了UDP连接，这个端口就会被用来放出负载数据包：
即一部分加密的IP数据帧的。
至于对于特定的UDP端口，哪个IP数据帧会从这里释放出来，则是随机选择的。

**代理**负责双向传送UDP数据包。
代理会试图与服务器和客户机的UDP端口建立连接。
之后它使用自己的机制，不管是HTTP/AJAX/WebSocket还是XMPP或者别的什么来传送UDP包。
代理不一定和最终的服务器处于同一台计算机（
例如，您可以连接到XMPP服务器，然后将数据包发送到一个由最终的服务器所拥有的固定帐号上
）。

注意到服务器和客户机的行为是相同的：它为每个IP数据帧选择一个随机的代理。
这样对于上层的连接来说，具体的IP数据帧在请求和应答时都很有可能穿过不同的隧道，
协议和IP地址都可能不同。这种方式应当可以迷惑一个观察者——例如防火墙。


## 安装

### 系统要求

1. 操作系统：当前在Fedora 21和Ubuntu 14.04下测试完成。
1. 安装了Python2的环境。
1. 对于有些代理，需要使用NodeJS。

### 依赖的文件

#### 核心程序

1. `python-pytun`, 一个用来设定TUN设备的模块。
   使用命令`sudo pip install python-pytun`来安装这一模块。
1. `salsa20`，一个Python模块，为我们的程序提供加密服务。
   使用`sudo pip install salsa20`来安装这一模块。

#### 代理程序

对于不同的代理，还有具体的依赖性。

如下代理使用NodeJS编写。
首先安装NodeJS，然后在各自的目录中用`npm install`安装依赖文件。

1. `proxies/websocket`

## 用法

### 配置

Fyuneru需要首先安装在服务器和客户端上。
在运行前，在`run_as.py`相同的路径下创建`config.json`文件。
服务器和客户端都需要这样一个文件，他们必须**完全相同**。

`config.json`文件的内容如下。请不要删除任何条目，只按照需要修改他们的值。

```
{
    "core": {
        "server": {
            "ip": "10.1.0.1",
            "ports": [17100,17101,17102,17103,17104]
        },
        "client": {
            "ip": "10.1.0.2",
            "ports": [7100,7101,7102,7103,7104]
        },
        "key": "<在这里输入一个很长而随机的密钥>"
    },
    "proxies": {
        "websocket": {
            "server": {
                "ip": "<运行着这个代理的服务器的地址>",
                "webport": 6000,
                "coreports": [17100, 17101]
            },
            "client": {
                "coreports": [7100, 7101]
            }
        }
    }
}
```

对上述文件的核心（core）部分的解释：

1. `core.server.ip`和`core.client.ip`分别是预计分配给服务器和客户端的虚拟网卡的IP地址。
1. `core.server.ports`和`core.client.ports`是一系列端口号的数组。
   代理将在这些端口号上与核心交互。
1. 务必记住修改`core.key`的值，使之为一个很长而随机的口令。
   这对于您的安全是十分重要的。

为了配置具体一个代理，需要在`proxies`部分中设定参数。

#### 配置一个WebSocket代理

配置文件中需要有`proxies.websocket`的部分。

`proxies.websocket.server`包含3个键值：`ip`, `webport`, `coreports`。

1. `ip`是客户端用以连接服务器的地址。
2. `webport`是服务器所监听的、位于互联网的TCP端口。客户端将向这个端口发起连接。
3. `coreport`必须是`core.server.ports`的一个子集。
   在此指定哪些UDP端口将会被此代理所使用。

`proxies.websocket.client`包含一个键值：`coreports`。
它必须是`core.client.ports`的一个子集。

注意修改防火墙的设置，使得端口`proxies.websocket.server.webport`可以通过。

### 运行

找到`run_as.py`，用命令`python run_as.py s`来启动服务器。
用`python run_as.py c`来启动客户端。您需要提供root权限。

在`run_as.py`之后添加`--debug`标志将会进入调试模式（当前只能调试核心）。
在此模式下，将会在控制台输出收发的IP数据帧。
