---
title: 加密解密算法
categories: 
- [ CTF ,逆向工程]
tags:
- 汇编
- Re
published: true
---
常见的加密解密算法实现汇总
<!--more-->
# HEX
```cpp hex加密解密
#include "stdio.h" // sprintf()函数声明所在的头文件
char data[] = "Hello World!!\r\n";
char res[32];

int str_2_hex_str(char *dest, char *src)
{
    int len = strlen(src); // 获取接收数据长度
    int i,j;
    for (i = 0, j = 0; i < len; i++) {
        sprintf(&dest[j], "%02X", src[i]);
        j+=2; // 每个16进制占2个长度
    }
    dest[j] = '\0'; // 添加字符串结束符
    return j; // 返回字符串长度
}

int len = str_2_hex_str(res, data);
///////////////////////////////
#include <stdlib.h> // 要使用strtol()库函数，需要包含头文件

char data[] = "48656C6C6F20576F726C6421210D0A"; // 假如，我们接收到这样的数据
char res[32]; // 储存转换后的结果

int hex_str_2_str(char *dest, char *src)
{
    int len = strlen(src); // 获取接收数据长度
    int i,j;
    for (i = 0, j = 0; i < len; i+=2) { // 每次取两个字符
        char tmp_buf[3]; // 每两个字符组成一个16进制字符串，同时结尾需要空字符来告诉编译器我们的是字符串
        char *endptr; // 保存已转换数值后的下一个字符
        // 以下为取待转换的16进制字符串
        tmp_buf[0] = src[i];
        tmp_buf[1] = src[i + 1];
        tmp_buf[2] = '\0'; // 记得添加空字符
        // 转换成16进制，base传16即可
        dest[j++] = strtol(tmp_buf, &endptr, 16);
    }
    dest[j] = '\0'; // 添加字符串结束符
    return j;
}

int len = hex_str_2_str(res, data);

```
# BASE64
```cpp base64加密解密算法
//base64加密
char base64[65] = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/";
void encodeBase64(char* str,int len,char** in){
    
    //读取3个字节zxc，转换为二进制01111010 01111000 01100011
    //转换为4个6位字节，011110 100111 100001 100011
    //不足8位在前补0，变成00011110 00100111 00100001 00100011
    //若剩余的字节数不足以构成4个字节，补等号
    int encodeStrLen = 1 + (len/3)*4 ,k=0;
    encodeStrLen += len%3 ? 4 : 0;
    char* encodeStr = (char*)(malloc(sizeof(char)*encodeStrLen));
    for(int i=0;i<len;i++){
        if(len - i >= 3){
            encodeStr[k++] = base64[(unsigned char)str[i]>>2];
            encodeStr[k++] = base64[((unsigned char)str[i]&0x03)<<4 | (unsigned char)str[++i]>>4];
            encodeStr[k++] = base64[((unsigned char)str[i]&0x0f)<<2 | (unsigned char)str[++i]>>6];
            encodeStr[k++] = base64[(unsigned char)str[i]&0x3f];
        }else if(len-i == 2){
            encodeStr[k++] = base64[(unsigned char)str[i] >> 2];
            encodeStr[k++] = base64[((unsigned char)str[i]&0x03) << 4 | ((unsigned char)str[++i] >> 4)];
            encodeStr[k++] = base64[((unsigned char)str[i]&0x0f) << 2];
            encodeStr[k++] = '=';
        }else{
            encodeStr[k++] = base64[(unsigned char)str[i] >> 2];
            encodeStr[k++] = base64[((unsigned char)str[i] & 0x03) << 4];                                                                                                              //末尾补两个等于号
            encodeStr[k++] = '=';
            encodeStr[k++] = '=';
        }
    }
    encodeStr[k] = '\0';
    *in = encodeStr;
}

/**
* 解码既编码的逆过程，先找出编码后的字符在编码之前代表的数字
* 编码中将3位个字符变成4个字符，得到这4个字符的每个字符代表的原本数字
* 因为在编码中间每个字符用base64码表进行了替换，所以这里要先换回来
* 在对换回来的数字进行位运算使其还原成3个字符
*/
void decodeBase64(char* str,int len,char** in){
    
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
char mm[]="UAsFvs3tDyTxFPGb7WbyBYSm05VWrJxgjArj9mx490pfH1LO";
char *mm1;
char *mm2;
encodeBase64(mm,strlen(mm),&mm1);
decodeBase64(mm1,strlen(mm1),&mm2);
puts(mm2);
}
```
# XXTEA
```cpp 加密解密算法
#include <stdint.h>
#define DELTA 0x9e3779b9
#define MX (((z>>5^y<<2) + (y>>3^z<<4)) ^ ((sum^y) + (key[(p&3)^e] ^ z)))
//容易魔改
void btea(uint32_t * v, int n, uint32_t const key[4]) {
//v为数据，n为数据长度(负时为解密)，key为密钥
    uint32_t y,  z, sum;  unsigned p, rounds, e;
    if (n > 1)
    /* Coding Part */
    {
        rounds = 6 + 52 / n;
        sum = 0;
        z = v[n - 1];
        do {
            sum += DELTA;
            e = (sum >> 2) & 3;
            for (p = 0; p < n - 1; p++) {
                y = v[p + 1];
                z = v[p] += MX;
            }
            y = v[0];
            z = v[n - 1] += MX;
        } while (-- rounds );
    } else if (n < -1)
    /* Decoding Part */
    {
        n = -n;
        rounds = 6 + 52 / n;
        sum = rounds * DELTA;
        y = v[0];
        do {
            e = (sum >> 2) & 3;
            for (p = n - 1; p > 0; p--) {
                z = v[p - 1];
                y = v[p] -= MX;
            }
            z = v[n - 1];
            y = v[0] -= MX;
            sum -= DELTA;
        } while (-- rounds );
    }
}
```

