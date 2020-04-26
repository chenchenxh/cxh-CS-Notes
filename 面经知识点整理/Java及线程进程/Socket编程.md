https://www.jianshu.com/p/fb4dfab4eec1

## 服务端示例

- 获取ServerSocket
- ServerSocket->Socket
- Socket -> getInputStream/getOutputStream -> write/read
- close

```java
public class Server {

    public static void main(String[] args) {
     
        try {
            // 为了看流程，我就把所有的代码都放在main函数里了,也没有捕捉异常，直接抛出去了。实际开发中不可取。
            // 1.新建ServerSocket对象，创建指定端口的连接
            ServerSocket serverSocket = new ServerSocket(12306);
            System.out.println("服务端监听开始了~~~~");
            // 2.进行监听
            Socket socket = serverSocket.accept();// 开始监听9999端口，并接收到此套接字的连接。
            // 3.拿到输入流（客户端发送的信息就在这里）
            InputStream is = socket.getInputStream();
            // 4.解析数据
            InputStreamReader reader = new InputStreamReader(is);
            BufferedReader bufReader = new BufferedReader(reader);
            String s = null;
            StringBuffer sb = new StringBuffer();
            while ((s = bufReader.readLine()) != null) {
                sb.append(s);
            }
            System.out.println("服务器：" + sb.toString());
            // 关闭输入流
            socket.shutdownInput();

            OutputStream os = socket.getOutputStream();
            os.write(("我是服务端,客户端发给我的数据就是："+sb.toString()).getBytes());
            os.flush();
            // 关闭输出流
            socket.shutdownOutput();
            os.close();

            // 关闭IO资源
            bufReader.close();
            reader.close();
            is.close();

            socket.close();// 关闭socket
            serverSocket.close();// 关闭ServerSocket

        } catch (IOException e) {
            e.printStackTrace();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

## 客户端

直接new Socket，然后跟服务端类似，从第二个开始

