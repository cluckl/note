# LRU

## 注意事项

* 实现方式：HashMap + 自定义双链表
* 在增加或者删除节点时注意对HashMap执行对应的操作
* 在双链表中，删除节点需要改变两条连线；增加一新节点需要改变四条连接，同时注意改变的先后顺序

## 代码

```java
class LRUCache {
    private int capacity;
    private Node head;
    private Node tail;
    HashMap<Integer, Node> memory;

    public LRUCache(int capacity) {
        this.capacity = capacity;
        this.memory = new HashMap<>();

        this.head = new Node();
        this.tail = new Node();
        this.head.postNode = this.tail;
        this.tail.preNode = this.head;
    }

    public int get(int key) {
        Node node = this.memory.get(key);

        if (node == null){
            return -1;
        }

        shiftToFirst(node);

        return node.value;

    }

    public void put(int key, int value) {
        Node node = this.memory.get(key);
        if (node != null){
            node.value = value;
        }
        else{
            node = new Node(value);
            node.key = key;
            if (this.memory.size() >= this.capacity){
                Node deletedNode = popTailNode();
                this.memory.remove(deletedNode.key);
            }
            this.memory.put(key, node);
        }
        shiftToFirst(node);
    }

    class Node{
        int value;
        int key;
        Node preNode;
        Node postNode;

        public Node(){
        }

        public Node(int val){
            this.value = val;
        }

        @Override
        public boolean equals(Object o) {
            if (this == o) {
                return true;
            }
            if (o == null || getClass() != o.getClass()) {
                return false;
            }
            Node node = (Node) o;
            return value == node.value;
        }

        @Override
        public int hashCode() {
            return Objects.hash(value);
        }

    }

    private Node popTailNode(){
        Node node = tail.preNode;
        node.preNode.postNode = tail;
        tail.preNode = node.preNode;
        node.preNode = null;
        node.postNode = null;
        return node;
    }

    private void shiftToFirst(Node node){
        if (node.preNode != null){
            node.preNode.postNode = node.postNode;
            node.postNode.preNode= node.preNode;
        }

        node.preNode = this.head;
        node.postNode = this.head.postNode;

        this.head.postNode.preNode = node;
        this.head.postNode = node;
    }
}
```

