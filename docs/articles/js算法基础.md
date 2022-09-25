---
title: js算法基础
date: 2014-11-16 11:28:21
tags: javascript
---
## 变量提升
``` javascript
console.log(a); // function a
var a = 10;
function a(){
  console.log("变量提升")
}
console.log(a); // 10
```
<!-- more -->
函数提前执行，且预先解析变量名，相当于先定义a，并未赋值，先赋值给function，后被10重写。
## 函数声明提前执行，超越return
``` javascript
(function f(){
  function f(){console.log('1');}
  return f(); 
  function f(){console.log('2');}
  function f(){console.log('3');}
  function f(){console.log('4');}
  console.log('我永远不会执行！');
})()   //4  
```
函数声明有提前执行的作用，return后面的函数也会在return前执行，但非函数则不行，所以最后的console永远也不会执行。
## 求素数
``` javascript
function getPrimeNumber(num){
  if (num == 1){return false};
  for(var i = 2; i < num; i++){  //依次循环比其小的数
    if(num % i == 0){
      return false;
    }
  }
  return true;
}
getPrimeNumber(11) // true
```
## 求素数2
``` javascript
function getPrimeNumber(num){
  if (num == 1){return false};
  var srqNum = Math.sqrt(num); 
  for(var i = 2; i < srqNum; i++){   //只循环到开根后的数
    if(num % i == 0){
      return false;
    }
  }
  return true;
}
getPrimeNumber(11) // true
```
## 求第二大
``` javascript
function getSencond(arr){
  if (arr.length <= 1){return;}
  var max = arr[0];
  var second = arr[0];
  for(var i = 1;i < arr.length;i++){
    if(max < arr[i]){
      second = max; //更新第二大
      max = arr[i]; // 更新最大
    }else{
      if(second < arr[i]){
        second = arr[i]  // 介于两者之间，更新第二大
      }
    }
  }
  console.log(max,second)
}

getSencond([1,0,8,6,7,9,7,6,7,8,9,8,0,0,10,9,2,2,3,1,9]);
```
用求最大的思想，新建变量容器存储第一大和第二大。
## 快速排序
``` javascript
function qsort(arr){
  if(arr.length <= 1){
    return arr;
  }
  var first = arr[0], rest = arr.slice(1);
  
  return qsort(rest.filter(function(o){ //递归
    return o < first;
  })).concat([first])
  .concat(qsort(rest.filter(function(o){
    return o >= first;
  })));
}

var arr = [3,4,6,1,2,5];
console.log(qsort(arr));
```
## 快排（2）
``` javascript
function quickSort(arr){
      if(arr.length<=1){
        return arr;
      }
      var num = Math.floor(arr.length/2);//中间位置索引，用做标记
      var numValue = arr.splice(num,1);//取出标记位置的值
      var left = [];
      var right = [];
      for(var i =0;i<arr.length;i++){
        if(arr[i]<numValue){
          left.push(arr[i]);
        }else{
          right.push(arr[i]);
        }
      }
      return quickSort(left).concat([numValue],quickSort(right));
  
}
 var arr = [12,5,37,6,22,40]; 
 //alert(quickSort(arr));
 console.log(quickSort(arr));
```
