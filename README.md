# DS_Module16

## AIM: 
~~~
To Implement the program for AVL,B TREE,B+TREE using c program.
~~~
## PROGRAM:
### AVL TREE 
~~~
#include<stdio.h>
#include<stdlib.h>

typedef struct node
{
int data;
struct node *left,*right;
int ht;
}node;
node *insert(node *,int);
void preorder(node *);
//void inorder(node *);
int height(node *);
node *rotateright(node *);
node *rotateleft(node *);
node *RR(node *);
node *LL(node *);
node *LR(node *);
node *RL(node *);
int BF(node *);
 
int main()
{
node *root=NULL;
int x,n,i;
scanf("%d",&n);
root=NULL;
for(i=0;i<n;i++)
{
scanf("%d",&x);
root=insert(root,x);
}
//printf("\nPreorder sequence:\n");
preorder(root);
//printf("\n\nInorder sequence:\n");
//inorder(root);
//printf("\n");
return 0;
}
 
node * insert(node *T,int x)
{
if(T==NULL)
{
T=(node*)malloc(sizeof(node));
T->data=x;
T->left=NULL;
T->right=NULL;
}
else
if(x > T->data) // insert in right subtree
{
T->right=insert(T->right,x);
if(BF(T)==-2)
{
if(x>T->right->data)
T=RR(T);
else
T=RL(T);
}
}
else
if(x<T->data)
{
T->left=insert(T->left,x);
if(BF(T)==2)
{
    if(x < T->left->data)
T=LL(T);
else
T=LR(T);
}
}
T->ht=height(T);
return(T);
}
 
int height(node *T)
{
int lh,rh;
if(T==NULL)
return(0);
if(T->left==NULL)
lh=0;
else
lh=1+T->left->ht;
if(T->right==NULL)
rh=0;
else
rh=1+T->right->ht;
if(lh>rh)
return(lh);
return(rh);
}

 
node * rotateright(node *x)
{
node *y;
y=x->left;
x->left=y->right;
y->right=x;
x->ht=height(x);
y->ht=height(y);
return(y);
}
 
node * rotateleft(node *x)
{
node *y;
y=x->right;
x->right=y->left;
y->left=x;
x->ht=height(x);
y->ht=height(y);
return(y);
}
 
node * RR(node *T)
{
T=rotateleft(T);
return(T);
}
 
node * LL(node *T)
{
T=rotateright(T);
return(T);
}
 
node * LR(node *T)
{
T->left=rotateleft(T->left);
T=rotateright(T);
return(T);
}
 
node * RL(node *T)
{
T->right=rotateright(T->right);
T=rotateleft(T);
return(T);
}
 
int BF(node *T)
{
int lh,rh;
if(T==NULL)
return(0);
 
if(T->left==NULL)
lh=0;
else
lh=1+T->left->ht;
 
if(T->right==NULL)
rh=0;
else
rh=1+T->right->ht;
 
return(lh-rh);
}
 
void preorder(node *T)
{
if(T!=NULL)
{
printf("%d(Bf=%d)",T->data,BF(T));
preorder(T->left);
preorder(T->right);
}
}
 
