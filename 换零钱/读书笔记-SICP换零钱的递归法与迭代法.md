咳咳..先说一段废话..
最近开始看SICP这本书，正看到了换零钱的部分。看到里面那么多简明生动的例子，还有作者的细心讲解，真是唤起了对学习的无限激情。之前也看过王垠的一些文章，提到了诸如`Lisp`、`scheme`等语言，不过当时对此是零概念（其实现在对`scheme`的了解，也止于SICP换零钱这部分章节）。在了解了`mit-scheme`的一些基本用法后，虽然也觉得用‘括号’这种语法的确很优雅，但当我真的写完一个长长的定义之后，游荡在许许多多括号中间，找个小bug，真心觉得头疼。

***
## 换零钱
回到正题，先说一下题目：

> 给了50、25、10、5、1美分的硬币，将1美元（100美分）换成零钱，总共有多少种换法？

在这之前，书中也提到了一般有两种方法，**线性递归**和**线性迭代**，书中给出的解法是一个最为直观的，效率也最低的解法——**线性递归**。

### 递归法

书中的思路这样描述：
> 将总数为a的现金换成n种硬币的不同方式的数目等于
>  * 将现金数a换成除第一种硬币之外的所有其他硬币的不同方式数目，加上<br>
>  * 将现金数a-d换成所有种类的硬币的不同方式数目，其中的d是第一种硬币的币值

按照这种规则，则可以总结出如下算法：
> * 如果a就是0，应该算作是有1种换零钱的方式。
> * 如果a小于0，应该算作是有0种换零钱的方式。
> * 如果n是0，应该算作是有0种换零钱的方式。

`n`应该代表的是，第几种硬币（有1、2、3、4、5这五种币种，0就是没有可分配的币种）。
这个递归过程的代码为：

```
(define (first-denomination kinds-of-coin)
  (cond ((= kinds-of-coin 1) 1)
	((= kinds-of-coin 2) 5)
	((= kinds-of-coin 3) 10)
	((= kinds-of-coin 4) 25)
	((= kinds-of-coin 5) 50)))

(define (cc amount kinds-of-coin)
  (cond ((= amount 0) 1)
	((or (< amount 0) (= kinds-of-coin 0)) 0)
	(else (+ (cc amount
		     (- kinds-of-coin 1))
		 (cc (- amount
			(first-denomination kinds-of-coin))
		     kinds-of-coin)))))

(define (count-change amount)
  (cc amount 5))
```
所以测试了一下，结果为：

```
1 ]=> (count-change 100)

;Value: 292

```
这个递归运算是指数级的，现在看起来没什么问题，但是当总钱数比较大时，问题就明显了。

### 迭代法

又是一段废话...
作者说，这一解法留给读者作为一个挑战。看到'挑战'这个字眼，凭着对求知的好胜心，我的心就痒痒了，然后接下来就是自虐的过程。虽然出身于数学系，但是大学期间，基本都把青春献给dota了。
面对这个一层套一层的问题，脑袋已经不够用了，想了许多种思路，几乎都是半途发现行不通又重新思考。在想了几个小时后，终于忍不住去网上搜索答案，英文不好，只能搜搜中文的。发现有个答案，大概看了一下，说是用数组先把一部分结果算出来，然后复用。因为我的`scheme`水平实在有限，没接触到那些用法，所以也没细看，而且我觉得这个解法，也不符合我想要的那种答案。
只好回来继续虐自己吧，光空想，脑袋是转不过来了。突然想起，书中一般都是通过画一些推导图之类的，来寻求规律，于是我也拿起笔画起来，终于找到了一种思路。

#### 思路
递归法是通过把每一个大问题划分成两个小问题，从而生一个树形解法，思路虽然比较好理解，但是运算时就要把树全部展开，而且子树当中存在大量重复计算，效率可想而知。而迭代法不能延续这种思路，只能通过自身的参数，来分析当前迭代的情况，并计算出来，进行下一次迭代。所以我的想法是，可不可以每次迭代通过每种币种的数量，计算出剩余的钱，然后分析出当前情况是否是一个有效的分法，然后进行下一次迭代，直至每种币种的所有数量情况遍历完毕。
怎么判断，什么是一个有效的分法呢？就是每种硬币，它的币值乘以它的数量，最后相加在一起，正好等于总钱数时，即剩余钱数是0时，就是一种有效的分法。
设有如下对应关系：

