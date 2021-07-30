# ST表

## 简介

ST 表是用于解决**可重复贡献问题**的数据结构。英文写作:``Sparse Table``

> #**可重复贡献问题**
**可重复贡献问题**是指对于运算$opt$，满足$xoptx=x$，则对应的区间询问就是一个可重复贡献问题。例如，最大值有 $max(x,x)=x$，gcd 有 $gcd(x,x)=x$，所以 RMQ 和区间 GCD 就是一个可重复贡献问题。像区间和就不具有这个性质，如果求区间和的时候采用的预处理区间重叠了，则会导致重叠部分被计算两次，这是我们所不愿意看到的。另外，$opt$ 还必须满足结合律才能使用 ST 表求解。
>>RMQ:Range Maximum/Minimum Query,表示区间最大（最小）值。其解决方法有多种。

ST表基于倍增思想，支持静态区间处理(**支持静态预处理不支持动态维护**)，离线预处理时间复杂度 θ（nlogn），在线查询时间θ（1），如果按照$2^{i}$倍增的话复杂度可以降至θ（logn），依旧没有线段树优。
然而，我们发现 $max(x,x)=x$，也就是说，区间最大值是一个具有“**可重复贡献**”性质的问题。即使用来求解的预处理区间有重叠部分，只要这些区间的**并**是所求的区间，最终计算出的答案就是正确的。

如果手动模拟一下，可以发现我们能使用至多两个预处理过的区间来覆盖询问区间，也就是说询问时的时间复杂度可以被降至 θ（1），在处理有大量询问的题目时十分有效。

## 基本实现

假设以$2^{i}$倍增，st[i][j]表示区间$[i,i+2^{j}-1]$内的最值，显然每次跳$2^{j}-1$步，那么就有：

预处理：

- $st[i][0]=a[i]$

- $st[i][j]=F(st[i][j-1],st[i+2^{j-1}][j-1])$

查询：

假设查询区间[l,r]，想办法找出两个子区间[l,x],[y,r]使得他们的并集等于[l,r]，那么可以找到一个最大的k使得$2^{k}<=(r-l+1)$作为两个区间长度。
如此可得$F([l,r])=F(F[l,x],F[y,r])$。
解析下可得$k=\lfloor \log_{2}(r-l+1) \rfloor$，两个区间分别为：$F[l,l+2^{k}-1]$，$F[r-2^{k}+1,r]$。
这种方法可行正是因为其“**可重复贡献**”性质。

### 注意点

由于这里的log是局限于整数范围，因此可以预处理出log函数值来。

- $Logn[1]=0$
- $Logn[i]=Logn[i/2]+1$

## 例子

[洛谷P3865[模板]ST表](https://www.luogu.com.cn/problem/P3865)

n个数m次询问，每次询问区间最大值。

注意数据范围和数组大小。

```c++
#define ll long long
#define IOS std::ios::sync_with_stdio(false),std::cin.tie(0),std::cout.tie(0);
#define SYP system("pause");
using namespace std;
const ll logn=21;
const ll N=1e6+5;
ll st[N][logn],Logn[N+1];
void initlog()
{
    Logn[1]=0,Logn[2]=1;
    for(ll i=3;i<N;i++)Logn[i]=Logn[i/2]+1;
}
int main()
{
    IOS
    initlog();
    ll n,m;cin>>n>>m;
    for(ll i=1;i<=n;i++){
        ll d;cin>>d;
        st[i][0]=d;
    }
    for(ll j=1;j<=logn;j++){
        for(ll i=1;i+(1<<j)-1<=n;i++){
            st[i][j]=max(st[i][j-1],st[i+(1<<(j-1))][j-1]);
        }
    }
    for(ll i=1;i<=m;i++){
        ll l,r;cin>>l>>r;
        ll k=Logn[r-l+1];
        cout<<max(st[l][k],st[r-(1<<k)+1][k])<<'\n';
    }
    return 0;
}
```
