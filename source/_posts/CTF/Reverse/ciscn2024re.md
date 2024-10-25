---
title: CISCN2024复现及总结
categories: 
- [ CTF ,逆向工程]
tags:
- 汇编
- Re
---

# Asm_re
题目给了一个文本文件包含arm汇编(是ida的片段)，大部分re手都只会x86汇编，arm汇编很少接触。
下面给出两种办法获得伪c片段

### 方法一 CHATGPT
```c
#include <stdio.h>
#include <string.h>

// 假设一个类似的全局变量
extern char __stack_chk_guard_ptr;
extern char unk_100003F10;
extern void __chkstk_darwin();

void main() {
    // 栈变量
    char stack_guard;
    char var_18;
    char var_B4;
    int index=0;
    char flag[] = "flag{xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx}";
    char var_C0[128];
    char lena;
    char var_D0[128];
    char var_8[128];
    char var_100[128];
    char var_D8[128];
    char var_DC[128];
    char var_E0[128];
    char var_E4[128];
    char var_E8[128];
    char var_EC[128];
    char var_F0[128];
    char var_F4[128];
    char var_F8[128];
    
    // 初始化
    memcpy(var_C0, unk_100003F10, 0x98);
    size_t len = strlen(flag);
    lena = len + 1;
    
    // 主循环
    while (index < lena) {
        char c = flag[index];
        c = (c * 'P' + 0x14) ^ 'M';
        c += 0x1E;
        var_100[index] = c;
        index++;
    }
    
    // 打印换行
    printf("\n");
    var_F4[0] = 1;
    var_F8[0] = 0;
    
    // 另一段主循环
    while (var_F8[0] < lena) {
        int comparison_result = var_100[var_F8[0]] == var_8[var_F8[0]];
        if (!comparison_result) {
            var_F4[0] = 0;
            break;
        }
        var_F8[0]++;
    }
    
    // 结果检查
    if (var_F4[0] == 1) {
        printf("Success\n");
    } else {
        printf("Failure\n");
    }
}

```
用gpt4o可以得到比较完善的代码(gpt3.5的话就有点烂了)

### 方法二 提取十六进制数据
```text
__text:0000000100003BBC FC 6F BE A9                   STP             X28, X27, [SP,#-0x10+var_10]!
__text:0000000100003BC0 FD 7B 01 A9                   STP             X29, X30, [SP,#0x10+var_s0]
__text:0000000100003BC4 FD 43 00 91                   ADD             X29, SP, #0x10
__text:0000000100003BC8 FF 03 04 D1                   SUB             SP, SP, #0x100
```
可以看出字节数据都是整齐偏移的
用vscode打开文本文件，shif+alt键进行矩形选择
得到这样的数据
```hex
FC 6F BE A9                  
FD 7B 01 A9                  
FD 43 00 91                  
FF 03 04 D1                  
08 00 00 B0                  
```
打开010编辑器
新建文件->编辑->粘贴自16进制文本->保存到桌面
用ida打开文件，选择汇编语言类型，小端序ARM。
可以看到汇编代码，按p键创建函数，F5即可看到c语言代码。

