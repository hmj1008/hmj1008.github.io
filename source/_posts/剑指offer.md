---
title: 剑指offer
date: 2017-04-05 11:22:06
toc: true
tags:
    - 代码
    - 阅读
    - python
---
> 
《剑指Offer》系统地总结了如何在面试时写出高质量代码，如何优化代码效率，以及分析、解决难题的常用方法。书中对于问题算法的给出C++描述。本人阅读该书，深感有所收获和启发。选取部分有意思的问题及算法，python实现。
<!-- more -->

# 数组
## 输入一个数字N，按顺序打印从1到最大的n位十进制数

```python
'''
问题：输入一个数字N，按顺序打印从1到最大的n位十进制数
eg:
	input:3
	output: 1,2, ...,999
'''
class Solution(object):
	# #########################
	#  错误解法：
	#  求的最大n位数，循环打印。
	#  忽略当n很大时，大数溢出。
	# #########################
	def Print1ToMaxOfN_wrong(self,n):
		number = 1
		i = 0
		while i < n:
			# n = n * 10
			number *= 10 
			i += 1
		for j in range(1,number):
			print (j)

	###################################################
	#  正确解法1：
	#  用数组保存各位数字模拟数字加法，绕过大数溢出陷阱
	####################################################
	def Print1ToMaxofN_right_1(self,n):
		if n < 0:
			return False
		# 准备一个N位的数组，里面有N个0，初始化一个N位的 "数字"
		number_str = []
		for i in range(0,n):
			number_str.append(0)
		# 给"数字"加1后，并打印出该"数字"
		while not self.IncrementNumberStr(number_str=number_str,length=len(number_str)):
			self.PrintNumberStr(number_str)

	# 模拟加法加1，遇10进位。
	def IncrementNumberStr(self,number_str,length):
		# 终止加1的判定条件，为True终止
		is_overflow = False
		# 从"数字"的最低位开始执行加1操作
		i = length - 1
		# 进位多少
		nTakeOver = 0
		# 开始对"数字"执行加1
		while i >= 0:
			nSum = number_str[i] + nTakeOver
			if i == length - 1:
				# 加1
				nSum += 1
			# 遇10进1，直到不用进位
			if nSum >= 10:
				if i == 0:
					is_overflow = True
				else:
					nSum -= 10
					nTakeOver = 1
					number_str[i] = nSum
			else:
				# 不用进位，赋值，退出while循环
				number_str[i] = nSum
				break
			# 指向"更高位"数字
			i -= 1
		return is_overflow

	# 打印"数字"，从"最高位"非0开始打印：
	# 1,2,3...,10,11,...,998,999
	def PrintNumberStr(self,number_str):
		length = len(number_str)
		is_begin_0 = True
		result_number = []
		# 从"最高位"下标为0开始遍历"数字"
		for i in range(0,length):
			if is_begin_0 and number_str[i] != 0:
				is_begin_0 = False
			if not is_begin_0:
				result_number.append(number_str[i])
		print (result_number)
	
	###################################################
	#  正确解法2：
	#  全排列递归数字的每一位,
	####################################################
	def Print1ToMaxN_rught_2(self,n):
		if n < 0:
			return False
		number = []
		for i in range(0,n):
			number.append(0)
		for j in range(0,10):
			number[0] = j
			self.Print1ToMaxN_Recursively(number,n,0)
	
	def Print1ToMaxN_Recursively(self,number,length,index):
		if index == length - 1:
			self.PrintNumberStr(number_str=number)
			return
		for k in range(0,10):
			number[index+1] = k
			self.Print1ToMaxN_Recursively(number=number,length=length,index=index+1)

```
## 把数组中的数字区分两部分，奇数在前部分，偶数在后

