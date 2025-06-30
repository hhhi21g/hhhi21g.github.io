---
layout: post

title: Com11_第十一届蓝桥杯大赛软件赛省赛Java大学A组

date: 2025-04-07 13:19:22 +0900

categories: [数据结构与算法]
tags: [刷题]
---

### 第十一届蓝桥杯大赛软件赛省赛Java大学A组

#### 1.  门牌制作

<p>
    <img src="https://hhhi21g.github.io/assets/img/Algorithm/lanqiao11/l0.png" alt="alt text" style="zoom:70%;" />
</p>



> 624

------

#### 2.  既约分数

<p>
    <img src="https://hhhi21g.github.io/assets/img/Algorithm/lanqiao11/l1.png" alt="alt text" style="zoom:70%;" />
</p>



```java
package lanQ11;

public class l2 {

	public static void main(String[] args) {
		int res = 0;
		for (int i = 1; i <= 2020; i++) {
			for (int j = 1; j <= 2020; j++) {
				if (gcd(i, j) == 1) {
					System.out.println(i + " " + j);
					res++;
				} else
					continue;
			}
		}
		System.out.print(res);
	}

	static int gcd(int a, int b) {
		if (a == 1 || b == 1)
			return 1;
		if (b == 0)
			return a;
		return gcd(b, a % b);
	}
}
```

> 2481215

回想一下最大公约数求法即可

------

#### 3.  蛇形填数

<p>
    <img src="https://hhhi21g.github.io/assets/img/Algorithm/lanqiao11/l3.png" alt="alt text" style="zoom:70%;" />
</p>



```java
package lanQ11;

public class l3 {

	public static void main(String[] args) {
		int[][] res = new int[200][200];
		boolean turn = false; // false: 向右上方向走
		int layer = 1;
		int num = 1;
		res[1][1] = 1;
		boolean flag = false;
		for (int i = 1, j = 2;;) {
			turn = !turn;
			layer++;
			if (flag)
				break;
			for (num = num + 1; num <= (1 + layer) * layer / 2; num++) {
				res[i][j] = num;
				if (num < (1 + layer) * layer / 2) {
					if (turn) {
						i++;
						j--;
					} else {
						i--;
						j++;
					}
				}
				if (i == 20 && j == 20)
					flag = true;
			}
			num--;
			if (turn)
				i++;
			else
				j++;

		}

		for (int i = 1; i <= 20; i++) {
			for (int j = 1; j <= 20; j++) {
				System.out.print(res[i][j] + " ");
			}
			System.out.println();
		}
		System.out.printf("I'm res:%d", res[20][20]);
	}
}
```

> 761

------

#### 4.  七段码

<p>
    <img src="https://hhhi21g.github.io/assets/img/Algorithm/lanqiao11/l2.png" alt="alt text" style="zoom:70%;" />
</p>



```python
import os
import sys

l1 = 7 #只亮一个管
l2 = 10  #ab, af, fg, bg, gc, ge, ed, cd, fe, bc
l3 = 16 #1abg, 2afg, 3fab,4fgb,5gcd,6ged,7edc,8egc,9afe,10abc,11fed,12bcd,13fgc,14fge,15bgc,16bge
l4 = 20 #1abgf,2gcde,3abcd,4afed,5afgc,6afge,7abgc,8abge,9fabe,10fabc,11fgbe,12fgbc,13gecf,14gecb,15gedf,16gedb,17gcdf,18gcdb,19edcf,20edcb
l5 = 19 #abgfe1,abgfc2,gedcb3,gedcf4,abcde5,bcdef6,cdefa7,defab8,efabc9,fabcd10,afgce11,afgcd12,afged13,abgcd14,abgce15,abged16,fgbed17,fgbec18,fbgcd19
l6 = 7 #abcdef1,abgfed2,abgfcd3,abgfec4,dcgefa5,dcgeba6,dcgefb7
l7 = 1
s = l1+l2+l3+l4+l5+l6+l7
print(s)
```

------

#### 5.  成绩分析

<p>
    <img src="https://hhhi21g.github.io/assets/img/Algorithm/lanqiao11/l4.png" alt="alt text" style="zoom:70%;" />
</p>



```java
package lanQ11;

import java.util.Scanner;

public class l5 {

	public static void main(String[] args) {
		Scanner sc = new Scanner(System.in);
		int n = sc.nextInt();
		int max = -1;
		int min = 101;
		int sum = 0;
		for (int i = 0; i < n; i++) {
			int score = sc.nextInt();
			if (score > max)
				max = score;
			if (score < min)
				min = score;
			sum += score;
		}
		System.out.println(max);
		System.out.println(min);
		System.out.printf("%.2f", 1.0 * sum / n);

		sc.close();
	}
}
```

------

#### 6.  回文日期

<p>
    <img src="https://hhhi21g.github.io/assets/img/Algorithm/lanqiao11/l5.png" alt="alt text" style="zoom:70%;" />
