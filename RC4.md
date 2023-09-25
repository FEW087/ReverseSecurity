# RC4

###### 算法关键变量

​		密钥流：RC4算法的关键，根据明文和密钥生成，密钥流的长度与明文的长度相对应。密文第i字节=明文第i字节^密钥流第i字节。

​		状态向量S：长度为256，每个单位也是一个字节，算法运行的任何时候，S都包括0-255的8比特数的排列组合，只是值的位置发生了变换。可以作为伪代码中判断RC4算法的依据。

​		临时向量T：长度也为256，每个单元也是一个字节。密钥的长度为256字节时直接将密钥的值付给T，不然的话，就轮转地将每个字节付给T,也就是将密钥循环输入T直到填满。

​		密钥K:长度为1-256字节，其长度keylen与明文长度和密钥流长度没有关系。

###### 原理

​		1.初始化S和T

```c
for(i=0;i<255;i++)
{
    S[i]=i;
    T[i]=K[i%keylen];
}
```

​		2.初始排列S

```c
int j=0;
for(i=0;i<255;i++)
{
    j=(j+S[i]+T[i])%256;
    swap(S[i],S[j]);
}
```

​		3.产生密钥流

```c
int i,j=0;
for(r=0;r<strlen(明文);r++)
{
    i=(i+1)%256;
    j=(j+S[i])%256;
    swap(S[i],S[j]);
    t=(S[i]+S[j])%256;
    K[r]=S[t];
}
```

加解密函数

```c
#include<stdio.h>
#include<random>
#include<time.h>
#include<string.h>
#define MAX 65534

int S[256]; //向量S
char T[256];    //向量T
int Key[256];   //随机生成的密钥
int KeyStream[MAX]; //密钥
char PlainText[MAX];
char CryptoText[MAX];
const char *WordList = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789";
void init_S()
// 初始化S;
{
    for(int i = 0; i < 256; i++){
        S[i] = i;
    }
}

void init_Key(){
    // 初始密钥
    int index;
    srand(time(NULL));  //根据当前时间，作为种子
    int keylen = int(double(random())/double(RAND_MAX)*256);    //随机获取一个密钥的长度
    for(int i = 0; i < keylen; i++){
        index = int(double(random())/double(RAND_MAX)*63);  //生产密钥数组
        Key[i] = WordList[index];
    }
    int d;
    for(int i = 0; i < 256; i++){   //初始化T[]
        T[i] = Key[i%keylen];
    }

    
}

void  permute_S()
{
    // 置换S;
    int temp;
    int j = 0;
    for(int i = 0; i < 256; i++){
        j = (j + S[i] + T[i]) % 256;
        temp = S[i];
        S[i] = S[j];
        S[j] = temp;
    }
}

void create_key_stream(char *text, int textLength)
{
    // 生成密钥流
    int i,j;
    int temp, t, k;
    int index = 0;
    i = j = 0;
    while(textLength --){   //生成密钥流
        i = (i+1)%256;
        j = (j + S[i]) % 256;
        temp = S[i];
        S[i] = S[j];
        S[j] = temp;
        t = (S[i] + S[j]) % 256;
        KeyStream[index] = S[t];
        index ++;
    }
 
}


 void Rc4EncryptText(char *text)
{
    //加密 && 解密
    int textLength = strlen(text);
    init_S();
    init_Key();
    permute_S();
    create_key_stream(text, textLength);
    int plain_word;
    printf("============开始加密============:\n 密文：");
    for(int i = 0; i < textLength; i++){
        CryptoText[i] = char(KeyStream[i] ^ text[i]); //加密
    }
    for(int i = 0; i < textLength; i++){
        printf("%c", CryptoText[i]);
    }
    printf("\n============加密完成============\n============开始解密============\n明文：");
    for(int i = 0; i < textLength; i++){
        PlainText[i] = char(KeyStream[i] ^ CryptoText[i]);   //解密
    }
    for(int i = 0; i < textLength; i++){
        printf("%c", PlainText[i]);
    }
    printf("\n============解密完成============\n");
    printf("\n");
    
}



int main()
{   
    char text[] = "密文";
    Rc4EncryptText(text);
    return 0;
}

```

