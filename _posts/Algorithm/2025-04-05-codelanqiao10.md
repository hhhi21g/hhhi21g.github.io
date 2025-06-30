---
layout: post

title: Com10_第十届蓝桥杯大赛软件赛省赛Java大学A组

date: 2025-04-05 22:36:22 +0900

categories: [Code]
---

### 第十届蓝桥杯大赛软件赛省赛Java大学A组

#### 1.  平方和

<p align="center">
    <img src="https://hhhi21g.github.io/assets/img/Algorithm/lanqiao10/l0.png" alt="alt text" style="zoom:70%;" />
</p>


> 2658417853

这才是真正的签到题！

------

#### 2.  数列求值

<p align="center">
    <img src="https://hhhi21g.github.io/assets/img/Algorithm/lanqiao10/l1.png" alt="alt text" style="zoom:70%;" />
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
    <img src="https://hhhi21g.github.io/assets/img/Algorithm/lanqiao10/l2.png" alt="alt text" style="zoom:70%;" />
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
    <img src="https://hhhi21g.github.io/assets/img/Algorithm/lanqiao10/l3.png" alt="alt text" style="zoom:70%;" />
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

#### 7.  外卖优先级

<p align="center">
    <img src="https://hhhi21g.github.io/assets/img/Algorithm/lanqiao10/l4.png" alt="alt text" style="zoom:70%;" />
</p>


```java
import java.util.Arrays;
import java.util.Scanner;

public class Main {

	public static void main(String[] args) {
		Scanner sc = new Scanner(System.in);
		int n = sc.nextInt();
		int m = sc.nextInt();
		int t = sc.nextInt();

		int[][] pay = new int[n + 1][t + 1]; // pay[i][t]: i店家，t时刻的订单数
		boolean[] priority = new boolean[n + 1]; // priority[i]: i店家是否优先1/0
		for (int i = 0; i < m; i++) {
			int num1 = sc.nextInt();
			int num2 = sc.nextInt();
			pay[num2][num1]++;
		}

		int[] priNum = new int[t + 1]; // 优先级
		for (int i = 1; i <= n; i++) {
		    Arrays.fill(priNum, 0);
			for (int j = 1; j <= t; j++) {
				if (pay[i][j] > 0) {
					priNum[j] = priNum[j - 1] + 2 * pay[i][j];
				} else {
					if (priNum[j - 1] - 1 >= 0)
						priNum[j] = priNum[j - 1] - 1;
				}
				if (priNum[j] > 5)
					priority[i] = true;
				if (priNum[j] <= 3)
					priority[i] = false;
			}
		}
		int cnt = 0;
		for (int i = 1; i <= n; i++) {
			if (priority[i])
				cnt++;
		}

		System.out.print(cnt);
	}
}
```

上面代码十个用例通过九个，剩下一个二维数组空间太大爆了

```java
package lanQ10;

import java.util.ArrayList;
import java.util.Collections;
import java.util.List;
import java.util.Scanner;

public class L7_2 {

	public static void main(String[] args) {
		Scanner sc = new Scanner(System.in);
		int n = sc.nextInt();
		int m = sc.nextInt();
		int t = sc.nextInt();

		List<List<Integer>> orders = new ArrayList<>();
		for (int i = 0; i <= n; i++) {
			orders.add(new ArrayList<>());
		}

		for (int i = 0; i < m; i++) {
			int ts = sc.nextInt();
			int id = sc.nextInt();
			orders.get(id).add(ts);
		}

		int count = 0;
		for (int id = 1; id <= n; id++) {
			List<Integer> list = orders.get(id);
			Collections.sort(list); // 必须是list,进行排序
			int currentTime = 0;
			int priority = 0;
			boolean inCache = false;
			int j = 0;
			while (j < list.size()) {
				int ts = list.get(j);
				int cnt = 0;
				// 处理连续时刻来多个订单
				while (j < list.size() && list.get(j) == ts) {
					cnt++;
					ts++;
				}

				int dt = ts - currentTime - 1; // 没人下单
				if (dt > 0) {
					int newPriority = Math.max(priority - dt, 0);
					if (inCache && newPriority <= 3)
						inCache = false;

					priority = newPriority;
				}

				priority += 2 * cnt;
				if (priority > 5) {
					inCache = true;
				} else if (priority <= 3) {
					inCache = false;
				}
				currentTime = ts;
			}

			// 处理最后时间段:处理完最后一个订单，可能还有空闲时间
			int dt = t - currentTime;
			if (dt > 0) {
				int newPriority = Math.max(priority - dt, 0);
				if (inCache && newPriority <= 3)
					inCache = false;
				priority = newPriority;
			}
			if (inCache) {
				count++;
			}
		}
		System.out.println(count);
	}
}
```

------

#### 8.  修改数组

<p align="center">
    <img src="https://hhhi21g.github.io/assets/img/Algorithm/lanqiao10/l5.png" alt="alt text" style="zoom:70%;" />
</p>

```java
import java.util.ArrayDeque;
import java.util.Deque;
import java.util.Scanner;

public class Main {

	public static void main(String[] args) {
		Scanner sc = new Scanner(System.in);
		int n = sc.nextInt();
		Deque<Integer> origin = new ArrayDeque<>();
		Deque<Integer> res = new ArrayDeque<>();
		for (int i = 0; i < n; i++) {
			int num = sc.nextInt();
			origin.offerLast(num);
		}
		for (int i = 0; i < n; i++) {
			int num = origin.pollFirst();
			while (res.contains(num)) {
				num++;
			}
			res.offerLast(num);
		}

		for (int i = 0; i < n; i++) {
			System.out.print(res.pollFirst());
			System.out.print(" ");
		}
	}
}
```

这里需要注意：**contains()**方法只是方便实现，但是**时间复杂度还是很高的**，因此上面代码对于大部分代码都出现运行超时(4/10); 这个问题可以通过引入一个**HashSet，查找复杂度为O(1)**

修改后，通过8/10

```java
package lanQ10;

import java.util.Scanner;

public class L8 {
	static int[] f = new int[2000000]; // 并查集，用于存储父节点

	public static void main(String[] args) {
		Scanner sc = new Scanner(System.in);
		int n = sc.nextInt();
		int[] origin = new int[n];

		for (int i = 1; i < f.length; i++) {
			f[i] = i; // 1. 将每个节点的父节点先初始化为自己
		}

		for (int i = 0; i < n; i++) {
			origin[i] = sc.nextInt();
		}

		for (int i = 0; i < origin.length; i++) {
			int k = find(origin[i]);
			origin[i] = k;
			f[origin[i]] = find(origin[i] + 1);
		}

		for (int i = 0; i < origin.length; i++) {
			System.out.print(origin[i] + " ");
		}

	}

	public static int find(int x) {
		if (x == f[x])
			return x; // 根节点的父节点初始化为了自己
		else {
			f[x] = find(f[x]); // 父节点设置为父节点的父节点，实现了路径压缩
			return f[x];
		}
	}
}
```

并查集知识补充：https://zhuanlan.zhihu.com/p/93647900

------

#### 9.  糖果

------

#### 10.  组合数问题