|   币种   |   币值   |   数量   |
|  :---:   |   :---:  |  :---:   |
| coin1   | 50       |    c1     |
| coin2   | 25       |    c2     |
| coin3   | 10       |    c3     |
| coin4   | 5         |    c4    |
| coin5   | 1         |    c5    |

总钱数为`amount`;
那么，当`amount = coin1*c1+coin2*c2+coin3*c3+coin4*c4+coin5*c5;`时，
就是一种有效的分法。

有几个问题需要考虑：
- 按照怎么样顺序去遍历各币种的数量，直至当剩余钱数符合一种特殊情况（小于0或者等于0）。
- 当剩余钱数等于或者小于0的时候，怎样安排下一次顺序。
- 怎么判断结束情况。

分钱这种事，大概一想也知道，如果你有100块，分别可分成2张50块的，4张25块的，10张10块的，20张5块的，100张1块的。币值越大，可分的数量就越少，如果给定有一张是50块的，那零外50又可分成，2张25块的，5张10块的等等...
按这个规律，我们又发现，全分成50的，有2张；50、25混着分，有3张；全分成25的，有4张。这样就是说，币种平均值越小，可分的张数就越多。
那我们就先从平均值最大的币种开始遍历（没试过从平均值最小的遍历），迭代就是只要剩余钱数大于0，就让数量增加，每当遇到特殊情况，剩余钱数小于0或者等于0，就开始遍历平均值第二大的，这样可分的张数就越来越多。直到只剩下平均值最小的币种了，那么就结束了。

#### 方法

