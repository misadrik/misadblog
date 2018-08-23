---
title: 使用分治法计算逆序数
date: 2014-10-25 22:06:52
categories: 
- 技术
- 算法
tags: [分治,技术,算法]
copyright: true
comments: true
reward: true
---

也许是第一次写这样的程序，略显生疏，希望通过以后学习能够有更多的进步，因为是在coursera上学算法，全英文课也着实坑爹了点，只能说继续努力吧！！相信自己。我看行！希望有大神可以多提意见，一定认真学习改进。
<!-- more -->
```cpp
/*----------------------------------------------- 
  名称：Use divide-and-couquer count the inversion 分治法计算逆序数  
  编写：MISaD 
  博客：http://blog.csdn.net/misadd  
  日期：2014.10.25 
  修改：2014.10.25 
        主要错误在于1.错误估计了逆序数的大小，先使用了long 
                    2.merge中错误计算了逆序数个数  
  内容：1.读入文件流 
        2.子程序1,sorted-and-count，将数组递归分为两部分从而分别计算每一部分的逆序数 。  
        3.子程序2,merge-and-count,使用归并排序计算两部分组合的逆序数。  
------------------------------------------------*/  
#include <iostream>  
#include <fstream>// 读入文件流  
#include <vector>//创建vector数组  
using namespace std;  
  
long long CountInversion(vector<long long>&vec);  
long long merge(vector<long long>& vec,vector<long long>& v1,vector<long long>& v2);  
  
int main()  
{  
    ifstream fin("IntegerArray.txt");  
    vector<long long> init;  
    long long temp;  
      
    while(fin>>temp){  
        init.push_back(temp);   
    }  
      
    long long  dataSize = init.size() ;  
    cout<<"Data Size:"<<dataSize<<endl;  
      
    long long x = CountInversion(init);  
      
    cout<<x<<endl;  
      
    return 0;  
      
}  
  
long long CountInversion(vector<long long>&vec)  
{  
    long long n = vec.size();  
    if( 1 == n )   
        return 0;  
    long long result = 0;  
    long long mid = n/2;  
      
    vector<long long> v1,v2;  
    long long k = 0;  
    while(k < n){  
        if(k < mid){  
            v1.push_back(vec[k]);  
            k++;  
        }  
        else{  
            v2.push_back(vec[k]);  
            k++;   
        }  
    }  
      
    long long leftNum,rightNum,splitNum;  
    leftNum = CountInversion(v1);  
    rightNum = CountInversion(v2);  
    splitNum = merge(vec,v1,v2);  
      
    result = leftNum+rightNum+splitNum;  
      
    return result;  
}  
  
long long  merge(vector<long long>& vec,vector<long long>& v1,vector<long long>& v2)  
{  
    long long n1 = v1.size();  
    long long n2 = v2.size();  
    long long n = n1 + n2;  
    long long num = 0;//split-inversion number  
      
    long long p1 = 0,p2 = 0;  
    long long k = 0;  
    while(k < n && p1 < n1 && p2 < n2)  
    {  
        if(v1[p1] > v2[p2]){  
            vec[k] = v2[p2];  
            num +=n1-p1;  
            p2++;  
        }  
        else{  
            vec[k] =v1[p1];  
            p1++;  
        }  
        k++;  
    }  
      
    while(p1 < n1){  
        vec[k] = v1[p1];  
        p1++;  
        k++;  
    }  
    while(p2 < n2){  
        vec[k] = v2[p2];  
        p2++;  
        k++;  
    }  
      
    return num;  
      
}  
```
首先讲一下基本的原理
首先用递归不断的将数组分成v1(0，mid-1)和v2(mid,n)两部分，具体计算逆序数则是通过归并排序来实现的，那么归并排序怎么实现计算逆序数的功能呢？
我们不妨假设v1,v2是两个分别已经按从小到达排好序的数组，所以如果v1[p1]>v2[p2],那么v1中所有从p1开始到mid-1的数都可以与v2[p2]这一个元素是逆序组，所以通过归并排序可以省时省力的计算每一部分中的逆序数，最后只要将各部分的逆序数相加，就可以得到最终结果。

具体写程序的时候有一些问题，首先是读入文件流的时候有下面几种方法
```cpp
while(fin>>temp)  
{  
    init.push_back(temp);  
}  
```
```cpp
for(int p = 0; p < 100000; p++) {  
    fin>>temp;  
    init.push_back(temp);  
}  
```
但是不能这样
```cpp
while(!fin.eof()) {  
       fin>>temp;  
       init.push_back(temp);   
}  
```
因为fin读到文件末尾的时候，再次进行读写才会发现已经到了文件的末尾无法读取了，因此才会返回true；
比如说读到了最后一个数据了，此时仍然为false；当再次读入时候发现不能再读了,这时候才会返回true结束循环，然而这时已经将最后一个数据写入了两次。

然后应用vector数组的优势是比较灵活，push_back（x）的作用是将x依次放入数组；
size()作用是表示数组的大小；

两个子函数的作用在思路中已经做了简单的描述再次不想多说，然后我还想说一下数据大小；
因为我做的是1-100000中所有的数的任意排序的逆序数，所以是一个超出了2^31次，所以用long是不行的，所以改成了long long；当然也可以用double,但是直接输出的话会输出科学计数法，所以需要用
```cpp
cout.setf(ios_base::fixed,ios_base::floatfield);  
```
当然还看到有大神是这么用的
```cpp
char str[20];  
sprintf(str,"%.8lf",x);//将格式化后的x存到字符串str中 （将浮点型转换成字符串）   
printf("%s",str);//不转换会按科学计数法输出   
```
简言之就是讲一个double的数字符串化之后输出。

