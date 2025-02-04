<div align=center>
	<img src="_v_images/20200904171558212_22234.png" width="300"> 
</div>

<br/>
<br/>
<br/>

<center><font size='20'>FD.io VPP：源码文档</font></center>

<br/>
<br/>
<center><font size='5'>荣涛</font></center>
<center><font size='5'>2020年9月</font></center>
<br/>
<br/>
<br/>
<br/>

[FD.io VPP ](https://docs.fd.io/vpp/20.05/index.html)



# 1. VPP用户文档

## 1.1. 英特尔AVF设备插件

[Intel AVF device plugin for VPP](https://docs.fd.io/vpp/20.05/d1/def/avf_plugin_doc.html)

## 1.2. 双向转发检测

[BFD module](https://docs.fd.io/vpp/20.05/de/d7a/bfd_doc.html)

## 1.3. vcl-ldpreload：使用VPP通信库（VCL）的LD_PRELOAD库。

这部分官方文档说明不是很明确。

[a LD_PRELOAD library that uses the VPP Communications Library (VCL).](https://docs.fd.io/vpp/20.05/d6/dcf/vcl_ldpreload_doc.html)

用户可以`LD_PRELOAD`使用`POSIX套接字API的任何应用程序`。

注意：源已移至`... / vpp / src / vcl`，并且`libvcl_ldpreload.so`是使用VPP构建的，可以在`.../vpp/build-root/install-vpp[_debug]-native/vpp/lib`中找到

### 1.3.1. 运行demo

运行不带参数的测试脚本以查看帮助菜单：

```bash
export WS_ROOT=<top level="" vpp="" git="" repo="" dir>=""> (e.g. /scratch/my_name/vpp) $WS_ROOT/test/scripts/socket_test.sh
```

### 1.3.2. Docker iPerf示例


这些启动`xterms`。 要退出，请关闭xterms并运行以下docker kill cmd（警告：这将杀死所有docker容器！）

```bash
docker kill $(docker ps -q)
```

使用默认Linux Bridge的Docker iPerf

```bash
$WS_ROOT/test/scripts/socket_test.sh -bi docker-kernel
```
使用VPP的Docker iPerf

```bash
$WS_ROOT/test/scripts/socket_test.sh -bi docker-preload
```

# 2. VPP开发者文档

## 2.1. 测试框架文档

[Test Framework Documentation](https://docs.fd.io/vpp/20.05/dd/d89/test_framework_doc.html)

* [Test framework documentation for VPP 20.05](https://docs.fd.io/vpp/20.05/vpp_make_test/html/)
* Test framework documentation for VPP 20.01
* Test framework documentation for VPP 19.08
* Test framework documentation for VPP 19.04
* Test framework documentation for VPP 19.01
* Test framework documentation for VPP 18.10
* Test framework documentation for VPP 18.07
* Test framework documentation for VPP 18.04
* Test framework documentation for VPP 18.01
* Test framework documentation for VPP 17.10
* Test framework documentation for VPP 17.04
* Test framework documentation for VPP 17.01

## 2.2. VPP插件示例

[Sample plugin for VPP](https://docs.fd.io/vpp/20.05/d9/de7/sample_plugin_doc.html)

官方文档中，这部分内容与```20.05```版本冲突。

![FD](_v_images/20200904131127984_632.png =499x)


## 2.3. 二进制接口

[Binary API support](https://docs.fd.io/vpp/20.05/df/d20/api_doc.html)

VPP提供了一种二进制API方案，以允许各种各样的客户端代码对数据平面表进行编程。 在撰写本文时，有数百种二进制API。

消息在* .api文件中定义。 如今，大约有50个api文件，随着人们添加可编程功能，更多文件将陆续到来。 API文件编译器源位于```src/tools/vppapigen```中。

在```src/vnet/interface.api```中，这是一个典型的请求/响应消息定义：

```c
autoreply define sw_interface_set_flags
{
  u32 client_index;
  u32 context;
  u32 sw_if_index;
  /* 1 = up, 0 = down */
  u8 admin_up_down;
};
```

首先，API编译器将此定义呈现到```build-root /.../ vpp / include / vnet / interface.api.h```中，如下所示：

```c
/****** Message ID / handler enum ******/
#ifdef vl_msg_id
vl_msg_id(VL_API_SW_INTERFACE_SET_FLAGS, vl_api_sw_interface_set_flags_t_handler)
vl_msg_id(VL_API_SW_INTERFACE_SET_FLAGS_REPLY, vl_api_sw_interface_set_flags_reply_t_handler)
#endif     
/****** Message names ******/
#ifdef vl_msg_name
vl_msg_name(vl_api_sw_interface_set_flags_t, 1)
vl_msg_name(vl_api_sw_interface_set_flags_reply_t, 1)
#endif      
/****** Message name, crc list ******/
#ifdef vl_msg_name_crc_list
#define foreach_vl_msg_name_crc_interface \
_(VL_API_SW_INTERFACE_SET_FLAGS, sw_interface_set_flags, f890584a) \
_(VL_API_SW_INTERFACE_SET_FLAGS_REPLY, sw_interface_set_flags_reply, dfbf3afa) \
#endif      
/****** Typedefs *****/
#ifdef vl_typedefs
typedef VL_API_PACKED(struct _vl_api_sw_interface_set_flags {
    u16 _vl_msg_id;
    u32 client_index;
    u32 context;
    u32 sw_if_index;
    u8 admin_up_down;
}) vl_api_sw_interface_set_flags_t;
typedef VL_API_PACKED(struct _vl_api_sw_interface_set_flags_reply {
    u16 _vl_msg_id;
    u32 context;
    i32 retval;
}) vl_api_sw_interface_set_flags_reply_t;
...
#endif /* vl_typedefs */
```

要更改接口的管理状态，二进制api客户端会将```vl_api_sw_interface_set_flags_t```发送到VPP，VPP会以```vl_api_sw_interface_set_flags_reply_t```消息进行响应。

多层软件，传输类型和共享库实现了多种功能：

* API消息分配，跟踪，精美打印和重播。
* 通过全局共享内存，成对/专用共享内存和套接字的消息传输。
* 跨线程不安全消息处理程序的工作线程的屏障同步。

正确编码的消息处理程序对用于向VPP传递消息或从VPP传递消息的传输一无所知。同时使用多种API消息传输类型是相当合理的。

由于历史原因，二进制api消息（可能）以网络字节顺序发送。在撰写本文时，我们正在认真考虑该选择是否有意义。

### 2.3.1. 消息分配

由于二进制API消息始终按顺序处理，因此我们会尽可能使用环形分配器分配消息。与传统的内存分配器相比，该方案非常快，并且不会引起堆碎片。请参见```src / vlibmemory / memory_shared.c``` ```vl_msg_api_alloc_internal（）```。

无论采用哪种传输方式，二进制api消息始终遵循```msgbuf_t```标头：

```c
typedef struct msgbuf_
{
  unix_shared_memory_queue_t *q;
  u32 data_len;
  u32 gc_mark_timestamp;
  u8 data[0];
} msgbuf_t;
```

这种结构使跟踪消息变得容易，而不必对其进行解码-只需保存```data_len```字节-并允许```vl_msg_api_free（）```快速处理消息缓冲区：

```c
void
vl_msg_api_free (void *a)
{
  msgbuf_t *rv;
  api_main_t *am = &api_main;
  rv = (msgbuf_t *) (((u8 *) a) - offsetof (msgbuf_t, data));
  /*
   * Here's the beauty of the scheme.  Only one proc/thread has
   * control of a given message buffer. To free a buffer, we just 
   * clear the queue field, and leave. No locks, no hits, no errors...
   */
  if (rv->q)
    {
      rv->q = 0;
      rv->gc_mark_timestamp = 0;
      return;
    }
}
```

### 2.3.2. 消息跟踪和重播

VPP可以捕获并重放相当大的二进制API跟踪，这一点非常重要。涉及数十万个API事务的系统级问题可以在一秒钟或更短的时间内重新运行。部分重播允许对车轮掉落的点进行二进制搜索。可以在数据平面上添加脚手架，以在遇到复杂情况时触发。

通过二进制API跟踪，打印和重播，系统级错误报告的格式为“在300,000次API事务之后，VPP数据平面停止转发流量，FIX IT！”可以离线解决。

人们经常发现，控制平面客户端经过很长时间或在复杂环境下对数据平面进行了错误编程。没有直接的证据，“这是一个数据平面问题！”

请参阅```src / vlibmemory / memory_vlib :: c vl_msg_api_process_file（）```和```src / vlibapi / api_shared.c```。另请参见调试CLI命令“ api跟踪”

### 2.3.3. 客户端连接详细信息

从C语言客户端到VPP建立二进制API连接很容易：

```c
int
connect_to_vpe (char *client_name, int client_message_queue_length)
{
  vat_main_t *vam = &vat_main;
  api_main_t *am = &api_main;
  if (vl_client_connect_to_vlib ("/vpe-api", client_name, 
                                client_message_queue_length) < 0)
    return -1;
  /* Memorize vpp's binary API message input queue address */
  vam->vl_input_queue = am->shmem_hdr->vl_input_queue;
  /* And our client index */
  vam->my_client_index = am->my_client_index;
  return 0;
} 
```

32是```client_message_queue_length```的典型值。 VPP在需要将API消息发送到二进制API客户端时无法阻止，并且VPP端的二进制API消息处理程序非常快。 发送异步消息时，请务必热情地刮擦二进制API rx环。

### 2.3.4. 二进制API消息RX pthread

调用```vl_client_connect_to_vlib```会启动二进制API消息RX pthread：

```c
static void *
rx_thread_fn (void *arg)
{
  unix_shared_memory_queue_t *q;
  memory_client_main_t *mm = &memory_client_main;
  api_main_t *am = &api_main;
  q = am->vl_input_queue;
  /* So we can make the rx thread terminate cleanly */
  if (setjmp (mm->rx_thread_jmpbuf) == 0)
    {
      mm->rx_thread_jmpbuf_valid = 1;
      while (1)
        {
          vl_msg_api_queue_handler (q);
        }
    }
  pthread_exit (0);
}
```

要自己处理二进制API消息队列，请使用```vl_client_connect_to_vlib_no_rx_pthread```。

反过来，```vl_msg_api_queue_handler（...）```使用```mutex/ condvar```信号唤醒，处理```VPP->client```流量，然后进入睡眠状态。当```VPP->Client```API消息队列从空变为非空时，VPP提供```condvar```广播。

VPP以很高的速度检查自己的二进制API输入队列。 VPP根据数据平面数据包处理要求，以可变的速率在“进程”上下文（也称为协作多任务线程上下文）中调用消息处理程序。

### 2.3.5. 客户端断开连接详细信息

要与VPP断开连接，请调用```vl_client_disconnect_from_vlib```。如果客户端应用程序异常终止，请调用此函数。 VPP竭尽全力为死客户举行一场体面的葬礼，但VPP无法保证释放共享二进制API段中泄漏的内存。

### 2.3.6. 将二进制API消息发送到VPP

重点是将二进制API消息发送到VPP，并接收来自VPP的答复。许多VPP二进制API包含一个客户端请求消息和一个简单的状态回复。例如，要设置界面的管理员状态，请编写以下代码：

```c
vl_api_sw_interface_set_flags_t *mp;
mp = vl_msg_api_alloc (sizeof (*mp));
memset (mp, 0, sizeof (*mp));
mp->_vl_msg_id = clib_host_to_net_u16 (VL_API_SW_INTERFACE_SET_FLAGS);
mp->client_index = api_main.my_client_index;
mp->sw_if_index = clib_host_to_net_u32 (<interface-sw-if-index>);
vl_msg_api_send (api_main.shmem_hdr->vl_input_queue, (u8 *)mp);
```

关键点：

* 使用```vl_msg_api_alloc```分配消息缓冲区
* 分配的消息缓冲区未初始化，必须假定包含垃圾箱。
* 不要忘记设置```_vl_msg_id```字段！
* 在撰写本文时，二进制API消息ID和数据以网络字节顺序发送
* 客户端库全局数据结构```api_main```跟踪用于与VPP通信的足够的指针和句柄

### 2.3.7. 从VPP接收二进制API消息

除非您进行了其他安排（请参见```vl_client_connect_to_vlib_no_rx_pthread```），否则将在单独的rx pthread上接收消息。 与客户端应用程序主线程同步是应用程序的责任！

如下设置消息处理程序：

```c
#define vl_typedefs         /* define message structures */
#include <vpp/api/vpe_all_api_h.h>
#undef vl_typedefs
/* declare message handlers for each api */
#define vl_endianfun                /* define message structures */
#include <vpp/api/vpe_all_api_h.h>
#undef vl_endianfun
/* instantiate all the print functions we know about */
#define vl_print(handle, ...)
#define vl_printfun
#include <vpp/api/vpe_all_api_h.h>
#undef vl_printfun
/* Define a list of all message that the client handles */
#define foreach_vpe_api_reply_msg                            \
   _(SW_INTERFACE_SET_FLAGS_REPLY, sw_interface_set_flags_reply)
   static clib_error_t *
   my_api_hookup (vlib_main_t * vm)
   {
     api_main_t *am = &api_main;
   #define _(N,n)                                                  \
       vl_msg_api_set_handlers(VL_API_##N, #n,                     \
                              vl_api_##n##_t_handler,              \
                              vl_noop_handler,                     \
                              vl_api_##n##_t_endian,               \
                              vl_api_##n##_t_print,                \
                              sizeof(vl_api_##n##_t), 1);
     foreach_vpe_api_msg;
   #undef _
     return 0;
    }
```

用于建立消息处理程序的关键API是```vl_msg_api_set_handlers```，它在```api_main_t```结构中的多个并行向量中设置值。 撰写本文时：并非所有矢量元素值都可以通过API设置。 您会看到零星的API消息注册，然后对该表单进行一些细微调整：

```c
/*
 * Thread-safe API messages
 */
am->is_mp_safe[VL_API_IP_ADD_DEL_ROUTE] = 1;
am->is_mp_safe[VL_API_GET_NODE_GRAPH] = 1;
```

## 2.4. VPP API模型

[VPP API module](https://docs.fd.io/vpp/20.05/d4/d09/vapi_doc.html)

### 2.4.1. 综述

VPP API模块允许通过共享内存接口与VPP通信。 该API包含3部分：

* 通用代码-低级API
* 生成的代码-高级API
* 代码生成器-生成自己的高级API，例如 用于自定义插件

#### 2.4.1.1. 通用代码

* C：C通用代码表示基本的低级API，提供连接/断开连接，执行消息发现以及发送/接收消息的功能。 C变体位于```vapi.h```中。
* C++：C ++由```vapi.hpp```提供，并包含高级API模板，这些模板由生成的代码专用。

#### 2.4.1.2. 生成代码

源代码树中存在的每个API文件都会自动转换为**JSON文件**，代码生成器会解析并生成C（`vapi_c_gen.py`）或C ++（`vapi_cpp_gen.py`）代码。

然后可以将其包含在客户端应用程序中，并提供与VPP进行交互的便捷方法。这包括：

* 自动字节交换
* 基于上下文的自动请求-响应匹配
* 调用回调时自动强制转换为适当的类型（类型安全）
* 自动发送转储邮件的控制命令

该API支持两种操作模式：

* 阻塞
* 非阻塞

**在阻塞模式下**，每当启动操作时，代码就会等待，直到操作完成为止。这意味着在发送消息时，调用会阻塞，直到消息可以写入共享内存为止。同样，接收消息会一直阻塞，直到有消息可用为止。在更高级别上，这还意味着在执行请求（例如show_version）时，调用将阻塞，直到响应返回（例如show_version_reply）为止。

**在非阻塞模式下**，它们是解耦的，每当无法执行操作时，API都会返回VAPI_EAGAIN，并且在发送请求后，由客户端等待和处理响应。

#### 2.4.1.3. 代码生成器

Python代码生成器有两种形式-`C和C ++`，并生成高级API标头。所有代码都存储在标题中。

### 2.4.2. 用法

#### 2.4.2.1. 低级API

有关功能的描述，请参阅`vapi.h`标头中`doxygen`格式的内联API文档。 建议使用专用标头（例如`vpe.api.vapi.h`或`vpe.api.vapi.hpp`）提供的更安全的高级API。

#### 2.4.2.2. C高级API

##### 2.4.2.2.1. 回调

C高级API严格基于回调，以实现最大效率。 每当启动操作时，带有回调上下文的回调就是该操作的一部分。 然后，当响应（或多个响应）与请求绑定在一起时，将调用回调。 另外，如果注册了回调，则每当事件到达时都会调用回调。 所有指向响应/事件的指针都指向共享内存，并在回调完成后立即释放，因此客户端需要提取/复制其感兴趣的任何数据。

##### 2.4.2.2.2. 阻塞模式

在简单阻塞模式下，整个操作（作为简单请求或转储）已完成，并且在函数调用期间调用了其回调（对于转储可能多次）。

在这种模式下，一个简单请求的伪代码示例：

`vapi_show_version（message, callback, callback_context）`

* 1.生成唯一的内部上下文并将其分配给`message.header.context`
* 2.将消息交换为网络字节顺序
* 3.向vpp发送消息（消息已被消耗，vpp将释放它）
* 4.创建内部“未完成的请求上下文”，用于存储回调，回调上下文和内部上下文值
* 5.呼叫分派，在这种模式下，它接收并处理响应，直到内部“未完成的请求”队列为空。 在阻止模式下，此队列始终最多包含一项。

注意：在某些情况下，可能会在调用响应回调之前调用不同的-不相关的回调。 事件存储在共享内存队列中。


##### 2.4.2.2.3. 非阻塞模式

在非阻塞模式下，所有请求仅被字节交换，上下文信息和回调一起存储在本地（因此，在以上示例中，仅执行步骤`1-4`，而跳过了步骤`5`。 调用调度取决于客户端应用程序。 这允许在发送/接收消息之间交替，或具有调用分派的专用线程。

#### 2.4.2.3. C ++高级API

##### 2.4.2.3.1. 回调

在C ++ API中，响应自动绑定到相应的`Request`，`Dump`或`Event_registration`对象。可以选择指定一个回调，然后在收到响应时调用该回调。

注意：响应占用共享内存空间，应在不再需要时手动释放（对于结果集）或自动释放（通过销毁拥有它们的对象）。一旦执行了`Request`或`Dump`对象，就无法重新发送该对象，因为该请求本身（存储在共享内存中）已被vpp占用，并且无法访问（设置为nullptr）。

##### 2.4.2.3.2. 用法

###### 2.4.2.3.2.1. 请求和转储

在Connection类型的对象上创建并调用`connect（）`以连接到vpp。

* 1.使用typedef创建一个`Request`或`Dump`类型的对象（例如`Show_version`）
* 2.如果需要，使用`get_request（）`获取和处理基础请求。
* 3.发出`execute（）`发送请求。
* 4.使用`wait_for_response（）`或`dispatch（）`等待响应。
* 5.使用`get_response_state（）`获取状态，并使用`get_response（）`读取响应。

###### 2.4.2.3.2.2. Events

创建一个连接并执行适当的请求以订阅事件（例如`Want_stats`）

* 1.使用模板参数创建一个`Event_registration`，模板参数是您要插入的事件的类型。
* 2.调用`dispatch（）`或`wait_for_response（）`等待事件。事件发生时（如果传递给`Event_registration（）`构造函数），将调用回调。或者，读取结果集。

注意：结果集中存储的事件会占用共享内存中的空间，因此应定期释放（例如，在处理事件后在回调中释放）。


## 2.5. ACL插件恒定时间查找设计

[ACL plugin constant-time lookup design](https://docs.fd.io/vpp/20.05/d8/d83/acl_hash_lookup.html)

ACL插件的初始实现执行一个琐碎的`for（）`周期，以每个数据包为基础遍历分配的ACL。 即使对于非常短的ACL（由于其简单性），它也可以击败更高级的方法，但这并不是非常有效。

但是，要涵盖具有可接受性能的较长ACL的情况，我们需要有一种更好的匹配方法。 本文提出了一种机制，用于从O（M）（其中M是条目数）到O（N）（其中N是不同掩码组合的数量）进行查找。

### 2.5.1. ACL(s)的准备

ACL插件将维护“掩码类型”的全局列表，即ACE中“无关”位的特定配置。创建新的ACL后，将通过所有ACE，以分配并可能分配“掩码类型编号”。

每个ACL都有一个`hash_acl_info_t`结构，表示与该ACL相关的信息的“基于哈希”的部分，主要是`hash_ace_info_t`结构的数组-该数组的每个成员都与原始ACL中的规则（ACE）相对应，例如他们有一对`*（acl_index，ace_index）*`可以跟踪，主要用于调试。

为什么我们需要一个完整的单独结构，而不是在现有规则结构中添加新字段？首先，进行封装，以最大程度地减少基于散列的查找工件对主ACL代码的污染。第二，一个规则可能对应于一个以上的“基于哈希”的ACE。实际上，大多数规则确实与其中两个相对应。为什么呢？

考虑到当前的ACL查找逻辑是：如果数据包不是初始片段，并且有一个L4条目作用于该数据包，则仅根据L4协议字段值而不是协议和端口值进行比较。此行为由`acl_main`中的`l4_match_nonfirst_fragment`标志控制，并且需要与现有软件交换机实现保持兼容性。

虽然对`single_acl_match_5tuple（）`中的顺序检查来说，很容易通过在适当的时机进行突破来实现，但是在基于散列的匹配的情况下，这要花费我们**两次检查**：一项是对完整的5元组的检查，而标志`pkt.is_nonfirst_fragment`是`0`，三元组中的第二个，标志`pkt.is_nonfirst_fragment`为`1`，第二个检查由`acl_main.l4_match_nonfirst_fragment`设置触发为默认值`1`。这表明在给定`hash_ace_info_t`元素中必须有一个`“ match”`字段，这将反映应用蒙版后应该匹配的值。

在其他情况下，将原始ACL中的给定规则扩展为多个可能会有所好处-例如，作为对小端口范围的端口范围处理的优化（截至撰写本文时尚未完成）。

### 2.5.2. 将ACL分配给接口

将ACL列表分配给接口后，或者将新的ACL添加到应用于该接口的现有ACL列表中之后，我们需要更新`bihash`来加速查找。

用于查找的所有条目都存储在单个`48_8 bihash`中，后者从数据包中捕获5元组以及其他每个数据包的信息标志，例如： `l4_valid，is_non_first_fragment`等。为了便于所有接口使用单个`bihash，is_ip6，is_input，sw_if_index和keymask_type_index`是密钥的一部分-后者是必需的，因为可以存在具有相同值但不同掩码的条目，例如：`permit ::/0, permit::/128.`。

在将ACL应用于接口时，我们需要遍历与该ACL对应的`hash_ace_info_t`条目的列表，并使用与这些条目中的匹配值相对应的键来更新bihash。

哈希匹配的值包含对`applied_ace_hash_entry_t`元素的每个`* sw_if_index *`向量的索引，以及几个标志：阴影（优化：如果匹配条目上的该标志为零，则意味着我们可以尽早停止查找并声明一个匹配项（请参见下文）和`need_portrange_check`-表示匹配项是实际匹配项的超集，我们需要执行一次额外的检查。

同样，在插入时，我们必须记住，同一密钥可以有多个`applied_ace_hash_entry_t`，并且必须保留这些列表。这对于将ACL作为ACL向量的一部分递增地应用/取消应用是必要的：例如，两个ACL具有“ `permit 2001：db8 :: 1/128 any`”-即使我们可以保留第二个ACL的条目，已经删除了第一个。另外，如果有两个条目具有相同的键但端口范围不同，例如0..42和142..65535-如果我们决定不将其扩展为特定于特定端口的端口，则我们需要能够顺序匹配这些条目条目。

`~~TODO~~`

## 2.6. VPP API语言

[VPP API Language](https://docs.fd.io/vpp/20.05/df/dbb/api_lang_doc.html)

VPP二进制API是消息传递API。 **`VPP API语言用于定义VPP及其控制平面之间的RPC接口`**。 API消息支持共享内存传输和Unix域套接字（SOCK_STREAM）。

有线格式本质上是网络格式（大端）打包C结构的格式。

VPP API编译器位于src / tools / vppapigen中，当前可以编译为JSON或C（由VPP二进制文件本身使用）。

### 2.6.1. 语言定义
#### 2.6.1.1. 定义消息

消息交换分为3种类型：

* **请求/答复客户端发送请求消息**，服务器通过一条答复消息进行答复。约定是将回复消息命名为`method_name + _reply`。
* **转储/详细信息客户端向服务器发送“批量”请求消息**，服务器用一组详细信息进行答复。这些消息可能是不同的类型。转储/详细调用必须包含在控制ping块中（否则客户端将不知道批量传输的结束）。方法名称必须以`method+“ \ _dump”`结尾，回复消息应命名为`method+“ \ _details”`。这里的例外是返回多种消息类型（例如`sw_interface_dump`）的方法。转储/详细信息方法通常用于获取批量信息，例如完整的FIB表。
* **事件客户端可以注册以从服务器获取异步通知**。这对于获取接口状态更改等很有用。用于请求通知的方法名称通常以“ want_”为前缀。例如。 “` want_interface_events`”。服务定义中定义了事件注册产生的通知类型。

来自客户端的消息必须包含“ `client_index`”，标识发件人的`不透明cookie`和“` context`”字段，以使客户端将请求与答复进行匹配。

消息定义的示例。客户端发送`show_version`请求，服务器回复`show_version_reply`。

**所有请求中都必须有`client_index`和`context`字段**。上下文由服务器返回，并由客户端用于匹配请求和答复消息。

```c
define show_version
{
  u32 client_index;
  u32 context;
};
define show_version_reply
{
  u32 context;
  i32 retval;
  string program [32];
  string version [32];
  string build_date [32];
  /* The final field can be a variable length argument */
  string build_directory [];
};
```
客户端不使用这些标志，但是对于API的某些跟踪和调试有特殊的含义。 自动回复标志是仅带有retval字段的回复消息的简写。

```c
define : DEFINE ID '{' block_statements_opt '}' ';'
define : flist DEFINE ID '{' block_statements_opt '}' ';'
flist : flag
      | flist flag
flag : MANUAL_PRINT
     | MANUAL_ENDIAN
     | DONT_TRACE
     | AUTOREPLY
block_statements_opt : block_statements
block_statements : block_statement
                 | block_statements block_statement
block_statement : declaration
                | option
declaration : type_specifier ID ';'
            | type_specifier ID '[' ID '=' assignee ']' ';'
declaration : type_specifier ID '[' NUM ']' ';'
            | type_specifier ID '[' ID ']' ';'
type_specifier : U8
               | U16
               | U32
               | U64
               | I8
               | I16
               | I32
               | I64
               | F64
               | BOOL
               | STRING
type_specifier : ID
```

**`TODO`**









<br/>
<div align=right>	以上内容由荣涛翻译整理自网络。</div>