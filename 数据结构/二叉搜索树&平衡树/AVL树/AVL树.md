# AVL树

## 简介

平衡树的鼻祖(但在竞赛界出场率貌似不高)。通过旋转来维持平衡。
保证任一节点的左右子树的最大高度差为1，所以也叫**高度平衡树**。与此同时也保证了具有n个节点的AVL树高度一定是```O(logn)```，也就是最坏时间复杂度```O(logn)```

## 平衡因子(BF)

AVL树借助平衡因子来维持树的平衡，一个节点的平衡因子为左子树高度减右子树高度(或者反过来)，容易知道所有节点的BF只能等于-1 0 1三个值。如果不满足说明不平衡需要调整。

## 调整

AVL树用树旋转来调整不平衡的节点，有四种情况对应四种旋转方法。字母表示就是LL，LR，RL，RR。假设BF=左子高度-右子高度。
LL:当前节点的左子树太高了，并且左子树的左子树比较高。**左旋**
LR:当前节点的左子树太高了，并且左子树的右子树比较高。**左旋左子树再右旋自己**
RL:当前节点的右子树太高了，并且左子树的左子树比较高。**右旋右子树再左旋自己**
RR:当前节点的右子树太高了，并且左子树的右子树比较高。**右旋**
左子树太高指BF>1，比较高指BF>0。
右子树太高指BF<-1，比较高指BF<0。

## 基本操作

### 插入

回溯递归插入之后检查是否调整

### 删除

基本与bst相同，回溯时检查是否调整。
一个方法是找到当前删除节点，然后找打它的后继，用后继来替换掉它，同时检查调整平衡。

### 其他操作

和普通平衡树差不多。

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
    int l,r;
    int val;
    int height,siz;//高度 大小
}avl[N];
int cnt,root;

inline void newnode(int &now,int val)
{
    avl[now=++cnt].val=val;
    avl[cnt].siz=1;
    //avl[cnt].height=1;
}
inline void update(int now)
{
    avl[now].siz=avl[avl[now].l].siz+avl[avl[now].r].siz+1;
    avl[now].height=max(avl[avl[now].l].height,avl[avl[now].r].height)+1;
}
inline int getBF(int now)
{
    return avl[avl[now].l].height-avl[avl[now].r].height;
}
inline void lrotate(int &now)
{
    int r=avl[now].r;
    avl[now].r=avl[r].l;
    avl[r].l=now;
    now=r;
    update(avl[now].l),update(now);
}
inline void rrotate(int &now)
{
    int l=avl[now].l;
    avl[now].l=avl[l].r;
    avl[l].r=now;
    now=l;
    update(avl[now].r),update(now);
}

/*检查旋转*/
inline void checkrotate(int &now)
{
    int nbf=getBF(now);
    if(nbf>1){
        int lbf=getBF(avl[now].l);
        if(lbf>0)rrotate(now);//LL
        else lrotate(avl[now].l),rrotate(now);//LR
    }
    else if(nbf<-1){
        int rbf=getBF(avl[now].r);
        if(rbf<0)lrotate(now);//RR
        else rrotate(avl[now].r),lrotate(now);//RL
    }
    else if(now)update(now);//平衡且非空更新大小
}

void ins(int &now,int val)
{
    if(!now) newnode(now,val);
    else if(val<avl[now].val) ins(avl[now].l,val);
    else ins(avl[now].r,val);
    checkrotate(now);
}

/*找后继*/
int getaft(int &now,int fa)//当前与父节点
{
    int ans;
    if(!avl[now].l){//找到后继
        ans=now;
        avl[fa].l=avl[now].r;//后继的右子顶替后继接上其父亲
    }
    else {
        ans=getaft(avl[now].l,now);//递归回溯
        checkrotate(now);//回溯过程检查调整
    }
    return ans;
}

void del(int &now,int val)
{
    if(val==avl[now].val){//找到
        int l=avl[now].l,r=avl[now].r;//temp
        if(!l||!r) now=l+r;//不存在
        else {
            now=getaft(r,r);//找后继换掉当前节点
            if(now!=r)//后继不是原来的右子
                avl[now].r=r;//如果是原来右子那就误成了自环
            avl[now].l=l;
        }
    }
    else if(val<avl[now].val) del(avl[now].l,val);
    else del(avl[now].r,val);
    checkrotate(now);//检查调整
}
void ldr(int now)
{
    if(!now) return;
    ldr(avl[now].l);
    cout<<avl[now].val<<' ';
    ldr(avl[now].r);
}
int getrank(int val)            //以下与替罪羊树同
{
    int now=root,rank=1;
    while(now)
    {
        if(val<=avl[now].val)
            now=avl[now].l;
        else
        {
            rank+=avl[avl[now].l].siz+1;
            now=avl[now].r;
        }
    }
    return rank;
}
int getnum(int rank)
{
    int now=root;
    while(now)
    {
        if(avl[avl[now].l].siz+1==rank)
            break;
        else if(avl[avl[now].l].siz>=rank)
            now=avl[now].l;
        else
        {
            rank-=avl[avl[now].l].siz+1;
            now=avl[now].r;
        }
    }
    return avl[now].val;
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
    while(n--)
    {
        int opt,x;
        cin>>opt>>x;
        switch(opt)
        {
        case 1:
            ins(root,x);
            break;
        case 2:
            del(root,x);
            break;
        case 3:
            cout<<getrank(x)<<'\n';
            break;
        case 4:
            cout<<getnum(x)<<'\n';
            break;
        case 5:
            cout<<getnum(getrank(x)-1)<<'\n';
            break;
        case 6:
            cout<<getnum(getrank(x+1))<<'\n';
            break;
        }
    }

    //======================================================================

    //SYP
    return 0;
}
```