### 解密
有了伪c代码解密就和洒洒水一样
```c
#include <stdio.h>
int main(){
    char flag[] = "flag{xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx}";
    char mm1[]={0xd7,0x1f,0x00,0x00,0xb7,0x21,0x00,0x00,0x47,0x1e,0x00,0x00,0x27,0x20,0x00,0x00,0xe7,0x26,0x00,0x00,0xd7,0x10,0x00,0x00,0x27,0x11,0x00,0x00,0x07,0x20,0x00,0x00,0xc7,0x11,0x00,0x00,0x47,0x1e,0x00,0x00,0x17,0x10,0x00,0x00,0x17,0x10,0x00,0x00,0xf7,0x11,0x00,0x00,0x07,0x20,0x00,0x00,0x37,0x10,0x00,0x00,0x07,0x11,0x00,0x00,0x17,0x1f,0x00,0x00,0xd7,0x10,0x00,0x00,0x17,0x10,0x00,0x00,0x17,0x10,0x00,0x00,0x67,0x1f,0x00,0x00,0x17,0x10,0x00,0x00,0xc7,0x11,0x00,0x00,0xc7,0x11,0x00,0x00,0x17,0x10,0x00,0x00,0xd7,0x1f,0x00,0x00,0x17,0x1f,0x00,0x00,0x07,0x11,0x00,0x00,0x47,0x0f,0x00,0x00,0x27,0x11,0x00,0x00,0x37,0x10,0x00,0x00,0x47,0x1e,0x00,0x00,0x37,0x10,0x00,0x00,0xd7,0x1f,0x00,0x00,0x07,0x11,0x00,0x00,0xd7,0x1f,0x00,0x00,0x07,0x11,0x00,0x00,0x87,0x27,0x00,0x00};
    int *mm=mm1;
    int index=0;
    size_t len = strlen(flag);
    int  lena = len + 1;
    while (index < lena){
    mm[index]-=0x1e;
    mm[index]^='M';
    mm[index] =(mm[index]-0x14)/'P';; 
    flag[index]=mm[index];
    index++;
    }
    puts(flag);
    
}
```


# Rust_baby
像这种用那些奇葩语言写的二进制文件靠静态分析是很难做出来的
动态调试寻找函数
一直`f8`步过，多窗口观察终端有没有输出什么信息
发现在`sub_7FF7D72B298A`的地方程序输出`where is your flag?:`
我们由此确定主逻辑在这里。
函数有里有发现明显的长base64字符串
拿去解密得到
```json
{
        "henhenaaa!":[1,1,4,5,1,4,1,9,1,9,8,1,0],
        "cryforhelp":"igdydo19TVE13ogW1AT5DgjPzHwPDQle1X7kS8TzHK8S5KCu9mnJ0uCnAQ4aV3CSYUl6QycpibWSLmqm2y/GqW6PNJBZ/C2RZuu+DfQFCxvLGHT5goG8BNl1ji2XB3x9GMg9T8Clatc=",
        "whatadoor":"1145141919810WTF",
        "iwantovisit":"O0PSwantf1agnow1"
        }{
    "where":"where is your flag?:",
    "omg":"correct flag",
    "nonono":"nope, wrong flag"
}
```
这很明显是和加密有关的重要数据。
动调找到输出函数输入函数
从这一段到函数开头有很长一串代码，我们就不用去管它了，这种题只需要关注代码对数据的加密，不然很容易就被绕昏了。
输入函数后面下好断点，在终端输入数据 _flagggggggggggg_ 然后再在输入函数周围寻找对这段字符串的引用,找到引用后观察对输入字符处理
```c
do
  {
    *(_QWORD *)&flagg[0] = 0i64;
    for ( j = 0i64; j != 8; ++j )
    {
      v26 = 'E';
      if ( v23 + j < v20 )
        v26 = flagggggggg[j];
      *((_BYTE *)flagg + j) = v26;
    }
    v197[0] = (LPVOID)sub_7FF7D72B28C3(
                        *(__int64 *)&flagg[0],
                        byte_7FF7D72D2978[(unsigned __int8)(v24 - 3 * (((unsigned __int8)v24 / 3u) & 0xFE))],
                        *((_QWORD *)v188 + v24));
    for ( k = 0i64; k != 8; ++k )
      flagm1[k] = *((_BYTE *)v197 + k) ^ 0x33;
    ++v24;
    flagggggggg += 8;
    v23 += 8i64;
    flagm1 += 8;
}
while ( v24 != 13 );
```
前段是把flag每8组复制一次，剩下的补'E'
中间是个加密函数，先跳过
后面是一个异或
flagm1存储最后的数据，可是对它的调用在`+=8`后就没有了。我们不能得知它在哪又被处理了。
这个时候需要妙用硬件断点分析，对加密后的前几个字节下断点，硬件断点可以下在stack，监听数据的读写，读写时提示触发代码
```asm
Stack[000008D8]:000000DC9FCFEA50                 db  56h ; V  <-breakpoint
Stack[000008D8]:000000DC9FCFEA51                 db  58h ; X
Stack[000008D8]:000000DC9FCFEA52                 db  52h ; R
Stack[000008D8]:000000DC9FCFEA53                 db  54h ; T
Stack[000008D8]:000000DC9FCFEA54                 db  5Bh ; [
Stack[000008D8]:000000DC9FCFEA55                 db  5Bh ; [
```
当触发断点的时候，我们已经从伪c代码的369行跳转到634行，这几百行看起来都极难分析，我们略过，直接观察对加密数据的处理
```c
 do
{
         v93[v97] = v96[v97] ^ *(_BYTE *)(v91 + v97);
          ++v97;
}
while ( v88 != v97 );
```
可以看出，v91是flag加密后的数据，只是个简单的对表异或，这个异或表，可以经过多次调试推断，是个固定的表
```asm
debug027:0000025E67C42FA0                 db 0DCh
debug027:0000025E67C42FA1                 db  5Fh ; _
debug027:0000025E67C42FA2                 db  20h
debug027:0000025E67C42FA3                 db  22h ; "
debug027:0000025E67C42FA4                 db 0C2h
debug027:0000025E67C42FA5                 db  79h ; y
debug027:0000025E67C42FA6                 db  19h
debug027:0000025E67C42FA7                 db  56h ; V
debug027:0000025E67C42FA8                 db  35h ; 5
debug027:0000025E67C42FA9                 db 0DAh
debug027:0000025E67C42FAA                 db  8Bh
debug027:0000025E67C42FAB                 db  47h ; G
debug027:0000025E67C42FAC                 db 0D3h
debug027:0000025E67C42FAD                 db  19h
debug027:0000025E67C42FAE                 db 0FCh
debug027:0000025E67C42FAF                 db  55h ; U
debug027:0000025E67C42FB0                 db 0ABh
debug027:0000025E67C42FB1                 db 0ABh
debug027:0000025E67C42FB2                 db 0ABh
debug027:0000025E67C42FB3                 db 0ABh
```
再次对异或后的数据下硬件断点
```c
do
      v101[v90++] = *(_BYTE *)(v99 + v102++);
while ( v100 != v102 );
```
触发断点的是一个数据复制操作
继续对复制后的数据位置下断点