# AES
```cpp 加密解密
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
		/*0*/{ 0x63, 0x7c, 0x77, 0x7b, 0xf2, 0x6b, 0x6f, 0xc5, 0x30, 0x01, 0x67, 0x2b, 0xfe, 0xd7, 0xab, 0x76 },
		/*1*/{ 0xca, 0x82, 0xc9, 0x7d, 0xfa, 0x59, 0x47, 0xf0, 0xad, 0xd4, 0xa2, 0xaf, 0x9c, 0xa4, 0x72, 0xc0 },
		/*2*/{ 0xb7, 0xfd, 0x93, 0x26, 0x36, 0x3f, 0xf7, 0xcc, 0x34, 0xa5, 0xe5, 0xf1, 0x71, 0xd8, 0x31, 0x15 },
		/*3*/{ 0x04, 0xc7, 0x23, 0xc3, 0x18, 0x96, 0x05, 0x9a, 0x07, 0x12, 0x80, 0xe2, 0xeb, 0x27, 0xb2, 0x75 },
		/*4*/{ 0x09, 0x83, 0x2c, 0x1a, 0x1b, 0x6e, 0x5a, 0xa0, 0x52, 0x3b, 0xd6, 0xb3, 0x29, 0xe3, 0x2f, 0x84 },
		/*5*/{ 0x53, 0xd1, 0x00, 0xed, 0x20, 0xfc, 0xb1, 0x5b, 0x6a, 0xcb, 0xbe, 0x39, 0x4a, 0x4c, 0x58, 0xcf },
		/*6*/{ 0xd0, 0xef, 0xaa, 0xfb, 0x43, 0x4d, 0x33, 0x85, 0x45, 0xf9, 0x02, 0x7f, 0x50, 0x3c, 0x9f, 0xa8 },
		/*7*/{ 0x51, 0xa3, 0x40, 0x8f, 0x92, 0x9d, 0x38, 0xf5, 0xbc, 0xb6, 0xda, 0x21, 0x10, 0xff, 0xf3, 0xd2 },
		/*8*/{ 0xcd, 0x0c, 0x13, 0xec, 0x5f, 0x97, 0x44, 0x17, 0xc4, 0xa7, 0x7e, 0x3d, 0x64, 0x5d, 0x19, 0x73 },
		/*9*/{ 0x60, 0x81, 0x4f, 0xdc, 0x22, 0x2a, 0x90, 0x88, 0x46, 0xee, 0xb8, 0x14, 0xde, 0x5e, 0x0b, 0xdb },
		/*a*/{ 0xe0, 0x32, 0x3a, 0x0a, 0x49, 0x06, 0x24, 0x5c, 0xc2, 0xd3, 0xac, 0x62, 0x91, 0x95, 0xe4, 0x79 },
		/*b*/{ 0xe7, 0xc8, 0x37, 0x6d, 0x8d, 0xd5, 0x4e, 0xa9, 0x6c, 0x56, 0xf4, 0xea, 0x65, 0x7a, 0xae, 0x08 },
		/*c*/{ 0xba, 0x78, 0x25, 0x2e, 0x1c, 0xa6, 0xb4, 0xc6, 0xe8, 0xdd, 0x74, 0x1f, 0x4b, 0xbd, 0x8b, 0x8a },
		/*d*/{ 0x70, 0x3e, 0xb5, 0x66, 0x48, 0x03, 0xf6, 0x0e, 0x61, 0x35, 0x57, 0xb9, 0x86, 0xc1, 0x1d, 0x9e },
		/*e*/{ 0xe1, 0xf8, 0x98, 0x11, 0x69, 0xd9, 0x8e, 0x94, 0x9b, 0x1e, 0x87, 0xe9, 0xce, 0x55, 0x28, 0xdf },
		/*f*/{ 0x8c, 0xa1, 0x89, 0x0d, 0xbf, 0xe6, 0x42, 0x68, 0x41, 0x99, 0x2d, 0x0f, 0xb0, 0x54, 0xbb, 0x16 }
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
		/* 0     1     2     3     4     5     6     7     8     9     a     b     c     d     e     f */
		{0x52, 0x09, 0x6a, 0xd5, 0x30, 0x36, 0xa5, 0x38, 0xbf, 0x40, 0xa3, 0x9e, 0x81, 0xf3, 0xd7, 0xfb},
		{0x7c, 0xe3, 0x39, 0x82, 0x9b, 0x2f, 0xff, 0x87, 0x34, 0x8e, 0x43, 0x44, 0xc4, 0xde, 0xe9, 0xcb},
		{0x54, 0x7b, 0x94, 0x32, 0xa6, 0xc2, 0x23, 0x3d, 0xee, 0x4c, 0x95, 0x0b, 0x42, 0xfa, 0xc3, 0x4e},
		{0x08, 0x2e, 0xa1, 0x66, 0x28, 0xd9, 0x24, 0xb2, 0x76, 0x5b, 0xa2, 0x49, 0x6d, 0x8b, 0xd1, 0x25},
		{0x72, 0xf8, 0xf6, 0x64, 0x86, 0x68, 0x98, 0x16, 0xd4, 0xa4, 0x5c, 0xcc, 0x5d, 0x65, 0xb6, 0x92},
		{0x6c, 0x70, 0x48, 0x50, 0xfd, 0xed, 0xb9, 0xda, 0x5e, 0x15, 0x46, 0x57, 0xa7, 0x8d, 0x9d, 0x84},
		{0x90, 0xd8, 0xab, 0x00, 0x8c, 0xbc, 0xd3, 0x0a, 0xf7, 0xe4, 0x58, 0x05, 0xb8, 0xb3, 0x45, 0x06},
		{0xd0, 0x2c, 0x1e, 0x8f, 0xca, 0x3f, 0x0f, 0x02, 0xc1, 0xaf, 0xbd, 0x03, 0x01, 0x13, 0x8a, 0x6b},
		{0x3a, 0x91, 0x11, 0x41, 0x4f, 0x67, 0xdc, 0xea, 0x97, 0xf2, 0xcf, 0xce, 0xf0, 0xb4, 0xe6, 0x73},
		{0x96, 0xac, 0x74, 0x22, 0xe7, 0xad, 0x35, 0x85, 0xe2, 0xf9, 0x37, 0xe8, 0x1c, 0x75, 0xdf, 0x6e},
		{0x47, 0xf1, 0x1a, 0x71, 0x1d, 0x29, 0xc5, 0x89, 0x6f, 0xb7, 0x62, 0x0e, 0xaa, 0x18, 0xbe, 0x1b},
		{0xfc, 0x56, 0x3e, 0x4b, 0xc6, 0xd2, 0x79, 0x20, 0x9a, 0xdb, 0xc0, 0xfe, 0x78, 0xcd, 0x5a, 0xf4},
		{0x1f, 0xdd, 0xa8, 0x33, 0x88, 0x07, 0xc7, 0x31, 0xb1, 0x12, 0x10, 0x59, 0x27, 0x80, 0xec, 0x5f},
		{0x60, 0x51, 0x7f, 0xa9, 0x19, 0xb5, 0x4a, 0x0d, 0x2d, 0xe5, 0x7a, 0x9f, 0x93, 0xc9, 0x9c, 0xef},
		{0xa0, 0xe0, 0x3b, 0x4d, 0xae, 0x2a, 0xf5, 0xb0, 0xc8, 0xeb, 0xbb, 0x3c, 0x83, 0x53, 0x99, 0x61},
		{0x17, 0x2b, 0x04, 0x7e, 0xba, 0x77, 0xd6, 0x26, 0xe1, 0x69, 0x14, 0x63, 0x55, 0x21, 0x0c, 0x7d}
	};	// 填充Sbox矩阵
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
void main()
{
	printf("**************AES加解密***************\n");
	int i = 0, k;
	unsigned char PlainText[16] = { 0x32,0x43,0xf6,0xa8,0x88,0x5a,0x30,0x8d,0x31,0x31,0x98,0xa2,0xe0,0x37,0x07,0x34 },
		CipherKey[16] = { 0x2b,0x7e,0x15,0x16,0x28,0xae,0xd2,0xa6,0xab,0xf7,0x15,0x88,0x09,0xcf,0x4f,0x3c },
		CipherKey1[16] = { 0x2b,0x7e,0x15,0x16,0x28,0xae,0xd2,0xa6,0xab,0xf7,0x15,0x88,0x09,0xcf,0x4f,0x3c },
		PlainText1[16] = { 0 },
		PlainText2[16] = { 0 };
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
		printf("\n第%d轮循环：\n", i + 1);
		SubBytes(PlainText, PlainText1, 16);/*S盒置换*/
		ShiftRows(PlainText1);	/*行移位*/
		MixColumns(PlainText1, PlainText2);	/*列混淆*/
		ExtendCipherKey(CipherKey_word + 4 * i, i);/*子密钥生成*/
		for (k = 0; k < 4; k++)  WordToChar(CipherKey_word[k + 4 * (i + 1)], CipherKey + 4 * k);
		printf("此时的子密钥为：    ");
		for (k = 0; k < 16; k++)  printf("%02X ", CipherKey[k]);
		AddRoundKey(PlainText2, CipherKey);/*轮密钥加*/
		for (k = 0; k < 16; k++)  PlainText[k] = PlainText2[k];
		printf("\n当前明文加密之后为：");
		for (k = 0; k < 16; k++)  printf("%02X ", PlainText2[k]);
		printf("\n");
	}
	printf("\n最后一次循环：\n");
	SubBytes(PlainText, PlainText1, 16);
	ShiftRows(PlainText1);
	ExtendCipherKey(CipherKey_word + 4 * i, i);
	for (k = 0; k < 4;WordToChar(CipherKey_word[k + 4 * (i + 1)], CipherKey + 4 * k), k++);
	printf("此时的子密钥为：     ");
	for (k = 0; k < 16; k++)  printf("%02X ", CipherKey[k]);
	AddRoundKey(PlainText1, CipherKey);
	printf("\n\n最终AES加密后的密文为：");
	for (i = 0; i < 16; i++)  printf("%02X ", PlainText1[i]);
	printf("\n\n**************开始解密***************");
	AddRoundKey(PlainText1, CipherKey);
	for (i = 0; i < 9; i++)
	{
		printf("\n第%d次循环：", i + 1);
	    SubBytesRe(PlainText1, PlainText, 16);/*S盒置换*/
		for (k = 0; k < 4; WordToChar(CipherKey_word[k + 40 - 4 * (i + 1)], CipherKey + 4 * k),k++);/*子密钥生成*/
		ShiftRowsRe(PlainText);/*行移位逆*/
		AddRoundKey(PlainText, CipherKey);/*轮密钥加*/
		MixColumnsRe(PlainText);/*列混淆逆运算*/
		for (k = 0; k < 16;PlainText1[k] = PlainText[k],k++);
		printf("\n当前密文解密之后为：");
		for (k = 0; k < 16; k++)printf("%02X ", PlainText[k]);
		printf("\n");
	}
	printf("\n最后一次循环：");
	ShiftRowsRe(PlainText);/*行移位逆*/
	SubBytesRe(PlainText, PlainText1, 16);/*S盒置换*/
	AddRoundKey(PlainText1, CipherKey1);
	printf("\n最终AES解密后的明文为：");
	for (i = 0; i < 16; i++)  printf("%02X ", PlainText1[i]);
	printf("\n");
	system("pause");
}

```
# BlowFish

