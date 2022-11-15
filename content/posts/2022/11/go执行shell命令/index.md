---
title: "go执行shell命令"
subtitle: ""
date: 2022-11-01T09:06:08+08:00
lastmod: 2022-11-01T09:06:08+08:00
draft: false
toc:
  enable: true
weight: false
categories: [""]
tags: [""]
---

#### 背景

```
是这样的，最近在研究一个定时任务系统的改造，可能有点像jenkins做到的那种吧。

可以输入shell命令，也可以执行py脚本等等，相比之前来说，也要能够及时停止！

但是遇到了这么个问题，golang执行py脚本的时候获取不到脚本的输出。
1首先来看看go里面怎么运行shell脚本吧，我比较喜欢执行全部命令。
```

### 普通用法（一次性获取所有输出）

```go
package main

import (
    "fmt"
    "os/exec"
)

func main() {
    Command("ls")
}

// 这里为了简化，我省去了stderr和其他信息
func Command(cmd string) error {
    c := exec.Command("bash", "-c", cmd)
    // 此处是windows版本
    // c := exec.Command("cmd", "/C", cmd)
    output, err := c.CombinedOutput()
    fmt.Println(string(output))
    return err
}
```

可以看到，当前命令执行的是输出当前目录下的文件/文件夹

![image.png](https://upload-images.jianshu.io/upload_images/6053915-90f1828a00b20b6e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 实时显示

效果图:

![image.png](https://upload-images.jianshu.io/upload_images/6053915-741679a68e60e7a7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```
package main

import (
    "bufio"
    "fmt"
    "io"
    "os/exec"
    "sync"
)

func main() {
    // 执行ping baidu的命令, 命令不会结束
    Command("ping www.baidu.com")

}

func Command(cmd string) error {
    //c := exec.Command("cmd", "/C", cmd)   // windows
    c := exec.Command("bash", "-c", cmd)  // mac or linux
    stdout, err := c.StdoutPipe()
    if err != nil {
        return err
    }
    var wg sync.WaitGroup
    wg.Add(1)
    go func() {
        defer wg.Done()
        reader := bufio.NewReader(stdout)
        for {
            readString, err := reader.ReadString('\n')
            if err != nil || err == io.EOF {
                return
            }
            fmt.Print(readString)
        }
    }()
    err = c.Start()
    wg.Wait()
    return err
}
```

### 可关闭+实时输出

```
package main

import (
    "bufio"
    "context"
    "fmt"
    "io"
    "os/exec"
    "sync"
    "time"
)

func main() {
    ctx, cancel := context.WithCancel(context.Background())
    go func(cancelFunc context.CancelFunc) {
        time.Sleep(3 * time.Second)
        cancelFunc()
    }(cancel)
    Command(ctx, "ping www.baidu.com")
}

func Command(ctx context.Context, cmd string) error {
    // c := exec.CommandContext(ctx, "cmd", "/C", cmd)
    c := exec.CommandContext(ctx, "bash", "-c", cmd) // mac linux
    stdout, err := c.StdoutPipe()
    if err != nil {
        return err
    }
    var wg sync.WaitGroup
    wg.Add(1)
    go func(wg *sync.WaitGroup) {
        defer wg.Done()
        reader := bufio.NewReader(stdout)
        for {
            // 其实这段去掉程序也会正常运行，只是我们就不知道到底什么时候Command被停止了，而且如果我们需要实时给web端展示输出的话，这里可以作为依据 取消展示
            select {
            // 检测到ctx.Done()之后停止读取
            case <-ctx.Done():
                if ctx.Err() != nil {
                    fmt.Printf("程序出现错误: %q", ctx.Err())
                } else {
                    fmt.Println("程序被终止")
                }
                return
            default:
                readString, err := reader.ReadString('\n')
                if err != nil || err == io.EOF {
                    return
                }
                fmt.Print(readString)
            }
        }
    }(&wg)
    err = c.Start()
    wg.Wait()
    return err
}
```

效果图:

![image.png](https://upload-images.jianshu.io/upload_images/6053915-8719dd1094a076fb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

可以看到输出了3次（1秒1次）之后程序就被终止了，确切的说是读取输出流的循环结束了。

### 执行Python脚本(阻塞)

其实很简单，只要`python -u xxx.py`这样执行就可以了, -u参数

```
简单的说就是python的输出是有缓存的，-u会强制往标准流输出，当Python脚本阻塞的时候

也不会拿不到输出！
```

### 其他

"bash" 和"-c"，据我的观察，这2个参数代表在当前cmd窗口执行，而不加这2个参数，直接上shell的话，会启动一个新窗口，目前观察是stdout拿不到数据。

### 仍有缺陷

```
上面的命令可以解决大部分问题，但是获取不到stderr的信息，所以我们需要改造一下。

下面是输出和错误一并输出的实时读取，类似于jenkins那种。
package main

import (
    "bufio"
    "context"
    "fmt"
    "io"
    "os/exec"
    "sync"
    "time"
)

func main() {
    ctx, cancel := context.WithCancel(context.Background())
    go func(cancelFunc context.CancelFunc) {
        time.Sleep(3 * time.Second)
        cancelFunc()
    }(cancel)
    Command(ctx, "ping www.baidu.com")
}

func read(ctx context.Context, wg *sync.WaitGroup, std io.ReadCloser) {
    reader := bufio.NewReader(std)
    defer wg.Done()
    for {
        select {
        case <-ctx.Done():
            return
        default:
            readString, err := reader.ReadString('\n')
            if err != nil || err == io.EOF {
                return
            }
            fmt.Print(readString)
        }
    }
}

func Command(ctx context.Context, cmd string) error {
    //c := exec.CommandContext(ctx, "cmd", "/C", cmd) // windows
    c := exec.CommandContext(ctx, "bash", "-c", cmd) // mac linux
    stdout, err := c.StdoutPipe()
    if err != nil {
        return err
    }
    stderr, err := c.StderrPipe()
    if err != nil {
        return err
    }
    var wg sync.WaitGroup
    // 因为有2个任务, 一个需要读取stderr 另一个需要读取stdout
    wg.Add(2)
    go read(ctx, &wg, stderr)
    go read(ctx, &wg, stdout)
    // 这里一定要用start,而不是run 详情请看下面的图
    err = c.Start()
    // 等待任务结束
    wg.Wait()
    return err
}
```

![image.png](https://upload-images.jianshu.io/upload_images/6053915-56224b858d32ad04.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### windows输出乱码问题

参考资料： https://blog.csdn.net/rznice/article/details/88122923

#### 最后给一个解决windows乱码的完整案例

```
需要下载golang.org/x/text/encoding/simplifiedchinese
package main

import (
    "bufio"
    "fmt"
    "io"
    "os/exec"
    "sync"
    "golang.org/x/text/encoding/simplifiedchinese"
)

type Charset string

const (
    UTF8    = Charset("UTF-8")
    GB18030 = Charset("GB18030")
)

func main() {
    // 执行ping baidu的命令, 命令不会结束
    Command("ping www.baidu.com")

}

func Command(cmd string) error {
    //c := exec.Command("cmd", "/C", cmd)   // windows
    c := exec.Command("bash", "-c", cmd)  // mac or linux
    stdout, err := c.StdoutPipe()
    if err != nil {
        return err
    }
    var wg sync.WaitGroup
    wg.Add(1)
    go func() {
        defer wg.Done()
        reader := bufio.NewReader(stdout)
        for {
            readString, err := reader.ReadString('\n')
            if err != nil || err == io.EOF {
                return
            }
            byte2String := ConvertByte2String([]byte(readString), "GB18030")
            fmt.Print(byte2String)
        }
    }()
    err = c.Start()
    wg.Wait()
    return err
}

func ConvertByte2String(byte []byte, charset Charset) string {
    var str string
    switch charset {
    case GB18030:
        var decodeBytes, _ = simplifiedchinese.GB18030.NewDecoder().Bytes(byte)
        str = string(decodeBytes)
    case UTF8:
        fallthrough
    default:
        str = string(byte)
    }
    return str
}
```

 

## 概述

远程执行命令有什么用？为什么要远程执行命令？ 如果你只有2，3台服务器需要管理的时候，远程执行命令确实没有没多大作用，你可以登录到每台服务器上去完成各种操作。 当你的服务器大于3台的时候，远程执行的命令的方式就可以大大提高你的生产力了。

如果你有一个可以远程执行命令的工具，那么就可以像操作单台机器那样操作多台机器，机器越多，效率提高的越多。 远程执行命令最常用的方法就是利用 SSH 协议，将命令发送到远程机器上执行，并获取返回结果。

本文介绍如何使用 golang 实现远程执行命令。

## 一般命令

所谓一般命令，就是在一定时间内会执行完的命令。比如 grep, cat 等等。 执行命令的步骤是：连接，执行，获取结果

### 连接

连接包含了认证，可以使用 password 或者 sshkey 2种方式来认证。下面的示例为了简单，使用了密码认证的方式来完成连接。

```
import (  
  "fmt"
  "time"

  "golang.org/x/crypto/ssh"
)

func connect(user, password, host string, port int) (*ssh.Session, error) {  
  var (
    auth         []ssh.AuthMethod
    addr         string
    clientConfig *ssh.ClientConfig
    client       *ssh.Client
    session      *ssh.Session
    err          error
  )
  // get auth method
  auth = make([]ssh.AuthMethod, 0)
  auth = append(auth, ssh.Password(password))

  clientConfig = &ssh.ClientConfig{
    User:    user,
    Auth:    auth,
    Timeout: 30 * time.Second,
  }

  // connet to ssh
  addr = fmt.Sprintf("%s:%d", host, port)

  if client, err = ssh.Dial("tcp", addr, clientConfig); err != nil {
    return nil, err
  }

  // create session
  if session, err = client.NewSession(); err != nil {
    return nil, err
  }

  return session, nil
}
```

连接的方法很简单，只要提供登录主机的 **用户\*， \*密码\*， \*主机名或者IP\*， \*SSH端口**

### 执行，命令获取结果

连接成功后，执行命令很简单

```
import (  
  "fmt"
  "log"
  "os"
  "time"

  "golang.org/x/crypto/ssh"
)

func main() {  
  session, err := connect("root", "xxxxx", "127.0.0.1", 22)
  if err != nil {
    log.Fatal(err)
  }
  defer session.Close()

  session.Run("ls /; ls /abc")
}
```

上面代码运行之后，虽然命令正常执行了，但是没有正常输出的结果，也没有异常输出的结果。 要想显示结果，需要将 **session** 的 Stdout 和 Stderr 重定向 修改 **func main** 为如下：

```
func main() {  
  session, err := connect("root", "xxxxx", "127.0.0.1", 22)
  if err != nil {
    log.Fatal(err)
  }
  defer session.Close()

  session.Stdout = os.Stdout
  session.Stderr = os.Stderr
  session.Run("ls /; ls /abc")
}
```

这样就能在屏幕上显示正常，异常的信息了。

## 交互式命令

上面的方式无法远程执行交互式命令，比如 **top** ， 远程编辑一个文件，比如 **vi /etc/nginx/nginx.conf** 如果要支持交互式的命令，需要当前的terminal来接管远程的 PTY。

```
func main() {  
  session, err := connect("root", "olordjesus", "dockers.iotalabs.io", 2210)
  if err != nil {
    log.Fatal(err)
  }
  defer session.Close()

  fd := int(os.Stdin.Fd())
  oldState, err := terminal.MakeRaw(fd)
  if err != nil {
    panic(err)
  }
  defer terminal.Restore(fd, oldState)

  // excute command
  session.Stdout = os.Stdout
  session.Stderr = os.Stderr
  session.Stdin = os.Stdin

  termWidth, termHeight, err := terminal.GetSize(fd)
  if err != nil {
    panic(err)
  }

  // Set up terminal modes
  modes := ssh.TerminalModes{
    ssh.ECHO:          1,     // enable echoing
    ssh.TTY_OP_ISPEED: 14400, // input speed = 14.4kbaud
    ssh.TTY_OP_OSPEED: 14400, // output speed = 14.4kbaud
  }

  // Request pseudo terminal
  if err := session.RequestPty("xterm-256color", termHeight, termWidth, modes); err != nil {
    log.Fatal(err)
  }

  session.Run("top")
}
```

这样就可以执行交互式命令了，比如上面的 **top** 也可以通过 **vi /etc/nginx/nignx.conf** 之类的命令来远程编辑文件。
