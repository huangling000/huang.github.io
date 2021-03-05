归并+合并两个有序链表

在本次做题中，主要对arraylist的用法还有点不熟悉，长度为size（），获取用get（）

```
public class Solution {
    public ListNode mergeKLists(ArrayList<ListNode> lists) {
        return merge(lists,0,lists.size()-1);
    }
    public ListNode merge(ArrayList<ListNode> lists,int left,int right){
        if(left > right){
            return null;
        }
        if(left == right){
            return lists.get(left);
        }
        if(left+1 == right){
            return mergeTwoLists(lists.get(left),lists.get(right));
        }
        return mergeTwoLists(merge(lists,left,(left+right)/2),merge(lists,(left+right)/2+1,right));
    }
    public ListNode mergeTwoLists(ListNode left,ListNode right){
        ListNode head = new ListNode(0);
        ListNode tmp = head;
        while(left != null && right != null){
            if(left.val < right.val){
                tmp.next = left;
                tmp = tmp.next;
                left = left.next;
            }else{
                tmp.next = right;
                tmp = tmp.next;
                right = right.next;
            }
        }
        if(left != null && right == null){
            tmp.next = left;
        }
        if(right != null && left == null){
            tmp.next = right;
        }
        return head.next;
    }
}
```

