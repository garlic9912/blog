`UDP`（`User Datagram Protocol`，用户数据报协议）是一种**无连接**的传输层协议。它不建立连接，不保证数据可靠到达，但具有**传输速度快、延迟低、资源消耗少**的特点，适合实时音视频、游戏、广播等场景
![[qianrushi_04.png]]

# 与`TCP`对比
| 对比项       | `TCP`      | `UDP`           |
| --------- | ---------- | --------------- |
| **连接状态**  | 面向连接（三次握手） | 无连接             |
| **可靠性**   | 可靠（确认重传）   | 不可靠（可能丢包、乱序）    |
| **速度**    | 较慢         | 快               |
| **资源消耗**  | 高（维护连接状态）  | 低（无状态）          |
| **数据边界**  | 流式（无边界）    | 数据报（有边界）        |
| **广播/组播** | 不支持        | 支持              |
| **适用场景**  | 文件传输、网页、邮件 | 视频流、游戏、`DNS`、广播 |

# 核心函数
| 函数           | 作用                                |
| ------------ | --------------------------------- |
| `socket()`   | 创建 `UDP` 套接字（`SOCK_DGRAM`）        |
| `bind()`     | 绑定本地地址和端口（服务器端必需）                 |
| `sendto()`   | 发送数据报（指定目标地址）                     |
| `recvfrom()` | 接收数据报（获取发送方地址）                    |
| `connect()`  | `UDP` 也可调用 `connect()` 绑定对端地址（可选） |
| `close()`    | 关闭套接字                             |
## `sendto()`发送数据报
```c
#include <sys/socket.h>
ssize_t sendto(int sockfd, const void *buf, size_t len, int flags,
               const struct sockaddr *dest_addr, socklen_t addrlen);
```

| 参数          | 说明                                            |
| ----------- | --------------------------------------------- |
| `sockfd`    | UDP 套接字描述符                                    |
| `buf`       | 要发送的数据缓冲区                                     |
| `len`       | 要发送的数据长度（字节）                                  |
| `flags`     | 通常设为 `0`（可选 `MSG_DONTWAIT`、`MSG_NOSIGNAL`）    |
| `dest_addr` | 目标地址结构体（包含 IP 和端口）                            |
| `addrlen`   | `dest_addr` 的大小（`sizeof(struct sockaddr_in)`） |
**返回值**：成功返回实际发送的字节数;  失败返回 -1，并设置 `errno`


## `recvfrom()`接收数据报
```c
#include <sys/socket.h>
ssize_t recvfrom(int sockfd, void *buf, size_t len, int flags,
                 struct sockaddr *src_addr, socklen_t *addrlen);
```

| 参数         | 说明                                     |
| ---------- | -------------------------------------- |
| `flags`    | 通常设为 `0`（可选 `MSG_DONTWAIT`、`MSG_PEEK`） |
| `src_addr` | 输出参数，存放发送方的地址信息（可为 `NULL`）             |
| `addrlen`  | 输入时指定 `src_addr` 的大小，输出时填充实际地址长度       |
**返回值**：成功返回实际接收的字节数;  失败返回 -1，并设置 `errno`


# `UDP`的`connect()`
虽然 `UDP` 是无连接的，但可以调用 `connect()` 来“绑定”对端地址。这不是真正的连接，而是让内核知道目标地址，以便使用 `send()` / `recv()` 代替 `sendto()` / `recvfrom()`
```c
// 绑定对端地址（UDP 的 connect）
struct sockaddr_in peer_addr;
peer_addr.sin_family = AF_INET;
peer_addr.sin_port = htons(8080);
inet_pton(AF_INET, "192.168.1.100", &peer_addr.sin_addr);
connect(sockfd, (struct sockaddr *)&peer_addr, sizeof(peer_addr));

// 之后可以使用 send() / recv()
send(sockfd, buffer, len, 0);
recv(sockfd, buffer, len, 0);

// 解除绑定
struct sockaddr_in any;
any.sin_family = AF_UNSPEC;
connect(sockfd, (struct sockaddr *)&any, sizeof(any));
```

# `UDP`广播
```c
// 开启广播选项
int broadcast = 1;
setsockopt(sockfd, SOL_SOCKET, SO_BROADCAST, &broadcast, sizeof(broadcast));

// 发送广播数据（目标地址设为广播地址）
struct sockaddr_in broadcast_addr;
broadcast_addr.sin_family = AF_INET;
broadcast_addr.sin_port = htons(8080);
broadcast_addr.sin_addr.s_addr = INADDR_BROADCAST;   // 255.255.255.255

sendto(sockfd, data, len, 0, (struct sockaddr *)&broadcast_addr, sizeof(broadcast_addr));
```
