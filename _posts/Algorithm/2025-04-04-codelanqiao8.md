---
layout: post

title: Com8_第八届蓝桥杯大赛软件赛省赛Java大学A组

date: 2025-04-04 22:27:23 +0900

categories: [数据结构与算法]
tags: [刷题]
---

提交的时候一定要记得，把代码中**用于测试的输出去掉**😅

#### 1.  迷宫：最短路径问题

<p>
    <img src="https://hhhi21g.github.io/assets/img/Algorithm/lanqiao8/l0.png" alt="alt text" style="zoom:70%;" />
</p>



```java
package lanQ8;

import java.util.Arrays;
import java.util.LinkedList;
import java.util.Queue;

public class L1 {
	static int n = 30, m = 50;
	static String[] maze = { "01010101001011001001010110010110100100001000101010",
			"00001000100000101010010000100000001001100110100101", "01111011010010001000001101001011100011000000010000",
			"01000000001010100011010000101000001010101011001011", "00011111000000101000010010100010100000101100000000",
			"11001000110101000010101100011010011010101011110111", "00011011010101001001001010000001000101001110000000",
			"10100000101000100110101010111110011000010000111010", "00111000001010100001100010000001000101001100001001",
			"11000110100001110010001001010101010101010001101000", "00010000100100000101001010101110100010101010000101",
			"11100100101001001000010000010101010100100100010100", "00000010000000101011001111010001100000101010100011",
			"10101010011100001000011000010110011110110100001000", "10101010100001101010100101000010100000111011101001",
			"10000000101100010000101100101101001011100000000100", "10101001000000010100100001000100000100011110101001",
			"00101001010101101001010100011010101101110000110101", "11001010000100001100000010100101000001000111000010",
			"00001000110000110101101000000100101001001000011101", "10100101000101000000001110110010110101101010100001",
			"00101000010000110101010000100010001001000100010101", "10100001000110010001000010101001010101011111010010",
			"00000100101000000110010100101001000001000000000010", "11010000001001110111001001000011101001011011101000",
			"00000110100010001000100000001000011101000000110011", "10101000101000100010001111100010101001010000001000",
			"10000010100101001010110000000100101010001011101000", "00111100001000010000000110111000000001000000001011",
			"10000001100111010111010001000110111010101101111000" };

	static class State {
		int x, y;
		String path;

		State(int x, int y, String path) {
			this.x = x;
			this.y = y;
			this.path = path;
		}
	}

	// 方向数组:按D<L<R<U顺序
	static int[] dx = { 1, 0, 0, -1 };
	static int[] dy = { 0, -1, 1, 0 };
	static char[] dirChar = { 'D', 'L', 'R', 'U' };

	public static void main(String[] args) {
		int[][] dist = new int[n][m]; // 到某个点的最短步数
		String[][] pathMap = new String[n][m]; // 到某个点的最短路径，记录方向

		for (int i = 0; i < n; i++) {
			Arrays.fill(dist[i], Integer.MAX_VALUE); // 用一个值填充整个数组
			Arrays.fill(pathMap[i], null);
		}

		Queue<State> queue = new LinkedList<>();
		queue.add(new State(0, 0, "")); // 队列先进先出，实现BFS
		dist[0][0] = 0;
		pathMap[0][0] = "";

		while (!queue.isEmpty()) {
			State cur = queue.poll();
			int x = cur.x, y = cur.y;
			String path = cur.path;

			for (int d = 0; d < 4; d++) { // 尝试每种方向
				int nx = x + dx[d];
				int ny = y + dy[d];
				if (nx < 0 || nx >= n || ny < 0 || ny >= m)
					continue;
				if (maze[nx].charAt(ny) == '1')
					continue;

				int newDist = dist[x][y] + 1;
				String newPath = path + dirChar[d]; // 拼接路径

				if (newDist < dist[nx][ny]) {
					dist[nx][ny] = newDist;
					pathMap[nx][ny] = newPath;
					queue.add(new State(nx, ny, newPath));
				} else if (newDist == dist[nx][ny] && newPath.compareTo(pathMap[nx][ny]) < 0) {
					pathMap[nx][ny] = newPath;
					queue.add(new State(nx, ny, newPath));
				}
			}
		}
		System.out.println(pathMap[n - 1][m - 1]);
	}
}
```

