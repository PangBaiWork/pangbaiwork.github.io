---
title: 2024浙江省大学生网络与信息安全竞赛Reverse-WP
categories: 
- [ CTF ,逆向工程]
tags:
- 汇编
- Re
- frida
date: 2024-11-10 14:00:25
---
圆梦省一了。
依稀还记得一年前第一次参加线下赛的无力感。感谢队友不离不弃，这次终于无憾了。
<!--more-->
嘿嘿，这次和杭师的 gets 师傅，r3kapig 的 yuro 师傅，还有其他师傅们见面了，还是很开心。

这次有幸 AK 掉 RE，想着很久没写WP了那就小写一下吧。

## Reverse1
RC4加密
把beforemain,aftermain逻辑复制下来，最后把减号改成加就行了。
```c
unsigned char cipher[32] = {
    0x4E, 0x47, 0x38, 0x47, 0x62, 0x0A, 0x79, 0x6A, 0x03, 0x66, 0xC0, 0x69, 0x8D, 0x1C, 0x84, 0x0F, 
    0x54, 0x4A, 0x3B, 0x08, 0xE3, 0x30, 0x4F, 0xB9, 0x6C, 0xAB, 0x36, 0x24, 0x52, 0x81, 0xCF,0
};
char key1[]="keykey";
char key[100]="ban_debug!";
char s[256];
#include <stdlib.h>
#include <stdint.h>
#include <stdio.h>
#include <string.h>
typedef char __int8;
typedef int64_t __int64;
 void init(char *a1, char *a2,int a3)
{
  char v4; // [rsp+23h] [rbp-41Dh]
  int i; // [rsp+24h] [rbp-41Ch]
  int v6; // [rsp+28h] [rbp-418h]
  int j; // [rsp+2Ch] [rbp-414h]
  int v8[258]; // [rsp+30h] [rbp-410h] BYREF



  memset(v8, 0, 0x400uLL);
  for ( i = 0; i <= 255; ++i )
  {
    a1[i] = i;
    v8[i] = (unsigned char)a2[i % a3];
  }
  v6 = 0;
  for ( j = 0; j <= 255; ++j )
  {
    v6 = (v8[j] + v6 + (unsigned char)a1[j]) % 256;
    v4 = a1[j];
    a1[j] = a1[v6];
    a1[v6] = v4;
  }
  return ;
}


void  crypt1(char *a1, char *a2, int a3)
{
  int result; // rax
  char v4; // [rsp+27h] [rbp-11h]
  int v5; // [rsp+28h] [rbp-10h]
  int v6; // [rsp+2Ch] [rbp-Ch]
  int i; // [rsp+30h] [rbp-8h]

  v5 = 0;
  v6 = 0;
  for ( i = 0; ; ++i )
  {
    result = i;
    if ( a3 <= i )
      break;
    v5 = (v5 + 1) % 256;
    v6 = (v6 + (unsigned char)a1[v5]) % 256;
    v4 = a1[v5];
    a1[v5] = a1[v6];
    a1[v6] = v4;
    a2[i] ^= a1[(unsigned char)(a1[v5] + a1[v6])];
  }

}



void crypt2(char *a1, char *a2, int a3)
{
  __int64 result; // rax
  char v4; // [rsp+27h] [rbp-11h]
  int v5; // [rsp+28h] [rbp-10h]
  int v6; // [rsp+2Ch] [rbp-Ch]
  int i; // [rsp+30h] [rbp-8h]

  v5 = 0;
  v6 = 0;
  for ( i = 0; ; ++i )
  {
    result = i;
    if ( a3 <= i )
      break;
    v5 = (v5 + 1) % 256;
    v6 = (v6 + (unsigned char)a1[v5]) % 256;
    v4 = a1[v5];
    a1[v5] = a1[v6];
    a1[v6] = v4;
    a2[i] += a1[(unsigned char)(a1[v5] + a1[v6])];
  }

}


int main(){
init(s,key1,strlen(key1));
crypt1(s,key,strlen(key));
 memset(s, 0, sizeof(s));
init(s,key,strlen(key));
crypt2(s,cipher,32);
puts(cipher);

}
```


