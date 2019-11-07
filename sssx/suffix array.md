# SUFFIX ARRAY
## 构建后缀数组
- 后缀数组的概念：名次为`i`的后缀在`S`中的起始位置
```C
int n,k;
int Rank[MAXN+1];
int tmp[MAXN+1];

//compare(rank[i], rank[i+k]) and (rank[j], rank[j+k])
//比较拼接体的两部分，第一块比不出来则比第二块
bool compare_sa(int i, int j) {
	if (Rank[i] != Rank[j]) return Rank[i] < Rank[j];
	else {
		int ri = i + k <= n ? Rank[i+k] : -1;
		int rj = j + k <= n ? Rank[j+k] : -1;
		return ri < rj; 
	}
}

void construct_sa(string S, int *sa) {
	n = S.length();	
	for (int i = 0; i <= n; i++) {
		sa[i] = i;                  //初始未排序，i 字符开始，位置就是 i
		Rank[i] = i < n ? S[i] : -1; 
        // S[i] 表示所有1-后缀的排名
	}
    //k每次扩大两倍，体现倍增思想
	for(k = 1; k <= n; k *=2) {
		sort(sa, sa + n + 1, compare_sa);
        //对更新了rank的sa重新排序
		tmp[sa[0]] = 0;
		for (int i = 1; i <= n; i++) {
			tmp[sa[i]] = tmp[sa[i - 1]] + (compare_sa(sa[i - 1], sa[i]) ? 1 : 0);
		}
        //从排名最小的位置开始，如果发现跟下一个的排名一样大，则+1（不一样大也+1）
		for (int i = 0; i <= n; i++) {
			Rank[i] = tmp[i];
		}
	}
}
```
- 步长为 `O(logn)`. 普通的 `sort` 排序复杂度为 `O(nlogn^2)`，加入基数排序后复杂度变为 `O(nlogn)`
## 构建 height 数组
- LCP(Longest Common Prefix)
- height 数组定义：`height[i]` 为排名 `i` 和排名在它之后 `i+1` 那个字符串的最长公共前缀。
- H 数组定义：位置为 `i` 的后缀和排名在它前一个后缀的最长公共前缀。
- H 定理：`H[i] >= H[i-1] - 1`，利用这个求 LCP
```C
void construct_lcp(string S, int *sa, int *lcp) {
	int n = S.length();
	for (int i = 0; i <= n; i++) Rank[sa[i]] = i;
	int h = 0;
	lcp[0] = 0;
	for (int i = 0; i < n; i++) {
		int j = sa[Rank[i] - 1];
		//因为 Rank[n] 排在第一位，此处不会越界
        //取排名在 rank[i] 前一位的 j
		if (h > 0) h--;             //由于 H 定理，只需要从 H[i-1]-1 开始
		for(; j + h < n && i + h < n; h++) {
			if (S[j + h] != S[i + h]) break;
		}
		lcp[Rank[i] - 1] = h;
	}
}
```
## Common Substrings
### 计算两个字符串所有长度大于等于 K 的公共子串的数目
- 单调栈：分两个串的情况讨论。计算 B 串对 A 串的影响时，如果遇到 B 串，其贡献为 `lcp[i] - K + 1 `；如果遇到 A 串，则增加之前累计的贡献。但是贡献的值随着height变化而变化，比如从 abaaa abaac 进入到 acccc 时，之前累积的贡献对该串不再起作用。因此需减少其贡献。
- 为什么扫两遍：第一次是前面的 B 对后面的 A 的影响，第二次前面的 A 对后面的 B 的影响。二者并不重复。
- 但不是很懂为什么这跟计算任意 i,j 之间的最长公共前缀（用 `min height` ）有关系。
