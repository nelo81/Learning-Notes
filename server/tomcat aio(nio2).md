## tomcat nio 和 aio(nio2) 的对比

1. Acceptor类

- nio: `socket = serverSock.accept();` 这里是同步阻塞的
- aio: `socket = serverSock.accept().get();` 这里的accept()方法返回的是future，即异步操作

2. 处理流程上

- nio: acceptor.setSocketOptions -> pollerEvent queue <- poller -> socketProcessor -> ConnectionHandler
- aio: acceptor.setSocketOptions -> socketProcessor -> CompletionHandler
- 附带aio源码：

```
protected boolean setSocketOptions(AsynchronousSocketChannel socket) {
    try {
        socketProperties.setProperties(socket);
        Nio2Channel channel = nioChannels.pop();
        if (channel == null) {
            SocketBufferHandler bufhandler = new SocketBufferHandler(
                    socketProperties.getAppReadBufSize(),
                    socketProperties.getAppWriteBufSize(),
                    socketProperties.getDirectBuffer());
            if (isSSLEnabled()) {
                channel = new SecureNio2Channel(bufhandler, this);
            } else {
                channel = new Nio2Channel(bufhandler);
            }
        }
        Nio2SocketWrapper socketWrapper = new Nio2SocketWrapper(channel, this);
        channel.reset(socket, socketWrapper);
        socketWrapper.setReadTimeout(getSocketProperties().getSoTimeout());
        socketWrapper.setWriteTimeout(getSocketProperties().getSoTimeout());
        socketWrapper.setKeepAliveLeft(Nio2Endpoint.this.getMaxKeepAliveRequests());
        socketWrapper.setSecure(isSSLEnabled());
        socketWrapper.setReadTimeout(getConnectionTimeout());
        socketWrapper.setWriteTimeout(getConnectionTimeout());
        // Continue processing on another thread
        return processSocket(socketWrapper, SocketEvent.OPEN_READ, true);
    } catch (Throwable t) {
        ...
    }
    return false;
}
```

3. 适用范围

- nio: 连接数目多且连接比较短（轻操作）的架构，比如聊天服务器，并发局限于应用
- aio: 连接数目多且连接比较长（重操作）的架构，比如相册服务器，充分调用OS参与并发操作