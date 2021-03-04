递归思想

count累加判断，如果等于sum则将当前的序号写入一个新的arraylist中加入结果，否则继续向下遍历左右子树。每一次将当前节点加入list中时，注意在完成本次函数后应当将当前节点移除。

```
res.add(new ArrayList<>(list));//不能直接添加list
list.remove(list.size()-1);//记得移除

//完整代码
public ArrayList<ArrayList<Integer>> pathSum (TreeNode root, int sum) {
        // write code here
        ArrayList<ArrayList<Integer>> res = new ArrayList<ArrayList<Integer>>();
        ArrayList<Integer> list = new ArrayList<>();
        isPSum(root,res,sum,0,list);
        return res;
    }
    public void isPSum(TreeNode node,ArrayList<ArrayList<Integer>> res,int sum,int count,ArrayList<Integer> list){
        if(node == null)
            return;
        list.add(node.val);
        if((count+node.val) == sum){
            if(node.left == null && node.right == null){
                res.add(new ArrayList<>(list));
            }
            
            //isPSum(node.left,res,sum,0,new ArrayList<Integer>());
            //isPSum(node.right,res,sum,0,new ArrayList<Integer>());
        }
        isPSum(node.left,res,sum,count+node.val,list);
        isPSum(node.right,res,sum,count+node.val,list);
        list.remove(list.size()-1);
    }
```

