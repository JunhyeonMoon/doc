# Zero Copy

https://developer.ibm.com/articles/j-zerocopy/



웹어플리케이션들은 디스크에서 데이터를 읽고 정확히 같은 데이터를 response socket에 담아 보낸다. 커널 영역에서 데이터를 옮길 때, CPU 사이클과 메모리 bandwidth를 소모하는 카피가 일어난다. 이 과정을 zero copy 기법을 통해 제거할 수 있다.(context switching을 줄임) Java에서는 ```java.nio.channels.FileChannel``` 의 메소드인 ```transferTo()```로 Linux와 UNIX 시스템에서 zero copy를 지원한다.



## 기존의 Data transfer

### Copying bytes from a file to a socket

~~~java
File.read(fileDesc, buf, len); //Read buffer, Application buffer
Socket.send(socket, buf, len); //Application buffer, Socket buffer
~~~

이 과정에서 copy operation은 kernel mode와 user mode사이에서 4번의 context switching를 일으킨다(4번 복사).



<img src="https://developer.ibm.com/developer/default/articles/j-zerocopy/images/figure1.gif"/>

**context switching 과정**

1. ```read()```은 user mode에서 kernel mode로 context swtiching를 일으킨다. 내부적으로 ```sys_call()```을 호출해서 direct memory access(DMA) 엔진에 의해 파일에서 데이터를 읽는 첫 복사가 일어난다. 디스크에서 읽은 데이터는 커널 공간에 저장된다.
2. Read buffer에서 user buffer로 데이터가 복사되고, ```read()```는 return한다. kernel에서 user mode로 context swtiching이 일어난다.
3. ```send()```에 의해 user mode에서 kernel mode로 context switching이 일어난다. 그리고 커널 공간으로 3번째 복사를 한다.
4. ```send()```가 return을 호출하면서 4번째 context switching을 일으킨다. DMA engine에 의해 4번째 복사를 하면서 데이터는 kernel buffer에서 protocol engine로 옮겨진다.



## Zero-copy approach

기존 과정을 보면, 2번째와 3번째 복사는 불필요하다. 데이터를 read buffer에서 socket buffer로 직접 보낸다면 불필요한 과정을 생략할 수 있고, ```transferTo()```가 그 기능을 지원한다.

~~~java
public void transferTo(long position, long count, WritableByteChannel target);
~~~

```transferTo()```는 데이터를 file channel에서 writable byte channel로 전달한다.

<img src="https://developer.ibm.com/developer/default/articles/j-zerocopy/images/figure3.gif"/>



1. DMA engine에 의해 read buffer로 복사를 한다. 복사된 데이터는 ```ssize_t sendfile(int out_fd, int in)fd, off_t *offset, size_t count)```syscall을 통해 output socket으로 옮겨진다.
2. DMA engine이 kernel socket buffer에서 protocol engine으로 데이터를 복사한다.

총 2번의 context switching(user -> kernel -> user), 3번의 copy가 일어났다.



### Improvement

위의 방법으로 context switching를 4번->2번, 데이터 복사를 4번->3번으로 줄였다. 하지만 kernel공간에서 데이터 중복을 없애면 (Read buffer에서 바로 데이터 전달) 복사를 한번 더 줄일 수 있다.

데이터 길이와 장소 정보를 가지고 있는 descriptor에 의해 socket buffer에 데이터를 복사할 필요가 없다.

<img src="https://developer.ibm.com/developer/default/articles/j-zerocopy/images/figure5.gif"/>

결과적으로, 2번의 context switching, 2번의 copy로 데이터를 전달할 수 있다.