```c
    v112 = _byteswap_uint64(*(_QWORD *)&v101[v109]);
    *(_BYTE *)(v106 + v108 - 32) = byte_7FF7D72D25F8[(v112 >> 58) + 3];
    *(_BYTE *)(v106 + v108 - 31) = byte_7FF7D72D25F8[((v112 >> 52) & 0x3F) + 3];
    *(_BYTE *)(v106 + v108 - 30) = byte_7FF7D72D25F8[((v112 >> 46) & 0x3F) + 3];
```
这次触发的是一大堆取表操作，数据被转化到v106
我们对v106硬件断点，继续探索逻辑
```C
    v86 = v135[*((_QWORD *)&Src[0] + 1)] == v135[v106];
    ++v135;
    if ( !v86 )
    {
      v136 = sub_7FF7D72C426C("nonono", 6ui64, (__int64)v174);
```
我们这次直接定位到一个循环的判断代码片段里，这里应该是判断密文的地方了，左边的src也确实藏着一个开头得到的密文数据，同时我们点击v106，发现里面的数据全是可见字符，并且最后一个字符是'=’,结合代码特征可知是base64加密。
整体分析到此结束，我们来梳理一下:
`flag分组补'E'->每组到sub_7FF7D72B28C3加密->异或0x33->异或一个固定的表->base64`
现在逻辑再清晰不过了，我们只差sub_7FF7D72B28C3这个加密就可得出flag
这里我引用别人的解法: 调试找规律
```c
Stack[000011E8]:000000A8A00FE5A8 offset          db  66h ; f
Stack[000011E8]:000000A8A00FE5A9                 db  66h ; f
Stack[000011E8]:000000A8A00FE5AA                 db  67h ; g
Stack[000011E8]:000000A8A00FE5AB                 db  67h ; g
Stack[000011E8]:000000A8A00FE5AC                 db  68h ; h
Stack[000011E8]:000000A8A00FE5AD                 db  68h ; h
Stack[000011E8]:000000A8A00FE5AE                 db  69h ; i
Stack[000011E8]:000000A8A00FE5AF                 db  69h ; i
Stack[000011E8]:000000A8A00FE5B0 flag            db  67h ; g
Stack[000011E8]:000000A8A00FE5B1                 db  67h ; g
Stack[000011E8]:000000A8A00FE5B2                 db  67h ; g
Stack[000011E8]:000000A8A00FE5B3                 db  67h ; g
Stack[000011E8]:000000A8A00FE5B4                 db  67h ; g
Stack[000011E8]:000000A8A00FE5B5                 db  67h ; g
Stack[000011E8]:000000A8A00FE5B6                 db  67h ; g
Stack[000011E8]:000000A8A00FE5B7                 db  67h ; g
```
```python
delta = -1				    		       

input+= delta 		       

if i % 2 =1	          

 	delta -= 1
```
简单的一个加减运算，可是看起来却非常难懂，
可知我们解ctf题有时也需要一些取巧的技巧,
唉,感觉自己修为还不够，学长调几次就看出了这个伪加密，道阻且长。
`tips: 这个函数也可以利用硬件断点来分析，可以分析出flag只在最后的循环语句中被读取，并且只执行了简单的加减操作，以前没发现硬件断点有如此妙用，如今是领教到了`
以下是解密脚本
```c
#include <stdio.h>
void decodeBase64(char* str,int len,char** in){
	char base64[65] = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/";
    char ascill[129];
    int k = 0;
    for(int i=0;i<64;i++){
        ascill[base64[i]] = k++;
    }
	int decodeStrlen = len / 4 * 3 + 1;
	char* decodeStr = (char*)malloc(sizeof(char)*decodeStrlen);
	k = 0;
	for(int i=0;i<len;i++){
        decodeStr[k++] = (ascill[str[i]] << 2) | (ascill[str[++i]] >> 4);
		if(str[i+1] == '='){
			break;
		}
        decodeStr[k++] = (ascill[str[i]] << 4) |  (ascill[str[++i]] >> 2);
		if(str[i+1] == '='){
			break;
		}
        decodeStr[k++] = (ascill[str[i]] << 6) | (ascill[str[++i]]);
	}
	decodeStr[k] = '\0';
	*in = decodeStr;
}

int main(){

unsigned char xortable[]={0xdc,0x5f,0x20,0x22,0xc2,0x79,0x19,0x56,0x35,0xda,0x8b,0x47,0xd3,0x19,0xfc,0x55,0x14,0xcd,0xd2,0x7b,0x58,0x59,0x09,0x42,0xde,0x2c,0xb4,0x48,0xd9,0xf2,0x1b,0xa9,0x40,0xe1,0xa6,0xfb,0xff,0x38,0xc1,0xd5,0xe2,0xe8,0x77,0x78,0x6f,0x22,0x04,0xe6,0x16,0x3e,0x0c,0x35,0x27,0x29,0x89,0xb5,0x92,0x2e,0x6a,0xa6,0xdb,0x2f,0xc6,0xa9,0x6e,0x8f,0x34,0x90,0x59,0xfc,0x2d,0x91,0x66,0xeb,0xbe,0x0d,0xf4,0x05,0x0b,0x1b,0xcb,0x18,0x74,0xf9,0x82,0x81,0xbc,0x04,0xd9,0x75,0x8e,0x2d,0x97,0x07,0x7c,0x7d,0x18,0xc8,0x3d,0x4f,0xc0,0xa5,0x6a,0xd7};
unsigned char *mm2;
decodeBase64(mm1,strlen(mm1),&mm2);
puts(mm2);
for(int i=0;i<strlen(mm1) / 4 * 3 + 1;i++){
    mm2[i]^=(unsigned char)xortable[i];
	mm2[i]^=0x33;
}
puts(mm2);
for(int i=0;i<strlen(mm2);i+=8){
	mm2[i]+=1;
	mm2[i+1]+=1;
	mm2[i+4]-=1;
	mm2[i+5]-=1;
	mm2[i+6]-=2;
	mm2[i+7]-=2;	
}
puts(mm2);
}
```