```python
'''
问题：把数组中的数字区分开来，奇数在前部分，偶数在后
eg:
	input:  [1,2,3,4,6,9]
	output: [1,3,9,2,4,6]
'''
class Solution(object):
	# 设置两个指针：偶指针，奇指针
	# 初始化偶指针指向第一个元素，奇指针指向最后一个元素
	# 偶指针往数组后面元素遍历，直到遇到偶数
	# 奇指针往数组前面元素遍历，直到遇到奇数
	# 交换两个指针指向的元素位置
	def ReorderOddEven(self,number_arr):
		length = len(number_arr)
		if length == 0:
			print ("数组为空！")
			return False
		p_begin = 0
		p_end = length - 1
		while p_begin < p_end:
			# p_begin 指向偶数
			#while p_begin < p_end and number_arr[p_begin] % 2 != 0:
			while p_begin < p_end and not self.PartCondition(number_arr[p_begin]):
				p_begin += 1
			# p_end 指向奇数
			#while p_begin < p_end and number_arr[p_end] % 2 == 0:
			while p_begin < p_end and self.PartCondition(number_arr[p_end]):
				p_end -= 1
			# 交换奇偶数
			temp = number_arr[p_begin]
			number_arr[p_begin] = number_arr[p_end]
			number_arr[p_end] = temp
		print ("完成划分：")
		print (number_arr)

	# 定义一个划分数组成两部分的条件函数
	def PartCondition(self,number):
		if number % 2 == 0:
			return True
		else:
			return False
```

## 在二维数组中找n，判断是否在其中

```python
'''
问题：输入一个整数n 和 二维数组matrix，行列有序，升序。
判断n是否在matrix中
e.g
    input: matrix =[[1,2, 8, 9],
                    [2,4, 9,12],
                    [4,7,10,13],
                    [6,8,11,15]]
            n = 11
    output: True
'''
class Solution(object):
    # 由于矩阵行列有序且升序，从矩阵中的右上角开始比较，
    # 即：从第一行最后一个数字开始比较,以缩小查找范围
    # 比如上面的矩阵中查找11的路线：
    # 9[0,3] -> 12[1,3] -> 9[1,2] -> 10[2,2] -> 11[3,2]
    def findOne(self, matrix,n):
        found = False
        if matrix != None:
            rows_num = len(matrix)       #行数
            columns_num = len(matrix[0]) #列数
            if rows_num > 0 and columns_num > 0:
                # 初始比较位置：[第一行,最后一列]
                row = 0
                col = columns_num - 1
                while row < rows_num and col >= 0:
                    # 正好匹配，返回True,结束循环
                    if matrix[row][col] == n:
                        found = True
                        break
                    # 该行该列元素小于目标元素，只能在下一行找，
                    elif matrix[row][col] < n:
                        row += 1
                    # 该行该列元素大于目标元素，只能在下一列找，
                    else:
                        col -= 1
        return found
```

## 快速排序

```python
'''
快速排序算法：
        input: arr = [2,4,1,6,3]

        随机[0,4]=3 => arr[3] => 6
        第一次划分结果: arr_left=[2,4,1,3] arr_right=[] => [[2,4,1,3,]6[]]
        如上原理对arr_left,arr_right进行划分

        output: [1,2,3,4,6]
'''
import random
class Solution(object):
        # 交换数组arr中第i和第j个元素
        def Swap(self,arr,i,j):
                temp = arr[i]
                arr[i] = arr[j]
                arr[j] = temp

        # 以[begin,end]中间某个值进行小的在它左边、大的在右边划分，
        def partition(self,data_arr,length,begin,end):
                if length <= 0 or begin < 0 or end >length:
                        print ("invaild parameters")
                        return False
                # 随机获数组中任意一个数字data_arr[index]
                index = random.randint(begin,end)
                # Swap
                self.Swap(data_arr,index,end)
                small = begin - 1
                for index in range(begin,end):
                        if data_arr[index] < data_arr[end]:
                                small += 1
                                if small != index:
                                        # Swap
                                        self.Swap(data_arr,small,index)
                small += 1
                # data_arr = [... < data_arr[small] < ...]
                self.Swap(data_arr,small,end)
                return small

        # 递归划分，直到数组有序
        def QuickSort(self,arr,length,begin,end):
                if begin == end:
                        return 
                index = self.partition(data_arr=arr,length=length,begin=0,end=end)
                if index > begin:
                        self.QuickSort(arr=arr,length=length,begin=begin,end=index-1)
                if index < end:
                        self.QuickSort(arr=arr,length=length,begin=index+1,end=end)

```

