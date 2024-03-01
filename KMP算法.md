```
kmp算法主要解决的是由一个字符串匹配另一个字符串问题。
```

```
kmp算法的next数组则可以用来计算该str包含多少个sub（连续）或者是否包含sub等，但必须是连续的比如acb和ab就不算acb包含ab
```

```java
public class KMP {
    public static int KMP(String Str,String Sub,int pos){//pos指的是从哪里开始
        if (Str == null || Sub == null){
            return -1;
        }
        if (pos >= Str.length() || pos < 0){
            return -1;
        }
        int i = pos;
        int j = 0;
        int[] next = new int[Sub.length()];
        getNext(next,Sub);
        while(i < Str.length() && j < Sub.length()){//用while，不能用for
            if (j == -1 || Str.charAt(i) == Sub.charAt(j)){
                i++;//只有匹配成功或者k==-1才i++
                j++;
            }else {
                j = next[j];
            }
        }
        if (j >= Sub.length()){
            return i-j;//返回原串包含子串的开始下标
        }else {
            return -1;
        }
        //如果要求出str包含了多少个sub，返回下标
//         if (j == patternStr.length()) {
             //超出了最大索引值
//             firstIndexList.add(i - j);//保存下标
 //            j = next[j - 1];
          }
    }
    public static void getNext(int[] next,String sub){
        next[0] = -1;
        if(next.length==1){
            return;
        }
        next[1] = 0;
        int i = 2;//必须从第三位开始
        int k = 0;
        //由于我们设定好了next数组前两位的值
        //所以我们使用我们上面所讲到的逻辑就可以很好的完成我们的填充
        while(i < next.length){//必须while，不能使用for
            if (k ==- 1 || sub.charAt(k) == sub.charAt(i-1)){//i-1是因为next数组是找前面的数有多少位前缀和从0开始的字段相同
                next[i] = k+1;
                i++;//只有匹配成功或者k==-1才i++
                k++;
            }else {
                k = next[k];
            }
        }
    }
}
```

![img](D:\typora\笔记\习题笔记\imagesr\7abe875d6fd64e2bbbbbf4366e83d5e9.png)

实际上next[j]维护的是0~j-1位置的值，有人可能会问，Why？因为这样写别人来和你比较，你和它不一样，你就可以把坐标指向上一个前缀一样的位置，比如上面坐标为10的位置为c，那么假设11是d，11维护的是0~10的串，那么需要比较下标为10的位置和下标为5的位置，a和c不一样，那么由于索引5之前的值都维护过，我们可以很快找到同样前缀的下标2，只需要和下标2位置的比较就可以了，当然比较不成功继续以下标2的next[2]的下标0比较，简单来说就是0~4的串和6~9的串相同，本来我是要比较5和10下标所在的字符是否相同，如果不相同，那我的前缀肯定要重新规划，也就是不要从0~4，6~9的前缀，而是找一个新的前缀，这个前缀的位置自然就是next标记所在位置的前缀与10的前缀相同，都是ab，已经在之前计算next就已经计算好了，所以说next[i]记录的就是i的前缀和next[i]的前缀一样。

![image-20230404213302586](D:\typora\笔记\习题笔记\imagesr\image-20230404213302586.png)

这张图和上面的区别不大，变化的知识下标是从1开始，且所在next[i]等于前面前缀相同的数量+1而已，相当于i和next[i]都比上面大1，这样相互抵消其实不影响。

kmp算法超进化-》ac自动机算法

