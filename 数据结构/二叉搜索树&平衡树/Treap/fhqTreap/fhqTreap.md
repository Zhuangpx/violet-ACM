# fhqTreap

## 简介

一种无旋式Treap，发明者为范浩强，其核心至于分裂(split)和合并(merge)操作。通过这两种操作来实现平衡树的功能。

## 简单实现

```c++
#define STD using namespace std;
#define ll long long
#define db double
#define ldb long double
#define IOS std::ios::sync_with_stdio(false),std::cin.tie(0),std::cout.tie(0);
#define SYP system("pause");
//#define endl '\n'
using namespace std;

//=====================================================================
//#define int long long

const int N=1e5+5;
struct Node
{
    int l,r;//左 右
    int val,key;//权 索引
    int siz;//大小
}fhq[N];//fhq范浩强
int cnt,root;//节点数 头结点

   /*这里的索引是随机数是用来随机化堆的上下级的，实际上如果有其他方式来实现这个功能那么索引其实可有可无*/

mt19937 rnd(233);//随机 索引

/*新建节点并放回*/
inline int newnode(int val)
{
    fhq[++cnt].val=val;
    fhq[cnt].key=rnd();//索引随机
    fhq[cnt].siz=1;
    return cnt;//返回当前节点
}

/*更新大小*/
inline void update(int now)
{
    fhq[now].siz=fhq[fhq[now].l].siz+fhq[fhq[now].r].siz+1;
}

/*分裂split:两种分裂方式(按值分裂与按大小分裂)*/
  /*一般正常的平衡树按值分裂,如果维护区间信息则按大小分裂(如文艺平衡树)*/

/*按值分裂:选定一个权值,分裂成两棵树,其中一棵树的值全部小于等于该值,另一棵树的值全部大于该值*/

//当前节点now,选定权值val,两棵树(尾节点)分别为x和y(进入时属于未选且预选的状态)
void split(int now,int val,int &x,int &y)//每次选后需要改变,因此引用
{
    if(!now) x=y=0;//当前节点不存在,那么两棵树也结束
    else {
        if(fhq[now].val<=val){//当前节点权值小于等于val
            x=now;//当前节点和当前的左子树都分到小树,那么整体接上x
            //右子树不定,递归分裂,小树预选为右子节点,大树不变
            split(fhq[now].r,val,fhq[now].r,y);
        }
        else {//当前节点权值大于val
            y=now;//当前节点分到大树,那么接上y
            //左子树不定,递归分裂,小树不变,大树预选为左子节点
            split(fhq[now].l,val,x,fhq[now].l);
        }
        update(now);//结束后更新大小
    }
}

/*按大小分裂:选定大小siz,分裂成两棵树,其中一棵树大小等于siz,其余部分在另外一棵树里*/


/*合并merge:两棵树合并*/
  /*把树x和树y合并需要满足:x树上所有的值都小于等于y树上所有的值,并且新合并出来的数需满足Treap的性质*/

//合并小树x和大树y,并返回结束后值的节点编号
int merge(int x,int y)//大小要相对
{
    if(!x||!y) return x+y;//不存在
    /*先维护堆的性质*/
    /*这里体现随机化建树，采用随机化索引的方式，事实上还有其他拓展方式，主要能够实现随机化建树或者说随机化合并即可，见下方拓展*/
    if(fhq[x].key>fhq[y].key)//这里> >= < <= 都可以 只要保持堆的性质即可 此处默认索引大的在上方
    {
        /*x权小索引大,那么y在x的右下方*/
        fhq[x].r=merge(fhq[x].r,y);
        update(x);
        return x;
    }
    else {
        /*x权小索引小,那么x在y的左下方*/
        fhq[y].l=merge(x,fhq[y].l);
        update(y);
        return y;
    }
}


int x,y,z;

/*插入:假设插入新节点权值val,那么按val拆树为x和y,然后合并x,新节点,y*/

inline void ins(int val)
{
    split(root,val,x,y);
    root=merge(merge(x,newnode(val)),y);
}

/*删除:假设删除权值为val的点,需要分裂成三棵树,删除点,再合并*/
  /*
    1.按val把树分裂为x和z
    2.按val-1把x分裂为x和y
       此时y树上的值全部等于val
    3.y树上删除一个点
       删去y树上的根节点:让 y 等于 合并y的左子树和右子树
    4.合并x y z
  */

inline void del(int val)
{
    split(root,val,x,z);//分裂出x z
    split(x,val-1,x,y);//x分裂x y
    y=merge(fhq[y].l,fhq[y].r);//删除
    root=merge(merge(x,y),z);//合并
}


/*查询值得排名(比值小的数个数加一)*/
  /*按值val-1分裂成x和y,那么val排名就是x大小+1,最后合并即可*/

int getrank(int val)
{
    split(root,val-1,x,y);
    int ans=fhq[x].siz+1;
    root=merge(x,y);
    return ans;
}

/*查询指定排名的值*/
int getnum(int rank)
{
    int now=root;
    while(now){
        if(fhq[fhq[now].l].siz+1==rank)break;
        else if(fhq[fhq[now].l].siz>=rank)now=fhq[now].l;
        else {
            rank-=fhq[fhq[now].l].siz+1;
            now=fhq[now].r;
        }
    }
    return fhq[now].val;
}


/*求前驱和后继*/
  /*
    前驱:按val-1分裂为x和y,那么x里最右边的数即是val的前驱
    后继:按val分裂为x和y,那么y里最左边的数即是val的后继
  */

int getpre(int val)
{
    split(root,val-1,x,y);
    int now=x;
    while(fhq[now].r){
        now=fhq[now].r;
    }
    int ans=fhq[now].val;
    root=merge(x,y);
    return ans;
}
int getaft(int val)
{
    split(root,val,x,y);
    int now=y;
    while(fhq[now].l){
        now=fhq[now].l;
    }
    int ans=fhq[now].val;
    root=merge(x,y);
    return ans;
}


int main()
{
// #ifdef LOCAL
//     freopen("in.in","r",stdin);
//     freopen("out.out","w",stdout);
// #endif
    //IOS

    //======================================================================

    int n;cin>>n;
    int opt,p;
    while(n--){
        cin>>opt>>p;
        switch (opt)
        {
        case 1:
            ins(p);//插入
            break;
        case 2:
            del(p);//删除
            break;
        case 3:
            cout<<getrank(p)<<'\n';//查询排名
            break;
        case 4:
            cout<<getnum(p)<<'\n';//查询值
            break;
        case 5:
            cout<<getpre(p)<<'\n';//前驱
            break;
        case 6:
            cout<<getaft(p)<<'\n';//后继
            break;
        }
    }


    //======================================================================

    //SYP
    return 0;
}
```

