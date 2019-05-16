  # ThreadTrees: 线索树

  ## 1. 线索树的基本概念和定义
  线索树是基于二叉树（或者二叉搜索树）上进行改进的一种数据结构。对于普通二叉树中的结点，在左右子结点中存在空结点时，该结点的left和right字段被置为NULL，这时left和right字段的所占用的内存空间被浪费了。    
  
  因此线索树的基本定义为: 线索树规定在左子结点/右子结点为空时，将left指向当前结点在中序遍历中的前驱结点，将right指向当前结点在中序遍历中的后继结点。  
  
  根据上述定义，线索树中的结点可以定义如下：  

  ```
  struct ThreadNode
  {
      ThreadNode(int ival=0):val(ival),left(NULL),right(NULL),lflag(0),rflag(0)
      {
      }

      int val;
      ThreadNode *left;        // 指向当前结点的左子结点或者是中序遍历中的前驱结点的指针
      ThreadNode *right;       // 指向当前结点的右子结点或者是中序遍历中的后继结点的指针
      int lflag;               // 标志变量，lflag==0表示left指向左子结点，lflag==1表示left指向中序遍历中的前驱结点
      int rflag;               // 标志变量，rflag==0表示right指向右子结点，rflag==1表示right指向中序遍历中的后继结点
  };
  ```


## 2. 线索树的构建——二叉树的线索化算法
线索树一半都是基于一个已经存在的二叉树构建的，二叉树的线索化算法的主要思想为: 
> 1. 制作给定的二叉树的一个拷贝，拷贝中的结点使用上述线索树的结点定义
> 2. 中序遍历一次该拷贝，对于中序遍历过程中访问到的每一个结点执行操作：若该结点的左子结点为空，则置标志变量lflag=1，并将left指向当前结点的前驱结点；若该结点的右子结点为空，则置标志变量rflag=1，并将right指向当前结点的后继结点

将上述主要思想和二叉树的非递归中序遍历结合起来，详细的线索化步骤如下。
> 1. 创建一个临时栈，创建临时变量nowprev指向当前访问结点的前驱结点，初始化nowprev=NULL
> 2. 从根结点开始一直向左子结点方向遍历直到遇到空结点为止，将途经的结点依次加入栈中
> 3. 循环进行以下步骤直到栈为空为止: 
>> (1) 初始化结点指针now指向栈顶结点，now就是当前正在访问和处理的结点
>> (2) 从now->right开始，一直向左子结点方向遍历直到遇到空结点为止，将途经的结点依次加入栈中
>> (3) 正式进行线索化: 若now->left==NULL，则令left指向中序遍历的前驱结点: now->left=nowprev; now->lflag=1；若now->right==NULL，则令right指向中序遍历的后继结点（当前结点的后继结点就是当前的栈顶结点，若栈为空则设为NULL）: now->right=sta.size()?sta.top():NULL; now->rflag=1。
>> (4) 更新临时变量nowprev: nowprev=now

具体代码实现样例如下:

```
/*
 * ThreadTree: 根据普通二叉搜索树生成对应的线索树
 * note: 从普通二叉搜索树生成线索树的算法步骤可以划分为简单两步：
 *       (1) 第一步，从原始二叉搜索树进行深拷贝；
 *       (2) 第二步，在拷贝结果上通过进行依次中序遍历进行线索化：若当前结点的left为NULL，则令left指向当前结点在中序遍历中的前驱结点；若当前结点的right为NULL，则令right指向当前结点在中序遍历中的后继结点；并修改对应的状态标记变量 
*/ 
ThreadTree::ThreadTree(const Tree &other)
{
	// 1. 首先从输入的源二叉搜索树进行深拷贝，得到一个二叉搜索树 
	treeroot=__copyTree(other.getroot());    
	
	// 2. 然后进行该二叉搜索树的线索化，通过一次中序遍历进行实现，这里使用非递归方法进行实现（关于普通二叉树的中序遍历请参见二叉搜索树的部分，本处不再重复介绍） 
	// note: 使用中序遍历每次访问一个结点，将正在访问的结点进行线索化 
	stack<ThreadNode *> sta;
	
	ThreadNode *temp=treeroot;     // 用于遍历的临时指针 
	ThreadNode *nowprev=NULL;      // 当前结点now的前驱结点指针 
	
	while(temp)
	{
		sta.push(temp);
		temp=temp->left;
	}
	
	while(sta.size())
	{
		// 2.1 进行中序遍历所需要的常规步骤 
		ThreadNode *now=sta.top();    // 当前正在访问的和准备进行线索化的结点 
		sta.pop();
		
		temp=now->right;
		
		while(temp)
		{
			sta.push(temp); 
			temp=temp->left;
		}
		
		// 2.2 对当前结点now进行线索化，线索化规则按照线索树的定义: 若now->left==NULL，则将now->left指向当前结点now的前驱结点nowprev；若now->right==NULL，则将now->right指向当前结点的后继结点（当前结点now后下一个访问的结点就是当前的栈顶结点）；并设置对应的标志变量 
		if(!now->left)
		{
			now->left=nowprev;
			now->lflag=1;
		}
		if(!now->right)
		{
			// 注意首先判断栈是否为空，若访问当前结点后栈为空，说明当前结点是中序遍历中的最后一个结点，没有后继结点 
			if(sta.size())
			now->right=sta.top();
			else
			now->right=NULL;
			
			now->rflag=1;
		}
		
		// 2.3 更新当前结点now的前驱结点nowprev
		nowprev=now; 
	}
}

ThreadNode *ThreadTree::__copyTree(TreeNode *root)
{
	if(!root)
	return NULL;
	else
	{
		ThreadNode *now=new ThreadNode(root->val);
		now->left=__copyTree(root->left);
		now->right=__copyTree(root->right);
		
		return now;
	}
}
```

## 3. 线索树的两种基本操作
#### 3.1. 获取中序遍历首结点——getFirstNode操作

#### 3.2 获取中序遍历下一个结点——getNextNode操作