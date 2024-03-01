# dp合集

## 树形dp

使用链式前向星的方式

![image-20230418223720375](D:\typora\笔记\习题笔记\imagesr\image-20230418223720375.png)

```java
import java.util.Scanner;
import java.util.Arrays;

public class Main {

    static final int MAXN = 6005;
    static int n, cnt;
    static int[] head = new int[MAXN];
    static Edge[] e = new Edge[MAXN];

    static int[][] f = new int[MAXN][2];
    static int[] is_h = new int[MAXN];
    static int[] vis = new int[MAXN];

    static class Edge {
        int v, next;
    }
    static void addedge(int u, int v) { // u上司，v下属
        e[++cnt] = new Edge();
        e[cnt].v = v;
        e[cnt].next = head[u];
        head[u] = cnt;
    }
    static void calc(int k) {
        vis[k] = 1;
        for (int i = head[k]; i != 0; i = e[i].next) { // 枚举该结点的每个子结点
            if (vis[e[i].v] == 1) continue;
            calc(e[i].v);
            f[k][1] += f[e[i].v][0];
            f[k][0] += Math.max(f[e[i].v][0], f[e[i].v][1]); // 转移方程
        }
        return;
    }

    public static void main(String[] args) {
        Scanner scan = new Scanner(System.in);
        n = scan.nextInt();
        for (int i = 1; i <= n; i++) {
            f[i][1] = scan.nextInt();
        }
        for (int i = 1; i < n; i++) {
            int l = scan.nextInt();
            int k = scan.nextInt();
            is_h[l] = 1;
            addedge(k, l);
        }
        for (int i = 1; i <= n; i++) {
            if (is_h[i] != 1) { // 从根结点开始DFS
                calc(i);
                System.out.println(Math.max(f[i][1], f[i][0]));
                return;
            }
        }
    }
}

```

## 数位dp

什么情况下可以想到使用数位dp？

```
一般就是问你从1/0~n的所有数中满足某某条件的有多少个？如果说从x~n的话，那么还需要加一个downLimit作为下界，当然最简单的方法是f(n)-f(x-1)
```

```java
public class Solution{
    char s[];
    int memo[][];
    int len=0;
    public int countSpecialNumbers(int n) {
        s=Integer.toString(n).toCharArray();
        len=(int) Math.log10(n)+1;
        memo=new int[len][1<<10];
        for (int i = 0; i < len; i++) {
            Arrays.fill(memo[i],-1);
        }
        return f(0,0,true,false);
    }
    /*
    i:当前遍历到数位的第i位
    mask:记录已经选择的数字，索引为1即为该索引值已被选取
    isLimit:当前是否受到n的约束
    isNum:i前面的数位是否填了数字
     */
    int f(int i,int mask,boolean isLimit,boolean isNum){
        //条件符合结果拦截
        if(i==len){
            if()//拦截，具体题目具体分析
            return isNum?1:0;
        }
        //记忆数组结果拦截，注意这里需要!isLimit&&isNum，毕竟前面的不同选择同样影响着后面
        if(!isLimit&&isNum&&memo[i][mask]!=-1){//最好在上面给memo初始化-1，因为0也是结果，可以用来记录
            return memo[i][mask];
        }
        //选择
        //1、不选择值进入下一位
        //前提：前面的值没有选
        int res=0;
        if(!isNum){
            res = f(i + 1, mask, false, false);
        }
        //选择
        //上限：选择值的上限，取决于前面，如果前面被限制了，那么up最大不大于s[i],
        //下限：否则如果前面没有数字，那么选择1~9，否则前面有数字选择0~9
        int up=isLimit?s[i]-'0':9;
        for (int d = isNum?0:1; d <= up; d++) {
            //这里具体事件具体分析
           /* if((mask>>d&1)==0){
                res+=f(i+1,mask|1<<d,isLimit&&d==s[i]-'0',true);
            }
            */
        }
        //记忆一下
        if(!isLimit&&isNum){
            memo[i][mask]=res;
        }
        return res;
    }
}


```

```
mask：mask是否使用，取决于你是否需要记录下每一个出现的数字，比如每一位数字不能一样，那么我就需要使用mask进行记录，用于for选择可选分支时的筛选条件。
isNum：这个参数要不要下，取决于使用多个前导0是否与条件有冲突，比如每位数字不能一样，这时候如果不使用isNum表示前面没数字直接跳过，那么就需要用0表示这一位跳过，也就是0放在for的递归选择范围内，高位选择0意味着这一位不选，但是问题来了，每一位不能相同是在for循环选择可选分支时的约束条件，这个时候如果前面不选使用0代替，那么后面的选0就会受到影响。
总结：isNum是否出现取决于0是否能用来做不选时候的选择而不会影响到后面的选择。
isLimit:这个一般是要下的，主要用来记录下前面是否已经达到极限值，如果前面达到极限值比如极限是123，那么前面两位已经选取了12，那么第三位不能超过3，个人认为只要他给了条件时不能超过n，那么就需要isLimit。
```