```cpp BlowFish伪代码
// 轮函数
DWORD Feistel(DWORD x)
{
	DWORD h = S[0][x >> 24] + S[1][x >> 16 & 0xff];
	return (h ^ S[2][x >> 8 & 0xff]) + S[3][x & 0xff];
}
void encrypt(DWORD& XL, DWORD& XR)
{
	for (int i = 0; i < 16; i += 2)
	{
		XL ^= P[i];
		XR ^= Feistel(XL);
		XR ^= P[i + 1];
		XL ^= Feistel(XR);
	}
	XL ^= P[16];
	XR ^= P[17];
	swap(XL, XR);
}

void decrypt(DWORD& XL, DWORD& XR)
{
	for (int i = 16; i > 0; i -= 2)
	{
		XL ^= P[i + 1];
		XR ^= Feistel(XL);
		XR ^= P[i];
		XL ^= Feistel(XR);
	}
	XL ^= P[1];
	XR ^= P[0];
	swap(XL, XR);
}

```

```python BlowFish加密解密
from Crypto.Cipher import Blowfish
import codecs

class BlowfishCipher:
    def __init__(self):
        pass

    def encrypt(self, plaintext, key):
        key = key.encode("utf-8")
        cipher = Blowfish.new(key, Blowfish.MODE_ECB)
        
        # 将明文填充到8字节的倍数
        plaintext = plaintext.ljust((len(plaintext) + 7) // 8 * 8)
        
        ciphertext = cipher.encrypt(plaintext.encode('utf-8'))
        hex_encode = codecs.encode(ciphertext, 'hex_codec').decode('utf-8')
        return hex_encode

    def decrypt(self, ciphertext, key):
        key = key.encode("utf-8")
        cipher = Blowfish.new(key, Blowfish.MODE_ECB)
        
        ciphertext = codecs.decode(ciphertext, 'hex_codec')
        decrypted_text = cipher.decrypt(ciphertext).decode('utf-8').rstrip()
        return decrypted_text

if __name__ == '__main__':
    plaintext = ''
    key = 'UzBtZTBuZV9EMGcz'
    
    blowfish_cipher = BlowfishCipher()
    encrypted_text='11a51f049550e2508f17e16cf1632b47'
    decrypted_text = blowfish_cipher.decrypt(encrypted_text, key)
    print(f"加密: {encrypted_text}, 解密: {decrypted_text}")

```

