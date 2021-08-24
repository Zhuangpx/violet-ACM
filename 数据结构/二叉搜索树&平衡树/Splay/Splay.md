# Splay(伸展树)

## 简介

Splay是一种自组织的数据结构，也就是使用时可以通过具体情况调整(如使用频率)

Splay之所以叫伸展树，是因为其核心操作就至于**伸展(Splaying)**:把一个结点通过旋转调整到某个结点处(一般是根结点)。通过伸展能够实现分裂，合并，区间操作，搞LCT等

树旋转在这里有了更高级的形态，普通的**单旋**即**左旋(ZAG)和右旋(ZIG)**。此外还有**双旋**。

### 双旋

讨论节点x，其父节点P，祖父节点G。先把x调整至根节点

一字形(同构调整):

1. 对G ZIG，再对P ZIG (ZIG-ZIG)---偏左链->G右 P右--->偏右链

2. 对G ZAG，再对P ZAG (ZAG-ZAG)---偏右链->G左 P左--->偏左链

之字形(异构调整):

1. 对P ZAG，再对G ZIG (ZAG-ZIG)---'<'---->P左 G右

2. 对P ZIG，再对G ZAG (ZIG-ZAG)---'>'---->P右 G左

## 伸展(splaying)

递归式伸展不需要维护父节点(常数大)，否则需要维护父节点。

## 插入

分情况递归插入

## 删除

1. 找到删除的节点，将其伸展至根节点

2. 如果根节点重复次数大于1，直接减1即可

3. 否则如根节点没有后继，让根节点左儿子替换根节点即可

4. 否则找到根节点的后继(右子树的最左点)，然后将其伸展到根节点的右子(此时根节点的右子一定没有左子)。让右子替换跟节点(即右子左子接上根节点左子，然后右子换掉根节点)。

## 拓展:半伸展

## 拓展:区间操作

假设对区间[l,r]操作，先把```l-1```伸展到根节点，再把```r+1```伸展到根节点右子，那么此时根节点右子的左子树就是区间[l,r]。
注意到```l-1```可能为0，因此可以在一开始插入一个无穷小的节点作为首位防止越界。

## 简单板子

简化版:

```c++
const int maxn = 1e5+5;
struct Node
{
    int fa,ch[2],val,cnt,size;  //ch[0]是左儿子，ch[1]是右儿子
}spl[maxn];
int cnt,root;
inline void update(int x)
{
    spl[x].size=spl[spl[x].ch[0]].size+spl[spl[x].ch[1]].size+spl[x].cnt;
}
inline bool ident(int x,int f) { return spl[f].ch[1]==x; }  //查询父子关系
inline void connect(int x,int f,int s)          //建立父子关系
{
    spl[f].ch[s]=x;
    spl[x].fa=f;
}
void rotate(int x)      //合二为一的旋转
{
    int f=spl[x].fa,ff=spl[f].fa,k=ident(x,f);
    connect(spl[x].ch[k^1],f,k);        //三次建立父子关系
    connect(x,ff,ident(f,ff));
    connect(f,x,k^1);
    update(f),update(x);                //别忘了更新大小信息
}
void splaying(int x,int top)//代表把x转到top的儿子，top为0则转到根结点
{
    if(!top) root=x;
    while(spl[x].fa!=top)
    {
        int f=spl[x].fa,ff=spl[f].fa;
        if(ff!=top) ident(f,ff)^ident(x,f)?rotate(x):rotate(f);
        rotate(x);      //最后一次都是旋转x
    }
}
void newnode(int &now,int fa,int val)   //新建节点，要注意fa指针的初始化
{
    spl[now=++cnt].val=val;
    spl[cnt].fa=fa;
    spl[cnt].size=spl[now].cnt=1;
}
void delnode(int x)                 //删除结点，要注意fa指针的维护
{
    splaying(x,0);
    if(spl[x].cnt>1) spl[x].cnt--;
    else if(spl[x].ch[1])
    {
        int p = spl[x].ch[1];
        while(spl[p].ch[0]) p=spl[p].ch[0];
        splaying(p,x);
        connect(spl[x].ch[0],p,0);
        root=p;
        spl[p].fa=0;
        update(root);
    }
    else root=spl[x].ch[0],spl[root].fa=0;
}
void ins(int val,int &now=root,int fa=0)    //递归式，要传fa指针
{
    if(!now) newnode(now,fa,val),splaying(now,0);
    else if(val<spl[now].val) ins(val,spl[now].ch[0],now);
    else if(val>spl[now].val) ins(val,spl[now].ch[1],now);
    else spl[now].cnt++,splaying(now,0);
}
void del(int val,int now=root)              //同上
{
    if(val==spl[now].val) delnode(now);
    else if(val<spl[now].val) del(val,spl[now].ch[0]);
    else del(val,spl[now].ch[1]);
}
```

