[toc]

## 35 | Trie树：如何实现搜索引擎的搜索关键词提示功能？

### Trie 树

1.  Trie 树

	- 也叫“字典树”
	- 它是一个树形结构。
	- 它是一种专门处理**字符串匹配**的数据结构，用来解决在一组字符串集合中快速查找某个字符串的问题。
2.  本质

	- 利用字符串之间的**公共前缀**，将重复的前缀合并在一起
3.  构造图

	- ![img](imgs/280fbc0bfdef8380fcb632af39e84b32.jpg)

### 实现

1.  Trie 树主要有两个操作

	1. 一个是将字符串集合**构造**成 Trie 树。
	2. 另一个是在 Trie 树中**查询**一个字符串。

2.  如何存储一个 Trie 树？

	- Trie 树是一个多叉树
	- 借助散列表的思想，我们通过一个**下标与字符**一一映射的数组，来存储子节点的指针
		- ![img](imgs/f5a4a9cb7f0fe9dcfbf29eb1e5da6d35.jpg)

3.  代码

	- ```java
		
		public class Trie {
		  private TrieNode root = new TrieNode('/'); // 存储无意义字符
		
		  // 往Trie树中插入一个字符串
		  public void insert(char[] text) {
		    TrieNode p = root;
		    for (int i = 0; i < text.length; ++i) {
		      int index = text[i] - 'a';
		      if (p.children[index] == null) {
		        TrieNode newNode = new TrieNode(text[i]);
		        p.children[index] = newNode;
		      }
		      p = p.children[index];
		    }
		    p.isEndingChar = true;
		  }
		
		  // 在Trie树中查找一个字符串
		  public boolean find(char[] pattern) {
		    TrieNode p = root;
		    for (int i = 0; i < pattern.length; ++i) {
		      int index = pattern[i] - 'a';
		      if (p.children[index] == null) {
		        return false; // 不存在pattern
		      }
		      p = p.children[index];
		    }
		    if (p.isEndingChar == false) return false; // 不能完全匹配，只是前缀
		    else return true; // 找到pattern
		  }
		
		  public class TrieNode {
		    public char data;
		    public TrieNode[] children = new TrieNode[26];
		    public boolean isEndingChar = false;
		    public TrieNode(char data) {
		      this.data = data;
		    }
		  }
		}
		```

4.  时间复杂度分析

	- 构建　Trie 树，O(n)（n 表示所有字符串的长度和）
	- 查询，O(k)，k 表示要查找的字符串的长度。

### 空间复杂度

1.  Trie 树是非常耗内存的，用的是一种**空间换时间**的思路
2.  Trie 树是用**数组**来存储一个节点的子节点的指针。

### 与散列表、红黑树比较

1.  Trie 树对要处理字符串的要求
     1. 字符串中包含的字符集**不能太大**，如果字符集太大，那存储空间可能就会浪费很多。
    2. 要求字符串的**前缀重合比较多**
    3. 实现复杂
    4. Trie 树中用到了**指针**，所以，**对缓存并不友好**
2.  小结
    1.  散列表和红黑树比较适合**精确匹配**查找
    2.  Trie 树比较适合的是查找**前缀匹配**的字符串



