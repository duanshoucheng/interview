
## 1、AddFrament和replaceFragment的区别  
 
都会调用private void doAddOp(int containerViewId, Fragment fragment, String tag, int opcmd) ；
add 是把一个fragment添加到一个容器 container 里
replace 是先remove掉相同id的所有fragment，然后在add当前的这个fragment。
在大部分情况下，这两个的表现基本相同。因为，一般，咱们会使用一个FrameLayout来当容器，而每个Fragment被add 或者 replace 到这个FrameLayout的时候，都是显示在最上层的。所以你看到的界面都是一样的。但是，使用add的情况下，这个FrameLayout其实有2层，多层肯定要比一层的来得浪费，所以还是推荐使用replace。当然有时候还是需要使用add的。比如要实现轮播图的效果，每个轮播图都是一个独立的Fragment，而他的容器FrameLayout需要add多个Fragment，这样他就可以根据提供的逻辑进行轮播了。

FragmentManager 是一个抽象类，FragmentManagerImpl 是 FragmentManager 的子类，在 FragmentManager 同一个 java 文件内，是一个内部类
获取到 FragmentManager 后，咱们一般就会调用 beginTransaction() 方法，返回一个 FragmentTransaction 。FragmentTransaction 是一个抽象类,FragmentManagerImpl.beginTransaction() 返回的是BackStackRecord就是FragmentTransaction的实现类，并且实现了Runnable接口
>所以用add方式实现fragment的效果就是：切换fragment时不会重新创建，是什么样子切换回来还是什么样子；用replace的效果就是：切换fragment时每次都会重新创建初始化。
## 2、快速排序
通过一趟排序将待排序记录分割成独立的两部分，其中一部分记录的关键字均比另一部分关键字小，则分别对这两部分继续进行排序，直到整个序列有序。
把整个序列看做一个数组，把第零个位置看做中轴，和最后一个比，如果比它小交换，比它大不做任何处理；交换了以后再和小的那端比，比它小不交换，比他大交换。这样循环往复，一趟排序完成，左边就是比中轴小的，右边就是比中轴大的，然后再用分治法，分别对这两个独立的数组进行排序。[链接](http://www.cnblogs.com/xiaoming0601/p/5862860.html)
```
package com.company;

import java.util.HashMap;

public class Main {
    public static void main(String args[]) {
        Main qs = new Main();
        Integer[] list={34,3,53,2,23,7,14,10};
        qs._quickSort(list,0, list.length-1);
        for(int i=0;i<list.length;i++){
            System.out.print(list[i]+" ");
        }
        System.out.println();
    }

    public int getMiddle(Integer[] list, int low, int high) {
        int tmp = list[low];    //数组的第一个作为中轴
        while (low < high) {
            while (low < high && list[high] > tmp) {
                high--;
            }
            list[low] = list[high];   //比中轴小的记录移到低端
            while (low < high && list[low] < tmp) {
                low++;
            }
            list[high] = list[low];   //比中轴大的记录移到高端
        }
        list[low] = tmp;              //中轴记录到尾
        return low;                   //返回中轴的位置
    }

    public void _quickSort(Integer[] list, int low, int high) {
        if (low < high) {
            int middle = getMiddle(list, low, high);  //将list数组进行一分为二
            _quickSort(list, low, middle - 1);        //对低字表进行递归排序
            _quickSort(list, middle + 1, high);       //对高字表进行递归排序
        }
    }
}

```
