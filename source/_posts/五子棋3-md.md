---
title: 实现自己的五子棋（三）——实现AI下棋
date: 2018-07-10 13:51:30
tags: [game]
categories: Java
updated: 2018-07-10 14:51:30
---

这一篇我们将解决人机对战的问题。解决方案是“权值法”。

我的代码里默认的是人先下，也可以加个条件更换次序。其实所谓的AI的下棋不是真的自己在思考，而是从人所下的位置算出周围8个顶点的权值，挑出最大的位置来下棋。<!-- more -->

所以显然，我们需要做的就是:

1. 为不同的情况定义权值;
2. 计算权值
3. 挑出最大权值
4. AI下棋 。OK，让我们开始吧





  ##  一、 考虑棋子相连的情况和对应的权值

|   活连    |   眠连   |
| :-------: | :------: |
|  1连 40   |  1连 20  |
|  2连 400  | 2连 200  |
| 3连 3000  | 3连 500  |
| 4连 10000 | 4连 3000 |

（至于什么叫活连，眠连自行百度吧）



关于权值的设置可以自己在合理的范围内设置，所谓合理就是：不能让AI已经到达四连或者的情况下还去堵人的二连甚至一连。



有个tip：

- AI棋子的权值偏高，人棋子的权值偏低。AI偏攻击型
- 人棋子的权值偏高，AI棋子的权值偏低。AI偏防御型



因为希望能够从连的情况得到其权值，所以我们采用哈希表HashMap来储存各种情况的权值，并将其设置为static，让其能方便被ChessAI类调用。此代码和ChessAI类的其他数据成员代码如下：



```java
package GoBang;
 
import java.awt.Color;
import java.awt.Graphics2D;
import java.awt.RenderingHints;
import java.awt.event.MouseAdapter;
import java.awt.event.MouseEvent;
import java.util.ArrayList;
import java.util.HashMap;
 
import javax.swing.JComboBox;
import javax.swing.JOptionPane;
 
public class ChessAI extends MouseAdapter implements GoBangconfig {
	public GoBang gb;
        int maxrc[] = new int[2]; //定义数组储存最大权值的行和列，具体见三
	public ChessAI(GoBang gb) {
		this.gb = gb;
	}
 
	static HashMap<String, Integer> map = new HashMap<String, Integer>();
	static {
		// 活连
		map.put("010", 40);    //黑棋为1
		map.put("0110", 400);
		map.put("01110", 3000);
		map.put("011110", 10000);
		map.put("020", 20);    //白棋为2
		map.put("0220", 200);
		map.put("02220", 500);
		map.put("022220", 3000);
 
		// 眠连
		map.put("012", 40);
		map.put("0112", 400);
		map.put("01112", 3000);
		map.put("011112", 10000);
		map.put("021", 40);
		map.put("0221", 400);
		map.put("02221", 3000);
		map.put("022221", 10000);
	}
	private int[][] weightArray = new int[row][column];   //权值数组
```



## 二、计算各个方向的权值

接下来我们定义一个权值数组用来储存8个位置的权值。某个棋子对应的权值等于把上下左右，左上，右上，左下，右下八个方向的权值全部算出来后求和。定义权值是为了**让AI知道下在哪里更容易赢得胜利**。



不过先别急着计算权值，因为可能人下一步棋就已经赢了。所以应当先判断输赢，如果人还没赢，就再算权值。AI算法要做的事情就是统计存储棋子的数组，根据棋子数组中棋子相连的情况来计算权值存入到数组中。



 以水平向左的情况为例子：