## 求字符串的全排列
```python
string = ['a','b','c','\n']  
def Permutation(string,i):  
    if string == None:  
        return  
    if string[i] == '\n':  
        print(str(i)+"%s\n"%string)  
    else:  
        for j in range(i,len(string)-1):   
            print (str(j)+":"+str(i))  
            string[j],string[i] = string[i],string[j]  
            Permutation(string,i+1)  
            string[j],string[i] = string[i],string[j]  
Permutation(string,0) 
```

# 链表

## 链表的部分基本方法实现
```python
'''
链表:
	head->{v1,next}->{v2,next}->{v3,next}->None
	目前实现方法有：
		给链表尾部添加节点: addNode(value) 
		在第index个节点,插入新节点值为value: addNodeToIndex(index,value)
		获取链表长度: getListNodeSize()
		顺序遍历: visiteListNode()
		逆序遍历: visitListReversingly()
		按值删除节点: removeNodeByValue
		按序删除第index节点: removeNodeByIndex()
'''
# 节点定义
class Node(object):
	m_Value = None
	m_Next = None
	def __init__(self,v):
		self.m_Value = v

class ListNode(object):
	head = None
	size = 0

	#给链表尾部添加节点
	def addNode(self,value):
		node = Node(value)
		if self.head:
			p = self.head
			while p.m_Next != None:
				p = p.m_Next
			p.m_Next = node
		else:
			self.head = node
		self.size = self.size + 1
	
	#在第index个节点，插入新节点
	def addNodeToIndex(self,index,value):
		print ("往第{}添加节点".format(index))
		if index < 1 or index > self.getListNodeSize():
			print ("不合法index，1<=index<={}".format(self.getListNodeSize()))
			return False
		if index == 1:
			node = Node(value)
			node.m_Next = self.head
			self.head = node
			self.size = self.size + 1
		else:
			p = self.head
			pre = self.head
			i = 1
			while p.m_Next!=None and i<index:
				pre = p
				p = p.m_Next
				i = i+1
			node = Node(value)
			node.m_Next = p
			pre.m_Next=node
			self.size = self.size + 1
		print ("往第{}添加节点，ok!".format(index))

	#获取链表长度
	def getListNodeSize(self):
		return self.size

	#顺序遍历
	def visiteListNode(self):
		res = []
		p = self.head
		if p:
			if p.m_Value:
				res.append(p.m_Value)
			while p.m_Next != None:
				res.append(p.m_Next.m_Value)
				p = p.m_Next
		print ("链表顺序遍历：{}".format(res))

	#逆序遍历,顺序遍历链表，递归算法，相当于用栈(先进后出)
	def visitListReversingly(self,node):
		pass

	#按值删除节点
	def removeNodeByValue(self,value):
		found = False
		if not self.head:
			print ("欲删除：{}，但是链表为空,删毛线！".format(value))
			return found
		print ("欲删除节点值：{}".format(value))
		#第一个节点匹配删除
		if self.head.m_Value == value:
			found = True
			if self.head.m_Next:
				self.head = self.head.m_Next
			else:
				self.head = None
			self.size = self.size - 1
		#非第一个节点匹配删除
		else:
			p = self.head
			while p.m_Next!=None and p.m_Next.m_Value!=value:
				p = p.m_Next
			if p.m_Next!=None and p.m_Next.m_Value == value:
				found = True
				self.size = self.size - 1
				p.m_Next = p.m_Next.m_Next
		if found:
			print ("删除节点值：{},ok!".format(value))
		else:
			print ("删除节点值：{},失败，没有匹配节点!".format(value))

	#按序删除第index节点
	def removeNodeByIndex(self,index):
		print ("把第{}个节点删除".format(index))
		if index < 1 or index > self.getListNodeSize():
			print ("不合法index:1<=index<={}".format(self.getListNodeSize())))
			return False
		if index == 1:
			self.head = self.head.m_Next
			self.size = self.size - 1
		else:
			p = self.head
			pre = self.head
			i = 1
			while p.m_Next!=None and i<index:
				pre = p
				p = p.m_Next
				i = i+1
			pre.m_Next = p.m_Next
			self.size = self.size - 1
		print ("把第{}个节点删除，ok!".format(index))
```