先从币值最大的币种（第一种币种）开始遍历，每次数量加1，计算出剩余钱数，直到剩余钱数等于0或者小于0时，就将此次的最大币种的数量减1，并将币值第二大的币种（第二种币种）数量加1；接下来每次第一种的数量保持不变，将第二种的数量加1，再次直到剩余钱数等于或者小于0，然后将第二种数量减1，第三种数量加1，此时重复之前的步骤，直至到最后一个币种。可以发现，我们需要有个标识记录，当前是哪种硬币的数量应该加1。总结后，遍历情况如下图：
![图1.jpg](http://upload-images.jianshu.io/upload_images/579224-7308fe7d30f3fb09.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
**思考：**
*到了图中最后一步时，游标已经走到了最后一个币种，但此时遍历还远没有结束。接下来就是需要找到比现在的币种平均值要小的搭配了。一开始，我直接把第一个币种的数量减1，并且后面的币种数量清0，进而开始迭代，结果发现，漏掉了许多情况。漏掉的情况就是当第一个币种数量不变（即上图为1）时，除第一种币种之外，剩余其他币种的随意搭配。所以，只能从后往前，开始减少币种的数量，这样就不会漏掉情况了。*

接下来继续，目前游标已经是第五种硬币了，所以要**将前面离的最近的且数量不为0的币种数量减1，将下一币种加1（将游标移动到此币种），再将这之后的硬币数清零**，然后继续用之前的方法迭代。
接着图1继续迭代，如下图：
![图2.jpg](http://upload-images.jianshu.io/upload_images/579224-7593fa1b9e867b56.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

最后一个问题，怎么判断结束呢？
由于是一步一步减少币值较大的币种的数量，所以最后会看见这样一幅画面：
![图3.jpg](http://upload-images.jianshu.io/upload_images/579224-b6f50555603d8410.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

很明显，当**前四种硬币的数量为0**时，第5种硬币数量为100，这就是最后一种有效的分法，可以结束了。

#### 代码

方法已经有了，接下来就是还原成代码吧。由于对`scheme`不熟，如果直接用`scheme`，估计我就崩溃了。为了快速验证我的想法，我首先用`C语言`实现了一下（我用的是`Xcode`），代码如下：

```
// 获取对应币种的币值
int get_coin(int index){
    switch (index) {
        case 1:{
            return 50;
            break;
        }
        case 2:{
            return 25;
            break;
        }
        case 3:{
            return 10;
            break;
        }
        case 4:{
            return 5;
            break;
        }
        case 5:{
            return 1;
            break;
        }
        default:
            break;
    }
    return 0;
}

// 为了计算执行了多少次迭代
static long long cn2 = 0;

//                  剩余钱数     有效分法      游标
static int c_c(int leftAmount, int count, int cur, int c1, int c2, int c3, int c4, int c5){
    
    cn2++;
    
    // 当剩余钱数等于或者小于0时
    if (leftAmount == 0 || leftAmount < 0){
        
        // 1.当游标走到4时，其实是从前往后遍历的时候
        // 2.当游标在5时，但是第4种硬币的数量大于0，属于从后往前遍历
        // 这两种情况处理是一样的，之后的判断与处理跟此处类似
        // 这里还有一个稍微特殊一点的地方：
        // 当属于第2种情况时，由于第5种硬币已经是最后一种硬币，所以直接加1，且之后并没有需要数量清零的币种
        if ((cur==5 && c4>0) ||
            cur==4){
            
            return c_c(leftAmount+get_coin(4)-get_coin(5),
                       (leftAmount<0)?count:count + 1,// 如果剩余钱数等于0，有效次数加1，反之不变
                       5,
                       c1,
                       c2,
                       c3,
                       c4-1,
                       c5+1);
        }
        else if ((cur==5 && c4==0 && c3>0) ||
                 cur==3){
            
            return c_c(leftAmount+get_coin(3)-get_coin(4)+get_coin(5)*c5,
                       (leftAmount<0)?count:count + 1,
                       4,
                       c1,
                       c2,
                       c3-1,
                       c4+1,
                       0);
        }
        else if ((cur==5 && c4==0 && c3==0 && c2>0) ||
                 cur==2){
            
            return c_c(leftAmount+get_coin(2)-get_coin(3)+get_coin(4)*c4+get_coin(5)*c5,
                       (leftAmount<0)?count:count + 1,
                       3,
                       c1,
                       c2-1,
                       c3+1,
                       0,
                       0);
        }
        else if ((cur==5 && c4==0 && c3==0 && c2==0 && c1>0) ||
                 cur==1){
            
            return c_c(leftAmount+get_coin(1)-get_coin(2)+get_coin(3)*c3+get_coin(4)*c4+get_coin(5)*c5,
                       (leftAmount<0)?count:count + 1,
                       2,
                       c1-1,
                       c2+1,
                       0,
                       0,
                       0);
        }
        // 结束情况： (cur==5 && c4==0 && c3==0 && c2==0 && c1==0)
        else{
            // 如果剩余钱数等于0，有效次数加1，并返回
            return (leftAmount<0)?count:count+1;
        }
    }
    // 剩余钱数大于0
    else{
        // 继续将游标处的币种数量加1
        return c_c(leftAmount-get_coin(cur),
                   count,
                   cur,
                   (cur==1)?c1+1:c1,
                   (cur==2)?c2+1:c2,
                   (cur==3)?c3+1:c3,
                   (cur==4)?c4+1:c4,
                   (cur==5)?c5+1:c5);
    }
}

int getCountChange(int amount){
  
    cn2 = 0;
    
    // 将第一种搭配当参数传入
    int count = c_c(amount-get_coin(1),0,1,1,0,0,0,0);
    
    printf("执行了%lld次",cn2);
    
    return count;
}
```
然后进行测试
```
printf("count: %d",getCountChange(100));

控制台打印：
->] 执行了1554次
->] count: 292

```

其实我也把递归法实现了，有对比才好说话，递归法 (`getCountChange2`) 的测试如下：

```
printf("count: %d",getCountChange2(100));

控制台打印：
->] 执行了15499次
->] count: 292
```

看来差距还不是非常的大（10倍），那增加到500试试...

```
printf("迭代法count: %d",getCountChange3(500));
printf("递归法count: %d",getCountChange2(500));

控制台打印：
->] 迭代法执行了346730次
->] 迭代法count: 59576

->] 递归法执行了12822611次
->] 递归法count: 2435

```

恩，看来差距还是有的。
细心的朋友可能发现了，迭代法用的是`getCountChange3`，这个并不是bug。其实当我用`getCountChange`方法时，总钱数到200多的时候，Xcode就阻止继续运行了，此时是迭代了3万次左右，这个原因我不太懂，可能是会检查函数的最大循环调用次数。所以我写了个`getCountChange3`方法，方法里面用一个循环代替了函数递归，其他方式都不变，以此模拟函数迭代法。

看来没什么问题了，可以用`scheme`测试了。代码如下：

```
(define (get-coin index)
  (cond ((= index 1) 50)
	((= index 2) 25)
	((= index 3) 10)
	((= index 4) 5)
	((= index 5) 1)))

(define (c-c leftAmount count cursor c1 c2 c3 c4 c5)
  (cond ((or (= leftAmount 0) (< leftAmount 0)) ; 当剩余的钱小于或等于0的时候
	 (cond ((or (= cursor 4)
		    (and (= cursor 5) (> c4 0))) ;如果cursor=4，或者 cursor=5且c4>0
		(c-c (- (+ leftAmount
			   (get-coin 4))
			(get-coin 5)) ;leftAmount将第4种硬币钱数加回来一个，且c4减1，紧接着加上第5种硬币钱数，且c5加1
		     (if (< leftAmount 0)
			 count
			 (+ count 1)) ;如果leftAmount=0，说明正好分完一次，所以count加1；反之，count不变
		     5 ;将游标指向第五种硬币
		     c1
		     c2
		     c3
		     (- c4 1) ;c4减1
		     (+ c5 1))) ;c5加1，不清零是因为c5是最后一种硬币，无需从零开始计算
	       ((or (= cursor 3)
		    (and (= cursor 5) (= c4 0) (> c3 0)))
		(c-c (- (+ leftAmount
			   (get-coin 3)
			   (* c5 (get-coin 5)));此时c5需要清零，所以把第五种硬币的钱数都加回来
			(get-coin 4))
		     (if (< leftAmount 0)
			 count
			 (+ count 1))
		     4
		     c1
		     c2
		     (- c3 1)
		     (+ c4 1)
		     0))
	       ((or (= cursor 2)
		    (and (= cursor 5) (= c4 0) (= c3 0) (> c2 0)))
		(c-c (- (+ leftAmount
			   (get-coin 2)
			   (* c4 (get-coin 4))
			   (* c5 (get-coin 5)))
			(get-coin 3))
		     (if (< leftAmount 0)
			 count
			 (+ count 1))
		     3
		     c1
		     (- c2 1)
		     (+ c3 1)
		     0 ;清零原理同上
		     0))
	       ((or (= cursor 1)
		    (and (= cursor 5) (= c4 0) (= c3 0) (= c2 0) (> c1 0)))
		(c-c (- (+ leftAmount
			   (get-coin 1)
			   (* c3 (get-coin 3))
			   (* c4 (get-coin 4))
			   (* c5 (get-coin 5)))
			(get-coin 2))
		     (if (< leftAmount 0)
			 count
			 (+ count 1))
		     2
		     (- c1 1)
		     (+ c2 1)
		     0
		     0
		     0))
	       (else (if (< leftAmount 0); 此时c4=0,c3=0,c2=0,c1=0,所以全部遍历完毕，结束。
			 count
			 (+ count 1)))))
	(else (c-c (- leftAmount
		       (get-coin cursor));此时如果leftAmount>0,继续将游标指向的硬币数量加1
		    count
		    cursor
		    (if (= cursor 1)
		        (+ c1 1)
		        c1)
		    (if (= cursor 2)
		        (+ c2 1)
		        c2)
		    (if (= cursor 3)
		        (+ c3 1)
		        c3)
		    (if (= cursor 4)
		        (+ c4 1)
			c4)
		    (if (= cursor 5)
			(+ c5 1)
		        c5)))))

(define (count-change2 amount)
  (c-c (- amount
	  (get-coin 1))
       0
       1
       1
       0
       0
       0
       0))
```
进行测试
```
1]=> (count-change2 100)

;Value: 292
 
```
为了与**递归法**对比，我分别执行了钱数为1000的情况，**迭代法**也是过了20秒左右才出结果（801451种），但是**递归法**至今还没有算出来（也许是我等的时间太短了？）...

### 总结

虽然最后代码看起来很长，但思路还算清晰，由于因为挑战成功吧，还是有点小高兴的，也算是给了自己一个比较满意的答案。
仔细研究研究这个算法，还是有优化的余地的，比如当游标处的币种数量减1时，下一币种数量不是加1，而是直接加上此游标处币种除以下一币种的倍数（需要四舍五入），这样可以减少迭代次数。
学习的过程比较漫长，情绪也是跌宕起伏，但我相信结果是美好的，继续往下学吧😄...