</p>



```java
package lanQ11;

import java.util.Scanner;

public class l6 {

	public static void main(String[] args) {
		Scanner sc = new Scanner(System.in);
		int s = sc.nextInt();
		int e = sc.nextInt();
		int sy = s / 10000;
		int sm = (s - 10000 * sy) / 100;
		int sd = s - 100 * sm - 10000 * sy;
		int ey = e / 10000;
		int em = (e - 10000 * ey) / 100;
		int ed = e - 100 * em - 10000 * ey;
		boolean flag = false;
		int cnt = 0;
		int m = sm;
		int d = sd;
		for (int y = sy;; y++) {
			int onem = y % 100 / 10;
			int tenm = y % 10;
			int oned = y / 1000;
			int tend = y / 100 % 10;
			if (y > ey)
				flag = true;

			if (flag)
				break;
			for (m = tenm * 10 + onem; m <= Math.min(12, tenm * 10 + onem); m++) {

				if (flag)
					break;
				int maxd = 0;
				switch (m) {
				case 1:
				case 3:
				case 5:
				case 7:
				case 8:
				case 10:
				case 12:
					maxd = 31;
					break;
				case 4:
				case 6:
				case 9:
				case 11:
					maxd = 30;
					break;
				}
				if (m == 2) {
					if (y % 4 == 0 && y % 100 != 0)
						maxd = 29;
					else if (y % 400 == 0)
						maxd = 29;
					else
						maxd = 28;
				}
				for (d = tend * 10 + oned; d <= Math.min(tend * 10 + oned, maxd); d++) {
					cnt++;
					if (y < ey)
						;
					else if (y == ey && m < em)
						;
					else if (y == ey && m == em && d <= ed)
						;
					else
						flag = true;

					if (flag)
						break;
				}

				if (flag)
					break;
			}

			if (flag)
				break;
		}

		System.out.print(cnt);
	}
}
```

------

#### 7.  子串分值

<p>
    <img src="https://hhhi21g.github.io/assets/img/Algorithm/lanqiao11/l6.png" alt="alt text" style="zoom:70%;" />
</p>



```java
// 运行超时
package lanQ11;

import java.util.Arrays;
import java.util.Scanner;

public class l7 {

	static int res = 0;
	static int[] times = new int[26];

	public static void main(String[] args) {
		Scanner sc = new Scanner(System.in);
		String s = sc.next();
		for (int i = 1; i <= s.length(); i++) {
			for (int j = 0; j - 1 + i < s.length(); j++) {
				String cs = s.substring(j, j + i);
				doSum(cs);
			}
		}
		System.out.print(res);
	}

	static void doSum(String cs) {
		Arrays.fill(times, 0);
		for (int i = 0; i < cs.length(); i++) {
			times[cs.subSequence(i, i + 1).charAt(0) - 'a']++;
		}
		int single = 0;
		for (int i = 0; i < 26; i++) {
			if (times[i] == 1)
				single++;
		}
		res += single;
	}
}
```

------

#### 8.  字串排序

<p>
    <img src="https://hhhi21g.github.io/assets/img/Algorithm/lanqiao11/l7.png" alt="alt text" style="zoom:70%;" />
</p>

```java
package lanQ11;

import java.util.ArrayDeque;
import java.util.Deque;
import java.util.Scanner;

public class l8 {

	public static void main(String[] args) {
		Scanner sc = new Scanner(System.in);
		int v = sc.nextInt();
		if (v == 0) {
			System.out.print("a");
			return;
		}
		int len = 0;
		for (int i = (int) Math.sqrt(v * 2.0);; i++) {
			if (i * (i - 1) >= v * 2 && (i - 1) * (1 - 2) < v * 2) {
				len = i;
				break;
			}
		}
		Deque<Integer> q = new ArrayDeque<>();
		int offerNum = 1;
		int size = 0;
		if (v <= 351) {
			for (int cnt = len * (len - 1) / 2; cnt > v; cnt--) {
				q.offerFirst(offerNum);
				q.offerFirst(offerNum);
				size += 2;
				offerNum++;
			}

			for (; size < len; size++) {
				q.offerFirst(offerNum++);
			}
		} else {
			int mod = len % 26;
			int div = len / 26;
			int addCnt = div;
			int nowv = len * (len - 1) / 2 - (v - 26);
			for (; size < len;) {
				addCnt = div;
				if (offerNum <= mod)
					addCnt = div + 1;
				if (nowv > v)
					addCnt = div + 1;
				for (int i = 0; i < addCnt; i++) {
					q.offerFirst(offerNum);
				}
				size += addCnt;
				offerNum++;
			}
		}
		for (int i = 0; i < len; i++) {
			char m = (char) ('a' + q.pollFirst() - 1);
			System.out.print(m);
		}
	}
}
```

------

#### 9.  荒岛探测

------

