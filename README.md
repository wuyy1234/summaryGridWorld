
# 实训总结
历时四周的实训算是结束了，从一开始的ant,junit，到修改gridworld到最后的拼图优化算法，大致还是有个循序渐进由浅入深的进度。  
从一开始的编译和熟悉环境，到熟悉大类和项目架构。到最后的利用优化算法得出最优解，整体的实训出题的思路和训练目的还是能把握的住，但是觉得还有一些可以改进的地方。  
## 希望改进之处  
例如在阶段二的问答题着实有点多，感觉不太吃得消。虽然希望我们能够把gridworld里面的方法和类都仔细看一遍，但类似的题目出十来道就可以了，这次实训的问答题有六七十道，其实大部分是在消磨我们的时间。这个实训题目好像也用了几年，希望能够在问答题这块有所改进。
## 个人感触  
个人的感触还是有不少，在阶段二的时候还是体验了一把程序员的生活作息时间，阶段三的时候初步接触了优化算法，之前对这个的应用一直很少接触到，这一次很接地气的了解了一遍，包括退火算法和粒子群算法。
```
s=argONE*arg1+argTWO*arg2+argTHREE*arg3;
//参数一：后续节点不正确的数码个数
//参数二：曼哈顿距离
//参数三：物理距离
```
因为一开始是直接调参数，宿友比较厉害给直接手动调了出来，但是我就没那么幸运，只能一个一个修改，但是调了很久没有什么进展，有时还是会爆29000，于是只好利用简单循环来代替人工手动调参数，但是又有新的问题就是循环的效率太慢，大部分时间消耗在了计算上面，虽然有个好处是可以捕捉所有可能的点，但是大部分计算其实是无用功，在考虑现有的笔记本运行速度下并不是一个好的策略。  
于是开始接触优化算法，采用了粒子群算法，就是模拟许多的粒子，通过当前速度，粒子自身最有位置和粒子群最有位置三个变量来调整粒子下一步要去的位置。这个优化方法考虑了自身以及其他粒子的相互影响，在实现上也相对容易理解，最终的目的是希望粒子能够往最优位置靠拢。  

![此处输入图片的描述][1]
![此处输入图片的描述][2]  
但难点，一是粒子的初始速度和位置如何确定，一是对各个因子和系数的确定也是比较模糊，所以也是走了不少弯路。  
综合了自己和舍友的一些想法，觉得影响因素最大的首先是solution里面的因子的确定以及正确性，例如曼哈顿距离和后续错误节点数量。其次才是各个因子前面的系数，所谓的优化算法本质其实也只是对系数进行调整而已，如果本身的因子在设立之初已经有误或者目的错误，那么即使利用再好的优化算法也可能是在错误的道路上越走越远。有一点点像数学建模，很多学生参加建模比赛，但是在选取数据以及影响因子的时候就已经出错，那么后面无论再怎么套用高级的数学模型，结果也可能是一张废纸。  
![此处输入图片的描述][3]  
上面是利用粒子群算法优化的，可以看到每个粒子的平均步数在逐渐往相对最优位置靠拢，但是始终在一万多，而且超过29000的概率高达30%，很明显不是理想的结果。    
于是回想起影响因子，修改影响因子，如下：  
```
 int s = 0; // 后续节点不正确的数码个数
        int arg1=0;
        int dimension = JigsawNode.getDimension();
        for (int index = 1; index < dimension * dimension; index++) {//后续错误节点
            if (jNode.getNodesState()[index] + 1 != jNode.getNodesState()[index + 1]) {
                arg1++;
            }
        }
        
        int arg4=0;
        for(int index = 1; index <= dimension * dimension; index++) {//不在正确位置的节点
        	if(jNode.getNodesState()[index]!=index && jNode.getNodesState()[index]!=0 ) {
        		arg4++;
        	}
        }
       
    	int arg3=0; //所有 放错位的数码与其正确位置的距离之和
    	int arg2=0;//物理距离
    	int rowGoal=0,rowRea=0;
    	int colGoal=0,colRea=0;
    	for(int index = 1; index <= dimension * dimension; index++) {
    		if(jNode.getNodesState()[index]!=0) {
    			rowGoal = (index-1) / dimension;
        		colGoal= (index-1) % dimension;
        		rowRea = (jNode.getNodesState()[index]-1) / dimension;
        		colRea = (jNode.getNodesState()[index]-1) % dimension;
        		arg3+= Math.abs(rowGoal -rowRea) + Math.abs(colGoal-colRea);	
        		arg2+= Math.sqrt(Math.abs(rowRea- rowGoal)*Math.abs(rowRea- rowGoal)+Math.abs(colRea-colGoal)*Math.abs(colRea-colGoal));
    		}
    		
    	}
    	
    	s=argONE*arg1+argTWO*arg2+argTHREE*arg3+0*arg4;//arg4暂时置为0
    	jNode.setEstimatedValue(s);
```
完善了影响因子之后，继续用粒子群算法优化参数：  
结果：平均路径长度从9000降到最低3800，稳定在4000左右，而且突破29000的概率已经大幅下降到了1%，看起来还是达到了要求。  