## 随机化拓展:不加索引

容易知道，索引的作用在于随机化堆的优先级从而随机化合并，通过这样子的随机化合并提高效率。那么我们可以找其他方式来实现这个功能从而不加索引。
也就是在merge那一步里将x合并到y还是y合并到x处，

从[洛谷的一篇帖子(123169)](https://www.luogu.com.cn/discuss/show/123169)学习到了两种:

```c++
//原本是默认大根堆
if(fhq[x].key>fhq[y].key){;;}
//第一种拓展
if(rand()&1){;;}
//第二种拓展 用一个玄学常数99999998
if(99999998%(fhq[x].siz+fhq[y].siz)<fhq[x].siz)
```

## 功能拓展:区间操作

fhqTreap也可以进行区间操作。
就行线段树一样，某些情况下fhqTreap也可以利用懒惰标记可以处理区间信息。

对区间操作其实就是分裂后操作再合并，大体步骤如下:

- 假设对区间[l,r]操作，按大小分裂

- 按大小```l-1```分裂成x和y

- 把y按大小```r-l+1```分裂成y和z

- 对y树进行操作(次数的y树就是区间[l,r])

- 最后把x y z合并

**懒惰操作**:每次操作进行懒惰标记(而不是实际操作)在分裂时如果对后续可能产生影响，那么需要把懒惰标记下传。

以洛谷的[文艺平衡树](https://www.bilibili.com/video/BV1ft411E7JW?p=2)为例，即区间翻转操作。

```c++
#define STD using namespace std;
#define ll long long
#define db double
#define ldb long double
#define IOS std::ios::sync_with_stdio(false),std::cin.tie(0),std::cout.tie(0);
#define SYP system("pause");
//#define endl '\n'
using namespace std;
//=====================================================================
//#define int long long

const int N=1e5+5;
struct Node
{
    int l,r;
    int val,key;
    int siz;
    bool lazytag;
}fhq[N];
int cnt,root;

mt19937 rnd(233);
inline int newnode(int val)
{
    fhq[++cnt].val=val;
    fhq[cnt].key=rnd();
    fhq[cnt].siz=1;
    return cnt;
}
inline void update(int now)
{
    fhq[now].siz=fhq[fhq[now].l].siz+fhq[fhq[now].r].siz+1;
}

/*懒惰下传*/
inline void pushdown(int now)
{
    swap(fhq[now].l,fhq[now].r);//左右交换
    fhq[fhq[now].l].lazytag^=1;
    fhq[fhq[now].r].lazytag^=1;
    fhq[now].lazytag=0;//懒惰标记置0
}

/*按大小分裂*/
void split(int now,int siz,int &x,int &y)
{
    if(!now) x=y=0;
    else {
        if(fhq[now].lazytag) pushdown(now);//懒惰下传
        if(fhq[fhq[now].l].siz<siz){
            x=now;
            split(fhq[now].r,siz-fhq[fhq[now].l].siz-1,fhq[now].r,y);
        }
        else {
            y=now;
            split(fhq[now].l,siz,x,fhq[now].l);
        }
        update(now);
    }
}

int merge(int x,int y)
{
    if(!x||!y) return x+y;
    if(fhq[x].key>fhq[y].key)
    {
        if(fhq[x].lazytag) pushdown(x);//懒惰下传
        fhq[x].r=merge(fhq[x].r,y);
        update(x);
        return x;
    }
    else {
        if(fhq[y].lazytag) pushdown(y);//懒惰下传
        fhq[y].l=merge(x,fhq[y].l);
        update(y);
        return y;
    }
}

void reverse(int l,int r)
{
    int x,y,z;
    split(root,l-1,x,y);
    split(y,r-l+1,y,z);
    fhq[y].lazytag^=1;//y翻转
    root=merge(merge(x,y),z);
}


void ldr(int now)
{
    if(!now) return ;
    if(fhq[now].lazytag) pushdown(now);
    ldr(fhq[now].l);
    cout<<fhq[now].val<<' ';
    ldr(fhq[now].r);
}


inline void init(int n)
{
    for(int i=1;i<=n;i++){//细节init
        root=merge(root,newnode(i));
    }
}

int main()
{
// #ifdef LOCAL
//     freopen("in.in","r",stdin);
//     freopen("out.out","w",stdout);
// #endif
    //IOS

    //======================================================================

    int n;cin>>n;
    int m;cin>>m;
    init(n);
    while(m--){
        int l,r;cin>>l>>r;
        reverse(l,r);
    }
    ldr(root);
    cout<<'\n';

    //======================================================================

    //SYP
    return 0;
}
```