## 反转链表

```python
'''
反转链表
定义一个函数，输入一个链表的头结点，
反转链表所有节点的顺序，返回反转后的头结点
eg:
	input: phead = {2 -> 3 -> 5 -> 1}
	output: head = {1 -> 5 -> 3 -> 2}
'''
# 节点定义
class Node(object):
	m_Value = None
	m_Next = None
	def __init__(self,v):
		self.m_Value = v

class Solution(object):
	# 反转链表，首先考虑不要出现断链，然后考虑头结点为空的情况
	def ReverseList(self,listHead):
		# 保存反转后的头结点
		pReverseListHead = None
		pPrevNode = None
		pNode = listHead
		# 从第一个结点开始遍历，进行反转
		while pNode:
			# 临时保存下一个结点的，防止出现断链
			pNodeNext = pNode.m_Next
			# 当链表的最后一个结点的m_Next为空时，即为反转完成，
			# 赋值给pReverseListHead
			if pNodeNext == None:
				pReverseListHead = pNode
			# 当前结点的m_Next指向上一个结点
			pNode.m_Next = pPrevNode
			# 当前结点变成上一个结点
			pPrevNode = pNode
			# 临时保存下一个结点发挥作用，继续循环下一个结点反转
			pNode = pNodeNext
		return pReverseListHead

	def visiteListNode(self,listHead):
		res = []
		p = listHead
		if p:
			if p.m_Value:
				res.append(p.m_Value)
			while p.m_Next != None:
				res.append(p.m_Next.m_Value)
				p = p.m_Next
		print ("链表顺序遍历：{}".format(res))

# TEST
listHead = Node(2)
listHead.m_Next = Node(1)
listHead.m_Next.m_Next = Node(4)
listHead.m_Next.m_Next.m_Next = Node(6)
s = Solution()
s.visiteListNode(listHead=listHead) 
# 链表顺序遍历：[2,1,4,6]
s.visiteListNode(listHead=s.ReverseList(listHead=listHead))
# 链表顺序遍历：[6,4,1,2]
s.visiteListNode(listHead=s.ReverseList(listHead=None)) 
# 链表顺序遍历：[]
```

## 合并两个升序的链表，返回合并之后的升序链表头结点
```python
'''
合并两个升序的链表，返回合并之后的升序链表头结点
eg:
	input: a_head = {2 -> 3 -> 5 -> 7}
		   b_head = {1 -> 2 -> 4 -> 8}
	output: c_head = {1 -> 2 -> 2 -> 3 -> 4 -> 5 -> 7 -> 8}
'''
# 节点定义
class Node(object):
	m_Value = None
	m_Next = None
	def __init__(self,v):
		self.m_Value = v

class Solution(object):
	# 两个链表都是升序的，所以从两个头结点开始，较小者为头结点，递归算法：
	def Merge(self,a_head,b_head):
		# 考虑边界条件
		if a_head == None:
			return b_head
		if b_head == None:
			return a_head
		pMergeListHead = None
		if a_head.m_Value < b_head.m_Value:
			pMergeListHead = a_head
			pMergeListHead.m_Next = self.Merge(a_head.m_Next,b_head)
		else:
			pMergeListHead = b_head
			pMergeListHead.m_Next = self.Merge(a_head,b_head.m_Next)
		return pMergeListHead

	# 遍历头结点为listHead的链表
	def visiteListNode(self,listHead):
		res = []
		p = listHead
		if p:
			if p.m_Value:
				res.append(p.m_Value)
			while p.m_Next != None:
				res.append(p.m_Next.m_Value)
				p = p.m_Next
		print ("链表顺序遍历：{}".format(res))

# TEST
a_listHead = Node(2)
a_listHead.m_Next = Node(3)
a_listHead.m_Next.m_Next = Node(5)
a_listHead.m_Next.m_Next.m_Next = Node(7)

b_listHead = Node(1)
b_listHead.m_Next = Node(2)
b_listHead.m_Next.m_Next = Node(4)
b_listHead.m_Next.m_Next.m_Next = Node(8)

s = Solution()
s.visiteListNode(a_listHead) 
# 链表顺序遍历：[2,3,5,7]
s.visiteListNode(b_listHead) 
# 链表顺序遍历：[1,2,4,8]
s.visiteListNode(s.Merge(a_head=a_listHead,b_head=b_listHead)) 
# 链表顺序遍历：[1,2,2,3,4,5,7,8]

```

