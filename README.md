# software-analysis-and-test
**first.c**:简单的加法程序，用于实验  
**first.ll**:使用LLVM将first.c转化为SSA后的程序，可见为部分SSA  

## 分析
first.ll的主体为  
```c
define void @add(i32, i32) #0 {
  %3 = alloca i32, align 4
  %4 = alloca i32, align 4
  %5 = alloca i32, align 4
  store i32 %0, i32* %3, align 4
  store i32 %1, i32* %4, align 4
  %6 = load i32, i32* %3, align 4
  %7 = load i32, i32* %4, align 4
  %8 = add nsw i32 %6, %7
  store i32 %8, i32* %5, align 4
  %9 = load i32, i32* %5, align 4
  %10 = load i32, i32* %4, align 4
  %11 = sub nsw i32 %9, %10
  store i32 %11, i32* %5, align 4
  ret void
}
```  
为展示效果明显，将编号用变量名表示(3-a1, 4-b1, 5-c1, 6-b2, 7-a1, 8-c2, 9-c3, 10-b3, 11-c3)可以得出  

```  c
define void @add(i32, i32) #0 {
  %a1 = alloca i32, align 4
  %b1 = alloca i32, align 4
  %c1 = alloca i32, align 4
  store i32 %0, i32* %a1, align 4
  store i32 %1, i32* %b1, align 4
  %a2 = load i32, i32* %a1, align 4
  %b2 = load i32, i32* %b1, align 4
  %c2 = add nsw i32 %a2, %b2
  store i32 %c2, i32* %c1, align 4
  %c3 = load i32, i32* %c1, align 4
  %b3 = load i32, i32* %b1, align 4
  %c4 = sub nsw i32 %c3, %b3
  store i32 %c4, i32* %c1, align 4
  ret void
}
```    
由上面可以看出c1被反复赋值，即赋值后内存位置不变，因为SSA中单一变量只能被赋值一次，因此是部分SSA。  
原因：为减少分支节点的应用（上述实验只是简单模拟部分SSA，并没有体现出该效果），llvm在涉及到内存上的值存储时没有采用SSA，而是反复赋值。