```java
// 根据棋子数组中棋子相连的情况来计算权值存入到数组中
public void WeightCount() {

    //System.out.println("sssssssssss" + gb.isArrive.length);

    for (int r = 0; r < gb.isArrive.length; r++) {

        for (int c = 0; c < gb.isArrive[r].length; c++) {

            // 判断此位置有无棋子，如果已经有棋子了就没必要下了
            if (gb.isArrive[r][c] == 0) {
                // 水平方向
                String code = "0";// 用来记录棋子相连情况
                int chess = 0;// 记录第一次出现的棋子
                int number = 0;// 记录空位出现的次数

                // 水平方向
                // 水平向左
                for (int c1 = c - 1; c1 >= 0; c1--) {
                    if (gb.isArrive[r][c1] == 0) {
                        if (c == c1 + 1) { // 在c1+1处就空了，说明是连续两个空位，就不用再讨论啦
                            break;
                        }
                        // 不是连续两个空位的情况
                        else if (number == 0) {// 表示第一次出现空位
                            code = code + gb.isArrive[r][c1];// 记录棋子相连的情况
                            number++;// 空位的次数加1
                        } else if (number == 1) {// 表示第二次出现空位
                            if (gb.isArrive[r][c1] == gb.isArrive[r][c1 + 1]) {
                                break; // 检测是否连续两个位置是否都为空位
                            }

                            code = code + gb.isArrive[r][c1];// 记录棋子相连的情况
                            number++;
                        } else if (number == 2) {// 表示第三次出现空位
                            if (gb.isArrive[r][c1] == gb.isArrive[r][c]) // 检测两个位置是否都为空位
                                break;
                        }
                    } else {
                        if (chess == 0) { // 表示第一次出现棋子
                            chess = gb.isArrive[r][c1];// 存储第一次出现棋子
                            code = code + gb.isArrive[r][c1]; // 记录棋子的相连情况
                        } else if (chess == gb.isArrive[r][c1]) {// 判断是否和第一次出现的棋子同色
                            code = code + gb.isArrive[r][c1]; // 记录棋子的相连情况
                        } else {// 表示此位置的棋子的颜色与第一次出现的颜色不同，所以记录
                            code = code + gb.isArrive[r][c1]; // 记录棋子的相连情况
                            break;
                        }

                    }
                }

                Integer value = map.get(code); 
                if (value != null)// 判断value是否不为null
                    weightArray[r][c] += value;// 在对应空位累加权值  }
```



其他方向的代码将放在文末，思路都是差不多的。

想提醒的是：

- 之所以判断是否是连续空位，是因为如果空位有连续地两个，那么可以当做没连的情况了，所以 break 遍历的循环；

- 用 Integer 而不用 int ，是因为下面的 if 条件用到了null，Java里int不能和null直接比较，需要将 int 装箱。

  

## 三、findMax() 函数的实现

现在我们得到了想要的权值数组，接下来就从中取到权值最大的数据的行和列，再将这个行和列传给监听器，就能在该位置让AI下棋了



```java
public int[] findmax() { // 得到最大权值的行列，用数组来储存
		int max = 0, i = 0, j = 0;
		for (i = 0; i < weightArray.length; i++) {
			for (j = 0; j < weightArray[i].length; j++) {
				//System.out.print(weightArray[i][j] + "\t");
				if (weightArray[i][j] > max) {
					max = weightArray[i][j];
					maxrc[0] = i;
					maxrc[1] = j;
				}
			}
			//System.out.println();
		}
		//System.out.println(maxrc[0] + "<>" + maxrc[1]);
		return maxrc;  //因为不能同时返回两个值，所以用数组来存储后返回数组即可
	}
```



至此，AI算法的核心部分已经完成了。需要做的就是把它和之前的 chessListener 联系起来，让监听能同时分情况调用 PERSONPLAY() 和 AIPLAY() （不知道你是否还有印象在前一篇文章里这里留了个坑，留在这节补上）。

我们把AI的算法封装为 AIPLAY()，到时候直接调用就行。代码：