# 二叉树

## 二叉树的构造和遍历
```python
'''
二叉树

'''
# 树节点定义
class TreeNode(object):
	m_value = None
	m_right = None
	m_left = None
	def __init__(self,value=None):
		self.m_value = value

# 二叉树
class Tree(object):
	root = None
	def __init__(self,node=None):
		if node == None:
			self.root = TreeNode()
		else:
			self.root = node

	# 添加节点，按层次添加
	def add(self,value):
		add_node = TreeNode(value)
		#如果树是空树，给根节点赋值
		if self.root.m_value == None:
			self.root = add_node
			print ("add "+str(value)+" to root")
			return True
		else:
			treeNodesQueue = []
			treeNodesQueue.append(self.root)
			while treeNodesQueue:
				# 弹出先入队的节点
				node = treeNodesQueue.pop(0)
				# 判断节点左右节点是否为空，
				# 如为空，添加节点，结束循环
				if node.m_left == None:
					node.m_left = add_node
					print ("add "+str(value)+"to "+str(node.m_value)+" left")
					return True
				if node.m_right == None:
					node.m_right = add_node
					print ("add "+str(value)+"to "+str(node.m_value)+" right")
					return True
				# 节点左右不为空，则继续
				treeNodesQueue.append(node.m_left)
				treeNodesQueue.append(node.m_right)
	
	# 先序递归遍历	
	def preOrderRecursiveVisite(self,root):
		if root == None:
			return
		print (root.m_value)
		self.preOrderRecursiveVisite(root.m_left)
		self.preOrderRecursiveVisite(root.m_right)

	# 中序递归遍历	
	def midleOrderRecursiveVisite(self,root):
		if root == None:
			return
		self.midelOrderRecursiveVisite(root.m_left)
		print (root.m_value)
		self.midelOrderRecursiveVisite(root.m_right)

	# 后序递归遍历	
	def afterOrderRecursiveVisite(self,root):
		if root == None:
			return
		self.afterOrderRecursiveVisite(root.m_left)
		self.afterOrderRecursiveVisite(root.m_right)
		print (root.m_value)
		
	# 利用队列按层次遍历
	def levelOrderVisiteByQueue(self,root):
		if root == None:
			return False
		treeNodesQueue = []
		treeNodesQueue.append(root)
		while treeNodesQueue:
			node = treeNodesQueue.pop(0)
			print (node.m_value)
			if node.m_left != None:
				treeNodesQueue.append(node.m_left)
			if node.m_right != None:
				treeNodesQueue.append(node.m_right)	
# TEST
s = Tree()
for i in range(0,8):
	s.add(i)
s.midelOrderRecursiveVisite(s.root)
s.preOrderRecursiveVisite(s.root)
s.afterOrderRecursiveVisite(s.root)
s.levelOrderVisiteByQueue(s.root)
```