# SM4
```cpp 加密解密算法
#include<stdio.h>
#define u8 unsigned char
#define u32 unsigned long

// S盒
const u8 Sbox[256] = {
	0xd6,0x90,0xe9,0xfe,0xcc,0xe1,0x3d,0xb7,0x16,0xb6,0x14,0xc2,0x28,0xfb,0x2c,0x05,
	0x2b,0x67,0x9a,0x76,0x2a,0xbe,0x04,0xc3,0xaa,0x44,0x13,0x26,0x49,0x86,0x06,0x99,
	0x9c,0x42,0x50,0xf4,0x91,0xef,0x98,0x7a,0x33,0x54,0x0b,0x43,0xed,0xcf,0xac,0x62,
	0xe4,0xb3,0x1c,0xa9,0xc9,0x08,0xe8,0x95,0x80,0xdf,0x94,0xfa,0x75,0x8f,0x3f,0xa6,
	0x47,0x07,0xa7,0xfc,0xf3,0x73,0x17,0xba,0x83,0x59,0x3c,0x19,0xe6,0x85,0x4f,0xa8,
	0x68,0x6b,0x81,0xb2,0x71,0x64,0xda,0x8b,0xf8,0xeb,0x0f,0x4b,0x70,0x56,0x9d,0x35,
	0x1e,0x24,0x0e,0x5e,0x63,0x58,0xd1,0xa2,0x25,0x22,0x7c,0x3b,0x01,0x21,0x78,0x87,
	0xd4,0x00,0x46,0x57,0x9f,0xd3,0x27,0x52,0x4c,0x36,0x02,0xe7,0xa0,0xc4,0xc8,0x9e,
	0xea,0xbf,0x8a,0xd2,0x40,0xc7,0x38,0xb5,0xa3,0xf7,0xf2,0xce,0xf9,0x61,0x15,0xa1,
	0xe0,0xae,0x5d,0xa4,0x9b,0x34,0x1a,0x55,0xad,0x93,0x32,0x30,0xf5,0x8c,0xb1,0xe3,
	0x1d,0xf6,0xe2,0x2e,0x82,0x66,0xca,0x60,0xc0,0x29,0x23,0xab,0x0d,0x53,0x4e,0x6f,
	0xd5,0xdb,0x37,0x45,0xde,0xfd,0x8e,0x2f,0x03,0xff,0x6a,0x72,0x6d,0x6c,0x5b,0x51,
	0x8d,0x1b,0xaf,0x92,0xbb,0xdd,0xbc,0x7f,0x11,0xd9,0x5c,0x41,0x1f,0x10,0x5a,0xd8,
	0x0a,0xc1,0x31,0x88,0xa5,0xcd,0x7b,0xbd,0x2d,0x74,0xd0,0x12,0xb8,0xe5,0xb4,0xb0,
	0x89,0x69,0x97,0x4a,0x0c,0x96,0x77,0x7e,0x65,0xb9,0xf1,0x09,0xc5,0x6e,0xc6,0x84,
	0x18,0xf0,0x7d,0xec,0x3a,0xdc,0x4d,0x20,0x79,0xee,0x5f,0x3e,0xd7,0xcb,0x39,0x48
};
	
// 密钥扩展算法的常数FK 
const u32 FK[4] = {
	0xa3b1bac6, 0x56aa3350, 0x677d9197, 0xb27022dc
};

// 密钥扩展算法的固定参数CK 
const u32 CK[32] = {
	0x00070e15, 0x1c232a31, 0x383f464d, 0x545b6269,
	0x70777e85, 0x8c939aa1, 0xa8afb6bd, 0xc4cbd2d9,
	0xe0e7eef5, 0xfc030a11, 0x181f262d, 0x343b4249,
	0x50575e65, 0x6c737a81, 0x888f969d, 0xa4abb2b9,
	0xc0c7ced5, 0xdce3eaf1, 0xf8ff060d, 0x141b2229,
	0x30373e45, 0x4c535a61, 0x686f767d, 0x848b9299,
	0xa0a7aeb5, 0xbcc3cad1, 0xd8dfe6ed, 0xf4fb0209,
	0x10171e25, 0x2c333a41, 0x484f565d, 0x646b7279
};

u32 functionB(u32 b); // 查S盒的函数B 
u32 loopLeft(u32 a, short length); // 循环左移函数 
u32 functionL1(u32 a); // 线性变换L
u32 functionL2(u32 a); // 线性变换L'
u32 functionT(u32 a, short mode); // 合成变换T
void extendFirst(u32 MK[], u32 K[]); // 密钥扩展算法第一步
void extendSecond(u32 RK[], u32 K[]); // 密钥扩展算法第二步
void getRK(u32 MK[], u32 K[], u32 RK[]); // 轮密钥获取算法
void iterate32(u32 X[], u32 RK[]); // 迭代算法
void reverse(u32 X[], u32 Y[]); // 反转函数 
void encryptSM4(u32 X[], u32 RK[], u32 Y[]); // 加密算法
void decryptSM4(u32 X[], u32 RK[], u32 Y[]); // 解密算法
 
/*
	查S盒的函数B 
	参数:	u32 b
	返回值:	查S盒的结果u32 b
*/ 
u32 functionB(u32 b) {
	u8 a[4];
	short i;
	a[0] = b / 0x1000000;
	a[1] = b / 0x10000;
	a[2] = b / 0x100;
	a[3] = b;
	b = Sbox[a[0]] * 0x1000000 + Sbox[a[1]] * 0x10000 + Sbox[a[2]] * 0x100 + Sbox[a[3]];
	return b;
}

/*
	循环左移算法
	参数：	u32 a    length：循环左移位数
	返回值：u32 b 
*/
u32 loopLeft(u32 a, short length) {
	short i;
	for(i = 0; i < length; i++) {
		a = a * 2 + a / 0x80000000;
	}
	return a;
}

/* 
	密钥线性变换函数L
	参数：	u32 a
	返回值：线性变换后的u32 a	
*/
u32 functionL1(u32 a) {
	return a ^ loopLeft(a, 2) ^ loopLeft(a, 10) ^ loopLeft(a, 18) ^ loopLeft(a, 24);
}

/* 
	密钥线性变换函数L'
	参数：	u32 a
	返回值：移位操作后的u32 a	
*/
u32 functionL2(u32 a) {
	return a ^ loopLeft(a, 13) ^ loopLeft(a, 23);
}

/*
	合成变换T
	参数：	u32 a    short mode：1表示明文的T，调用L；2表示密钥的T，调用L' 
	返回值：合成变换后的u32 a 
*/
u32 functionT(u32 a, short mode) {
	return mode == 1 ? functionL1(functionB(a)) : functionL2(functionB(a));
}
 
/* 
	密钥扩展算法第一步
	参数：	MK[4]：密钥  K[4]:中间数据，保存结果	（FK[4]：常数） 
	返回值：无 
*/ 
void extendFirst(u32 MK[], u32 K[]) {
	int i;
	for(i = 0; i < 4; i++) {
		K[i] = MK[i] ^ FK[i]; 
	} 
}

/* 
	密钥扩展算法第二步
	参数：	RK[32]：轮密钥，保存结果    K[4]：中间数据 （CK[32]：固定参数） 
	返回值：无
*/ 
void extendSecond(u32 RK[], u32 K[]) {
	short i;
	for(i = 0; i <32; i++) {
		K[(i+4)%4] = K[i%4] ^ functionT(K[(i+1)%4] ^ K[(i+2)%4] ^ K[(i+3)%4] ^ CK[i], 2);
		RK[i] = K[(i+4)%4];
	} 
}

/*
	密钥扩展算法 
	参数：	MK[4]：密钥     K[4]：中间数据    RK[32]：轮密钥，保存结果 
	返回值：无 
*/ 
void getRK(u32 MK[], u32 K[], u32 RK[]) {
	extendFirst(MK, K);
	extendSecond(RK, K);
}

/*
	迭代32次
	参数：	u32 X[4]：迭代对象，保存结果    u32 RK[32]：轮密钥
	返回值：无	  
*/
void iterate32(u32 X[], u32 RK[]) {
	short i;
	for(i = 0; i < 32; i++) {
		X[(i+4)%4] = X[i%4] ^ functionT(X[(i+1)%4] ^ X[(i+2)%4] ^ X[(i+3)%4] ^ RK[i], 1);
	}
}

/*
	反转函数 
	参数；	u32 X[4]：反转对象    u32 Y[4]：反转结果
	返回值：无 
*/
void reverse(u32 X[], u32 Y[]) {
	 short i;
	 for(i = 0; i < 4; i++){
	 	Y[i] = X[4 - 1 - i];
	 } 
} 

/*
	加密算法
	参数：	u32 X[4]：明文    u32 RK[32]：轮密钥    u32 Y[4]：密文，保存结果 
	返回值：无 
*/
void encryptSM4(u32 X[], u32 RK[], u32 Y[]) {
	iterate32(X, RK);
	reverse(X, Y);
} 

/*
	解密算法
	参数： 	u32 X[4]：密文    u32 RK[32]：轮密钥    u32 Y[4]：明文，保存结果
	返回值：无 
*/
void decryptSM4(u32 X[], u32 RK[], u32 Y[]) {
	short i;
	u32 reverseRK[32];
	for(i = 0; i < 32; i++) {
		reverseRK[i] = RK[32-1-i];
	}
	iterate32(X, reverseRK);
	reverse(X, Y);
}
 
/*
	测试数据：
	明文：	01234567 89abcdef fedcba98 76543210
	密钥：	01234567 89abcdef fedcba98 76543210
	密文：	681edf34 d206965e 86b3e94f 536e4246 
*/
int main(void) {
	u32 X[4]; // 明文 
	u32 MK[4]; // 密钥 
	u32 RK[32]; // 轮密钥  
	u32 K[4]; // 中间数据 
	u32 Y[4]; // 密文 
	short i; // 临时变量 
	printf("明文："); 
	scanf("%8x%8x%8x%8x", &X[0], &X[1], &X[2], &X[3]);
	printf("密钥："); 
	scanf("%8x%8x%8x%8x", &MK[0], &MK[1], &MK[2], &MK[3]);
	printf("**************生成轮密钥*****************\n"); 
	getRK(MK, K, RK);
	for(i = 0; i < 32; i++) {
		printf("[%2d]：%08x    ", i, RK[i]);
		if(i%4 == 3)	printf("\n"); 
	}
	printf("************** 生成密文 *****************\n"); 
	encryptSM4(X, RK, Y);
	printf("%08x %08x %08x %08x\n", Y[0], Y[1], Y[2], Y[3]);
	printf("************** 生成明文 *****************\n");  
	decryptSM4(Y, RK, X);
	printf("%08x %08x %08x %08x\n", X[0], X[1], X[2], X[3]);
	return 0;	
} 

```

