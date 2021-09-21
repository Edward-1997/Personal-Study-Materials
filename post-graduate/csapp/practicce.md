第二章：

编写过程is_little_endian，当在小端机器上编译运行时返回1，在大端机器上编译运行时返回0；

```c
typedef unsigned char *byte;
int is_little_endian(){
    unsigned num = 0x12345678;
    int len = sizeof(int);
    byte p0 = (byte)&num;
    int i;
    for (i = 0; i < len; i++){
        unsigned char lower = (num >> (i << 3)) & 0xFF;
        if (p0[i] != lower){
            return 0;
        }
    }
    return 1;
}
```

2.61写一个C表达式，在下列描述的条件下产生1，而在其他情况下得到0，假设x是int类型

A x的任何位都等于1

B x的任何位都等于0

C x的最低有效字节中的位都等于1

D x的最高有效字节中的位都等于0

```c
int any_bit_eq_one(int x){
    return !(~x);
}

int any_bit_eq_zero(int x){
    return !x;
}

int lowest_byte_eq_one(int x){// x = 0x123456FF
    unsigned char i = x & 0xff; // x = 0xFF
    i = ~i; // if x == 0xFF x = 0x00 else x != 0x00
    return !i;
}
int highest_byte_eq_zero(int x){
    int w = sizeof(int) << 3;
    unsigned mask = 0xff;
    mask = mask << (w - 8);
    return !(mask & x);
}
```

2.62 将下面的C函数代码补充完整。函数 srl 用算术右移来完成逻辑右移；函数sra用逻辑左移完成算术右移。后面操作不包括除法和右移。

