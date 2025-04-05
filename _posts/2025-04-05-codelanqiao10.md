---
layout: post

title: Com10_第十届蓝桥杯大赛软件赛省赛Java大学A组

date: 2025-04-05 22:36:22 +0900

categories: [Code]
---

### 第八届蓝桥杯大赛软件赛省赛Java大学A组

#### 1.  平方和

<p align="center">
    <img src="https://hhhi21g.github.io/public/img/lanqiao10/l0.png" alt="alt text" style="zoom:70%;" />
</p>

> 2658417853

这才是真正的签到题！

------

#### 2.  数列求值

<p align="center">
    <img src="https://hhhi21g.github.io/public/img/lanqiao10/l1.png" alt="alt text" style="zoom:70%;" />
</p>

```java
package lanQ10;

import java.util.ArrayList;
import java.util.List;

public class L2 {

	static long idx = 0;

	public static void main(String[] args) {
		List<Long> a = new ArrayList<>();
		a.add((long) 1);
		a.add((long) 1);
		a.add((long) 1);
		idx = 3;
		for (long i = idx; i < 20190324; i++) {
			long addNum = a.get((int) (i - 1)) % 10000 + a.get((int) (i - 2)) % 10000 + a.get((int) (i - 3)) % 10000;
			a.add(addNum);
		}
		long res = a.get(20190323);
		System.out.print(res % 10000);
	}
}
```

数字太大只要求最末几位的情况下，**在中间步骤就取模**，不然结果出来也是错的

------

#### 3.  迷宫

------

#### 4.  最大降雨量

<p align="center">
    <img src="https://hhhi21g.github.io/public/img/lanqiao10/l2.png" alt="alt text" style="zoom:70%;" />
</p>

```java
public class Main {
    public static void main(String[] args) {
        /*
        * [][][][a][][][]
        * [][][][b][][][]
        * [][][][c][][][]
        * [][][][max][][][]
        * [][][][d][][][]
        * [][][][e][][][]
        * [][][][f][][][]
        * 
        * 此题意思为将49分为7组数字，求取七组数字中每组数字的中位数所构成的数列的中位数的最大值
        * 即如图所示，最大化[max]
        * 49个数字中需要比[max]大的有【max】行的后三位，d、e、f行的后四位
        * 即结果如下
        * */
        System.out.println(49 - (3 * 4) - 3);
    }
}
```

------

#### 5.  RSA解密

------

#### 6.  完全二叉树的权值

<p align="center">
    <img src="https://hhhi21g.github.io/public/img/lanqiao10/l3.png" alt="alt text" style="zoom:70%;" />
</p>

```java
package lanQ10;

import java.util.Scanner;

public class L6 {

	public static void main(String[] args) {
		Scanner sc = new Scanner(System.in);
		int n = sc.nextInt();
		int[] w = new int[n + 1];
		for (int i = 1; i <= n; i++) {
			w[i] = sc.nextInt();
		}
		int max = 0;
		int resDep = 1;
		int total = 0;
		int resd = 1;

		for (int i = 1; total < n; i++, resd++) {
			total = total + Math.min((1 << (i - 1)), n - total);

			int resw = 0;

			for (int j = (1 << (i - 1)); j < (1 << i) && j <= n; j++) {
				resw += w[j];
			}
			if (resw > max) {
				max = resw;
				resDep = resd;
			}
		}

		System.out.print(resDep);
	}
}
```

思路很简单，需要注意**细节问题**：

- 使用左移代替幂的计算；
- **满二叉树与完全二叉树：**
  - 满二叉树：叶子节点只能在最后一层；
  - 完全二叉树：最后一层可以不被铺满；

- 到n就退出

------