**思路：**

- 题目要求：找出最短路径，且字典序最前；

- **BFS：**层级遍历，可以保证第一次到达终点一定是最短路径；

  DFS： 适合用于找所有解

- 使用**class**存储每一个点的状态：坐标值，及从起点到达的方式；

- 使用**三个数组表示方向**，从来没想过的思路；

- **两个辅组数组**：
  - dist:  到某个点的最短步数；
  - pathMap: 到某个点的最短路径；
- 使用**队列**表示还能继续处理的点，先进先出；
- **更新**逻辑:
  - 如果出界或是墙，跳过；
  - 如果`newDist < dist[nx][ny]` : 找到更短路径，更新；
  - 如果路径长度相同，`newPath < pathMap[nx][ny]`, 字典序更小，更新；

**注意：**

队列和栈可以统一使用`Deque<Integer> q = new ArrayDeque<>()`

**总结：**

刚好是没复习到的图论，很会考😁

------

#### 2.  9数算式

<p>
    <img src="https://hhhi21g.github.io/assets/img/Algorithm/lanqiao8/l1.png" alt="alt text" style="zoom:70%;" />
</p>



```java
package lanQ8;

import java.util.ArrayList;
import java.util.List;

public class L2 {

	static int cnt = 0;

	public static void main(String[] args) {
		for (int i = 1; i <= Math.sqrt((double) 98765432); i++) {
			List<Integer> left = new ArrayList<>();
			int temp = i;
			boolean flag = false;
			while (temp > 0) {
				int mid = temp % 10;
				temp /= 10;
				if (left.contains(mid) || mid == 0) {
					flag = true;
					break;
				}
				left.add(mid);
			}
			if (flag)
				continue;

			for (int j = 98765432; j > (int) Math.sqrt((double) 98765432); j--) {
				List<Integer> mid = new ArrayList<>();
				j = Math.min(j, 987654321 / i);
				int temp1 = j;
				boolean flag1 = false;
				if (i * j < 123456789)
					break;
				if (i * j > 987654321 || i * j < 123456789)
					continue;
				while (temp1 > 0) {
					int mid1 = temp1 % 10;
					temp1 /= 10;
					if (left.contains(mid1) || mid.contains(mid1) || mid1 == 0) {
						flag1 = true;
						break;
					}
					mid.add(mid1);
				}

				if (flag1)
					continue;
				else {
					List<Integer> right = new ArrayList<>();  // 动态数组
					long resNum = i * j;
					if (resNum < 123456789)
						continue;
					long temp2 = resNum;
					boolean flag2 = false;
					while (temp2 > 0) {
						int mid2 = (int) (temp2 % 10);
						temp2 /= 10;
						if (right.contains(mid2) || mid2 == 0) {
							flag2 = true;
							break;
						}
						right.add(mid2);
					}
					if (flag2)
						continue;
					else {
						cnt++;
						System.out.println(i + "*" + j + "=" + resNum);
					}
				}
			}
		}
		System.out.println(cnt);
	}
}
```

唯一AC，建三个动态数组，用来判断是否有重复的数字；

把无法进行下去的条件筛的干净一点，即可快速获得结果

------

#### 3.  魔方状态

<p>
    <img src="https://hhhi21g.github.io/assets/img/Algorithm/lanqiao8/l2.png" alt="alt text" style="zoom:70%;" />
</p>



> 229878