## 概率dp

### DP 求概率

题目大意：袋子里有w只白鼠和b只黑鼠，公主和龙轮流从袋子里抓老鼠。谁先抓到白色老鼠谁就赢，如果袋子里没有老鼠了并且没有谁抓到白色老鼠，那么算龙赢。公主每次抓一只老鼠，龙每次抓完一只老鼠之后会有一只老鼠跑出来。每次抓的老鼠和跑出来的老鼠都是随机的。公主先抓。问公主赢的概率。

![image-20230426224523060](D:\typora\笔记\习题笔记\imagesr\image-20230426224523060.png)

```java
for (int i = 1; i <= w; i++) dp[i][0] = 1;  // 初始化
for (int i = 1; i <= b; i++) dp[0][i] = 0;
  for (int i = 1; i <= w; i++) {
    for (int j = 1; j <= b; j++) {  // 以下为题面概率转移
      dp[i][j] += (double)i / (i + j);
      if (j >= 3) {
        dp[i][j] += (double)j / (i + j) * (j - 1) / (i + j - 1) * (j - 2) /
                    (i + j - 2) * dp[i][j - 3];
      }
      if (i >= 1 && j >= 2) {
        dp[i][j] += (double)j / (i + j) * (j - 1) / (i + j - 1) * i /
                    (i + j - 2) * dp[i - 1][j - 2];
      }
    }
  }
  printf("%.9lf\n", dp[w][b]);
```

### DP 求期望

一个软件有s个子系统，会产生n种 bug。某人一天发现一个 bug，这个 bug 属于某种 bug 分类，也属于某个子系统。每个 bug 属于某个子系统的概率是1/s，属于某种 bug 分类的概率是 1/n。求发现 n种 bug，且s个子系统都找到 bug 的期望天数。

![image-20230426225441970](D:\typora\笔记\习题笔记\imagesr\image-20230426225441970.png)

```java
for (int i = n; i >= 0; i--) {
    for (int j = s; j >= 0; j--) {
      if (i == n && s == j) continue;
      dp[i][j] = (dp[i][j + 1] * i * (s - j) + dp[i + 1][j] * (n - i) * j +
                  dp[i + 1][j + 1] * (n - i) * (s - j) + n * s) /
                 (n * s - i * j);  // 概率转移
    }
  }
  printf("%.4lf\n", dp[0][0]);
```

```
概率dp最重要的一点是确定初始值，比如抓老鼠的题目，我们可以知道当袋子里没有白鼠算龙赢，公主赢的条件则是抓到白鼠，换句话来说轮到公主时袋子里全是白鼠算公主赢，那上面的条件换成，轮到公主时袋子里没有白鼠算龙赢即可。这就是初始条件，所以做题之前应该先找到初始条件，然后选择顺推还是逆推，比如这道题，我们知道f(0,j)=0,f(i,0)=1那么当i更大，j更大就需要进一步推出来，那么就是顺推，但是比如像之前求取期望的那道题，我们知道f(n,s),那么接下来就是n和s不断减小，那么就是逆推求期望，直到推到f(0,0)就是最后的结果。
```

## 状压DP（实际上就是把状态使用二进制压缩）

模板

```java
for (int i = 0; i < n; i++) {//当前行
            for (int j = 0; j <=cnt ; j++) {//当前行的所有状态
                for (int l = 0; l <cnt ; l++) {//上一行的所有状态
                    if(){//如果当前行和上一行不冲突
                        //f[i][j]+=f[i-1][l];看情况而定
                    }
                }
            }
        }
```

![image-20230429170432572](D:\typora\笔记\习题笔记\imagesr\image-20230429170432572.png)

![image-20230429170750654](D:\typora\笔记\习题笔记\imagesr\image-20230429170750654.png)

解释：当前状态左移一位如果与当前状态相与==0表示当前状态没有相邻的1。

![image-20230429171010561](D:\typora\笔记\习题笔记\imagesr\image-20230429171010561.png)

给定一张 n(n≤20) 个点的带权无向图，点从 0~n-1 标号，求起点 0 到终点 n-1 的最短Hamilton路径。 Hamilton路径的定义是从 0 到 n-1 不重不漏地经过每个点恰好一次。

输入：

```
4
0 2 1 3
2 0 2 1
1 2 0 1
3 1 1 0
```

输出：

```
4
```

![image-20230429212429766](D:\typora\笔记\习题笔记\imagesr\image-20230429212429766.png)