```java
public void AIPLAY(int x, int y) {
    // 一开始人下
    // 计算棋子要落的交叉点
    int countx = (y - 20 + size / 2) / size;
    int county = (x - 20 + size / 2) / size;
    g = (Graphics2D) gf.getGraphics();
    g.setRenderingHint(RenderingHints.KEY_ANTIALIASING,
                       RenderingHints.VALUE_ANTIALIAS_ON); // 抗锯齿
    int arrivex, arrivey;
    // 棋盘上的具体位置
    arrivex = 20 + county * size;
    arrivey = 20 + countx * size;

    if (countx >= 0 && countx <= 15 && county >= 0 && county <= 15
        && gf.isArrive[countx][county] != 0) // 有棋子
    {
        JOptionPane.showMessageDialog(gf, "此处已有棋子，请换一个地方");
    } else {
        // 人下了黑棋
        g.setColor(Color.black);
        g.fillOval(arrivex - size / 2, arrivey - size / 2, size, size);
        gf.isArrive[countx][county] = 1;
        list.add(new chess(countx, county)); // 有序地存储每个棋子的行列，为悔棋做准备
        // 判断输赢
        if (Gobangwin.judge(gf.isArrive, countx, county)) {
            JOptionPane.showMessageDialog(gf, "黑棋胜利");
            gf.removeMouseListener(this);   box.setEnabled(true); //解封box
            return;
        }
        ChessAI c = new ChessAI(gf);
        // System.out.println(">>>"+gf.isArrive[0].length+"<<<");
        // 轮到AI下
        c.WeightCount();
        int maxrc[] = c.findmax();
        arrivex = 20 + maxrc[1] * size;
        arrivey = 20 + maxrc[0] * size;
        g.setColor(Color.white);
        g.fillOval(arrivex - size / 2, arrivey - size / 2, size, size);
        gf.isArrive[maxrc[0]][maxrc[1]] = 2;
        list.add(new chess(maxrc[0], maxrc[1]));// 有序地存储每个棋子的行列，原因同上

        // 判断输赢
        if (Gobangwin.judge(gf.isArrive, maxrc[0], maxrc[1])) {
            JOptionPane.showMessageDialog(gf, "白棋胜利");
            gf.removeMouseListener(this);  box.setEnabled(true); //解封box
        }
    }

}
```



同样的，我们也可以把人下棋的方法封装起来，便于调用。代码如下：（注释与 AI 相似，不再重复

```java
public void PERSONPLAY(int x, int y) {
 
		// 人下的方法
		// 计算棋子要落的交叉点
		int countx = (y - 20 + size / 2) / size;
		int county = (x - 20 + size / 2) / size;
		g = (Graphics2D) gf.getGraphics();
		g.setRenderingHint(RenderingHints.KEY_ANTIALIASING,
				RenderingHints.VALUE_ANTIALIAS_ON); // 抗锯齿
		int arrivex, arrivey; // 棋盘上
		arrivex = 20 + county * size;
		arrivey = 20 + countx * size;
 
		if (gf.isArrive[countx][county] != 0) // 有棋子
		{
			JOptionPane.showMessageDialog(gf, "此处已有棋子，请换一个地方");
		} else {// 当前位置可以下棋
			if (turn == 1) {
				// 设置颜色
				g.setColor(Color.black);
 
				// 下棋
				g.fillOval(arrivex - size / 2, arrivey - size / 2, size, size);
				gf.isArrive[countx][county] = 1;
				turn++;
			} else {
				// 设置颜色
				g.setColor(Color.white);
 
				// 下棋
				g.fillOval(arrivex - size / 2, arrivey - size / 2, size, size);
				gf.isArrive[countx][county] = 2;
				turn--;
 
			}
			//
 
			list.add(new chess(countx, county));// 有序地存储每个棋子的行列
 
			// 判断输赢
			if (Gobangwin.judge(gf.isArrive, countx, county)) {
				if (turn == 2)
					JOptionPane.showMessageDialog(gf, "黑棋胜利");
				else
					JOptionPane.showMessageDialog(gf, "白棋胜利");
				gf.removeMouseListener(this);  box.setEnabled(true); //解封box
			}
		}
 
	}
```



## 四、大功告成

好的，到此为止呢整个五子棋程序就结束了。运行后的界面如图：

![五子棋](/images/wuziqi.jpg)

<center>图1 游戏界面</center>