```java
//AcNode结点结构
class AcNode
{
    //孩子节点用HashMap存储，能够在O(1)的时间内查找到，效率高
	Map<Character,AcNode> children=new HashMap<>();
	AcNode failNode;
    //使用set集合存储字符长度，防止敏感字符重复导致集合内数据重复
	Set<Integer> wordLengthList = new HashSet<>();
}
//字典树构建
public static void insert(AcNode root,String s){
		AcNode temp=root;
		char[] chars=s.toCharArray();
		for (int i = 0; i < s.length(); i++) {
				if (!temp.children.containsKey(chars[i])){ //如果不包含这个字符就创建孩子节点
					temp.children.put(chars[i],new AcNode());
				}
				temp=temp.children.get(chars[i]);//temp指向孩子节点
		}
		temp.wordLengthList.add(s.length());//一个字符串遍历完了后，将其长度保存到最后一个孩子节点信息中
	}
//构建fail指针
public static void buildFailPath(AcNode root,int n,String[] s){
		//第一层的fail指针指向root,并且让第一层的节点入队，方便BFS
		Queue<AcNode> queue=new LinkedList<>();
		Map<Character,AcNode> childrens=root.children;
		Iterator iterator=childrens.entrySet().iterator();
		while(iterator.hasNext()){
			Map.Entry<Character, AcNode> next = (Map.Entry<Character, AcNode>) iterator.next();
			queue.offer(next.getValue());
			next.getValue().failNode=root;
		}
		//构建剩余层数节点的fail指针,利用层次遍历
		while(!queue.isEmpty()){
			AcNode x=queue.poll();
			childrens=x.children; //取出当前节点的所有孩子
			iterator=childrens.entrySet().iterator();
			while(iterator.hasNext()){   
				Map.Entry<Character, AcNode> next = (Map.Entry<Character, AcNode>) iterator.next();
				AcNode y=next.getValue();  //得到当前某个孩子节点
				AcNode fafail=x.failNode;  //得到孩子节点的父节点的fail节点
                //如果 fafail节点没有与 当前节点父节点具有相同的转移路径，则继续获取 fafail 节点的失败指针指向的节点，将其赋值给 fafail
				while(fafail!=null&&(!fafail.children.containsKey(next.getKey()))){
					fafail=fafail.failNode;
				}
                //回溯到了root节点，只有root节点的fail才为null
				if (fafail==null){
					y.failNode=root;
				}
				else {
                    //fafail节点有与当前节点父节点具有相同的转移路径，则把当前孩子节点的fail指向fafail节点的孩子节点
					y.failNode=fafail.children.get(next.getKey());
				}
                //如果当前节点的fail节点有保存字符串的长度信息，则把信息存储合并到当前节点
				if (y.failNode.wordLengthList!=null){
						y.wordLengthList.addAll(y.failNode.wordLengthList);
					}
					queue.offer(y);//最后别忘了把当前孩子节点入队
			}
		}

	}
//查找
	public static void query(AcNode root,int n,String s){
		AcNode temp=root;
		char[] c=s.toCharArray();
		for (int i = 0; i < s.length(); i++) {
            //如果这个字符在当前节点的孩子里面没有或者当前节点的fail指针不为空，就有可能通过fail指针找到这个字符
            //所以就一直向上更换temp节点
			while(temp.children.get(c[i])==null&&temp.failNode!=null){
				temp=temp.failNode;
			}
            //如果因为当前节点的孩子节点有这个字符，则将temp替换为下面的孩子节点
			if (temp.children.get(c[i])!=null){
				temp=temp.children.get(c[i]);
			}
            //如果temp的failnode为空，代表temp为root节点，没有在树中找到符合的敏感字，故跳出循环，检索下个字符
			else{
				continue;
			}
            //如果检索到当前节点的长度信息存在，则代表搜索到了敏感词，打印输出即可
			if (temp.wordLengthList.size()!=0){
				handleMatchWords(temp,s,i);
			}
		}
	}

	//利用节点存储的字符长度信息，打印输出敏感词及其在搜索串内的坐标
	public static void handleMatchWords(AcNode node, String word, int currentPos)
	{
		for (Integer wordLen : node.wordLengthList)
		{
			int startIndex = currentPos - wordLen + 1;
			String matchWord = word.substring(startIndex, currentPos + 1);
			System.out.println("匹配到的敏感词为:"+matchWord+",其在搜索串中下标为:"+startIndex+","+currentPos);
		}
	}

```

