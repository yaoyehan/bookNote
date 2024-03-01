ST表

ST表是使用倍增的方式求出区间最值

```java
import java.util.Scanner;

public class Main {
    //lxy[i][j]表示以i为起点，扩展（包括自身）2^j个数字，这样就可以组成所有区间比如（1,10）可以用lxy[1][3]表示（1,8）和lxy[3][3]表示（3,10）组成，当然我们可以发现区间重叠，可是在求最值的情况下区间重叠并不影响最后的结果。
    static int[][] fmax = new int[50010][16];
    static int[][] fmin = new int[50010][16];

    public static void main(String[] args) {
        Scanner in = new Scanner(System.in);
        int n = in.nextInt(), m = in.nextInt();

        for (int i = 1; i <= n; i++) {
            fmax[i][0] = in.nextInt();
            fmin[i][0] = fmax[i][0];
        }

        for (int i = 1; i <= 16; i++) {
            for (int j = 1; j+(1<<i)<=n+1 ; j++) {
                //为什么是n+1是因为lxy[j][i]是从j到j+(1<<i)-1的位置刚好(1<<i)个数字，所以j+(1<<i)-1<=n就等于j+(1<<i)<=n+1
                fmax[j][i]=Math.max(fmax[j][i-1],fmax[j+(1<<(i-1))][i-1]);
                fmin[j][i]=Math.min(fmin[j][i-1],fmin[j+(1<<(i-1))][i-1]);
            }
        }

        for (int i = 1; i <= m; i++) {
            int l = in.nextInt(), r = in.nextInt();
            System.out.println(ST(l, r));
        }
    }

    static int ST(int l, int r) {
        int s = (int) (Math.log(r - l + 1) / Math.log(2)), x, y;
      //上面实际上是求log2(r-l+1)的结果，由公式loga(b)/loga(c)=logc(b)
        x = Math.max(fmax[l][s], fmax[r - (1 << s) + 1][s]);
        y = Math.min(fmin[l][s], fmin[r - (1 << s) + 1][s]);
        return x - y;
    }
}
```