~~~
### B TREE
```
#include <stdio.h>
#include <stdlib.h>

#define MAX 3
#define MIN 2

struct BTreeNode {
  int val[MAX + 1], count;
  struct BTreeNode *link[MAX + 1];
};

struct BTreeNode *root;

// Create a node
struct BTreeNode *createNode(int val, struct BTreeNode *child) {
  struct BTreeNode *newNode;
  newNode = (struct BTreeNode *)malloc(sizeof(struct BTreeNode));
  newNode->val[1] = val;
  newNode->count = 1;
  newNode->link[0] = root;
  newNode->link[1] = child;
  return newNode;
}

// Insert node
void insertNode(int val, int pos, struct BTreeNode *node,
        struct BTreeNode *child) {
  int j = node->count;
  while (j > pos) {
    node->val[j + 1] = node->val[j];
    node->link[j + 1] = node->link[j];
    j--;
  }
  node->val[j + 1] = val;
  node->link[j + 1] = child;
  node->count++;
}

// Split node
void splitNode(int val, int *pval, int pos, struct BTreeNode *node,
         struct BTreeNode *child, struct BTreeNode **newNode) {
  int median, j;

  if (pos > MIN)
    median = MIN + 1;
  else
    median = MIN;

  *newNode = (struct BTreeNode *)malloc(sizeof(struct BTreeNode));
  j = median + 1;
  while (j <= MAX) {
    (*newNode)->val[j - median] = node->val[j];
    (*newNode)->link[j - median] = node->link[j];
    j++;
  }
  node->count = median;
  (*newNode)->count = MAX - median;

  if (pos <= MIN) {
    insertNode(val, pos, node, child);
  } else {
    insertNode(val, pos - median, *newNode, child);
  }
  *pval = node->val[node->count];
  (*newNode)->link[0] = node->link[node->count];
  node->count--;
}

// Set the value
int setValue(int val, int *pval,
           struct BTreeNode *node, struct BTreeNode **child) {
  int pos;
  if (!node) {
    *pval = val;
    *child = NULL;
    return 1;
  }

  if (val < node->val[1]) {
    pos = 0;
  } else {
    for (pos = node->count;
       (val < node->val[pos] && pos > 1); pos--)
      ;
    if (val == node->val[pos]) {
      //printf("Duplicates are not permitted\n");
      return 0;
    }
  }
  if (setValue(val, pval, node->link[pos], child)) {
    if (node->count < MAX) {
      insertNode(*pval, pos, node, *child);
    } else {
      splitNode(*pval, pval, pos, node, *child, child);
      return 1;
    }
  }
  return 0;
}

// Insert the value
void insert(int val) {
  int flag, i;
  struct BTreeNode *child;

  flag = setValue(val, &i, root, &child);
  if (flag)
    root = createNode(i, child);
}

// Search node
void search(int val, int *pos, struct BTreeNode *myNode) {
  if (!myNode) {
    return;
  }

  if (val < myNode->val[1]) {
    *pos = 0;
  } else {
    for (*pos = myNode->count;
       (val < myNode->val[*pos] && *pos > 1); (*pos)--)
      ;
    if (val == myNode->val[*pos]) {
      printf("%d is found",val);
      return;
    }
  }
  search(val, pos, myNode->link[*pos]);

  return;
}

// Traverse then nodes
void traversal(struct BTreeNode *myNode) {
  int i;
  if (myNode) {
    for (i = 0; i < myNode->count; i++) {
      traversal(myNode->link[i]);
      printf("%d ", myNode->val[i + 1]);
    }
    traversal(myNode->link[i]);
  }
}

int main() {
   int ch,n,x,y;
  scanf("%d",&n);
  for(int i=0;i<n;i++)
  {
      scanf("%d",&x);
      insert(x);
  }
  scanf("%d",&y);

 // traversal(root);

  //printf("\n");
  search(y, &ch, root);
}

```
### B+ TREE
```
#include<stdio.h>
#include<stdio.h>
#include<stdlib.h>

struct B_TreeNode
{
    int *data;
    struct B_TreeNode **child_ptr;
    int leaf;
    int n;
};
struct B_TreeNode *root = NULL, *np = NULL, *x = NULL;
struct B_TreeNode * init()
{
    int i;
    np = (struct B_TreeNode *)malloc(sizeof(struct B_TreeNode));
    np->data =(int*)malloc(5*sizeof(int));
    np->child_ptr = (struct B_TreeNode **)malloc(6*sizeof(struct B_TreeNode**));
    np->leaf = 1;
    np->n = 0;
    for (i = 0; i < 6; i++)
    {
        np->child_ptr[i] = NULL;
    }
    return np;
}
void traverse(struct B_TreeNode *p)
{
    //printf("\n");
    int i;
    for (i = 0; i < p->n; i++)
    {
        if (p->leaf == 0)
        {
            traverse(p->child_ptr[i]);
        }
        printf("%d ",p->data[i]);
    }
    if (p->leaf == 0)
    {
        traverse(p->child_ptr[i]);
    }
    //printf("\n");
}
void sort(int *p, int n)
{
    int i, j, temp;
    for (i = 0; i < n; i++)
    {
        for (j = i; j <= n; j++)
        {
            if (p[i] > p[j])
            {
                temp = p[i];
                p[i] = p[j];
                p[j] = temp;
            }
        }
    }
}
int split_child(struct B_TreeNode *x, int i)
{
    int j, mid;
    struct B_TreeNode *np1, *np3, *y;
    np3 = init();
    np3->leaf = 1;
    if (i == -1)
    {
        mid = x->data[2];
        x->data[2] = 0;
        x->n--;
        np1 = init();
        np1->leaf = 0;
        x->leaf = 1;
        for (j = 3; j < 5; j++)
        {
            np3->data[j - 3] = x->data[j];
            np3->child_ptr[j - 3] = x->child_ptr[j];
            np3->n++;
            x->data[j] = 0;
            x->n--;
        }
        for(j = 0; j < 6; j++)
        {
            x->child_ptr[j] = NULL;
        }
        np1->data[0] = mid;
        np1->child_ptr[np1->n] = x;
        np1->child_ptr[np1->n + 1] = np3;
        np1->n++;
        root = np1;
    }
    else
    {
        y = x->child_ptr[i];
        mid = y->data[2];
        y->data[2] = 0;
        y->n--;
        for (j = 3; j < 5; j++)
        {
            np3->data[j - 3] = y->data[j];
            np3->n++;
            y->data[j] = 0;
            y->n--;
        }
        x->child_ptr[i + 1] = y;
        x->child_ptr[i + 1] = np3;
    }
    return mid;
}
void insert(int a)
{
    int i, temp;
    x = root;
    if (x == NULL)
    {
        root = init();
        x = root;
    }
    else
    {
        if (x->leaf == 1 && x->n == 5)
        {
            temp = split_child(x, -1);
            x = root;
            for (i = 0; i < (x->n); i++)
            {
                if ((a > x->data[i]) && (a < x->data[i + 1]))
                {
                    i++;
                    break;
                }
                else if (a < x->data[0])
                {
                    break;
                }
                else
                {
                    continue;
                }
            }
            x = x->child_ptr[i];
        }
        else
        {
            while (x->leaf == 0)
            {
            for (i = 0; i < (x->n); i++)
            {
                if ((a > x->data[i]) && (a < x->data[i + 1]))
                {
                    i++;
                    break;
                }
                else if (a < x->data[0])
                {
                    break;
                }
                else
                {
                    continue;
                }
            }
                if ((x->child_ptr[i])->n == 5)
                {
                    temp = split_child(x, i);
                    x->data[x->n] = temp;
                    x->n++;
                    continue;
                }
                else
                {
                    x = x->child_ptr[i];
                }
            }
        }
    }
    x->data[x->n] = a;
    sort(x->data, x->n);
    x->n++;
}
int main()
{
    int i, n, t;
    scanf("%d",&n);
    for(i = 0; i < n; i++)
    {
      
        scanf("%d",&t);
        insert(t);
    }
    //cout<<"traversal of constructed tree\n";
    traverse(root);
 
}

```
## OUTPUT:
### AVL TREE
![image](https://github.com/user-attachments/assets/d108bc24-7b05-4079-a03a-23492291a717)
### B TREE 
![image](https://github.com/user-attachments/assets/ce76b368-3098-4aad-b190-1a1d0d1a2258)
### B+ TREE
![image](https://github.com/user-attachments/assets/fed97501-bbb0-4599-bfbe-7ae54786458e)