## Reverse2
魔改UPX，用010对照标志改回来就行（去年初赛决赛也是，出题人真喜欢UPX）
![BASE64](https://shp.qpic.cn/collector/1642981619/91f02db2-44a1-4cf3-9b3f-e475e9772ae0/0)
base64加密
换表
`/+9876543210zyxwvutsrqponmlkjihgfedcbaZYXWVUTSRQPONMLKJIHGFEDCBA`
密文
`u76svKu5hJvLzpvHnJvGx5nPz53Nz8uaxsfGxseanJnHy83ImoL=`
直接解密就行了

## Reverse3
程序有花指令和反调试。去掉花指令后发现并没有什么逻辑。
在 `TlsCallback_0` 里发现字符串 `FileName` 异或0x11。
![TlsCallback_0](https://shp.qpic.cn/collector/1642981619/ae18a2f8-a001-409c-a6f5-3e99915a15ad/0)
异或回来找到了 `1.exe` 这个程序，加密应该主要位于这里，原本的 exe 只是一个加载器。
观察 `1.exe` 逻辑，是一个 `AES加密` ，但是有换盒。
把密钥和密文取出，反出逆s盒，进行 `AES解密` ，发现无法解密。
比对内存中的数据和脚本加密数据发现是一样的， `AES解密` 不可能存在问题。
猜测是加载器修改了 key 或者其他数据。 `frida` 的注入不会被反调试检测，用 `frida` 去hook得到key的数据，发现 key 被改变。换用改变的key仍然不可解密。
接下来 hook 了 `sbox` ，`sbox` 也发生了改变，替换改变后的 `sbox` 还是不可解密。
再 hook 密文，发现密文没有改变，那改变的可能是输入的 flag 数据。hook 后确实是输入的 flag 被加密了。
多次 hook 发现输入的 flag 每个字节都会有一定偏移。我们把可打印字符串表作为 flag 输入，获得字符变化的对应关系。

```0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ!\"#$%&'()*+,-./:;<=>?@[\\]^_``` 输入后
![fridaHOok](https://shp.qpic.cn/collector/1642981619/439a671a-4f28-4ed3-b8e3-9453ff001ca8/0)
按照对应关系输出 `AES解密` 的数据即可得到 flag。

frida hook脚本
```js

var inter=setInterval(function () {
    var sgame = Process.findModuleByName("1.exe");
    if(sgame==null){
        console.log("无");
        return;
    }

    
    clearInterval(inter);
    console.log(""+sgame.base.add(0x001EC9  ));
    Interceptor.attach(sgame.base.add(0x001EC9  ), { 
    onEnter: function (args) {
        var rax=this.context.rax; 
        console.log("" +rax);
		// 检查密文有没有被修改
           console.log(hexdump(ptr(rax),{length: 50,ansi:true}));
      
         
    }
})

Interceptor.attach(sgame.base.add(0x001E2C  ), {
    onEnter: function (args) {
       
        console.log("input flag");
  // 检查输入flag的时候有没有被修改
           console.log(hexdump(ptr(this.context.rdx),{length: 16,ansi:true}));
         
    }
})

Interceptor.attach(sgame.base.add(0x1380), { 
    onEnter: function (args) {
        var rdx=this.context.rdx; 
     // 检查key ,sbox ,进入函数的flag有没有被修改
           console.log("key" );
           console.log(hexdump(ptr(rdx),{length: 16,ansi:true}));
           console.log("flag:" );
           console.log(hexdump(ptr(this.context.rcx),{length: 16,ansi:true}));
           console.log("sbox:" );
           console.log(hexdump(ptr(sgame.base.add(0x5160)),{length: 256,ansi:true}));
    }
})
} , 1) 
```
逆盒脚本
```python
new_s_box = [
    0x63,0x7c,0x77,0x7b,0xf2,0x6b,0x6f,0xc5,0x30,0x01,0x67,0x2b,0xfe,0xd7,0xab,0x76,0xca,0x82,0xc9,0x7d,0xfa,0x59,0x47,0xf0,0xad,0xd4,0xa2,0xaf,0x9c,0xa4,0x72,0xc0,0xb7,0xfd,0x93,0x26,0x36,0x3f,0xf7,0xcc,0x34,0xa5,0xe5,0xf1,0x71,0xd8,0x31,0x15,0x04,0xc7,0x23,0xc3,0x18,0x96,0x05,0x9a,0x07,0x12,0x80,0xe2,0xeb,0x27,0xb2,0x75,0x09,0x83,0x2c,0x1a,0x1b,0x6e,0x5a,0xa0,0x52,0x3b,0xd6,0xb3,0x29,0xe3,0x2f,0x84,0x53,0xd1,0x00,0xed,0x20,0xfc,0xb1,0x5b,0x6a,0xcb,0xbe,0x39,0x4a,0x4c,0x58,0xcf,0xd0,0xef,0xaa,0xfb,0x43,0x4d,0x33,0x85,0x45,0xf9,0x02,0x7f,0x50,0x3c,0x9f,0xa8,0x51,0xa3,0x40,0x8f,0x92,0x9d,0x38,0xf5,0xbc,0xb6,0xda,0x21,0x10,0xff,0xf3,0xd2,0xcd,0x0c,0x13,0xec,0x5f,0x97,0x44,0x17,0xc4,0xa7,0x7e,0x3d,0x64,0x5d,0x19,0x73,0x60,0x81,0x4f,0xdc,0x22,0x2a,0x90,0x88,0x46,0xee,0xb8,0x14,0xde,0x5e,0x0b,0xdb,0xe0,0x32,0x3a,0x0a,0x49,0x06,0x24,0x5c,0xc2,0xd3,0xac,0x62,0x91,0x95,0xe4,0x79,0xe7,0xc8,0x37,0x6d,0x8d,0xd5,0x4e,0xa9,0x6c,0x56,0xf4,0xea,0x65,0x7a,0xae,0x08,0xba,0x78,0x25,0x2e,0x1c,0xa6,0xb4,0xc6,0xe8,0xdd,0x74,0x1f,0x4b,0xbd,0x8b,0x8a,0x70,0x3e,0xb5,0x66,0x48,0x03,0xf6,0x0e,0x61,0x35,0x57,0xb9,0x86,0xc1,0x1d,0x9e,0xe1,0xf8,0x98,0x11,0x69,0xd9,0x8e,0x94,0x9b,0x1e,0x87,0xe9,0xce,0x55,0x28,0xdf,0x8c,0xa1,0x89,0x0d,0xbf,0xe6,0x42,0x68,0x41,0x99,0x2d,0x0f,0xb0,0x54,0xbb,0x16
]
import string
new_contrary_sbox = [0]*256
print(len(new_s_box))
for i in range(256):
    line = (new_s_box[i]&0xf0)>>4
    rol = new_s_box[i]&0xf
    new_contrary_sbox[(line*16)+rol] = i
print(len(new_contrary_sbox))
print(new_contrary_sbox)
print(string.printable)
```

aes解密脚本，每次解密一段
```c 

#define _CRT_SECURE_NO_WARNINGS
#include<stdio.h>
void AddRoundKey(unsigned char *plaintext, unsigned char * CipherKey)/*轮密钥加*/
{
	for (int j = 0; j < 16; j++)  plaintext[j] = plaintext[j] ^ CipherKey[j];
}
void SubBytes(unsigned char *plaintext, unsigned char *plaintextencrypt, int count)/*S盒置换*/
{
	unsigned int row, column;
	unsigned char Sbox[16][16] = {
		      /* 0     1     2     3     4     5     6     7     8     9     a     b     c     d     e     f */
		 0x63,0x7c,0x77,0x7b,0xf2,0x6b,0x6f,0xc5,0x30,0x01,0x67,0x2b,0xfe,0xd7,0xab,0x76,0xca,0x82,0xc9,0x7d,0xfa,0x59,0x47,0xf0,0xad,0xd4,0xa2,0xaf,0x9c,0xa4,0x72,0xc0,0xb7,0xfd,0x93,0x26,0x36,0x3f,0xf7,0xcc,0x34,0xa5,0xe5,0xf1,0x71,0xd8,0x31,0x15,0x04,0xc7,0x23,0xc3,0x18,0x96,0x05,0x9a,0x07,0x12,0x80,0xe2,0xeb,0x27,0xb2,0x75,0x09,0x83,0x2c,0x1a,0x1b,0x6e,0x5a,0xa0,0x52,0x3b,0xd6,0xb3,0x29,0xe3,0x2f,0x84,0x53,0xd1,0x00,0xed,0x20,0xfc,0xb1,0x5b,0x6a,0xcb,0xbe,0x39,0x4a,0x4c,0x58,0xcf,0xd0,0xef,0xaa,0xfb,0x43,0x4d,0x33,0x85,0x45,0xf9,0x02,0x7f,0x50,0x3c,0x9f,0xa8,0x51,0xa3,0x40,0x8f,0x92,0x9d,0x38,0xf5,0xbc,0xb6,0xda,0x21,0x10,0xff,0xf3,0xd2,0xcd,0x0c,0x13,0xec,0x5f,0x97,0x44,0x17,0xc4,0xa7,0x7e,0x3d,0x64,0x5d,0x19,0x73,0x60,0x81,0x4f,0xdc,0x22,0x2a,0x90,0x88,0x46,0xee,0xb8,0x14,0xde,0x5e,0x0b,0xdb,0xe0,0x32,0x3a,0x0a,0x49,0x06,0x24,0x5c,0xc2,0xd3,0xac,0x62,0x91,0x95,0xe4,0x79,0xe7,0xc8,0x37,0x6d,0x8d,0xd5,0x4e,0xa9,0x6c,0x56,0xf4,0xea,0x65,0x7a,0xae,0x08,0xba,0x78,0x25,0x2e,0x1c,0xa6,0xb4,0xc6,0xe8,0xdd,0x74,0x1f,0x4b,0xbd,0x8b,0x8a,0x70,0x3e,0xb5,0x66,0x48,0x03,0xf6,0x0e,0x61,0x35,0x57,0xb9,0x86,0xc1,0x1d,0x9e,0xe1,0xf8,0x98,0x11,0x69,0xd9,0x8e,0x94,0x9b,0x1e,0x87,0xe9,0xce,0x55,0x28,0xdf,0x8c,0xa1,0x89,0x0d,0xbf,0xe6,0x42,0x68,0x41,0x99,0x2d,0x0f,0xb0,0x54,0xbb,0x16
        
        };// 填充Sbox矩阵
	for (int i = 0; i < count; i++)
	{
		row = (plaintext[i] & 0xF0) >> 4;
		column = plaintext[i] & 0x0F;
		plaintextencrypt[i] = Sbox[row][column];
	}
}
void SubBytesRe(unsigned char *plaintext, unsigned char *plaintextencrypt, int count)/*S盒逆置换*/
{
	unsigned int row, column;
	unsigned char Sbox[16][16] = {
		82, 9, 106, 213, 48, 54, 165, 56, 191, 64, 163, 158, 129, 243, 215, 251, 124, 227, 57, 130, 155, 47, 255, 135, 52, 142, 67, 68, 196, 222, 233, 203, 84, 123, 148, 50, 166, 194, 35, 61, 238, 76, 149, 11, 66, 250, 195, 78, 8, 46, 161, 102, 40, 217, 36, 178, 118, 91, 162, 73, 109, 139, 209, 37, 114, 248, 246, 100, 134, 104, 152, 22, 212, 164, 92, 204, 93, 101, 182, 146, 108, 112, 72, 80, 253, 237, 185, 218, 94, 21, 70, 87, 167, 141, 157, 132, 144, 216, 171, 0, 140, 188, 211, 10, 247, 228, 88, 5, 184, 179, 69, 6, 208, 44, 30, 143, 202, 63, 15, 2, 193, 175, 189, 3, 1, 19, 138, 107, 58, 145, 17, 65, 79, 103, 220, 234, 151, 242, 207, 206, 240, 180, 230, 115, 150, 172, 116, 34, 231, 173, 53, 133, 226, 249, 55, 232, 28, 117, 223, 110, 71, 241, 26, 113, 29, 41, 197, 137, 111, 183, 98, 14, 170, 24, 190, 27, 252, 86, 62, 75, 198, 210, 121, 32, 154, 219, 192, 254, 120, 205, 90, 244, 31, 221, 168, 51, 136, 7, 199, 49, 177, 18, 16, 89, 39, 128, 236, 95, 96, 81, 127, 169, 25, 181, 74, 13, 45, 229, 122, 159, 147, 201, 156, 239, 160, 224, 59, 77, 174, 42, 245, 176, 200, 235, 187, 60, 131, 83, 153, 97, 23, 43, 4, 126, 186, 119, 214, 38, 225, 105, 20, 99, 85, 33, 12, 125        };	// 填充Sbox矩阵
	for (int i = 0; i < count; i++)
	{
		row = (plaintext[i] & 0xF0) >> 4;
		column = plaintext[i] & 0x0F;
		plaintextencrypt[i] = Sbox[row][column];
	}
}
void ShiftRowsRe(unsigned char *plaintextencrypt)/*行移位的逆*/
{
	unsigned char temp = 0;
	for (int i = 0; i < 4; i++)//第i行
	{
		for (int j = 0; j < 4 - i; j++)//第j次左移
		{
			temp = plaintextencrypt[i];
			for (int k = 0; k < 4; k++)
				plaintextencrypt[i + 4 * k] = plaintextencrypt[i + 4 * (k + 1)];
			plaintextencrypt[i + 12] = temp;
		}
	}
}
void ShiftRows(unsigned char *plaintextencrypt)/*行移位*/
{
	unsigned char temp = 0;
	for (int i = 0; i < 4; i++)//第i行
	{
		for (int j = 0; j < i; j++)//第j次左移
		{
			temp = plaintextencrypt[i];
			for (int k = 0; k < 4; k++)
				plaintextencrypt[i + 4 * k] = plaintextencrypt[i + 4 * (k + 1)];
			plaintextencrypt[i + 12] = temp;
		}
	}
}
unsigned char Mult2(unsigned char num)/*列混淆*/
{
	unsigned char temp = num << 1;
	if ((num >> 7) & 0x01)
		temp = temp ^ 27;
	return temp;
}
unsigned char Mult3(unsigned char num)
{
	return Mult2(num) ^ num;
}
void MixColumns(unsigned char *plaintextencrypt, unsigned char *plaintextcrypt)
{
	int i;
	for (i = 0; i < 4; i++)
		plaintextcrypt[4 * i] = Mult2(plaintextencrypt[4 * i]) ^ Mult3(plaintextencrypt[4 * i + 1]) ^ plaintextencrypt[4 * i + 2] ^ plaintextencrypt[4 * i + 3];
	for (i = 0; i < 4; i++)
		plaintextcrypt[4 * i + 1] = plaintextencrypt[4 * i] ^ Mult2(plaintextencrypt[4 * i + 1]) ^ Mult3(plaintextencrypt[4 * i + 2]) ^ plaintextencrypt[4 * i + 3];
	for (i = 0; i < 4; i++)
		plaintextcrypt[4 * i + 2] = plaintextencrypt[4 * i] ^ plaintextencrypt[4 * i + 1] ^ Mult2(plaintextencrypt[4 * i + 2]) ^ Mult3(plaintextencrypt[4 * i + 3]);
	for (i = 0; i < 4; i++)
		plaintextcrypt[4 * i + 3] = Mult3(plaintextencrypt[4 * i]) ^ plaintextencrypt[4 * i + 1] ^ plaintextencrypt[4 * i + 2] ^ Mult2(plaintextencrypt[4 * i + 3]);
}
/*逆列混淆*/ 
#define xtime(x)   ((x<<1) ^ (((x>>7) & 1) * 0x1b))
#define Multiply(x,y) (((y & 1) * x) ^ ((y>>1 & 1) * xtime(x)) ^ ((y>>2 & 1) * xtime(xtime(x))) ^ ((y>>3 & 1) * xtime(xtime(xtime(x)))) ^ ((y>>4 & 1) * xtime(xtime(xtime(xtime(x))))))
void MixColumnsRe(unsigned char *state)
{
	
	unsigned char a, b, c, d;
	for (int i = 0; i < 4; i++)
	{
		a = state[4*i];
		b = state[4*i+1];
		c = state[4*i+2];
		d = state[4*i+3];
		state[4 * i] = Multiply(a, 0x0e) ^ Multiply(b, 0x0b) ^ Multiply(c, 0x0d) ^ Multiply(d, 0x09);
		state[4 * i + 1] = Multiply(a, 0x09) ^ Multiply(b, 0x0e) ^ Multiply(c, 0x0b) ^ Multiply(d, 0x0d);
		state[4 * i + 2] = Multiply(a, 0x0d) ^ Multiply(b, 0x09) ^ Multiply(c, 0x0e) ^ Multiply(d, 0x0b);
		state[4 * i + 3] = Multiply(a, 0x0b) ^ Multiply(b, 0x0d) ^ Multiply(c, 0x09) ^ Multiply(d, 0x0e);
	}
}
int CharToWord(unsigned char *character, int first)/*字节转字*/
{
	return (((int)character[first] & 0x000000ff) << 24) | (((int)character[first + 1] & 0x000000ff) << 16) | (((int)character[first + 2] & 0x000000ff) << 8) | ((int)character[first + 3] & 0x000000ff);
}
void WordToChar(unsigned int word, unsigned char *character)/*字转字节*/
{
	for (int i = 0; i < 4; character[i++] = (word >> (8 * (3 - i))) & 0xFF);
}
void ExtendCipherKey(unsigned int *CipherKey_word, int round)/*密钥扩展*/
{
	unsigned char CipherKeyChar[4] = { 0 },CipherKeyCharEncrypt[4] = { 0 };
	unsigned int Rcon[10] = { 0x01000000,0x02000000,0x04000000,0x08000000,0x10000000,0x20000000,0x40000000,0x80000000,0x1B000000,0x36000000 };	//轮常量
	for (int i = 4; i < 8; i++)
	{
		if (!(i % 4))
		{
			WordToChar((CipherKey_word[i - 1] >> 24) | (CipherKey_word[i - 1] << 8), CipherKeyChar);
			SubBytes(CipherKeyChar, CipherKeyCharEncrypt, 4);
			CipherKey_word[i] = CipherKey_word[i - 4] ^ CharToWord(CipherKeyCharEncrypt, 0) ^ Rcon[round];
		}
		else
			CipherKey_word[i] = CipherKey_word[i - 4] ^ CipherKey_word[i - 1];
	}
}
#include <stdlib.h>
  void genString(int size){
       char *text = (char *)malloc(size * sizeof(char));
       for (size_t i = 0; i < size; i++)
       {
       text[i]=35+i;
       }
       puts(text);
       
}

char table[]="0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ!\"#$%&'()*+,-./:;<=>?@[\\]^_`{|}~ ";
char res[]={0x36,0x35,0x34,0x33,0x3a,0x39,0x38,0x37,0x3e,0x3d,0x85,0x84,0x83,0x8a,0x89,0x88,0x87,0x8e,0x8d,0x8c,0x8b,0x92,0x91,0x90,0x8f,0x76,0x75,0x74,0x73,0x7a,0x79,0x78,0x77,0x7e,0x7d,0x7c,0xa5,0xa4,0xa3,0xaa,0xa9,0xa8,0xa7,0xae,0xad,0xac,0xab,0xb2,
0xb1,0xb0,0xaf,0x96,0x95,0x94,0x93,0x9a,0x99,0x98,0x97,0x9e,0x9d,0x9c,0x45,0x44,0x43,0x4a,0x49,0x48,0x47,0x4e,0x4d,0x4c,0x4b,0x52,0x51,0x50,0x4f,0x3c,0x3b,0x42,0x41,0x40,0x3f,0xa6,0x9b,0xa2,0xa1,0xa0,0x9f,0x86,0x7b,0x82,0x81,0x80,0x66,0x66
};
char getcc(char in){
for (size_t i = 0; i < 96; i++)
{
   if (in==res[i])
   {
    return table[i];
   }
   
}

}
void main()
{
	printf("**************AES加解密***************\n");
	int i = 0, k;

unsigned mm[]={0x71,0x55,0x7f,0xa8,0xfa,0x0e,0xa3,0x19,0xa0,0x5c,0xf9,0x0e,0x9b,0x0b,0x5e,0xfc,0xb5,0xa8,0x49,0xfd,0x90,0x99,0x74,0xc7,0x77,0x02,0x6a,0xf5,0x9a,0x6a,0xba,0x7f,0xfb,0xe7,0x68,0xda,0x54,0xee,0xe8,0xbb,0x78,0x01,0xe7,0xbb,0xa2,0x95,0x95,0xfa};

   unsigned char PlainText[48] ="aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa"
    ,
		CipherKey[16] = { 0x05,0x06,0x07,0x08,0x37,0x42,0x4d,0x58,0x63,0x00,0x0a,0x0c,0x0d,0x0e,0x0f,0x10 },
		CipherKey1[16] = {  0x05,0x06,0x07,0x08,0x37,0x42,0x4d,0x58,0x63,0x00,0x0a,0x0c,0x0d,0x0e,0x0f,0x10},
		PlainText1[48] = { 
              0x71, 0x55, 0x7F, 0xA8, 0xFA, 0x0E, 0xA3, 0x19, 0xA0, 0x5C, 0xF9, 0x0E, 0x9B, 0x0B, 0x5E, 0xFC, 
    0xB5, 0xA8, 0x49, 0xFD, 0x90, 0x99, 0x74, 0xC7, 0x77, 0x02, 0x6A, 0xF5, 0x9A, 0x6A, 0xBA, 0x7F, 
    0xFB, 0xE7, 0x68, 0xDA, 0x54, 0xEE, 0xE8, 0xBB, 0x78, 0x01, 0xE7, 0xBB, 0xA2, 0x95, 0x95, 0xFA
 
         },
		PlainText2[48] = {   0x71, 0x55, 0x7F, 0xA8, 0xFA, 0x0E, 0xA3, 0x19, 0xA0, 0x5C, 0xF9, 0x0E, 0x9B, 0x0B, 0x5E, 0xFC, 
    0xB5, 0xA8, 0x49, 0xFD, 0x90, 0x99, 0x74, 0xC7, 0x77, 0x02, 0x6A, 0xF5, 0x9A, 0x6A, 0xBA, 0x7F, 
    0xFB, 0xE7, 0x68, 0xDA, 0x54, 0xEE, 0xE8, 0xBB, 0x78, 0x01, 0xE7, 0xBB, 0xA2, 0x95, 0x95, 0xFA
  };
	unsigned int CipherKey_word[44] = { 0 };
	for (i = 0; i < 4; CipherKey_word[i++] = CharToWord(CipherKey, 4 * i));
	printf("密钥：");
	for (k = 0; k < 16; k++) printf("%2X ", CipherKey[k]);
	printf("\n明文：");
	for (k = 0; k < 16; k++) printf("%02X ", PlainText[k]);
	printf("\n**************开始加密****************");
	AddRoundKey(PlainText, CipherKey);
	for (i = 0; i < 9; i++)
	{
		SubBytes(PlainText, PlainText1, 16);/*S盒置换*/
		ShiftRows(PlainText1);	/*行移位*/
		MixColumns(PlainText1, PlainText2);	/*列混淆*/
		ExtendCipherKey(CipherKey_word + 4 * i, i);/*子密钥生成*/
		for (k = 0; k < 4; k++)  WordToChar(CipherKey_word[k + 4 * (i + 1)], CipherKey + 4 * k);
		for (k = 0; k < 16; k++)  printf("%02X ", CipherKey[k]);
		AddRoundKey(PlainText2, CipherKey);/*轮密钥加*/
		for (k = 0; k < 16; k++)  PlainText[k] = PlainText2[k];

	}
	
	SubBytes(PlainText, PlainText1, 16);
	ShiftRows(PlainText1);
	ExtendCipherKey(CipherKey_word + 4 * i, i);
	for (k = 0; k < 4;WordToChar(CipherKey_word[k + 4 * (i + 1)], CipherKey + 4 * k), k++);
	AddRoundKey(PlainText1, CipherKey);
	printf("\n\n最终AES加密后的密文为：");
	for (i = 0; i < 16; i++)  printf("%02X ", PlainText1[i]);

 
    for (size_t i = 0; i < 16; i++)
    {
    PlainText1[i]=mm[i];   // 解密第二段第三段偏移16，32即可
    }
    
 
	printf("\n\n**************开始解密***************");
	AddRoundKey(PlainText1, CipherKey);
	for (i = 0; i < 9; i++)
	{

	    SubBytesRe(PlainText1, PlainText, 16);/*S盒置换*/
		for (k = 0; k < 4; WordToChar(CipherKey_word[k + 40 - 4 * (i + 1)], CipherKey + 4 * k),k++);/*子密钥生成*/
		ShiftRowsRe(PlainText);/*行移位逆*/
		AddRoundKey(PlainText, CipherKey);/*轮密钥加*/
		MixColumnsRe(PlainText);/*列混淆逆运算*/
		for (k = 0; k < 16;PlainText1[k] = PlainText[k],k++);
	}
	printf("\n最后一次循环：");
	ShiftRowsRe(PlainText);/*行移位逆*/
	SubBytesRe(PlainText, PlainText1, 16);/*S盒置换*/
	AddRoundKey(PlainText1, CipherKey1);
	printf("\n最终AES解密后的明文为：");
   
	for (i = 0; i < 16; i++)  printf("%c",getcc( PlainText1[i]));
	printf("\n");
   
}
```
