# Trees

> **Core idea:** Hierarchical structure. Almost every tree problem is a DFS where you return something from each subtree and combine it at the current node.

---

## 1. Node Structure

```cpp
struct TreeNode {
    int val;
    TreeNode *left, *right;
    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
};
```

---

## 2. All Traversals

### Recursive (easy to write, understand)
```cpp
void inorder(TreeNode* root, vector<int>& res) {
    if (!root) return;
    inorder(root->left, res);
    res.push_back(root->val);   // IN: left → root → right
    inorder(root->right, res);
}
// Preorder: root → left → right  (root->val first)
// Postorder: left → right → root  (root->val last)
```

### Iterative Inorder (Morris / Stack)
```cpp
// Stack-based inorder
vector<int> inorderIter(TreeNode* root) {
    vector<int> res;
    stack<TreeNode*> st;
    TreeNode* cur = root;
    while (cur || !st.empty()) {
        while (cur) { st.push(cur); cur = cur->left; }
        cur = st.top(); st.pop();
        res.push_back(cur->val);
        cur = cur->right;
    }
    return res;
}
```

### Level Order (BFS)
```cpp
vector<vector<int>> levelOrder(TreeNode* root) {
    vector<vector<int>> res;
    if (!root) return res;
    queue<TreeNode*> q;
    q.push(root);
    while (!q.empty()) {
        int sz = q.size();
        res.push_back({});
        while (sz--) {
            auto node = q.front(); q.pop();
            res.back().push_back(node->val);
            if (node->left)  q.push(node->left);
            if (node->right) q.push(node->right);
        }
    }
    return res;
}
```

---

## 3. BST Operations

**BST invariant:** left subtree < root < right subtree (strict).

### Search — O(h)
```cpp
TreeNode* search(TreeNode* root, int key) {
    if (!root || root->val == key) return root;
    return key < root->val ? search(root->left, key) : search(root->right, key);
}
```

### Insert — O(h)
```cpp
TreeNode* insert(TreeNode* root, int key) {
    if (!root) return new TreeNode(key);
    if (key < root->val) root->left  = insert(root->left, key);
    else                 root->right = insert(root->right, key);
    return root;
}
```

### Delete — O(h)
```cpp
TreeNode* deleteNode(TreeNode* root, int key) {
    if (!root) return nullptr;
    if (key < root->val)      root->left  = deleteNode(root->left, key);
    else if (key > root->val) root->right = deleteNode(root->right, key);
    else {
        if (!root->left)  return root->right;
        if (!root->right) return root->left;
        // find inorder successor (min of right subtree)
        TreeNode* succ = root->right;
        while (succ->left) succ = succ->left;
        root->val = succ->val;
        root->right = deleteNode(root->right, succ->val);
    }
    return root;
}
```

---

## 4. Lowest Common Ancestor

### BST — O(h)
```cpp
TreeNode* lcaBST(TreeNode* root, TreeNode* p, TreeNode* q) {
    if (p->val < root->val && q->val < root->val) return lcaBST(root->left, p, q);
    if (p->val > root->val && q->val > root->val) return lcaBST(root->right, p, q);
    return root;  // split point = LCA
}
```

### Generic Binary Tree — O(n)
```cpp
TreeNode* lcaBT(TreeNode* root, TreeNode* p, TreeNode* q) {
    if (!root || root == p || root == q) return root;
    TreeNode* L = lcaBT(root->left, p, q);
    TreeNode* R = lcaBT(root->right, p, q);
    if (L && R) return root;  // p and q in different subtrees
    return L ? L : R;
}
```

---

## 5. Height, Balance, Diameter

```cpp
// Height
int height(TreeNode* root) {
    if (!root) return 0;
    return 1 + max(height(root->left), height(root->right));
}

// Is Balanced (O(n) — single pass)
int heightCheck(TreeNode* root) {
    if (!root) return 0;
    int L = heightCheck(root->left);
    if (L == -1) return -1;
    int R = heightCheck(root->right);
    if (R == -1) return -1;
    if (abs(L - R) > 1) return -1;
    return 1 + max(L, R);
}
bool isBalanced(TreeNode* root) { return heightCheck(root) != -1; }

// Diameter (longest path between any two nodes)
int ans;
int diam(TreeNode* root) {
    if (!root) return 0;
    int L = diam(root->left), R = diam(root->right);
    ans = max(ans, L + R);     // path going through root
    return 1 + max(L, R);
}
int diameterOfBinaryTree(TreeNode* root) { ans = 0; diam(root); return ans; }
```

---

## 6. Complexity

| Operation | BST (avg) | BST (worst, skewed) | Balanced BST |
|-----------|-----------|---------------------|--------------|
| Search | O(log n) | O(n) | O(log n) |
| Insert | O(log n) | O(n) | O(log n) |
| Delete | O(log n) | O(n) | O(log n) |
| Traversal | O(n) | O(n) | O(n) |

AVL tree and Red-Black tree guarantee O(log n) height by rebalancing.

---

## 7. Practice Problems

| Problem | Technique | LC # |
|---------|-----------|------|
| Invert Binary Tree | DFS | 226 |
| Maximum Depth | DFS height | 104 |
| Diameter of Binary Tree | DFS return height | 543 |
| Balanced Binary Tree | DFS -1 sentinel | 110 |
| Level Order Traversal | BFS | 102 |
| Right Side View | BFS last of each level | 199 |
| Validate BST | In-order check / range DFS | 98 |
| Kth Smallest in BST | In-order iterator | 230 |
| Construct from Pre+In | Hash map of inorder indices | 105 |
| Binary Tree Max Path Sum | DFS, track global max | 124 |
| Serialize / Deserialize | Pre-order + null markers | 297 |