# DES
```cpp DES加密解密算法
#include<stdio.h>  
#include<string.h> 
#include<stdlib.h> 
#include"table.h"//存储各种数据表的头文件 

// 十六轮子密钥 
static bool SubKey[16][48] = { 0 }; 
/*-----------------------------自定义函数-----------------------------*/
 void SetKey(char My_key[8]); //生成16轮的子密钥 
 void ByteToBit(bool * Data_out, char * Data_in, int Num); //字节转换成位
 void BitToByte(char My_message[8], bool * Message_in, int Num); //位转换成字节 
 void TableReplace(bool *Data_out, bool *Data_in, const char *Table, int Num);  //各种表的置换算法 
 void Bitcopy(bool * Data_out, bool * Data_in, int Num);  //二进制数组的拷贝 
 void Loop_bit(bool * Data_out, int movstep, int len);  //左移
 void Run_Des(char My_message[8], char CIPhertext[64]);//des的轮加密算法
 void Xor(bool * Message_out, bool * Message_in, int Num); //执行异或 
 void S_change(bool * Data_out, bool * Data_in);  // S盒变换 
 void Run_desDes(char My_message[8], char CIPhertext[64]);// DES轮解密算法 
/*--------------------------*/

/*--------------------------主函数----------------------------------*/
   int main()
1
{    
	int i = 0, j;    
	char My_key[8] = { 0 };  //记录加密密钥    
	char You_key[8] = { 0 }; //解密密钥    
	char My_message[8] = { 0 }; //明文    
	char CIPhertext[64] = { 0 };//密文    
	printf("请输入你要加密的内容(8 Byte):\n");    
	scanf("%s", My_message);    
	printf("请输入你的加密密钥(8 Byte):\n");    
	scanf("%s", My_key);    
	i = strlen(My_key);    
	while (i != 8)//确保密钥长度为8byte    
	{      
	    printf("请输入加密密钥(8 Byte)\n");      
	    scanf("%s", My_key);      
	    i = 0;       
	    i = strlen(My_key);   
	}    
	SetKey(My_key);  //生成16轮的加密子密钥  
	Run_Des(My_message, CIPhertext); //des的轮加密过程   
	printf("经过加密的密文为(二进制):\n");    
	printf("%s", CIPhertext);    
	printf("\n");    
	Run_desDes(My_message, CIPhertext);//解密;    
	printf("解密结果为:\n");    
	printf("%s", My_message);    
	printf("\n");
	return 0; 
}

 /*--------------------具体函数定义----------------------*/ 
void Bitcopy(bool * Data_out, bool * Data_in, int Num) //二进制数组拷贝 
{    
	int i = 0;    
	for (i = 0; i < Num; i++)       
	Data_out[i] = Data_in[i]; 
} 
void ByteToBit(bool * Data_out, char * Data_in, int Num) //字节转位，num为二进制位数 
{    
	int i, j;    
	for (i = 0; i < Num; i++)       
	Data_out[i] = (Data_in[i / 8] >> (i % 8)) & 0x01; 
}
void BitToByte(char My_message[8], bool * Message_in, int Num) //位转换成字节，num为位数 
{    
	int i = 0;    
	for (i = 0; i < (Num / 8); i++)    
		My_message[i] = 0;
	for (i = 0; i < Num; i++)   
		My_message[i / 8] |= Message_in[i] << (i % 8);
} 
void TableReplace(bool *Data_out, bool * Data_in, const char *Table, int Num) // 置换算法,Num表示置换表的长度 
{    
	int i = 0;    
	static bool Temp[256] = { 0 };    
	for (i = 0; i < Num; i++)    
	{       
	 Temp[i] = Data_in[Table[i] - 1];//将输入数据的指定位置作为输出数据的第i位    
	 }    
	 Bitcopy(Data_out, Temp, Num);
}

void Loop_bit(bool * Data_out, int movstep, int len)//左循环移位 
{   
	 static bool Temp[256] = { 0 };   
	 Bitcopy(Temp, Data_out, movstep);//将前movstep位数据放入temp缓存   
	 Bitcopy(Data_out, Data_out + movstep, len - movstep);//数据依次向左移位    
	 Bitcopy(Data_out + len - movstep, Temp, movstep);//将temp中缓存的数据放入最后
 } 
void Xor(bool * Message_out, bool * Message_in, int Num)//执行异或 
{    
	int i;
	for (i = 0; i < Num; i++)    
	{       
	 Message_out[i] = Message_out[i] ^ Message_in[i];   
	} 
} 
void SetKey(char My_key[8])//8字节的初始密钥 
{    
	int i, j;    
	static bool Key_bit[64] = { 0 }; //Key的二进制缓存    
	static bool *Key_bit_L, *Key_bit_R;    
	Key_bit_L = &Key_bit[0]; //key的左边28位    
	Key_bit_R = &Key_bit[28]; //key的右边28位  
	ByteToBit(Key_bit, My_key, 64);    
	TableReplace(Key_bit, Key_bit, PC1_Table, 56);//pc-1 置换    
	for (i = 0; i < 16; i++)//生成第i个子密钥Subkey[i]  
	{        
		Loop_bit(Key_bit_L, Move_Table[i], 28);//左边28位进行左循环移位        
		Loop_bit(Key_bit_R, Move_Table[i], 28);//右边28位进行左循环移位        
		TableReplace(SubKey[i], Key_bit, PC2_Table, 48);//pc-2置换    
	} 
} 
void S_change(bool * Data_out, bool * Data_in) //S盒变换 
{    
	int i;    
	int r = 0, c = 0;//S盒的行和列    
	for (i = 0; i < 8; i++, Data_in = Data_in + 6, Data_out = Data_out + 4) // 每6bit输入进行一次代换    
	{        
		r = Data_in[0] * 2 + Data_in[5] * 1;//将m1m6转换为十进制数，作为行数        
		c = Data_in[1] * 8 + Data_in[2] * 4 + Data_in[3] * 2 + Data_in[4] * 1;//将m2m3m4m5转换为十进制数，作为列数        	
		ByteToBit(Data_out, &S_Box[i][r][c], 4);//取S盒数据,将其二进制形式的每一位填到输出的指定位置    
	} 
} 
void F_change(bool Data_out[32], bool Data_in[48])   // f函数 
{    
	int i;    
	static bool Message_E[48] = { 0 };  //存放E置换的结果    
	TableReplace(Message_E, Data_out, E_Table, 48);//E表置换    
	Xor(Message_E, Data_in, 48);//与密钥K[i]异或    
	S_change(Data_out, Message_E);  // S盒变换    
	TableReplace(Data_out, Data_out, P_Table, 32);  //P置换 
}
void Run_Des(char My_message[8], char CIPhertext[64])//des轮加密算法 
{    
	int i;    
	static bool Message_bit[64] = { 0 };    
	static bool *Message_bit_L = &Message_bit[0], *Message_bit_R = &Message_bit[32];//分成左右两部分，每部分32位    
	static bool Temp[32] = { 0 };    
	ByteToBit(Message_bit, My_message, 64);    		
	TableReplace(Message_bit, Message_bit, IP_Table, 64);//初始置换    
	for (i = 0; i < 16; i++)//每一轮的加密    
	{        
		Bitcopy(Temp, Message_bit_R, 32);//缓存右32位        
		F_change(Message_bit_R, SubKey[i]);//轮函数F        
		Xor(Message_bit_R, Message_bit_L, 32);//左32位与轮函数F输出进行异或        
		Bitcopy(Message_bit_L, Temp, 32);//右32位R(i-1)作为下一轮的Li    
	}    
	TableReplace(Message_bit, Message_bit, IPR_Table, 64);//逆初始置换    
	for (i = 0; i < 64;i++)        
	CIPhertext[i] = Message_bit[i]+'0';//由于是将bool型数据赋给字符型数据，因此要+'0'    
	CIPhertext[i] = '\0'; 
} 
void Run_desDes(char My_message[8], char CIPhertext[64])// DES轮解密算法 
{    
	int i = 0;    
	static bool Message_bit[64] = { 0 };    
	static bool * Message_bit_L = &Message_bit[0], *Message_bit_R = &Message_bit[32];    
	static bool Temp[32] = { 0 };    
	for (i = 0; i < 64; i++)        
	Message_bit[i] = CIPhertext[i]-'0';    
	TableReplace(Message_bit, Message_bit, IP_Table, 64);    
	for (i = 15; i >= 0; i--)//与加密过程的区别是子密钥使用顺序相反    
	{        
		Bitcopy(Temp, Message_bit_L, 32);        
		F_change(Message_bit_L, SubKey[i]);        
		Xor(Message_bit_L, Message_bit_R, 32);	
		Bitcopy(Message_bit_R, Temp, 32);    }    			
		TableReplace(Message_bit, Message_bit, IPR_Table, 64);    
		BitToByte(My_message, Message_bit, 64); 
	}
}

```


