## Preface

This document describes the DoD Standard Transmission Control Protocol(TCP). There have been nine earlier editions of the ARPA TCP specification on which this standard is based, and the present text draws heavily from them.There have been many contributors to this work
both in terms of concepts and in terms of text. This edition clarifies several details and removes the end-of-letter buffer-size adjustments,and redescribes the letter mechanism as a push function.

这篇文档描述了DoD标准下的TCP协议。这个标准是基于早期的9篇关于TCP的文章，并且着重的描述了这些文档。有相当多无私奉献的人致力于创建这些文档和概念。作为推广功能，这篇文章澄清了一些细节，移除了以字母结尾的缓存区大小适应，并且重新描述了传送机制。



基础数据转换：

TCP协议能够在用户之间直接传输一个连续的字节流。为了能够在网络系统中传输，这些字节流将会被包装进报文段中。通常来说，TCP协议决定了什么时候阻塞和分发数据。

有时用户需要确保，他们发送给TCP协议的所有数据都会被传输。因此，为了实现这个目的，TCP协议需要定