莫名其妙版:

```c++
const int maxn = 1e5+5;
struct Node
{
    int l,r;
    int val,size;
    int cnt;        //当前结点重复次数，默认为1
}spl[maxn];         //内存池
int cnt,root;       //内存池计数器与根节点编号
inline void newnode(int &now,int &val)
{
    spl[now=++cnt].val=val;
    spl[cnt].size++;
    spl[cnt].cnt++;
}
inline void update(int now) //更新size
{
    spl[now].size=spl[spl[now].l].size+spl[spl[now].r].size+spl[now].cnt;
}
inline void zig(int &now)
{
    int l = spl[now].l;
    spl[now].l = spl[l].r;
    spl[l].r = now;
    now = l;
    update(spl[now].r),update(now);
}
inline void zag(int &now)
{
    int r = spl[now].r;
    spl[now].r = spl[r].l;
    spl[r].l = now;
    now = r;
    update(spl[now].l),update(now);
}
void splaying(int x,int &y) //我要把x伸展到y那个位置！
{
    if(x==y) return;        //如果到了终点，return
    int &l = spl[y].l, &r = spl[y].r;   //temp
    if(x==l) zig(y);        //如果左儿子是终点，那就单旋
    else if(x==r) zag(y);   //右儿子是终点也是单旋
    else        //否则就一定是双旋
    {
        if(spl[x].val<spl[y].val)
        {
            if(spl[x].val<spl[l].val)
                splaying(x,spl[l].l),zig(y),zig(y);     //zigzig情况
            else splaying(x,spl[l].r),zag(l),zig(y);    //zagzig情况
        }
        else
        {
            if(spl[x].val>spl[r].val)
                splaying(x,spl[r].r),zag(y),zag(y);     //zagzag情况
            else splaying(x,spl[r].l),zig(r),zag(y);    //zigzag情况
        }
    }
}
inline void delnode(int now)
{
    splaying(now,root);     //将要删除的结点伸展至根结点
    if(spl[now].cnt>1) spl[now].size--,spl[now].cnt--;  //如果有重复，令重复次数--
    else if(spl[root].r)    //否则如果当前结点有后继
    {
        int p = spl[root].r;
        while(spl[p].l) p=spl[p].l;     //找到后继
        splaying(p,spl[root].r);        //将其伸展至根结点右儿子
        spl[spl[root].r].l=spl[root].l; //右儿子左儿子变为根结点
        root=spl[root].r;               //根结点变为根结点右儿子
        update(root);                   //更新一下size信息
    }
    else root = spl[root].l;    //伸展之后没有后继，说明它是最大的了，那就直接删除
}
void ins(int &now,int &val)
{
    if(!now) newnode(now,val),splaying(now,root);
    else if(val<spl[now].val) ins(spl[now].l,val);
    else if(val>spl[now].val) ins(spl[now].r,val);
    else spl[now].size++,spl[now].cnt++,splaying(now,root); //特判相同的情况
}
void del(int now,int &val)
{
    if(spl[now].val==val) delnode(now);
    else if(val<spl[now].val) del(spl[now].l,val);
    else del(spl[now].r,val);
}
//以下大致与以前的代码相同，有大变动的地方给出了注释
int getrank(int val)
{
    int now = root, rank = 1;
    while(now)
    {
        if(spl[now].val==val)   //找到了要的结点，这个之前的没有
        {
            rank+=spl[spl[now].l].size;
            splaying(now,root);
            break;
        }
        if(val<=spl[now].val)
            now=spl[now].l;
        else
        {
            rank+=spl[spl[now].l].size+spl[now].cnt;
            now=spl[now].r;
        }
    }
    return rank;
}
int getnum(int rank)
{
    int now = root;
    while(now)
    {
        int lsize = spl[spl[now].l].size;
        if(lsize+1<=rank&&rank<=lsize+spl[now].cnt) //如果在这个范围内，那就是当前结点
        {
            splaying(now,root);
            break;
        }
        else if(lsize>=rank)
            now=spl[now].l;
        else
        {
            rank-=lsize+spl[now].cnt;
            now=spl[now].r;
        }
    }
    return spl[now].val;
}
```