## 判断树的子结构
```python
'''
输入两棵二叉树A和B,判断B是不是A的子结构：
	输入TreeA 和TreeB 的根节点，
	若TreeB 的结构是TreeA的一部分，返回ture,否则false
'''
class Solution(object):
	# 判断A中某个节点是否有和B树的根节点相等
	def HasSubTree(self,treeA,treeB):
		result = False
		# 防止空指针
		if treeA == None or treeB == None:
			return result
		# 遍历树A的每个节点，和树B的根节点比较，这是子树的基础条件
		if treeA.m_value == treeB.m_value:
			# 从树A匹配的节点开始，和树B的剩余节点依次匹配
			result = self.DoesTreeAHaveTreeB(treeA,treeB)
		if not result:
			result = self.HasSubTree(treeA.m_left,treeB)
		if not result:
			result = self.HasSubTree(treeA.m_right,treeB)
		return result

	def DoesTreeAHaveTreeB(self,treeA,treeB):
		# 当子树遍历完了后，可以确定是母树A的子结构
		if treeB == None:
			return True
		# 当母树A遍历到为空节点，子树仍有节点，判断两者结构不是包含关系
		if treeA == None:
			return False
		# 节点值不匹配，直接返回不匹配
		if treeA.m_value != treeB.m_value:
			return False
		# 递归依次匹配两树的左右子节点
		result_left = self.DoesTreeAHaveTreeB(treeA.m_left,treeB.m_left)
		result_right = self.DoesTreeAHaveTreeB(treeA.m_right,treeB.m_right)
		return result_left and result_right
# TEST
a = Tree()
for i in range(0,8):
	a.add(i)
b = Tree()
for i in range(0,4):
	b.add(i)
s = Solution()
print(s.HasSubTree(a.root,b.root))
```
## 镜像二叉树
```python
'''
输入一棵二叉树的根节点
以根节点为中心，对这课二叉树进行各子左右节点镜像
把非叶子节点的子节点进行左右对换，递归进行
'''
# 树节点定义
class TreeNode(object):
	m_value = None
	m_right = None
	m_left = None
	def __init__(self,value=None):
		self.m_value = value

class Solution(object):

	def mirrorTree(self,tree):
		if tree == None:
			return False
		# 叶子节点不作处理
		if tree.m_left == None and tree.m_right == None:
			return False
		# 交换左右节点
		tempNode = tree.m_left
		tree.m_left = tree.m_right
		tree.m_right = tempNode
		# 递归左右非叶子节点
		if tree.m_right:
			self.mirrorTree(tree.m_right)
		if tree.m_left
			self.mirrorTree(tree.m_left)
```
## 二叉搜索树的后序遍历序列与数组比对
```python
'''
二叉搜索树的后序遍历序列
问题：
	输入一个整数数组，判断该数组是不是某二叉搜索树的后序遍历结果.
	如果是返回 true, 否则返回 false.
	假设输入的数组任意两个元素互不相同.

二叉搜索树： 左子节点比根节点小，右子节点比根节点大

eg:
	data_arr = [5,7,6,9,11,10,8] 7
	True
'''
class Solution(object):

	def VserifySquenceOfBST(self,data_arr,beginIndex,endIndex):

		# 当数组中只有一个元素，直接返回True
		if endIndex - beginIndex == 0:
			return True
		# 取出数组中最后一个元素作为二叉搜索树的根节点
		root = data_arr[endIndex]
		i = beginIndex
		# 找比根节点大的元素开始位置
		while i < endIndex:
			if data_arr[i] > root:
				break
			i += 1
		# 从开始比根节点大的元素开始遍历剩余元素，
		# 如果出现比根节点小的元素，则不符合二叉搜索树的定义
		j = i
		while j < endIndex:
			if data_arr[j] < root:
				return False
			j += 1
		# 判断左子树是不是二叉搜索树
		left = True
		if i > 0:
			left = self.VserifySquenceOfBST(data_arr=data_arr,beginIndex=beginIndex,endIndex=i-1)
		# 判断右子树是不是二叉搜索树
		right = True
		if i < endIndex:
			right = self.VserifySquenceOfBST(data_arr=data_arr,beginIndex=i,endIndex=endIndex-1)
		return left and right

# TEST
s = Solution()
arr = [5,7,6,9,11,10,8]
print (s.VserifySquenceOfBST(data_arr=arr,beginIndex=0,endIndex=len(arr)-1)) # True
```
## 二叉树中和为某一值得路径
```python
'''
 二叉树中和为某一值得路径
 输入一棵二叉树和一个整数，打印出二叉树节点值的和为输入数值的所有路径。
 路径：从树根节点开始往下一直到叶节点所经过的节点形成以条路径。
'''
class Solution(object):
	# 利用python中的list append() 尾插和 pop()尾弹
	# 来保存先序遍历二叉树所经过的节点值，利用递归调用的特性。
	# 适当时候从路径中删除节点。
	def findPath(self,TreeRoot,expectSum):
		if TreeRoot == None:
			return
		path = []
		c = 0
		self.findPadthDeatil(TreeRoot=TreeRoot,path=path,expectSum=expectSum,currentSum=c)

	def findPadthDeatil(self,TreeRoot,path,expectSum,currentSum):
		currentSum += TreeRoot.m_value
		# 往数组中尾插入一个元素
		path.append(TreeRoot.m_value)
		# 到达叶子节点，所经历的节点值累加和为期望值
		is_leaf = TreeRoot.m_right == None and TreeRoot.m_left == None
		if currentSum == expectSum and is_leaf:
			print ("find a path:")
			print (path)
		# 不是叶子节点，遍历其子节点
		if TreeRoot.m_left!= None:
			self.findPadthDeatil(TreeRoot=TreeRoot.m_left,path=path,expectSum=expectSum,currentSum=currentSum)
		if TreeRoot.m_right != None:
			self.findPadthDeatil(TreeRoot=TreeRoot.m_right,path=path,expectSum=expectSum,currentSum=currentSum)
		# 在返回父节点之前，在路径中删除当前节点
		# 并在currentSum中减去当前节点值
		currentSum -= TreeRoot.m_value
		# 从数组中弹出最后一个元素
		path.pop()

# 树节点定义
class TreeNode(object):
	m_value = None
	m_right = None
	m_left = None
	def __init__(self,value=None):
		self.m_value = value
# TEST
root = TreeNode(10)
root.m_left = TreeNode(5)
root.m_right = TreeNode(12)
root.m_left.m_left = TreeNode(4)
root.m_left.m_right = TreeNode(7)
s = Solution()
s.findPath(TreeRoot=root,expectSum=22)
```
# 栈
## 栈内最小元素的mini函数时间复杂度为O(1)
```python
'''
定义栈的数据结构，在该类型中实现一个能够得到栈内最小元素的mini函数,
调用该栈中的pop,push,mini的时间复杂度为O(1)
'''
# 利用辅助栈，在每次对栈的push操作时，
# 把最小元素push到辅助栈中保存
# m_data 数据栈
# m_mini 辅助栈

class Solution(object):
	# 初始化m_data 数据栈，m_mini 辅助栈
	def __init__(self):
		self.m_data = Strack()
		self.m_mini = Strack()
	# 压入数据
	def push(self,value):
		if value == None:
			return False
		self.m_data.push(value)
		# 把最小的元素压入辅助栈中
		if self.m_mini.size() == 0 or value < self.m_mini.top():
			self.m_mini.push(value)
		else:
			self.m_mini.push(self.m_mini.top())
		return True
	
	# 弹出数据
	def pop(self):
		if self.m_mini.size() > 0 and self.m_mini.size() > 0:
			self.m_mini.pop()
			return self.m_data.pop()
		else:
			return False

	# 取栈中的最小元素
	def mini(self):
		if self.m_mini.size() > 0 and self.m_mini.size() > 0:
			return self.m_mini.top()
		else:
			print ("栈空!")
			return False

# 其实python里的list已经有栈的思想，这里在其基础上
# 栈的定义
class Strack(object):
	# 初始化一个list来作为栈的存储数据
	def  __init__(self):
		self.s = []
	# 入栈
	def push(self,value):
		self.s.append(value)
	# 出栈
	def pop(self):
		if self.size() == 0:
			print ("栈空,无法pop!")
			return False
		print ("弹出栈顶元素：{}".format(self.top()))
		return self.s.pop()
	# 取栈顶元素
	def top(self):
		if self.size() == 0:
			print ("栈空,无法top!")
			return False
		return self.s[self.size()-1]
	# 获取栈的长度
	def size(self):
		return len(self.s)

# TEST
# s = Solution()
# s.push(3)
# s.push(1)
# s.push(4)
# s.pop()
# print (s.mini())
# s.pop()
# print (s.mini())
```