# BASE58
```cpp base58加密解密
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
 
unsigned char* en_base58(unsigned char* input)  // 编码
{
    static char* nb58 = (char*)"123456789ABCDEFGHJKLMNPQRSTUVWXYZabcdefghijkmnopqrstuvwxyz";
    size_t len = strlen((char*)input);
    size_t rlen = (len / 2 + 1) * 3;
    unsigned char* ret = (unsigned char*)malloc(rlen + len);
    unsigned char* src = ret + rlen;
    unsigned char* rptr = ret + rlen;
    unsigned char* ptr, * e = src + len - 1;
    size_t i;
    memcpy(src, input, len);
    while (src <= e)
    {
        if (*src)
        {
            unsigned char rest = 0;
            ptr = src;
            while (ptr <= e)
            {
                unsigned int c = rest * 256;
                rest = (c + *ptr) % 58;
                *ptr = (c + *ptr) / 58;
                ptr++;
            }
            --rptr;
            *rptr = nb58[rest];
        }
        else
        {
            src++;
        }
    }
    for (i = 0; i < ret + rlen - rptr; i++)
        ret[i] = rptr[i];
    ret[i] = 0;
    return ret;
}
 
bool de_base58(unsigned char* src)  // 解码
{
    static char b58n[] =
    {
        -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1,
        -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1,
        -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1,
        -1,  0,  1,  2,  3,  4,  5,  6,  7,  8, -1, -1, -1, -1, -1, -1,
        -1,  9, 10, 11, 12, 13, 14, 15, 16, -1, 17, 18, 19, 20, 21, -1,
        22, 23, 24, 25, 26, 27, 28, 29, 30, 31, 32, -1, -1, -1, -1, -1,
        -1, 33, 34, 35, 36, 37, 38, 39, 40, 41, 42, 43, -1, 44, 45, 46,
        47, 48, 49, 50, 51, 52, 53, 54, 55, 56, 57, -1, -1, -1, -1, -1,
        -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1,
        -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1,
        -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1,
        -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1,
        -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1,
        -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1,
        -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1,
        -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1,
    };
    size_t len = strlen((char*)src);
    size_t rlen = (len / 4 + 1) * 3;
    unsigned char* ret = (unsigned char*)malloc(rlen);
    unsigned char* rptr = ret + rlen;
    size_t i;
    unsigned char* ptr;
    for (i = 0; i < len; i++)
    {
        char rest = b58n[src[i]];
        if (rest < 0)
        {
            free(ret);
            return NULL;
        }
        for (ptr = ret + rlen - 1; ptr >= rptr; ptr--)
        {
            unsigned int c = rest + *ptr * 58;
            *ptr = c % 256;
            rest = c / 256;
        }
        if (rest > 0)
        {
            rptr--;
            if (rptr < ret)
            {
                free(ret);
                return NULL;
            }
            *rptr = rest;
        }
    }
    for (i = 0; i < ret + rlen - rptr; i++)
        ret[i] = rptr[i];
    ret[i] = 0;
    memcpy(src, ret, strlen((char*)src));
}
 
int main()
{
    char str[] = "9EGJCxbxRGT";
    printf("%s\n", en_base58((unsigned char*)"12345678"));
    de_base58((unsigned char*)str);
    printf("%s", str);
    getchar();
    return 0;
}
```