PS：代码中有很多被注释掉的输出语句。因为代码比较长，我们不知道是在哪里出现了bug，所以在关键节点加上输出语句能更有效的排查掉错误。这也是程序员的必备技能之一（血与泪的教训++）



THE  END.



## 五、附：其他方向的代码

```java
	public void WeightCount() {
 
		//System.out.println("sssssssssss" + gb.isArrive.length);
 
		for (int r = 0; r < gb.isArrive.length; r++) {
 
			for (int c = 0; c < gb.isArrive[r].length; c++) {
 
				// 判断此位置有无棋子
				if (gb.isArrive[r][c] == 0) {
					// 水平方向
					String code = "0";// 用来记录棋子相连情况
					int chess = 0;// 记录第一次出现的棋子
					int number = 0;// 记录空位出现的次数
 
					// 水平方向
					// 水平向左
					for (int c1 = c - 1; c1 >= 0; c1--) {
						if (gb.isArrive[r][c1] == 0) {
							if (c == c1 + 1) {// 在c1+1处就空了，说明是连续两个空位，就不用再讨论啦
								break;
							}
							// 不是连续两个空位的情况
							else if (number == 0) {// 表示第一次出现空位
								code = code + gb.isArrive[r][c1];// 记录棋子相连的情况
								number++;// 空位的次数加1
							} else if (number == 1) {// 表示第二次出现空位
								if (gb.isArrive[r][c1] == gb.isArrive[r][c1 + 1]) {
									break; // 检测两个位置是否都为空位
								}
 
								code = code + gb.isArrive[r][c1];// 记录棋子相连的情况
								number++;
							} else if (number == 2) {// 表示第三次出现空位
								if (gb.isArrive[r][c1] == gb.isArrive[r][c]) // 检测两个位置是否都为空位
									break;
							}
						} else {
							if (chess == 0) {// 表示第一次出现棋子
								chess = gb.isArrive[r][c1];// 存储第一次出现棋子
								code = code + gb.isArrive[r][c1]; // 记录棋子的相连情况
							} else if (chess == gb.isArrive[r][c1]) {// 判断是否和第一次出现的棋子同色
								code = code + gb.isArrive[r][c1]; // 记录棋子的相连情况
							} else {// 表示此位置的棋子的颜色与第一次出现的颜色不同，所以记录
								code = code + gb.isArrive[r][c1]; // 记录棋子的相连情况
								break;
							}
 
						}
					}
 
					Integer value = map.get(code);
					if (value != null)// 判断value是否不为null
						weightArray[r][c] += value;// 在对应空位累加权值
 
					// 水平向右
					for (int c1 = c + 1; c1 < gb.isArrive[r].length; c1++) {
						if (gb.isArrive[r][c1] == 0) {
							if (c == c1 - 1) {// 在c1+1处就空了，说明是连续两个空位，就不用再讨论啦
								break;
							}
							// 不是连续两个空位的情况
							else if (number == 0) {// 表示第一次出现空位
								code = code + gb.isArrive[r][c1];// 记录棋子相连的情况
								number++;// 空位的次数加1
							} else if (number == 1) {// 表示第二次出现空位
								if (c1 + 1 < gb.isArrive[r].length
										&& gb.isArrive[r][c1] == gb.isArrive[r][c1 + 1]) {
									break; // 检测两个位置是否都为空位
								}
 
								code = code + gb.isArrive[r][c1];// 记录棋子相连的情况
								number++;
							} else if (number == 2) {// 表示第三次出现空位
								if (gb.isArrive[r][c1] == gb.isArrive[r][c]) // 检测两个位置是否都为空位
									break;
							}
						} else {
							if (chess == 0) {// 表示第一次出现棋子
								chess = gb.isArrive[r][c1];// 存储第一次出现棋子
								code = code + gb.isArrive[r][c1]; // 记录棋子的相连情况
							} else if (chess == gb.isArrive[r][c1]) {// 判断是否和第一次出现的棋子同色
								code = code + gb.isArrive[r][c1]; // 记录棋子的相连情况
							} else {// 表示此位置的棋子的颜色与第一次出现的颜色不同，所以记录
								code = code + gb.isArrive[r][c1]; // 记录棋子的相连情况
								break;
							}
 
						}
					}
					value = map.get(code);
					if (value != null)// 判断value是否不为null
						weightArray[r][c] += value;// 在对应空位累加权值
 
					// 竖直方向
					// 竖直向上检查
					for (int r1 = r - 1; r1 >= 0; r1--) {
						if (gb.isArrive[r1][c] == 0) {
							if (r == r1 + 1) {// 在c1+1处就空了，说明是连续两个空位，就不用再讨论啦
								break;
							}
							// 不是连续两个空位的情况
							else if (number == 0) {// 表示第一次出现空位
								code = code + gb.isArrive[r1][c];// 记录棋子相连的情况
								number++;// 空位的次数加1
							} else if (number == 1) {// 表示第二次出现空位
								if (gb.isArrive[r1][c] == gb.isArrive[r1 + 1][c]) {
									break; // 检测两个位置是否都为空位
								}
 
								code = code + gb.isArrive[r1][c];// 记录棋子相连的情况
								number++;
							} else if (number == 2) {// 表示第三次出现空位
								if (gb.isArrive[r1][c] == gb.isArrive[r][c]) // 检测两个位置是否都为空位
									break;
							}
						} else {
							if (chess == 0) {// 表示第一次出现棋子
								chess = gb.isArrive[r1][c];// 存储第一次出现棋子
								code = code + gb.isArrive[r1][c]; // 记录棋子的相连情况
							} else if (chess == gb.isArrive[r1][c]) {// 判断是否和第一次出现的棋子同色
								code = code + gb.isArrive[r1][c]; // 记录棋子的相连情况
							} else {// 表示此位置的棋子的颜色与第一次出现的颜色不同，所以记录
								code = code + gb.isArrive[r1][c]; // 记录棋子的相连情况
								break;
							}
 
						}
					}
					value = map.get(code);
					if (value != null)// 判断value是否不为null
						weightArray[r][c] += value;// 在对应空位累加权值
 
					// 竖直向下检查
					for (int r1 = r + 1; r1 < gb.isArrive.length; r1++) {
						if (gb.isArrive[r1][c] == 0) {
							if (r == r1 - 1) {// 在c1+1处就空了，说明是连续两个空位，就不用再讨论啦
								break;
							}
							// 不是连续两个空位的情况
							else if (number == 0) {// 表示第一次出现空位
								code = code + gb.isArrive[r1][c];// 记录棋子相连的情况
								number++;// 空位的次数加1
							} else if (number == 1) {// 表示第二次出现空位
								if (r1 + 1 < gb.isArrive.length
										&& gb.isArrive[r1][c] == gb.isArrive[r1 + 1][c]) {
									break; // 检测两个位置是否都为空位
								}
 
								code = code + gb.isArrive[r1][c];// 记录棋子相连的情况
								number++;
							} else if (number == 2) {// 表示第三次出现空位
								if (gb.isArrive[r1][c] == gb.isArrive[r][c]) // 检测两个位置是否都为空位
									break;
							}
						} else {
							if (chess == 0) {// 表示第一次出现棋子
								chess = gb.isArrive[r1][c];// 存储第一次出现棋子
								code = code + gb.isArrive[r1][c]; // 记录棋子的相连情况
							} else if (chess == gb.isArrive[r1][c]) {// 判断是否和第一次出现的棋子同色
								code = code + gb.isArrive[r1][c]; // 记录棋子的相连情况
							} else {// 表示此位置的棋子的颜色与第一次出现的颜色不同，所以记录
								code = code + gb.isArrive[r1][c]; // 记录棋子的相连情况
								break;
							}
 
						}
					}
					value = map.get(code);
					if (value != null)// 判断value是否不为null
						weightArray[r][c] += value;// 在对应空位累加权值
 
					// 右斜方向
					// 向左上
					for (int r1 = r - 1, c1 = c - 1; r1 > 0 && c1 > 0; r1--, c1--) {
 
						if (gb.isArrive[r1][c] == 0) {
							if (r == r1 + 1 && c == c1 + 1) {// 在c1+1处就空了，说明是连续两个空位，就不用再讨论啦
								break;
							}
							// 不是连续两个空位的情况
							else if (number == 0) {// 表示第一次出现空位
								code = code + gb.isArrive[r1][c1];// 记录棋子相连的情况
								number++;// 空位的次数加1
							} else if (number == 1) {// 表示第二次出现空位
								if (r1 + 1 < gb.isArrive.length
										&& c1 + 1 < gb.isArrive[r].length
										&& gb.isArrive[r1][c1] == gb.isArrive[r1 + 1][c1 + 1]) {
									break; // 检测两个位置是否都为空位
								}
 
								code = code + gb.isArrive[r1][c1];// 记录棋子相连的情况
								number++;
							} else if (number == 2) {// 表示第三次出现空位
								if (gb.isArrive[r1][c1] == gb.isArrive[r][c]) // 检测两个位置是否都为空位
									break;
							}
						} else {
							if (chess == 0) {// 表示第一次出现棋子
								chess = gb.isArrive[r1][c1];// 存储第一次出现棋子
								code = code + gb.isArrive[r1][c1]; // 记录棋子的相连情况
							} else if (chess == gb.isArrive[r1][c1]) {// 判断是否和第一次出现的棋子同色
								code = code + gb.isArrive[r1][c1]; // 记录棋子的相连情况
							} else {// 表示此位置的棋子的颜色与第一次出现的颜色不同，所以记录
								code = code + gb.isArrive[r1][c1]; // 记录棋子的相连情况
								break;
							}
 
						}
					}
					value = map.get(code);
					if (value != null)// 判断value是否不为null
						weightArray[r][c] += value;// 在对应空位累加权值
 
					// 向右下
					for (int r1 = r + 1, c1 = c + 1; r1 < gb.isArrive.length
							&& c1 < gb.isArrive[r].length; r1++, c1++) {
 
						if (gb.isArrive[r1][c1] == 0) {
							if (r == r1 - 1 && c == c1 - 1) {// 在c1+1处就空了，说明是连续两个空位，就不用再讨论啦
								break;
							}
							// 不是连续两个空位的情况
							else if (number == 0) {// 表示第一次出现空位
								code = code + gb.isArrive[r1][c1];// 记录棋子相连的情况
								number++;// 空位的次数加1
							} else if (number == 1) {// 表示第二次出现空位
								if (r1 + 1 < gb.isArrive.length
										&& c1 + 1 < gb.isArrive[r1].length
										&& (gb.isArrive[r1][c1] == gb.isArrive[r1 + 1][c1 + 1])) {
									break; // 检测两个位置是否都为空位
								}
 
								code = code + gb.isArrive[r1][c1];// 记录棋子相连的情况
								number++;
							} else if (number == 2) {// 表示第三次出现空位
								if (gb.isArrive[r1][c1] == gb.isArrive[r][c]) // 检测两个位置是否都为空位
									break;
							}
						} else {
							if (chess == 0) {// 表示第一次出现棋子
								chess = gb.isArrive[r1][c1];// 存储第一次出现棋子
								code = code + gb.isArrive[r1][c1]; // 记录棋子的相连情况
							} else if (chess == gb.isArrive[r1][c1]) {// 判断是否和第一次出现的棋子同色
								code = code + gb.isArrive[r1][c1]; // 记录棋子的相连情况
							} else {// 表示此位置的棋子的颜色与第一次出现的颜色不同，所以记录
								code = code + gb.isArrive[r1][c1]; // 记录棋子的相连情况
								break;
							}
 
						}
 
					}
					value = map.get(code);
					if (value != null)// 判断value是否不为null
						weightArray[r][c] += value;// 在对应空位累加权值
 
					// 左斜方向
					// 向右上
					for (int r1 = r - 1, c1 = c + 1; r1 >= 0
							&& c1 < gb.isArrive[r].length; r1--, c1++) {
						if (gb.isArrive[r1][c1] == 0) {
							if (r == r1 + 1 && c == c1 - 1) {// 在c1+1处就空了，说明是连续两个空位，就不用再讨论啦
								break;
							}
							// 不是连续两个空位的情况
							else if (number == 0) {// 表示第一次出现空位
								code = code + gb.isArrive[r1][c1];// 记录棋子相连的情况
								number++;// 空位的次数加1
							} else if (number == 1) {// 表示第二次出现空位
								if (c1 + 1 < gb.isArrive[r].length
										&& r1 - 1 > 0
										&& gb.isArrive[r1][c1] == gb.isArrive[r1 - 1][c1 + 1]) {
									break; // 检测两个位置是否都为空位
								}
 
								code = code + gb.isArrive[r1][c1];// 记录棋子相连的情况
								number++;
							} else if (number == 2) {// 表示第三次出现空位
								if (gb.isArrive[r1][c1] == gb.isArrive[r][c]) // 检测两个位置是否都为空位
									break;
							}
						} else {
							if (chess == 0) {// 表示第一次出现棋子
								chess = gb.isArrive[r1][c1];// 存储第一次出现棋子
								code = code + gb.isArrive[r1][c1]; // 记录棋子的相连情况
							} else if (chess == gb.isArrive[r1][c1]) {// 判断是否和第一次出现的棋子同色
								code = code + gb.isArrive[r1][c1]; // 记录棋子的相连情况
							} else {// 表示此位置的棋子的颜色与第一次出现的颜色不同，所以记录
								code = code + gb.isArrive[r1][c1]; // 记录棋子的相连情况
								break;
							}
 
						}
 
					}
					value = map.get(code);
					if (value != null)// 判断value是否不为null
						weightArray[r][c] += value;// 在对应空位累加权值
 
					// 向左下
					for (int r1 = r + 1, c1 = c - 1; r1 < gb.isArrive.length
							&& c1 > 0; r1++, c1--) {
						if (gb.isArrive[r1][c1] == 0) {
							if (r == r1 - 1 && c == c1 + 1) {// 在c1+1处就空了，说明是连续两个空位，就不用再讨论啦
								break;
							}
							// 不是连续两个空位的情况
							else if (number == 0) {// 表示第一次出现空位
								code = code + gb.isArrive[r1][c1];// 记录棋子相连的情况
								number++;// 空位的次数加1
							} else if (number == 1) {// 表示第二次出现空位
								if (r1 + 1 < gb.isArrive.length
										&& c1 - 1 > 0
										&& gb.isArrive[r1][c1] == gb.isArrive[r1 + 1][c1 - 1]) {
									break; // 检测两个位置是否都为空位
								}
 
								code = code + gb.isArrive[r1][c1];// 记录棋子相连的情况
								number++;
							} else if (number == 2) {// 表示第三次出现空位
								if (gb.isArrive[r1][c1] == gb.isArrive[r][c]) // 检测两个位置是否都为空位
									break;
							}
						} else {
							if (chess == 0) {// 表示第一次出现棋子
								chess = gb.isArrive[r1][c1];// 存储第一次出现棋子
								code = code + gb.isArrive[r1][c1]; // 记录棋子的相连情况
							} else if (chess == gb.isArrive[r1][c1]) {// 判断是否和第一次出现的棋子同色
								code = code + gb.isArrive[r1][c1]; // 记录棋子的相连情况
							} else {// 表示此位置的棋子的颜色与第一次出现的颜色不同，所以记录
								code = code + gb.isArrive[r1][c1]; // 记录棋子的相连情况
								break;
							}
 
						}
 
					}
 
					value = map.get(code);
					if (value != null)// 判断value是否不为null
						weightArray[r][c] += value;// 在对应空位累加权值
 
				}
			}
		}
	}
```