**思路：(根本看不懂，也并没有找到很完整的解题思路）**

<p>
    <img src="https://hhhi21g.github.io/assets/img/Algorithm/lanqiao8/l3.png" alt="alt text" style="zoom:70%;" />
</p>

<p>
    <img src="https://hhhi21g.github.io/assets/img/Algorithm/lanqiao8/l4.jpeg" alt="alt text" style="zoom:70%;" />
</p>



------

#### 4.  方格分割

<p>
    <img src="https://hhhi21g.github.io/assets/img/Algorithm/lanqiao8/l5.png" alt="alt text" style="zoom:70%;" />
</p>



```java
package lanQ8;

public class L4 {
	static int[][] v = new int[7][7];
	static int[] dx = { 0, 1, 0, -1 };
	static int[] dy = { 1, 0, -1, 0 };
	static int cut = 0;

	static void dfs(int x, int y) {
		if (x == 0 || x == 6 || y == 0 || y == 6) {
			cut++;
			return;
		}

		for (int i = 0; i < 4; i++) {
			int tx = x + dx[i];
			int ty = y + dy[i];

			if (v[tx][ty] == 0) {
				v[tx][ty] = 1;
				v[6 - tx][6 - ty] = 1;
				dfs(tx, ty);
				v[tx][ty] = 0;
				v[6 - tx][6 - ty] = 0;
			}
		}
	}

	public static void main(String[] args) {
		v[3][3] = 1;
		dfs(3, 3);
		System.out.print(cut / 4);  // 旋转有4种
	}
}
```

**思路：**

- 不考虑格子，而是考虑边框，那么形状其实是7*7；
- 由于本题要求找出所有路径，就像第1题所说，使用**dfs**;
- 中心点是一定要经过的，`v[3][3]`设置为1：
  - 向边界探索，这里也同第一题，试探上下左右四个方向；
  - 走到边界，种类加1；

- 试探体现在：bfs递归调用完，要把当前节点的状态设置为0；

**总结：**

又考了dfs, 哪里不会考哪里😅

------

#### 5.  正则问题

<p>
    <img src="https://hhhi21g.github.io/assets/img/Algorithm/lanqiao8/l6.png" alt="alt text" style="zoom:70%;" />
</p>



```java
package lanQ8;

import java.util.Scanner;

public class L5_2 {

	static int pos = -1; // 下标
	static String s = null;

	public static void main(String[] args) {
		Scanner sc = new Scanner(System.in);
		s = sc.nextLine();
		System.out.println(dfs());
		sc.close();
	}

	private static int dfs() {
		int current = 0; // 当前积攒的x数目
		int max = 0; // 最终x的最大个数
		while (pos < s.length() - 1) {
			pos++;
			if (s.charAt(pos) == '(') {
				current += dfs(); // 去到递归下一层
			} else if (s.charAt(pos) == 'x') {
				current += 1; // 遇到x，积攒的x数目就加一
			} else if (s.charAt(pos) == '|') {
				max = Math.max(current, max);
				current = 0; // 当前x处理完，归零
			} else { // 遇到),回到上一层
				break;
			}
		}
		return Math.max(current, max);
	}
}
```

本来用的算术表达式的计算方法，即使用符号栈、数字栈，只对了一半没法AC

**思路：**

- 设置两个状态量：
  - current: 积攒的还未参与过运算的x数目；
  - max: 每次都保留最大值，可作为结果；

- 每一组括号视为递归的一层

**总结：**

妙啊！！

------

#### 6.  包子凑数

<p>
    <img src="https://hhhi21g.github.io/assets/img/Algorithm/lanqiao8/l7.png" alt="alt text" style="zoom:70%;" />
</p>



```java
// 个人AC代码
import java.util.Scanner;

public class Main {

	public static void main(String[] args) {
		Scanner sc = new Scanner(System.in);
		int n = sc.nextInt();
		int[] a = new int[n];
		int min = 10000;
		boolean flag = false;
		for (int i = 0; i < n; i++) {
			a[i] = sc.nextInt();
			if (a[i] % 2 != 0)
				flag = true;
			if (a[i] < min)
				min = a[i];
		}

		if (!flag) {
			System.out.println("INF");
			return;
		}

		int[] dp = new int[200000];
		for (int i = 0; i < min; i++) {
			dp[i] = 0;
		}
		for (int i = 0; i < n; i++) {
			dp[a[i]] = 1;
		}

		for (int i = min; i < 200000; i++) {
			for (int j = 0; j < n; j++) {
				if (i >= a[j]) {
					dp[i] += dp[i - a[j]];
				}
			}
		}

		int cnt = 0;
		for (int i = 1; i < 200000; i++) {
			if (dp[i] == 0) {
				cnt++;
			}
		}
    if(cnt > 5000) System.out.println("INF");
		else System.out.println(cnt);
	}
}
```

**思路：**

- 很明显的dp题，这里**dp[i]表示买i个包子的凑数法**，但并没想到状态转移方程；

- 想到本题只是判断是否能够凑出，并未要求给出正确方案数，因此想出以下错误方程：

  `dp[i] += dp[i - a[j]]`

- 这样只判断不为0，即表示能够凑出；

- 本代码的问题是，不知道要遍历到多少个包子，才能把所有凑不出来的都遍历到。(可以设为1e7)

------

#### 7.  分巧克力

<p>
    <img src="https://hhhi21g.github.io/assets/img/Algorithm/lanqiao8/l8.png" alt="alt text" style="zoom:70%;" />
</p>



```java
// 个人AC代码
package lanQ8;

import java.util.Scanner;

public class L7 {

	public static void main(String[] args) {
		Scanner sc = new Scanner(System.in);
		int n = sc.nextInt();
		int k = sc.nextInt();
		int[][] a = new int[n][2];
		int max = 0;
		for (int i = 0; i < n; i++) {
			a[i][0] = sc.nextInt();
			a[i][1] = sc.nextInt();
			if (a[i][0] > max)
				max = a[i][0];
			if (a[i][1] > max)
				max = a[i][1];
		}
		for (int i = max; i >= 1; i--) {
			int sum = 0;
			for (int j = 0; j < n; j++) {
				int hCnt = a[j][0] / i;
				int wCnt = a[j][1] / i;
				sum = sum + hCnt * wCnt;
				if (sum >= k) {
					System.out.println(i);
					break;
				}
			}
			if (sum >= k) {
				break;
			}
		}
	}
}
```

个人感觉最简单的一道题，看来不一定越后面的题越难

**放一个二分查找的代码**，前面的太暴力了

```java
package lanQ8;

import java.util.Scanner;

public class L7 {

	public static void main(String[] args) {
		Scanner sc = new Scanner(System.in);
		int n = sc.nextInt();
		int k = sc.nextInt();
		int[][] a = new int[n][2];
		int max = 0;
		for (int i = 0; i < n; i++) {
			a[i][0] = sc.nextInt();
			a[i][1] = sc.nextInt();
			if (a[i][0] > max)
				max = a[i][0];
			if (a[i][1] > max)
				max = a[i][1];
		}

		int left = 1, right = max;
		int ans = 0;

		while (left <= right) {
			int mid = (left + right) / 2;
			if (canCut(a, n, k, mid)) {
				ans = mid;
				left = mid + 1;
			} else {
				right = mid - 1;
			}
		}

		System.out.print(ans);
	}

	private static boolean canCut(int[][] a, int n, int k, int len) {
		int count = 0;
		for (int i = 0; i < n; i++) {
			count += (a[i][0] / len) * (a[i][1] / len);
			if (count >= k)
				return true;
		}
		return false;
	}
}
```

------

#### 8.  油漆面积

<p>
    <img src="https://hhhi21g.github.io/assets/img/Algorithm/lanqiao8/l9.png" alt="alt text" style="zoom:70%;" />
</p>

```java
package lanQ8;

import java.util.Scanner;

public class L8 {

	public static void main(String[] args) {
		Scanner sc = new Scanner(System.in);
		boolean[][]a = new boolean[10000][10000];
		int n = sc.nextInt();
		int sum = 0;
		
		for(int i = 0;i<n;i++) {
			int x1 = sc.nextInt();
			int y1 = sc.nextInt();
			int x2 = sc.nextInt();
			int y2 = sc.nextInt();
			
			if(x1 > x2) {
				int t = x1;
				x1 = x2;
				x2 = t;
			}
			if(y1 > y2) {
				int t = y1;
				y1 = y2;
				y2 = t;
			}
			
			for(int j = x1;j<x2;j++) {
				for(int k = y1;k<y2;k++) {
					if(a[j][k] == false) sum ++;
					a[j][k] =true;
				}
			}
		}
		System.out.print(sum);
	}
}
```

第二简单的一题，注意是算面积不是算点。

------

