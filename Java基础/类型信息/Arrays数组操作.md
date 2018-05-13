## 常用操作

1. Arrays.fll()填充整个数组，或者填充个指定位置的元素

        int  [] a = new int [10];
        Arrays.fill(a,1);
        print("a==" + Arrays.toString(a));

        int [] b = new int [10];
        Arrays.fill(b,1,8,1);
        print("b==" + Arrays.toString(b));
        //a==[1, 1, 1, 1, 1, 1, 1, 1, 1, 1]
        //b==[0, 1, 1, 1, 1, 1, 1, 1, 0, 0]
2. equals()判断数组中元素的data是否相等
3. deepEquals()判断多维数组是否相等
4. sort()数组排序
5. binarySearch()//用于已经排序的数组中查找元素
6. toString()//产生数组的String表示
7. hashcode()//产生数字的散列码
8. asList()//接受任意的数组或者序列号将其转化成List容器
