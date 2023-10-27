

![linkedList.svg](https://cdn.nlark.com/yuque/0/2020/svg/1512483/1592275761196-92bd7860-2cb1-4baf-b3b7-decd205858ee.svg#align=left&display=inline&height=49&originHeight=41&originWidth=408&size=12690&status=done&style=none&width=483)

链表结构中主要包含头结点，尾结点和中间结点  一个结点包含数据以及指向下个结点的指针
## LinkedListNode
```javascript
export default class LinkedListNode {
  constructor(value, next = null) {
    this.value = value; //存放值
    this.next = next; // 指向下一个结点
  }

  toString(callback) {
    return callback ? callback(this.value) : `${this.value}`;
  }
}

```


## LinkedList
```javascript


export default class LinkedList {
  /**
   * @param {Function} [comparatorFunction]
   */
  constructor(comparatorFunction) {
    // 初始化 头结点 尾结点 引入值对比函数
    this.head = null;  
    this.tail = null;
    this.compare = new Comparator(comparatorFunction);
  }

  /**
   * @param {*} value
   * @return {LinkedList}
   * 前置结点添加
   */
  prepend(value) {
    // 新结点成为一个头结点
    const newNode = new LinkedListNode(value, this.head);
    this.head = newNode;

    // 如果是第一个结点 那么既是头也是尾
    if (!this.tail) {
      this.tail = newNode;
    }

    return this;
  }

  /**
   * @param {*} value
   * @return {LinkedList}
   * 后置结点添加
   */
  append(value) {
    const newNode = new LinkedListNode(value);

    // If there is no head yet let's make new node a head.
    if (!this.head) {
      this.head = newNode;
      this.tail = newNode;

      return this;
    }

    // 上一个尾部结点指向新结点 新结点成为尾部结点
    this.tail.next = newNode;
    this.tail = newNode;

    return this;
  }

  /**
   * @param {*} value
   * @return {LinkedListNode}
   */
  delete(value) {
    // 没有头结点 还遍历个p
    if (!this.head) {
      return null;
    }

    let deletedNode = null;

    // If the head must be deleted then make next node that is differ
    // from the head to be a new head.
    while (this.head && this.compare.equal(this.head.value, value)) {
      deletedNode = this.head;
      this.head = this.head.next;
    }

    let currentNode = this.head;

    if (currentNode !== null) {
    
      while (currentNode.next) {
        if (this.compare.equal(currentNode.next.value, value)) {
          // 满足删除条件 把当前结点指针指向下结点的下个结点
          deletedNode = currentNode.next;
          currentNode.next = currentNode.next.next;
        } else {
          // 否则 继续遍历
          currentNode = currentNode.next;
        }
      }
    }

    // 如果是尾结点满足删除条件 上面已经处理了 currentNode是尾结点上个结点
    // 这里重新指下尾结点
    if (this.compare.equal(this.tail.value, value)) {
      this.tail = currentNode;
    }

    return deletedNode;
  }

  /**
   * @param {Object} findParams
   * @param {*} findParams.value
   * @param {function} [findParams.callback]
   * @return {LinkedListNode}
   */
  find({ value = undefined, callback = undefined }) {
    if (!this.head) {
      return null;
    }

    let currentNode = this.head;

    while (currentNode) {
      // 若自定义查找方法callback 使用其查找 
      if (callback && callback(currentNode.value)) {
        return currentNode;
      }

      // If value is specified then try to compare by value..
      if (value !== undefined && this.compare.equal(currentNode.value, value)) {
        return currentNode;
      }

      currentNode = currentNode.next;
    }

    return null;
  }

  /**
   * @return {LinkedListNode}
   */
  deleteTail() {
    const deletedTail = this.tail;

    if (this.head === this.tail) {
      // There is only one node in linked list.
      this.head = null;
      this.tail = null;

      return deletedTail;
    }

    // If there are many nodes in linked list...

    // Rewind to the last node and delete "next" link for the node before the last one.
    let currentNode = this.head;
    while (currentNode.next) {
      if (!currentNode.next.next) {
        currentNode.next = null;
      } else {
        currentNode = currentNode.next;
      }
    }

    this.tail = currentNode;

    return deletedTail;
  }

  /**
   * @return {LinkedListNode}
   */
  deleteHead() {
    if (!this.head) {
      return null;
    }

    const deletedHead = this.head;

    if (this.head.next) {
      this.head = this.head.next;
    } else {
      this.head = null;
      this.tail = null;
    }

    return deletedHead;
  }

  /**
   * @param {*[]} values - Array of values that need to be converted to linked list.
   * @return {LinkedList}
   * 数组转链表
   */
  fromArray(values) {
    values.forEach(value => this.append(value));

    return this;
  }

  /**
   * @return {LinkedListNode[]}
   * 链表转数组
   */
  toArray() {
    const nodes = [];

    let currentNode = this.head;
    while (currentNode) {
      nodes.push(currentNode);
      currentNode = currentNode.next;
    }

    return nodes;
  }

  /**
   * @param {function} [callback]
   * @return {string}
   */
  toString(callback) {
    return this.toArray().map(node => node.toString(callback)).toString();
  }

  /**
   * Reverse a linked list.
   * @returns {LinkedList}
   * 链表倒序
   */
  reverse() {
    let currNode = this.head;
    let prevNode = null;
    let nextNode = null;

    while (currNode) {
      // 存储下一个结点
      nextNode = currNode.next;

      // 当前指针指向前一个
      currNode.next = prevNode;

      // 为下一个循环准备
      prevNode = currNode;
      currNode = nextNode;
    }

    // 重置头尾指向
    this.tail = this.head;
    // prevNode是最后一个节点
    this.head = prevNode;

    return this;
  }
}

```


## 测试

```javascript
    const linkedList = new LinkedList();

    // 测试空链表
		linkedList.toString()===''  // true
    linkedList.head===null      // true
    linkedList.tail===null      // true

   // append

    linkedList.append(1);
    linkedList.append(2);   
		linkedList.toString()==='1,2'  //true
    linkedList.tail.next===null   //true

  // prepend
   const linkedList1 = new LinkedList();
   linkedList1.prepend(2);
   linkedList1.head.toString()==='2' //true
   linkedList1.tail.toString()==='2' //true
	 linkedList1.append(1);
   linkedList1.prepend(3);
   linkedList1.toString()==='3,2,1' //true

   // delete

    const linkedList2 = new LinkedList();
    linkedList2.delete(5)===null //true
    linkedList2.append(1);
    linkedList2.append(1);
    linkedList2.append(2);
    linkedList2.append(3);
    linkedList2.append(3);
    linkedList2.append(3);
    linkedList2.append(4);
    linkedList2.append(5);
    const deletedNode= linkedList2.delete(3)
    deletedNode.value===3 //true
    linkedList2.toString()==='1,1,2,4,5' //true

   // delete tail
   const linkedList3 = new LinkedList();
   linkedList3.append(1)
   linkedList3.append(2)
   linkedList3.append(3)
   const tailNode= linkedList3.deleteTail()
   tailNode.value===3 //true
   linkedList3.toString()==='1,2' //true

  // delete head
  linkedList3.deleteHead()
  linkedList3.toString()==='2'
  
  // objects value
  const linkedList4= new LinkedList();
  const nodeValue1 = { value: 1, key: 'key1' };
  const nodeValue2 = { value: 2, key: 'key2' };  
  linkedList4.append(nodeValue1).prepend(nodeValue2)
  const nodeStringfier=value=>`key:${value.key} value:${value.value}`
  linkedList4.toString(nodeStringfier)==='key:key1 value:1,key:key2 value:2'

  // find 
   const linkedList5=new LinkedList();
   linkedList5.append(1).append(2.append(3).append(4)
   const node= linkedList5.find({value:3})
   node.value===3 //true

   // find callback
   const cbNode= linkedList4.find({callback:value=>value.key==='key1'})
   cbNode.value.value===1 //true

 // custom compare Function
   const customFunc=(a,b)=>{
    if(_.isEqual(a,b)){
       return 0
    }else{
      return -1
    }
   }
  const linkedList6=new LinkedList(customFunc)
  linkedList6.append( { value: 1, key: 'key1' }).append( { value: 2, key: 'key2' })
  const node =  linkedList6.find({value: { value: 1, key: 'key1' }})
  node.value.value===1 //true
  node.value.key==='key1'

  // reverse
  const linkedList7 = new LinkedList();
  linkedList7
      .append(1)
      .append(2)
      .append(3);
  linkedList7.toString()==='1,2,3'
  linkedList7.reverse()
  linkedList7.toString()==='3,2,1'


```