# SHA1
```cpp sha1加密解密实现
#include<stdio.h>
#include<stdlib.h>
#include<string.h> 
#define rol(x,y) ((x<<y)|(x>>(32-y)))  //循环左移 

//一次循环过程，str为填充后的数据或是数据中的一部分 
void round(unsigned char str[64],unsigned int h[5]){
	unsigned int a, b, c, d, e,tmp,w[80];
	unsigned int i;
	for(i=0;i<16;i++){
		w[i]=((unsigned int)str[i*4]<<24)|(((unsigned int)str[i*4+1])<<16)|
						(((unsigned int)str[i*4+2])<<8)|(((unsigned int)str[i*4+3])<<0);
	}
	for (i=16;i<80;i++ ){
		tmp = w[i-3]^w[i-8]^w[i-14]^w[i-16];
		w[i]=rol(tmp,1);
    }
    
    a=h[0];b=h[1];c=h[2];d=h[3];e=h[4];
    for(i=0;i<80;i++){
    	switch(i/20){
    		case 0:tmp=rol(a,5)+((b&c)|(d&~b))+e+w[i]+0x5a827999;break;
    		case 1:tmp=rol(a,5)+(b^c^d)+e+w[i]+0x6ed9eba1;break;
    		case 2:tmp=rol(a,5)+((b&c)|(b&d)|(c&d))+e+w[i] +0x8f1bbcdc;break;
    		case 3:tmp=rol(a,5)+(b^c^d)+e+w[i] + 0xca62c1d6;break;
		}
		e=d;d=c;
		c=rol(b,30);
		b=a;a=tmp;
	}
	h[0]+=a;h[1]+=b;h[2]+=c;h[3]+=d;h[4]+=e;
}

//sha-1算法 
void sha1(unsigned char*input,long long len,unsigned char*output){
	unsigned char temp[64];
	unsigned int h[5]={0x67452301,0xefcdab89,0x98badcfe,0x10325476,0xc3d2e1f0};
	unsigned int i,n=len,tmp;
	while(n>=64){
		memcpy(temp,input+len-n,64);
		round(temp,h);
		n-=64;
	}
	
	if(n>=56){
		memset(temp,0,64);
		memcpy(temp,input+len-n,n);temp[n]=128;
		round(temp,h);
		memset(temp,0,64);
		for(i=56;i<64;i++)
			temp[i]=((len*8)>>(63-i)*8)&0xff;
		round(temp,h);
	}
	else{
		memset(temp,0,64);
		memcpy(temp,input+len-n,n);temp[n]=128;
		for(i=56;i<64;i++)
			temp[i]=((len*8)>>(63-i)*8)&0xff;
		round(temp,h);	
	}
	
	for(i=0;i<20;i++){
		tmp=(h[i/4]>>((3-i%4)*8))&0xff;
		sprintf((char*)output+2*i,"%02x",tmp);
	}
}

//测试 
int main(){
	unsigned char input[]="this is a test aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa",output[40]={0};
	sha1(input,strlen((char*)input),output);
	printf("%s\n",output);
}


```

