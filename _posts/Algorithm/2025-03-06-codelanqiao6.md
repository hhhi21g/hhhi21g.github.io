---
layout: post

title: Com6_第六届蓝桥杯大赛软件赛省赛Java大学A组

date: 2025-03-06 20:01:23 +0900

categories: [数据结构与算法]
tags: [刷题]
---

#### 1. 熊怪吃核桃（15）

<p>
    <img src="https://hhhi21g.github.io/assets/img/Algorithm/lanqiao6/l0.png" alt="alt text" style="zoom:70%;" />
</p>



```java
package lanQ6;

public class sec {

	public static void main(String[] args) {
		int remain = 1543;
		int res = 0;

		while (remain > 1) {
			if (remain % 2 == 1) {
				res++;
			}
			remain /= 2;
		}
		res++;
		System.out.println(res);  //5
	}
}
```

------

#### 2. 星系炸弹（15）

<p>
    <img src="https://hhhi21g.github.io/assets/img/Algorithm/lanqiao6/l1.png" alt="alt text" style="zoom:70%;" />
</p>



硬减

> 2017-08-05

------

#### 3. 九数分三组（25）

<p>
    <img src="https://hhhi21g.github.io/assets/img/Algorithm/lanqiao6/l2.png" alt="alt text" style="zoom:70%;" />
</p>



**思路：**

- 定位三个数的范围：

  C最大 = 987；A最小=123，最大329；

- 通过循环排除一波：

  范围，不能有数字相同，不能有0；这里两两不能相同的组合太多，不一一列举，只做初步筛选；

- 剩下的计算排除

> 192 &nbsp;219 &nbsp;273 &nbsp;327

------

#### 4. 移动距离（25）

<p>
    <img src="https://hhhi21g.github.io/assets/img/Algorithm/lanqiao6/l3.png" alt="alt text" style="zoom:70%;" />
</p>



```java
import java.util.Scanner;

public class Main {

	public static void main(String[] args) {
		int w, m, n;
		Scanner scanner = new Scanner(System.in);
		w = scanner.nextInt();
		m = scanner.nextInt();
		n = scanner.nextInt();

		int[][] floor = new int[10000 / w + 1][w];
		int idx = 1;
		int mw = 0, mh = 0;
		int nw = 0, nh = 0;
		while (idx <= 10000) {
			for (int i = 0; i < w; i++) {
				if ((idx - 1) / w % 2 == 0) {
					floor[(idx - 1) / w][i] = idx;
					if (idx == m) {
						mw = i;
						mh = (idx - 1) / w;
					}
					if (idx == n) {
						nw = i;
						nh = (idx - 1) / w;
					}
				} else {
					floor[(idx - 1) / w][w - 1 - i] = idx;
					if (idx == m) {
						mw = w - 1 - i;
						mh = (idx - 1) / w;
					}
					if (idx == n) {
						nw = w - 1 - i;
						nh = (idx - 1) / w;
					}
				}
				idx++;
			}
		}
		int width = 0;
		int height = 0;

		width = Math.abs(mw - nw);
		height = Math.abs(nh - mh);
		System.out.println(width + height);
	}
}
```

**思路：**

- 设两个楼号对应的数组下标分别为w1,h1和w2,h2；易知，它们最短移动距离为abs(w1-w2)+abs(h1-h2);
- 问题转化为求两个数字在数组中对应的下标；
- 找到蛇形数组数字与索引的规律，当数字与两个楼号相同时，将下标保存即可。

------

#### 5. 垒骰子（35，未解决）

<p>
    <img src="https://hhhi21g.github.io/assets/img/Algorithm/lanqiao6/l4.png" alt="alt text" style="zoom:70%;" />
</p>



分析思路：

1. 先固定一个，加入第二个：

   对于一组不能紧贴的数字：固定一个骰子的接触面，相应接触面有5个选择；

   两个骰子通过旋转都有4种不同摆放方法，`5*4*4*m*2`

   随便紧贴的：`(6 - m * 2) * 4 * 6 * 4`

2. 当加入第三个，把前两个视为整体：

   两个骰子不能紧贴的数字朝上的情况？

**补充内容**：

##### 1. **常数快速幂**(quickPow)

将“二进制指数”拆分，将O(n)降到O(log(n))；当n很大时，适用于该方法。

- 当指数为偶数，res = (base<sup>2</sup>)<sup>n/2</sup>, 记录`base = base * base`, n = n /2, 则res = base<sup>n</sup>;

- 当指数为奇数，由于指数无法被2整除，则结果应先乘上底数，再按照偶数情况拆分；

  此时由第一种情况，**拆分后底数为base**, 则`res = res * base`;

-  指数为0，则退出循环结束运算。

##### 2. 矩阵乘法(**multiplyMatrix**)