代码

```java
import java.util.*;
import java.io.*;

public class Main {
    static final int Inf = (int)1e9 + 7;
    static int[][] maze = new int[25][25];
    static int[][] dp = new int[1<<20|5][25];

    public static void main(String[] args) throws Exception {
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        int n = Integer.parseInt(br.readLine());
        for(int i = 0; i < n; i++) {
            StringTokenizer st = new StringTokenizer(br.readLine());
            for(int j = 0; j < n; j++) {
                maze[i][j] = Integer.parseInt(st.nextToken());
            }
        }
        Arrays.stream(dp).forEach(a -> Arrays.fill(a, Inf));
        dp[1][0] = 0;
        for(int i = 1; i < (1 << n); i++) {
            for(int j = 0; j < n; j++) {//这里起点终点谁先for都可以
                if((i & (1 << j)) > 0) { // 如果i的第j位是1，也就是如果经过点j
                    for(int k = 0; k < n; k++) {
                        if((i & (1 << k)) > 0) { // 如果i的第k位是1，也就是如果经过点k
                            dp[i][j] = Math.min(dp[i][j], dp[i^(1<<j)][k] + maze[k][j]);
                        }
                    }
                }
            }
        }
        System.out.println(dp[(1<<n)-1][n-1]);
    }
}
//这里之前有个疑问，就是如果当前状态是01111，0~3都拿了，4还没拿，那么dp[01111][0]表示从0开始，但是不对呀，假设我要从0开始拿这里就得0~3都走完后重新从0出发，走0-4这条路，但是这道题是从0出发到4，不能出现两条路，实际上这里确实维护了两条路，可问题是没有从0出发到3后再到0这条路，所以这条路的dp值非常大，就算+maze[01111][4]也没用，实际上只要是路走的通的，值都会很小，路走不通值就会很大，即便维护了也没关系。
//还有一点，就是这里为什么先for状态，再for终点，最后for起点，其实理论上先起点还是终点其实也没差别，但是为什么是先状态呢，明明我知道以0开头啊，问题是你不知道下一步走哪一个编号，可能有人会说，我for上当前行，for上当前行的所有状态，再for上所有上一行，再for上一行的所有状态，可以吗，只要把0开头的初始化，其他开头的给个大值，那么那些维护了也没有用，额，想想也可以，但是这样第一个for还有意义吗，没有，所以删除，所以我先维护第一行也就是以0开始，然后再按照上面的第二个for开始可以吗，额，拿除了最后一个for就和上面给的没什么差别了，那我们看看最后一个for，实际上就是for上一个编号还是for上一个状态的区别，可问题是我知道当前状态和当前编号我就可以推出上一个编号，实际上也是一个道理。
```

### 轮廓线DP

和状压dp的时间复杂度区别

![image-20230429221310882](D:\typora\笔记\习题笔记\imagesr\image-20230429221310882.png)

题目：

![image-20230429221225249](D:\typora\笔记\习题笔记\imagesr\image-20230429221225249.png)

使用状压dp解决：

![image-20230429221455693](D:\typora\笔记\习题笔记\imagesr\image-20230429221423561.png)

```java
public class 状态压缩dp_蒙德里安的梦想 {
    //N这里稍微开大了点是因为dp我们从1开始，自然就要开大于N
    //st[i]表示某一行的状态是否合法
    public static int N = 15, M = 1 << N;
    public static boolean[] st = new boolean[M];
	public static void main(String[] args) {
    Scanner in = new Scanner(System.in);
    while (true) {
        //接收每次的数据
        int n = in.nextInt(), m = in.nextInt();
        if (n == 0 && m == 0) break;
        //预处理,判断合并列的状态i是否合法
        //进制1表示横放，0表示竖放，如果不存在连续奇数个0，那么合法
        //1<<n表示2的n次方，乘法原理，对于这一列的每一行，都应该有0或1两种选法，所有可能性就是n个2(组合数)
        for (int i = 0; i < (1 << n); i++) {
            int count = 0;              //当前连续0的个数
            boolean is_valid = true;
            for (int j = 0; j<n; j++) {
                if (((i >> j) & 1) == 1) {      //如果为1
                    if ((count & 1) == 1) {     //并且连续0的个数是奇数，不合法
                        is_valid = false;
                        break;
                    }
                }
                else count++;   //不为1，说明是0
            }
            //这里是防止前置0的情况，比如4=0100，最后一次count++后for循环结束，但是最前端的0没有被算到
            if ((count & 1) == 1) is_valid = false;
            st[i] = is_valid;    //更新状态
        }

        //状态计算
        //记得每次都重置一下f，切记预处理，第0列不横放是一种合法方案，这个还是很难想到的
        long[][] f = new long[N][M];
        f[0][0] = 1;
        //三个循环分别是：枚举列i，枚举第i列的状态j，枚举第i-1列的状态k
        for (int i = 1; i<=m; i++) {
            for (int j = 0; j < (1 << n); j++) {
                for (int k = 0; k < (1 << n); k++) {
                    //判断两个条件（j和k不能有重叠的1）（j和k合并后必须合法，也就是不能有奇数个0，通过预处理的st判断）
                    //符合的话就状态转移即可，因为是方案数量，也就是dp中的count，所以是加起来
                    if ((j & k) == 0 && st[j | k]) {
                        f[i][j] += f[i-1][k];
                    }
                }
            }
        }

        System.out.println(f[m][0]); //最后一行肯定不可能出现有上半格竖着放的情况
    }
}
}
```
轮廓线dp：