# RC4
```cpp rc4加密解密
#include<stdio.h> 
#include<stdlib.h>
#include<string.h>

//s表的长度取256
#define size 256

unsigned char sbox[257]={0};

//初始化s表
void init_sbox(unsigned char*key){
	unsigned int i,j,k;
	int tmp;
	
	for(i=0;i<size;i++){
		sbox[i]=i;
	}
	
	j=k=0;
	for(i=0;i<size;i++){
		tmp=sbox[i];
		j=(j+tmp+key[k])%size;
		sbox[i]=sbox[j];
		sbox[j]=tmp;
		if(++k>=strlen((char*)key))k=0;
	}
}

//加解密函数
void enc_dec(unsigned char*key,unsigned char*data){
	int i,j,k,R,tmp;
	
	init_sbox(key);
	
	j=k=0;	
	for(i=0;i<strlen((char*)data);i++){
		j=(j+1)%size;
		k=(k+sbox[j])%size;
		
		tmp=sbox[j];
		sbox[j]=sbox[k];
		sbox[k]=tmp;
		
		R=sbox[(sbox[j]+sbox[k])%size];
		
		data[i]^=R;
	}	
}

int main(){
	unsigned char key[100]={0};
	unsigned char data[100]={0};
	printf("输入你要加密的字符：");
	scanf("%100s",data);
	printf("输入密钥：");
	scanf("%40s",key);
	enc_dec(key,data);
	printf("enc: %s\n",data);
	enc_dec(key,data);
	printf("dec: %s\n",data);
	return 0;
}

```