三重循环:

```java
for(int i = 0;i<size;i++){
    for(int j = 0;j<size;j++){
        for(int k = 0;k<size;k++){
            res[i][j] = res[i][j] + a[i][k] * b[k][j]
        }
    }
}
```

##### 3. **矩阵快速幂**(matrixPower)

原理与常数快速幂相同，这里需要将结果初始化为单位矩阵，且乘法需要通过2.实现。

```java
package lanQ6;

import java.util.Scanner;

public class fir01 {
	static final int MOD = 1000000007;

	public static void main(String[] args) {
		Scanner sc = new Scanner(System.in);

		long n = sc.nextLong(); // 骰子的个数
		int m = sc.nextInt(); // 互斥面的组数

		boolean[][] b = new boolean[7][7]; // 存储互斥面信息
		int[] shaiZi = { 0, 4, 5, 6, 1, 2, 3 }; // 记录每个面的对面

		// 读取互斥面数据
		for (int i = 0; i < m; i++) {
			int x = sc.nextInt();
			int y = sc.nextInt();
			b[x][y] = true;
			b[y][x] = true;
		}

		// 构造 6x6 转移矩阵
		long[][] matrix = new long[6][6];
		for (int i = 0; i < 6; i++) {
			for (int j = 0; j < 6; j++) {
				if (!b[shaiZi[i + 1]][j + 1]) { // 只有不互斥的情况下才允许转移
					matrix[i][j] = 1;
				}
			}
		}

		// 计算矩阵的 n-1 次幂
		long[][] resultMatrix = matrixPower(matrix, n - 1);

		// 初始 dp
		long sum = 0;
		for (int i = 0; i < 6; i++) {
			for (int j = 0; j < 6; j++) {
				sum = (sum + resultMatrix[i][j]) % MOD;
			}
		}

		// 计算 4^n % MOD
		long power = quickPow(4, n, MOD);
		sum = (sum * power) % MOD;

		System.out.println(sum);
	}

	// 矩阵快速幂
	public static long[][] matrixPower(long[][] matrix, long exp) {
		int size = matrix.length;
		long[][] res = new long[size][size];
		long[][] base = matrix;

		// 初始化单位矩阵
		for (int i = 0; i < size; i++) {
			res[i][i] = 1;
		}

		while (exp > 0) {
			if ((exp & 1) == 1) {  // exp为奇数
				res = multiplyMatrix(res, base);
			}
			base = multiplyMatrix(base, base);
			exp >>= 1;
		}
		return res;
	}

	// 矩阵相乘
	public static long[][] multiplyMatrix(long[][] a, long[][] b) {
		int size = a.length;
		long[][] res = new long[size][size];

		for (int i = 0; i < size; i++) {
			for (int j = 0; j < size; j++) {
				for (int k = 0; k < size; k++) {
					res[i][j] = (res[i][j] + a[i][k] * b[k][j]) % MOD;
				}
			}
		}
		return res;
	}

	// 快速幂计算 4^n % MOD
	public static long quickPow(long base, long exp, int mod) {
		long result = 1;
		while (exp > 0) {
			if ((exp & 1) == 1) { // 如果是奇数
				result = (result * base) % mod;
			}
			base = (base * base) % mod;
			exp >>= 1; // 右移，相当于 exp / 2
		}
		return result;
	}
}
```

**思路：**

- n : 骰子个数； m ：互斥面的组数

  `boolean [][]b`: 存储互斥面信息，true为互斥

  int []shaiZi = {0,4,5,6,1,2,3},存储骰子信息，按照相对面存储

- 接收互斥的一组数字x,y;

  `b[x][y]`互斥，`b[y][x]`也互斥，二者都设置为true;

- <p>
      <img src="https://hhhi21g.github.io/assets/img/Algorithm/lanqiao6/l5.svg" alt="alt text" style="zoom:70%;" />
  </p>

​      matrix[i] = M · matrix[i-1] = M<sup>n-1</sup> · matrix[1],  matrix[i]为第i层的方案数 (该matrix与代码中意义不同)

​      M为矩阵：当其元素下标+1对应的两个数字是互斥面时，该点元素为0；否则，该点元素为1；

- 使用矩阵快速幂，求出resultMatrix作为结果矩阵；
- 遍历求出该结果矩阵中所有元素之和，即为不考虑四周的摆放方案总数；
- 骰子通过旋转，在上下面确定时有四个不同的摆放方式，n个骰子则有4<sup>n</sup>个，通过常数的快速幂计算并乘在结果中，%1e9+7得到结果

****

#### 6. 灾后重建（35，未解决）

<p>
    <img src="https://hhhi21g.github.io/assets/img/Algorithm/lanqiao6/l5.png" alt="alt text" style="zoom:70%;" />
</p>

****
