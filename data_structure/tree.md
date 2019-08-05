

###[二叉树的遍历](https://www.cnblogs.com/llguanli/p/7363657.html)
![二叉树图片](https://github.com/zhhwang/gitbook-java/assets/二叉树.png)
#### 前序遍历
根-左-右
##### 递归实现
````$xslt

public void preOrderTraverse1(TreeNode root) {  
    if (root != null) {  
        System.out.print(root.val+"  ");  
        preOrderTraverse1(root.left);  
        preOrderTraverse1(root.right);  
    }  
}
````
##### 非递归实现
````$xslt
public void preOrderTraverse2(TreeNode root) {  
    LinkedList<TreeNode> stack = new LinkedList<>();  
    TreeNode pNode = root;  
    while (pNode != null || !stack.isEmpty()) {  
        if (pNode != null) {  
            System.out.print(pNode.val+"  ");  
            stack.push(pNode);  
            pNode = pNode.left;  
        } else { //pNode == null && !stack.isEmpty()  
            TreeNode node = stack.pop();  
            pNode = node.right;  
        }  
    }  
}
````
#### 中序遍历
左根右
##### 递归实现
````$xslt
public void inOrderTraverse1(TreeNode root) {  
    if (root != null) {  
        inOrderTraverse1(root.left);  
        System.out.print(root.val+"  ");  
        inOrderTraverse1(root.right);  
    }  
} 
````
##### 非递归实现
````$xslt
public void inOrderTraverse2(TreeNode root) {  
    LinkedList<TreeNode> stack = new LinkedList<>();  
        TreeNode pNode = root;  
            while (pNode != null || !stack.isEmpty()) {  
                if (pNode != null) {  
                stack.push(pNode);  
                pNode = pNode.left;  
            } else { //pNode == null && !stack.isEmpty()  
                TreeNode node = stack.pop();  
                System.out.print(node.val+"  ");  
                pNode = node.right;  
        }  
    }  
}  
````
#### 后序遍历
左右根
##### 递归实现
````$xslt
public void postOrderTraverse1(TreeNode root) {  
    if (root != null) {  
        postOrderTraverse1(root.left);  
        postOrderTraverse1(root.right);  
        System.out.print(root.val+"  ");  
    }  
}  
````
##### 非递归实现
````$xslt
void postOrder2(BinTree *root)    //非递归后序遍历
{
    stack<BTNode*> s;
    BinTree *p=root;
    BTNode *temp;
    while(p!=NULL||!s.empty())
    {
        while(p!=NULL)              //沿左子树一直往下搜索。直至出现没有左子树的结点 
        {
            BTNode *btn=(BTNode *)malloc(sizeof(BTNode));
            btn->btnode=p;
            btn->isFirst=true;
            s.push(btn);
            p=p->lchild;
        }
        if(!s.empty())
        {
            temp=s.top();
            s.pop();
            if(temp->isFirst==true)     //表示是第一次出如今栈顶 
             {
                temp->isFirst=false;
                s.push(temp);
                p=temp->btnode->rchild;    
            }
            else                        //第二次出如今栈顶 
             {
                cout<<temp->btnode->data<<" ";
                p=NULL;
            }
        }
    }    
} 
复制代码
复制代码
        另外一种思路：要保证根结点在左孩子和右孩子訪问之后才干訪问，因此对于任一结点P。先将其入栈。假设P不存在左孩子和右孩子。则能够直接訪问它；或者P存在左孩子或者右孩子。可是其左孩子和右孩子都已被訪问过了。则相同能够直接訪问该结点。若非上述两种情况。则将P的右孩子和左孩子依次入栈。这样就保证了每次取栈顶元素的时候，左孩子在右孩子前面被訪问。左孩子和右孩子都在根结点前面被訪问。

复制代码
复制代码
void postOrder3(BinTree *root)     //非递归后序遍历
{
    stack<BinTree*> s;
    BinTree *cur;                      //当前结点 
    BinTree *pre=NULL;                 //前一次訪问的结点 
    s.push(root);
    while(!s.empty())
    {
        cur=s.top();
        if((cur->lchild==NULL&&cur->rchild==NULL)||
           (pre!=NULL&&(pre==cur->lchild||pre==cur->rchild)))
        {
            cout<<cur->data<<" ";  //假设当前结点没有孩子结点或者孩子节点都已被訪问过 
              s.pop();
            pre=cur; 
        }
        else
        {
            if(cur->rchild!=NULL)
                s.push(cur->rchild);
            if(cur->lchild!=NULL)    
                s.push(cur->lchild);
        }
    }    
}
````
#### 层次遍历