粒子群算法源码：  
```
//粒子群算法,设为15个粒子
        int ParticleNum=15;
        int c1=2,c2=2;
        int dim=4;
        //double wInit=0.9;
        //double wEnd=0.4;
        double w=0.9;
        double[] gbest=new double[dim];//群最佳值对应的位置
        double gbestNum=0;//存取群体最优值
        double[] pbestNum=new double[ParticleNum];//存储粒子最优值
       
        double[][] pbest=new double[ParticleNum][dim];//粒子最优值对应的位置
        double[][] v=new double[ParticleNum][dim];//速度
        double[][] x=new double[ParticleNum][dim];//位置
        
        //设置初始速度和最优值
        for(int i=0;i<ParticleNum;i++) {
        	x[i][0]=Math.random()*ParticleNum;
        	x[i][1]=Math.random()*ParticleNum;
        	x[i][2]=Math.random()*ParticleNum;
        	x[i][3]=Math.random()*ParticleNum;
        	
        	v[i][0]=0;
        	v[i][1]=0;
        	v[i][2]=0;
        	v[i][3]=0;
        	pbestNum[i]=29000;
        	gbestNum=29000;
        }
        
        for(int y=0;y<40;y++) {//迭代20次,每迭代一次更新每个粒子的状态
        	 for(int i=0;i<ParticleNum;i++) {
             	
             	Solution.argONE=(int) x[i][0];
         	    Solution.argTWO=(int) x[i][1];
         	    Solution.argTHREE=(int) x[i][2];
         	    Solution.argFOUR=(int) x[i][3];
         	    //算出当前的值
         	    temp=0;
         	    for(int j=0;j<25;j++) {	
         		    JigsawNode startNode = Solution.scatter(destNode, 1000);
             		jigsaw.ASearch(startNode, destNode);
         		    //temp存取当前节点的结果
                 }
         	    temp=temp/25;
         	   //更新pbest和gbest
         	    if(pbestNum[i]>temp) {//越小越好
         	    	pbestNum[i]=temp;
         	    	pbest[i][0]=x[i][0];//更新粒子最优位置
         	    	pbest[i][1]=x[i][1];
         	    	pbest[i][2]=x[i][2];
         	    	pbest[i][3]=x[i][3];
         	    	
         	    }
         	    System.out.println(" 迭代次数： "+y+" pbest["+i+"][0]: "+ pbest[i][0] +" pbest["+i+"][1]: "+pbest[i][1]+" pbest["+i+"][2]: "
         	    		+pbest[i][2]+" pbest["+i+"][3]: "+pbest[i][3]+" 粒子"+i+"最优: "+pbestNum[i]);
         	    
         	    if(gbestNum>pbestNum[i]) {//更新群粒子最优位置
         	    	gbestNum=pbestNum[i];
         	    	gbest[0]=pbest[i][0];
         	    	gbest[1]=pbest[i][1];
         	    	gbest[2]=pbest[i][2];
         	    	gbest[3]=pbest[i][3];
         	    	
         	    }
         	   System.out.println("gbest[0]: "+ gbest[0] +" gbest[1]: "+gbest[1]+" gbest[2]: "+gbest[2]+" gbest[3]: "+gbest[3]+" 群最优: "+gbestNum);
         	    
         	  //四维
             	v[i][0]=w*v[i][0]+c1*Math.random()*(pbest[i][0]-x[i][0])+c2*Math.random()*(gbest[0]-x[i][0]);
             	v[i][1]=w*v[i][1]+c1*Math.random()*(pbest[i][1]-x[i][1])+c2*Math.random()*(gbest[1]-x[i][1]);
             	v[i][2]=w*v[i][2]+c1*Math.random()*(pbest[i][2]-x[i][2])+c2*Math.random()*(gbest[2]-x[i][2]);
             	v[i][3]=w*v[i][3]+c1*Math.random()*(pbest[i][3]-x[i][3])+c2*Math.random()*(gbest[3]-x[i][3]);
             	
             	x[i][0]+=v[i][0];
             	x[i][1]+=v[i][1];
             	x[i][2]+=v[i][2];
             	x[i][3]+=v[i][3];
             }
        	
        }
      
```

  [1]: https://img-blog.csdn.net/20160526160353248
  [2]: https://img-blog.csdn.net/20160526163529885
  [3]: http://imglf3.nosdn.127.net/img/Z281REhERnhNZlhtdG1NSnF0UVNocVk3QWk4d1FmajJtaFFNaDN3Q0I2Mk5jUnhaMk1UYjJRPT0.png?imageView&thumbnail=500x0&quality=96&stripmeta=0