![img](https://s3.ax1x.com/2021/02/01/ye3h3n.png)

注意轮廓线dp的起始位是左边格子，结束位是上面格子

```java
import com.sun.org.apache.bcel.internal.generic.NEW;

import java.util.Arrays;
import java.util.Scanner;

public class Solution {
    static int m,n,cur;
    static long[][] f;
    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);
        f = new long[2][(1 << 11)+1];//因为m<=11
        while ((n = sc.nextInt())>0&&(m = sc.nextInt())>0){
            Arrays.fill(f[0],0);
            Arrays.fill(f[1],0);
            cur=0;
            f[0][(1<<m)-1]=1;
            for (int i = 0; i < n; i++) {
                for (int j = 0; j < m; j++) {
                    cur^=1;
                    Arrays.fill(f[cur], 0);
                    for (int s = 0; s < 1<<m; s++) {//每一行的每一列为一格，该格的轮廓线可能性由上一格推出得到
                        if((s&(1<<(m-1)))>0){
       //最高位为1（也就是当前点的上面一格被占用了），那我不放或者横着放，这里是不放的情况，横着放在第三个条件
                            update(s,s<<1);
                        }
                        if(i>0&&(s&(1<<m-1))==0){
                            //当前点的上面一格为0（也就是没被占用），那我就可以竖着放
                            update(s,(s<<1)+1);
                        }
                        if(j>0&&((s&1)==0)&&(s&(1<<(m-1)))>0){
                            //当前列要>0,因为我要往左边倒，左边必须没被占用，上面比如被占用
                            update(s,(s<<1)+3);
                        }
                    }
                }
            }
            System.out.println(f[cur][(1<<m)-1]);//因为我要最后一行全被占用
        }
    }

    private static void update(int a, int b) {
        if((b&(1<<m))>0){//b最高位为1
            b-=1<<m;
        }
        f[cur][b]+=f[cur^1][a];
    }
}

```

换根dp

![image-20231122215222643](D:\typora\笔记\习题笔记\imagesr\image-20231122215222643.png)

```
输入：
3 3
1 1
1 2 3
输出：
2 3 3
```

解题思路：先求出每个节点有多少个子节点，和每个节点的到所有子节点的距离和，然后通过换根的方式求出以其他结点为根时该节点到其他节点的距离和，相当于拿当父节点为根时其到所有结点的距离和来维护当前节点到所有结点的距离和。

```java
import java.util.*;
public class Main {
    public static int n;
    public static ArrayList<Integer>[] child;
    public static int[] size;
    public static int[] sum;
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        n = scanner.nextInt();
        int m = scanner.nextInt();
        child = new ArrayList[n+1];
        size = new int[n+1];
        sum = new int[n+1];
        Arrays.setAll(child,e->new ArrayList<Integer>());
        for (int i = 2; i <= n; i++) {
            int par = scanner.nextInt();
            child[par].add(i);
        }
        dfs(1,-1);//维护sum[i]和size[i]
        dfs1(1,-1);
        for (int i = 0; i < m; i++) {
            int nextInt = scanner.nextInt();
            System.out.print(sum[nextInt]+" ");
        }
    }

    private static void dfs1(int root, int par) {
        if(par!=-1){
            sum[root]=sum[par]-size[root]+n-size[root];
        }
        for (Integer child : child[root]) {
            if(child!=par){
                dfs1(child,root);
            }
        }
    }

    private static int dfs(int root,int par) {
        int res=0;
        for (Integer child : child[root]) {
            if(child!=par){
                res+=dfs(child,root);
                size[root]+=size[child];
            }
        }
        res+=size[root];
        size[root]++;//加上自己
        sum[root]=res;
        return res;
    }
}
```

使用dp常见错误：

1、给dp初始-1，本来是为了说这个dp[i] [j]的方案不行，但是却会导致后面计算方案数从-1算起而不是从1算起，所以最好用0初始化。

