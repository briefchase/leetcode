# LeetCode Problems

## General Thoughts & Heuristics

- **Always implement a non-optimal solution first.** Even if you hate it. Write it out in plain English first, then translate it directly into Java. Getting a working brute-force solution on the board breaks the mental block and often reveals the exact bottleneck you need to optimize.
- **Does sorting help us solve this problem?** (Sorting can turn $O(N^2)$ lookups into $O(N)$ Two-Pointer squeezes, naturally group duplicates together for easy skipping/pruning, or allow for greedy choices).

## Arrays, Strings & Pointers

### Two Pointers: Two Sum (1)

```
To solve Two Sum:
    Create a map to store (required complement -> index)
    For each number in the array:
        If the current number is in the map:
            Return the index from the map and the current index
        Otherwise:
            Calculate the complement (target - current) and store it in the map with the current index
```

**Generalization Strategy (Two Sum / HashMap):**

- **Trigger**: "Find a pair/subarray that sums to X", or "Count pairs that meet a condition".
- **Method**: As you loop through numbers, figure out what the "missing piece" (complement) is, and store it in a HashMap. This turns nested $O(N^2)$ lookups into a single $O(N)$ pass.

```java
public int[] twoSum(int[] nums, int target) {
    Map<Integer, Integer> map = new HashMap<>();
    for (int i = 0; i < nums.length; i++) {
        if (map.containsKey(nums[i])) return new int[] {map.get(nums[i]), i};
        map.put(target - nums[i], i);
    }
    return null;
}
```

### Two Pointers: Valid Palindrome (125)

```
To solve Valid Palindrome:
    Convert the string to lowercase and remove all non-alphanumeric characters
    Loop through the first half of the cleaned string:
        Compare the character at the current index with its mirror index at the end
        If they don't match:
            Return false
    If the loop finishes without mismatches:
        Return true
```

**Generalization Strategy (Two Pointers / Ends):**

- **Trigger**: "Palindromes", "Find pairs in a *sorted* array", "Reverse an array".
- **Method**: Place one pointer at the start and one at the end. Move them inward based on constraints (e.g., skip symbols, or shrink if a sum is too big/small).

```java
public boolean isPalindrome(String s) {
    s = s.toLowerCase();
    s = s.replaceAll("[^a-z0-9]", "");
    int p = (int)(s.length()/2);
    for (int i = 0; i < p; i++) {
        if (s.charAt(i) != s.charAt(s.length()-i-1)) return false;
    }
    return true;
}
```

### Sliding Window: Longest Substring Without Repeating Characters (3)

```
To find the length of the longest substring without repeating characters:
    Set the start position to 0
    If the text is empty:
        Return 0
    Set the longest found string to the first character of the text
    For each position in the text from beginning to end:
        Get the character at the current position
        Look at the active window of text (from the start position to just before the current position)
        If the current character is already inside that active window:
            Find the exact spot of that duplicate character within the window
            Move the start position to the space immediately following the duplicate
        Create the current substring from the updated start position up to the current position
        If the current substring is longer than the longest found string:
            Update the longest found string to be this current substring
    Return the length of the longest found string
```

**Generalization Strategy (Sliding Window):**

- **Trigger**: "Longest/Shortest contiguous subarray or substring", "Max sum of a subarray of size K".
- **Method**: Use two pointers (`left` and `right`) to represent a window. Expand `right` to consume elements. When your window breaks the rules (e.g., duplicate found, sum too big), shrink `left` until the window is valid again.
- **Fixed-Size Window / Anagrams (Find All Anagrams - LC 438):** Anagrams can be identified by comparing character frequency HashMaps. Instead of rebuilding the map for every window, use a fixed-size window and update the map incrementally in $O(1)$ time (add the new right character, remove the old left character), then check if `currentMap.equals(targetMap)`.

```java
public int lengthOfLongestSubstring(String s) {
    int j = 0;
    // Note: In Java, always use !s.isEmpty() or !s.equals("") instead of != ""
    if (!s.isEmpty()) {
        String longest = s.charAt(0) + "";
        for (int i = 0; i < s.length(); i++) {
            String sub = s.substring(j, i+1);
            char next = sub.charAt(sub.length()-1);
            // Check if 'next' exists in the window BEFORE index i
            if (s.substring(j, i).contains(next + "")) {
                // Search for 'next' ONLY up to index i so we don't accidentally find the current character
                j = j + s.substring(j, i).indexOf(next) + 1;
            }
            // Standalone IF: Recalculate the window content with the new 'j' pointer applied
            sub = s.substring(j, i+1);
            if (sub.length() > longest.length()) {
                longest = sub;
            }
        }
        System.out.println(longest);
        return longest.length();
    } else return 0;
}
```

### Prefix Sum: Subarray Sum Equals K (560)

```
To find the count of subarrays that sum to k:
    Track prefix sum frequencies in a map // Start with {0: 1}
    Maintain running currentSum // Distance from start
    For each number:
        Update currentSum // currentSum += num
        If (currentSum - k) exists in map: // Find how many times a previousSum of (previousSum = currentSum - k) occurred behind us
            Add frequency to count // Each occurrence is an instance of k = currentSum - previousSum
        Update map with currentSum // Save for future gaps
```

**Generalization Strategy (Prefix Sum):**

- **Trigger**: "Subarray sums", "Continuous range equals K", "Number of subarrays meeting a math condition".
- **Method**: You can't use Sliding Window for sums if the array has negative numbers. Instead, keep a running total. If you want a subarray sum of `K`, you look backwards in a HashMap to see if you previously passed a sum of `(Running Total - K)`.

```java
public int subarraySum(int[] nums, int k) {
    Map<Integer, Integer> map = new HashMap<>();
   
    map.put(0, 1); // Base case: prefix sum 0 occurs once
   
    int currentsum = 0;
    int count = 0;
   
    for (int i = 0; i < nums.length; i++) {
        currentsum += nums[i];
       
        if (map.containsKey(currentsum - k)) { // If currentSum(12) - k(5) = prefixSum(7) exists
            count += map.get(currentsum - k); // Add freq of that prefixSum
        }
       
        map.put(currentsum, map.getOrDefault(currentsum, 0) + 1); // Track this new prefixSum
    }
   
    return count;
}
```

### Merge Intervals: Merge Intervals (56)

```
To merge overlapping intervals:
    Sort the intervals based on their start times
    Create a list to hold the merged result
    For each interval in the sorted list:
        If the result list is empty or the current interval starts after the last merged interval ends:
            Add the current interval to the result list
        Otherwise (they overlap):
            Update the end time of the last merged interval to the maximum of its current end and the new interval's end
```

**Generalization Strategy (Merge Intervals):**

- **Trigger**: "Overlapping meetings", "Scheduling", "Merging ranges".
- **Method**: ALWAYS sort the intervals by their start time first. Once sorted, any overlap will be strictly adjacent. You only ever need to compare the `start` of the current interval with the `end` of the last processed interval.

```java
// Here, time can be reduced by sorting first
public int[][] merge(int[][] intervals) {
    // Base case: if there's 1 or 0 intervals, they are already merged
    if (intervals.length <= 1) {
        return intervals;
    }

    // 1. Sort the intervals by their start index
    Arrays.sort(intervals, (a, b) -> Integer.compare(a[0], b[0])); //IMPORTANT
    // 2. Create an empty list to build our final answer
    List<int[]> merged = new ArrayList<>();
    // 3. Iterate through the sorted intervals
    for (int[] interval : intervals) {
        // If the answer list is empty OR the current interval's start is after the last merged interval's end (no overlap)
        if (merged.isEmpty() || merged.get(merged.size() - 1)[1] < interval[0]) {
            merged.add(interval);
        }
        // Otherwise, they overlap. Update the end of the last merged interval.
        else {
            merged.get(merged.size() - 1)[1] = Math.max(merged.get(merged.size() - 1)[1], interval[1]);
        }
    }

    // 4. Convert the ArrayList back to a standard 2D array
    return merged.toArray(new int[merged.size()][]);
}
```

### Fast & Slow Pointers: Linked List Cycle (141)

```
To detect a cycle in a linked list:
    Create a set to track visited node references
    Traverse the list starting from the head:
        If the current node is already in the set:
            A cycle exists; return true
        Otherwise:
            Add the node to the set and move to the next node
    If you reach the end of the list (null):
        Return false
```

**Generalization Strategy (Fast & Slow Pointers):**

- **Trigger**: "Cycle in a Linked List", "Find the Middle of a Linked List", "Linked List Cycle Start Point".
- **Method**: Use a `slow` pointer moving 1 step and a `fast` pointer moving 2 steps. If they ever equal each other, there is a cycle. If `fast` hits null, the list is finite. (Also used to perfectly find the middle: when `fast` reaches the end, `slow` is exactly halfway).

```java
public boolean hasCycle(ListNode head) {
    Set<ListNode> visited = new HashSet<>();
    ListNode current = head;
   
    // Safely handles empty lists by checking current != null directly
    while (current != null) {
        // HashSet natively compares object memory references in O(1) time
        if (visited.contains(current)) {
            return true;
        }
        visited.add(current);
        current = current.next;
    }
   
    return false;
}
```

### Math / Gauss's Formula: Missing Number (268)

```
To find the missing number in a range [0, n]:
    Calculate the expected sum of all numbers from 0 to n using Gauss's formula: n * (n + 1) / 2
    Calculate the actual sum of all numbers present in the array
    The difference between the expected and actual sums is the missing number
```

**Generalization Strategy (Math / Index Mapping):**

- **Trigger**: "Array containing numbers from 1 to N", "Find missing/duplicate number".
- **Method**: If the numbers map exactly to indices (0 to N), you can use Gauss's formula for sums, XOR logic, or swap each number `nums[i]` to its correct index `nums[nums[i]-1]`. It turns O(N) space hash-maps into O(1) space math.

```java
public int missingNumber(int[] nums) {
    int n = nums.length;
   
    // Gauss's Formula: The expected sum of a sequence from 0 to n
    int expectedSum = n * (n + 1) / 2;
   
    // The actual sum of the elements present in the array
    int actualSum = 0;
    for (int num : nums) {
        actualSum += num;
    }
   
    // The difference is mathematically guaranteed to be the missing number
    return expectedSum - actualSum;
}
```

## Advanced Data Structures

### Monotonic Stack: Daily Temperatures (739) HARD TO GENERALIZE

```
To find how many days until a warmer temperature:
    Create a result array and a stack to store indices of temperatures
    For each day's temperature:
        While the stack isn't empty and the current temperature is warmer than the temperature at the top index:
            Pop the index and calculate the distance (current index - popped index)
            Store this distance in the result array for that popped index // because its a warmer day than than the top STORE THE INDEX
        Push the current day's index onto the stack // each day on the stack is colder than the day below it on the stack
```

**Generalization Strategy (Monotonic Stack):**

- **Trigger**: "Next greater element", "Next warmer day", "Nearest smaller element".
- **Method**: Use a stack to store *indices*. As you iterate, if the current element "breaks" the sorting order you want to maintain (e.g., it's warmer than the top of the stack), you keep popping from the stack. The current element is the "answer" for everything you just popped.

```java
public int[] dailyTemperatures(int[] temperatures) {
    int[] result = new int[temperatures.length];
    // The stack stores the INDICES of temperatures, not the temperatures themselves.
    Stack<Integer> stack = new Stack<>();
   
    for (int i = 0; i < temperatures.length; i++) {
        // If the current day's temperature is warmer than the temperature at the index
        // sitting on top of the stack, we found our warmer day.
        while (!stack.isEmpty() && temperatures[i] > temperatures[stack.peek()]) {
            int prevIndex = stack.pop();
            result[prevIndex] = i - prevIndex; // Calculate the distance in days
        }
        // Push the current day's index onto the stack to wait for a warmer day.
        stack.push(i);
    }
   
    return result;
}
```

### Heap / Priority Queue: Top K Frequent Elements (347) HARD TO IMPLEMENT

```
To find the k most frequent elements:
    Count the frequency of each number using a map
    Create a min-heap to track the most frequent elements, comparing based on their map counts
    For each unique number in the map:
        Add it to the heap
        If the heap size exceeds k:
            Remove the least frequent element (the top of the min-heap)
    Extract the k remaining elements from the heap into a result array
```

**Generalization Strategy (Heap / Priority Queue):**

- **Trigger**: "Top K", "Kth largest/smallest", "K closest items", or "Streaming data where you need the current best without re-sorting". "Top-Heavy" Optimization: K is much smaller than N (O(N log K) is faster than O(N log N)).
- **Method**: To find the Top/Largest K, use a **Min-Heap** of size K (kick out the smallest elements). To find the Bottom/Smallest K, use a **Max-Heap** of size K (kick out the largest elements). This drops $O(N \log N)$ sorting time down to $O(N \log K)$.
- **Heap vs. HashMap + Heap (973):** Use *only a Heap* if the priority is an **intrinsic property** calculated on the fly (e.g., distance), but use a *HashMap first* if the priority is an **aggregate property** (e.g., frequency) that requires scanning the entire dataset to count or group elements to determine priority.

```java
public int[] topKFrequent(int[] nums, int k) {
    // 1. Build a frequency map to count occurrences of each number
    Map<Integer, Integer> frequencyMap = new HashMap<>();
    for (int num : nums) {
        if (frequencyMap.containsKey(num)) {
            frequencyMap.put(num, frequencyMap.get(num) + 1);
        } else {
            frequencyMap.put(num, 1);
        }
    }

    // 2. Initialize a Min-Heap (PriorityQueue) to track the top k elements.
    // The heap will store numbers, but we order them based on their frequency in frequencyMap.
    PriorityQueue<Integer> minHeap = new PriorityQueue<>((a, b) -> {
        int freqA = frequencyMap.get(a);
        int freqB = frequencyMap.get(b);
        return Integer.compare(freqA, freqB); // Ascending order: smallest frequency at the top
    });

    // 3. Iterate through unique numbers and maintain exactly k most frequent ones in the heap.
    for (int num : frequencyMap.keySet()) {
        minHeap.add(num);
        // If heap size exceeds k, remove the number with the SMALLEST frequency currently in the heap.
        if (minHeap.size() > k) {
            minHeap.poll(); 
        }
    }

    // 4. Extract the k most frequent numbers from the heap into the final result array.
    int[] topK = new int[k];
    for (int i = 0; i < k; i++) {
        topK[i] = minHeap.poll();
    }
   
    return topK;
}
```

### In-Place Linked List Reversal: Reverse Linked List (206)

```
To reverse a linked list in-place:
    Initialize a pointer for the previous node as null and current node as the head
    While the current node is not null:
        Temporarily save the next node
        Reverse the current node's pointer to point to the previous node
        Move the previous pointer to the current node
        Move the current pointer to the saved next node
    Return the previous pointer as the new head
```

**Generalization Strategy (In-Place Linked List Reversal):**

- **Trigger**: "Reverse a linked list", "Reverse nodes in K-groups", "Palindrome Linked List".
- **Method**: You only need 3 pointers (`prev`, `curr`, `nextTemp`). The core loop is always: Save `next`, flip `curr.next` to `prev`, then slide `prev` and `curr` forward one position.

```java
public ListNode reverseList(ListNode head) {
    ListNode prev = null;
    ListNode curr = head;
   
    while (curr != null) {
        // 1. Save the next node so we don't lose the rest of the list
        ListNode nextTemp = curr.next;
       
        // 2. Break the forward link and point it backwards
        curr.next = prev;
       
        // 3. Slide both pointers forward one step
        prev = curr;
        curr = nextTemp;
    }
   
    // prev lands on the last node, which is the new head
    return prev;
}
```

## Trees, Graphs & Matrices

### Tree BFS: Binary Tree Level Order Traversal (102)

```
To traverse a tree level by level:
    Create a result list and a queue for BFS, starting with the root node
    While the queue is not empty:
        Get the current level's size (number of nodes in the queue)
        Create a list for the current level
        For each node in the current level:
            Remove it from the queue and add its value to the level list
            If a child (left or right) exists:
                Add it to the queue for the next level
        Add the completed level list to the final result list
```

**Generalization Strategy (Tree BFS):**

- **Trigger**: "Level by level", "Shortest path in unweighted graph", "Tree Depth".
- **Method**: Use a `Queue`. The secret is the `for (int i = 0; i < queue.size(); i++)` loop inside the `while` loop. By locking in the size at the start of the level, you can process an entire "row" at once before moving deeper.

```java
public List<List<Integer>> levelOrder(TreeNode root) {
    // create the result
    List<List<Integer>> list = new ArrayList<>();
    // create the sturcture we will pack the tree into in order to get level by level access (queue)
    Queue<TreeNode> q = new LinkedList<>();
    if (root != null) q.add(root);
    // while ther is still nodes in the queue to process
    while (q.size() > 0) {
        int l = q.size();
        List<Integer> level = new ArrayList<>();
        // loop through one level
        for (int i = 0; i < l; i++) {
            // process the level into the result
            TreeNode node = q.remove();
            level.add(node.val);
            // pack the next level into the queue
            if (node.left != null) q.add(node.left);
            if (node.right != null) q.add(node.right);
        }
        // add the level to the result
        list.add(level);
    }
    return list;
}
```

### Tree DFS: Lowest Common Ancestor of a Binary Tree (236)

```
To find the lowest common ancestor of two nodes:
    Recursively search for nodes p and q in the left and right subtrees
    Base cases:
        If the current node is null, or matches p or q:
            Return the current node
    If both subtrees return a non-null result:
        The current node is the lowest common ancestor
    Otherwise:
        Return whichever subtree returned a non-null result (or null if both were null)
```

**Generalization Strategy (Tree DFS):**

- **Trigger**: "Path from root to leaf", "Search all paths", "Lowest Common Ancestor", "Tree properties".
- **Method**: Use Recursion. The key is answering: "What does this node need to tell its parent?" Handle the base cases (null node or target found), recurse on children, and then combine those child results to pass back up the chain.

```java
public TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
    TreeNode llca = null, rlca = null;
    // here, we must pack the tree into a stack to get DFS access
    // here we can use the call stack implicitly since DFS requires a stack (not a queue)
    // using an explicit stack requires much more code and the use of a hashmap
    if (root != null) {
        if (root.left != null) llca = lowestCommonAncestor(root.left, p, q);
        if (root.right != null) rlca = lowestCommonAncestor(root.right, p, q);
        if ((root == p || root == q) || (llca != null && rlca != null)) return root;
        if (llca != null) return llca;
        if (rlca != null) return rlca;
    }
    return null;
}
```

### Matrix Traversal (DFS/BFS): Number of Islands (200)

```
To count the number of islands in a grid:
    Iterate through every cell in the grid
    When a '1' (land) is encountered:
        Increment the island count
        Trigger a recursive function to "sink" the entire island by changing all connected '1's to '0's
    The recursive function checks all four directions (up, down, left, right) and stops when it hits water ('0') or the grid boundary
    Return the total island count
```

**Generalization Strategy (Matrix Traversal DFS/BFS):**

- **Trigger**: "Connected components in a grid", "Path finding in a 2D array", "Regions".
- **Method**: Use a nested loop to check every cell. When you find your target (e.g., '1'), increment your answer counter, then launch a recursive DFS (or BFS) from that cell to "infect", "sink", or "visit" all connected cells so you don't count them again.

```java
class Solution {
    public int numIslands(char[][] grid) {
        int count = 0;
        for (int i = 0; i < grid.length; i++) {
            for (int ii = 0; ii < grid[i].length; ii++) {
                if (grid[i][ii] == '1') {
                    sinkIsland(grid, i, ii);
                    count++;
                }
            }
        }
        return count;
    }

    private void sinkIsland(char[][] grid, int x, int y) {
        if (grid[x][y] == '1') {
            grid[x][y] = '0';
            if (x + 1 < grid.length) sinkIsland(grid, x+1, y);
            if (x - 1 >= 0) sinkIsland(grid, x-1, y);
            if (y + 1 < grid[x].length) sinkIsland(grid, x, y+1);
            if (y - 1 >= 0) sinkIsland(grid, x, y-1);
        }
    }
}
```

### Topological Sort: Course Schedule (207) HARD TO GENERALIZE

```
To determine if all courses can be finished:
    Build an adjacency list for the graph and an array to track the "in-degree" (number of prerequisites) for each course
    Add all courses with an in-degree of 0 (no prerequisites) to a queue
    While the queue is not empty:
        Remove a course, increment the count of courses taken, and look at all courses that depend on it
        Decrease their in-degree; if a dependent course's in-degree reaches 0:
            Add it to the queue
    If the count of courses taken equals the total number of courses:
        Return true
    Otherwise (a cycle exists):
        Return false
```

**Generalization Strategy (Topological Sort / Kahn's):**

- **Trigger**: "Prerequisites", "Dependencies", "Order of Tasks", "Detect cycle in directed graph".
- **Method**: 1. Build Adjacency List (Unlocks) & In-Degree Array (Padlocks). 2. Add all nodes with 0 In-Degree to a Queue. 3. Pop the queue, subtract 1 In-Degree from all its neighbors. If a neighbor reaches 0, queue it up.

**In Degree**: the number of incoming edges in a DAG (number of padlocks)  
**Out Degree**: (Unimportant) the number of outgoing edgees in a DAG (number of keys)  
**Adjacency List**: A directory that maps every node in a graph to a list of its "neighbors" (the nodes it points to)  
**Topological Sort**: Linear ordering of verticies in a graph such that for every directed edge from vertex U to V, U comes before V 

- A graph with a cycle cannot have a valid topological sort.  
- Multiple sortings can be valid for the same graph.

This question asks: Is this directed graph a DAG    

1. build the adjacency list (what rooms unlock what padlocks), count the indegrees of each node (the amount of padlocks)
2. queue up all nodes that have 0 indegree (no padlocks), and while the queue is not empty:
  pop off a node and reduce the indegrees of its outdegree nodes by one (unlock the padlocks)  
    any of the outdegree nodes that now have an indegree of 0 get added to the queue

```java
public boolean canFinish(int numCourses, int[][] prerequisites) {
    // 1. Build the graph (Adjacency List) and Indegree array
    List<List<Integer>> adj = new ArrayList<>();
    for (int i = 0; i < numCourses; i++) {
        adj.add(new ArrayList<>());
    }
   
    int[] indegree = new int[numCourses]; // Tracks how many prerequisites a course has
   
    for (int[] pre : prerequisites) {
        int course = pre[0];
        int prerequisite = pre[1];
        adj.get(prerequisite).add(course); // Edge from prerequisite -> course
        indegree[course]++;                // Increase prerequisite count for the course
    }
   
    // 2. Queue up all courses that have ZERO prerequisites
    Queue<Integer> queue = new LinkedList<>();
    for (int i = 0; i < numCourses; i++) {
        if (indegree[i] == 0) {
            queue.offer(i);
        }
    }
   
    // 3. Process the queue (Kahn's Algorithm / Topological Sort)
    int coursesTaken = 0;
    while (!queue.isEmpty()) {
        int current = queue.poll();
        coursesTaken++;
       
        // "Unlock" the dependent courses by reducing their indegree
        for (int neighbor : adj.get(current)) {
            indegree[neighbor]--;
            // If all prerequisites are met, add to the queue
            if (indegree[neighbor] == 0) {
                queue.offer(neighbor);
            }
        }
    }
   
    // 4. If we successfully took every course, there were no cyclical dependencies
    return coursesTaken == numCourses;
}
```

## Logic, Math & Search

### Binary Search: Search in Rotated Sorted Array (33)

```
To find a target in a rotated sorted array:
    Use binary search to find the "pivot index" where the sorted array was rotated (the smallest element)
    Determine which of the two sorted sub-arrays the target should be in by comparing it to the pivot and the array boundaries
    Perform a standard binary search on the chosen sub-array to find the target's index
```

**Generalization Strategy (Binary Search):**

- **Trigger**: "Sorted array", "O(log n) time requirement", "Find a pivot point".
- **Method**: Use `left` and `right` pointers to calculate `mid`. The key is determining *which half is properly sorted* (or which half meets your criteria) so you can definitively discard the other half.

```java
class Solution {
    public int search(int[] nums, int target) {
        if (nums == null || nums.length == 0) {
            return -1;
        }
       
        int pivot = findPivotIndex(nums);
       
        // Route the search to the correct fully sorted sub-array
        if (target >= nums[pivot] && target <= nums[nums.length - 1]) {
            return standardBinarySearch(nums, target, pivot, nums.length - 1);
        } else {
            return standardBinarySearch(nums, target, 0, pivot - 1);
        }
    }

    private int findPivotIndex(int[] nums) {
        int left = 0;
        int right = nums.length - 1;
       
        while (left < right) {
            int mid = left + (right - left) / 2;
            if (nums[mid] > nums[right]) {
                left = mid + 1;
            } else {
                right = mid;
            }
        }
        return left;
    }

    private int standardBinarySearch(int[] nums, int target, int left, int right) {
        while (left <= right) {
            int mid = left + (right - left) / 2;
            if (nums[mid] == target) {
                return mid;
            } else if (nums[mid] < target) {
                left = mid + 1;
            } else {
                right = mid - 1;
            }
        }
        return -1;
    }
}
```

### 1D Dynamic Programming: Coin Change (322) HARD TO APPLY

```
To find the minimum coins for a target amount:
    Create a dynamic programming array to store the minimum coins needed for each amount from 0 to target, initialized with a value greater than target
    Set the base case: 0 coins needed for an amount of 0
    Iterate through every amount from 1 to target:
        For each coin type:
            If the coin is smaller than or equal to the current amount:
                Check if using it (1 + coins for the remainder) is better than the current minimum
    If the target amount's value was never updated:
        It's impossible (return -1)
    Otherwise:
        Return its value
```

**Generalization Strategy (1D Dynamic Programming):**

- **Trigger**: "Minimum/Maximum ways", "Count all ways", "True/False possible". **The Golden Rule:** The problem asks for a *single aggregate number or boolean* (e.g., return `int`), NOT the actual paths.
- **Method**: Define a `dp` array where `dp[i]` holds the optimal answer for state `i`. Set your base cases (e.g., `dp[0] = 0`). Iterate forward, testing all choices (like trying each coin) and taking the `min`/`max` of your current value vs. the optimal past value + 1.
- **The "Left-to-Right" State Machine (House Robber - LC 198):** Stop trying to find the "best combination" globally. Put on blinders and walk left-to-right. At each step `i`, ask: *"If the array ended here, what is my max accumulated value?"* You make a localized binary choice: `Math.max(Take Current + dp[i-2], Skip Current + dp[i-1])`. If you only ever look back 2 steps, drop the array and just slide two variables forward (`prev1`, `prev2`) for $O(1)$ space.

**The 1D DP Decision Matrix:**
Ask these three questions to determine your DP array structure and state transitions:


| Question                                                                       | Condition                                            | Tool / Syntax                                                   |
| ------------------------------------------------------------------------------ | ---------------------------------------------------- | --------------------------------------------------------------- |
| **1. What are we optimizing/counting?** *(Determines the transition equation)* | **Min / Max** (Coin Change)                          | `dp[i] = Math.min/max(dp[i], dp[i-step] + cost)`                |
|                                                                                | **Total Ways** (Climbing Stairs)                     | `dp[i] += dp[i-step]`                                           |
|                                                                                | **Possible / Impossible** (Word Break)               | `dp[i] = dp[i]                                                  |
| **2. Does the order of choices matter?** *(Determines loop nesting)*           | **Order Matters** (Permutations / Min / Max)         | **Outer:** `for i to target` **Inner:** `for choice in choices` |
|                                                                                | **Order Doesn't Matter** (Combinations)              | **Outer:** `for choice in choices` **Inner:** `for i to target` |
| **3. How far back do we look?** *(Determines space complexity)*                | **Variable steps** (e.g., `i - coin`)                | Keep the full `dp[]` array. $O(N)$ space.                       |
|                                                                                | **Fixed steps** (e.g., `i-1`, `i-2` in House Robber) | Drop the array. Use variables (`prev1`, `prev2`). $O(1)$ space. |


```java
public int coinChange(int[] coins, int amount) {
    // 1D dynamic programing (forward). if we calculate the amount of coins required for each amount, and just consider adding one additional coin for each amount, we can just add one coin and look back at our previous counts.

    // The array goes up to 'amount', so size is amount + 1
    int[] dp = new int[amount + 1];
   
    // Fill with a value strictly greater than any possible answer
    int max = amount + 1;
    Arrays.fill(dp, max);
   
    // Base case
    dp[0] = 0;
   
    // Calculate the optimal answer for every amount from 1 to the target
    for (int i = 1; i <= amount; i++) {
        for (int coin : coins) {
            if (coin <= i) {
                // Make the choice: keep current minimum, or use this coin + the optimal remainder
                dp[i] = Math.min(dp[i], dp[i - coin] + 1);
            }
        }
    }
   
    // If the target amount's value was never updated, it's impossible to make
    return dp[amount] > amount ? -1 : dp[amount];
}
```

### Backtracking: Permutations (46)

```
To generate all possible permutations:
    Use recursion and backtracking to build all possible permutations
    In each recursive step:
        If the current path has all numbers:
            Add it to the final result list and return
        Iterate through all numbers in the original array:
            If the number is already used in the current path:
                Skip it
            Mark the number as used, add it to the current path, and recurse
            Un-mark the number and remove it from the path to explore other possibilities
```

**Generalization Strategy (Backtracking):**

- **Trigger**: "All combinations", "All permutations", "Generate all valid paths/subsets". **The Golden Rule:** The problem asks you to return the *actual lists/paths themselves* (e.g., return `List<List<Integer>>`).
- **Method**: Build solutions incrementally using recursion. The core loop is always: 1. **Choose** (add to path, mark used). 2. **Explore** (recursive call). 3. **Un-choose** (remove from path, unmark). Add copies of the path to results when a base case is hit.
- **Combinations with Reuse (Combination Sum - LC 39):** Order does not matter (`[2,3]` is identical to `[3,2]`). Pass a `startIndex` so the loop only looks forward, naturally preventing duplicates.
- **Permutations without Reuse (Permutations - LC 46):** Order matters, but you can only use each number once. Drop `startIndex` (always loop from `i = 0`), but pass a `boolean[] used` array to ensure you don't pick the exact same index twice in a single path.
- **Permutations with Reuse (Combination Sum IV - LC 377):** Order matters, and infinite reuse is allowed. Drop `startIndex` (loop from `i = 0`) and do not use a `used` array.
- **Subsets without Reuse (Subsets - LC 78):** Order does not matter, and elements cannot be reused. Pass a `startIndex` and recurse with `i + 1` to only look forward. **The Leap:** Unlike permutations or combinations that wait for a target length or sum, *every single partial state* in the recursion tree is a valid answer. Add the `path` to the results immediately at the top of the function, without any base case `if` statement.
- **Pruning Duplicates (Combination Sum II - LC 40):** When the input array has duplicate numbers and the result cannot contain duplicate combinations, do NOT use a `HashSet` (it causes TLE). Instead, **sort the array first** and skip identical siblings horizontally inside the `for` loop: `if (i > startIndex && nums[i] == nums[i-1]) continue;`.
- **Crucial Loop Rule:** NEVER mutate the `startIndex` variable inside your `for` loop (e.g., do not write `startIndex = i + 1`). Doing so breaks your pruning conditions (like `i > startIndex`) for subsequent iterations. Always treat `startIndex` as read-only and pass the math directly into the recursive call: `backtrack(..., i + 1)`.

**The Backtracking Decision Matrix:**
Ask these three questions to determine exactly which tools your `for` loop needs:


| Question                                                                    | Condition                      | Tool / Syntax                                                                                                                                                                       |
| --------------------------------------------------------------------------- | ------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **1. Is the target a Set or Sequence?** *(Does* `[1,2] == [2,1]`*?)*        | **Set** (Combinations/Subsets) | Use `startIndex`. `for (int i = startIndex; ...)`                                                                                                                                   |
|                                                                             | **Sequence** (Permutations)    | No `startIndex`. `for (int i = 0; ...)`                                                                                                                                             |
| **2. Can I reuse the exact same element?** *(Can index 0 be picked twice?)* | **No** (Standard)              | *If Set:* Recurse with `i + 1`. *If Sequence:* Use `boolean[] used`.                                                                                                                |
|                                                                             | **Yes** (Infinite Reuse)       | *If Set:* Recurse with `i`. *If Sequence:* Drop `used` array.                                                                                                                       |
| **3. Are there duplicates in the input?** *(e.g.,* `nums = [1, 2a, 2b]`*)*  | **Yes**                        | Sort array first. Add skip condition: *If Set:* `if (i > startIndex && nums[i] == nums[i-1]) continue;` *If Sequence:* `if (i > 0 && nums[i] == nums[i-1] && !used[i-1]) continue;` |
|                                                                             | **No**                         | Do nothing.                                                                                                                                                                         |


```java
public List<List<Integer>> permute(int[] nums) {
    List<List<Integer>> perms = new ArrayList<>();
   
    // recursively add permutations using a shared path and tracking array
    generatePermutations(nums, perms, new ArrayList<>(), new boolean[nums.length]);
   
    return perms;
}

private void generatePermutations(int[] nums, List<List<Integer>> perms, List<Integer> currentPath, boolean[] used) {
    // base case: add completed perms to the master list
    if (currentPath.size() == nums.length) {
        perms.add(new ArrayList<>(currentPath));
        return;
    }

    for (int i = 0; i < nums.length; i++) {
        // skip the ith index if it is already in the current permutation
        if (used[i]) continue;

        // put nums[i] at the end of the current array (Choose)
        used[i] = true;
        currentPath.add(nums[i]);

        // permute the array with the new index added (Explore)
        generatePermutations(nums, perms, currentPath, used);

        // remove the ith index to try the next available number (Un-choose)
        used[i] = false;
        currentPath.remove(currentPath.size() - 1);
    }
}
```

### Greedy Algorithm: Maximum Subarray (53) HARD TO RECOGNISE

```
To find the contiguous subarray with the largest sum:
    Maintain a running sum and a global maximum result, starting both with the first element
    Iterate through the array starting from the second element:
        If the running sum is negative:
            Discard it and start fresh (set to 0)
        Add the current number to the running sum
        If the current running sum is larger than the global maximum:
            Update the global maximum
    Return the global maximum
```

**Generalization Strategy (Greedy / Kadane's Algorithm):**

- **Trigger**: "Maximum sum contiguous subarray", "Local optimal choice leads to global optimal".
- **Method**: Maintain a running state (e.g., `currentSum`). If the state becomes actively harmful to future progress (like a sum dipping below 0), ruthlessly discard it and start fresh. Track your `globalMax` separately.

```java
public int maxSubArray(int[] nums) {
    // 1D Dynamic Programming (The Baseline): The foundational logic. You break the problem down by asking, "What is the maximum subarray ending at index i?" The DP realization is that the answer at i depends entirely on the answer at i-1.
    // Greedy (The Optimization): The space-saver. Instead of storing the history of every index in a full O(N) DP array, you apply a greedy decision to track only the immediate previous state. You make the locally optimal choice at every step to drop a negative running sum.
    // Kadane’s Algorithm (The Name): The historical title given to this specific optimized solution.

    // initialize strictly with primitives using the first element
    int currentsum = nums[0];
    int result = nums[0];
   
    // start the loop at index 1 since index 0 is already accounted for
    for (int i = 1; i < nums.length; i++) {
        if (currentsum < 0) currentsum = 0;
        currentsum = currentsum + nums[i];
        result = Math.max(currentsum, result);
    }
   
    return result;
}
```

## PROBLEM SET 2

### Two Pointers (Squeeze): 3Sum (15)

```
To find all unique triplets that sum to zero:
    Sort the array to handle duplicates and enable two-pointer math
    For each number in the array (up to the third-to-last):
        If this number is the same as the previous one, skip it (avoid duplicate triplets)
        Set a left pointer just after the current number, and a right pointer at the end
        While left pointer is less than right pointer:
            Calculate the sum of the three numbers
            If the sum is 0:
                Add the triplet to the results
                Skip any duplicate numbers for the left pointer
                Skip any duplicate numbers for the right pointer
                Move both pointers inward
            If the sum is too small:
                Move the left pointer right to increase the sum
            If the sum is too big:
                Move the right pointer left to decrease the sum
```

**Generalization Strategy (Two Pointers / Squeeze):**

- **Trigger**: "Find 2 or 3 elements that sum to X", "Return actual values (not indices)", "Avoid duplicate sets".
- **Method**: 1. **Sort** the array. 2. Place pointers at the ends (`left = i + 1`, `right = nums.length - 1`). 3. **Squeeze** inward based on math: if sum is too small, `left++`; if sum is too big, `right--`.
- **Two Pointers vs. Sliding Window:** 
  - *Sliding Window:* Pointers start at `0` and move in the *same direction*. You care about the contiguous chunk of elements *between* the pointers.
  - *Two Pointers (Squeeze):* Pointers start at *opposite ends* and move *inward*. You only care about the two specific elements the pointers are sitting on, ignoring the middle.

```java
class Solution {
    public List<List<Integer>> threeSum(int[] nums) {
        List<List<Integer>> result = new ArrayList<>();
        Arrays.sort(nums); // 1. Sort the array
        
        for (int i = 0; i < nums.length - 2; i++) {
            // Skip duplicate values for the locked number to ensure we don't generate the exact same triplet twice. 
            // (Note: The problem requires unique *triplets*, but the numbers *inside* the triplet can be identical, e.g., [-1, -1, 2] is valid).
            if (i > 0 && nums[i] == nums[i - 1]) continue;
            
            int left = i + 1;
            int right = nums.length - 1;
            
            while (left < right) { // 2. Squeeze
                int sum = nums[i] + nums[left] + nums[right];
                
                if (sum == 0) {
                    result.add(Arrays.asList(nums[i], nums[left], nums[right]));
                    
                    // Skip duplicates for left and right pointers
                    while (left < right && nums[left] == nums[left + 1]) left++;
                    while (left < right && nums[right] == nums[right - 1]) right--;
                    
                    left++;
                    right--;
                } else if (sum < 0) {
                    left++; // Need a bigger number
                } else {
                    right--; // Need a smaller number
                }
            }
        }
        return result;
    }
}
```

**Bonus: Two Sum using Squeeze (Two Sum II - LC 167)**
If an array is *already sorted* (or if you only need to return the actual values instead of original indices), you can solve Two Sum in $O(N)$ time and $O(1)$ space using the exact same squeeze mechanic from the inside of the 3Sum loop:

```java
public List<List<Integer>> twoSumSqueezeAll(int[] numbers, int target) {
    List<List<Integer>> result = new ArrayList<>();
    int left = 0;
    int right = numbers.length - 1;
    
    while (left < right) {
        int sum = numbers[left] + numbers[right];
        
        if (sum == target) {
            result.add(Arrays.asList(numbers[left], numbers[right])); // Found a pair!
            
            // Skip duplicates to avoid identical pairs in the result
            while (left < right && numbers[left] == numbers[left + 1]) left++;
            while (left < right && numbers[right] == numbers[right - 1]) right--;
            
            // Move both pointers inward to look for the next unique pair
            left++;
            right--;
        } else if (sum < target) {
            left++; // Need a bigger number
        } else {
            right--; // Need a smaller number
        }
    }
    return result;
}
```

### 2D Dynamic Programming: Unique Paths (62)

```
To find the number of unique paths in a grid:
    Create a 2D array (dp) of size m x n
    Initialize the first row and first column to 1 (only one way to travel straight down or straight right)
    For each remaining cell (i, j) in the grid:
        The number of ways to reach this cell is the sum of the ways to reach the cell above it (i-1, j) and the cell to its left (i, j-1)
    Return the value in the bottom-right cell
```

**Generalization Strategy (2D Dynamic Programming):**

- **Trigger**: "Number of paths in a grid", "Move only right and down", "Matrix optimization".
- **Method**: 2D Dynamic Programming. `dp[i][j]` represents the optimal state (or count) at cell `(i, j)`. Build the solution top-down and left-to-right by looking at adjacent previous cells.

```java
class Solution {
    public int uniquePaths(int m, int n) {
        // 1. Create the 2D DP array
        int[][] dp = new int[m][n];
        
        // 2. Base cases: Fill the first row and first column with 1s
        // (There is only 1 way to reach any cell in the top row: go straight right)
        // (There is only 1 way to reach any cell in the left col: go straight down)
        for (int i = 0; i < m; i++) {
            dp[i][0] = 1;
        }
        for (int j = 0; j < n; j++) {
            dp[0][j] = 1;
        }
        
        // 3. Build the rest of the grid
        for (int i = 1; i < m; i++) {
            for (int j = 1; j < n; j++) {
                // The number of ways to reach this cell is the sum of the ways 
                // to reach the cell directly above it and directly to its left.
                dp[i][j] = dp[i-1][j] + dp[i][j-1];
            }
        }
        
        // 4. Return the bottom-right corner
        return dp[m-1][n-1];
    }
}
```

### Tree DFS (Bottom-Up): Diameter of Binary Tree (543)

```
To find the longest path between any two nodes:
    Initialize a global variable to track the maximum diameter found so far
    Create a recursive DFS function to calculate the height of a tree:
        If the node is null, return 0
        Recursively find the height of the left subtree and the right subtree
        Update the global maximum diameter with the sum of the left and right heights (the "bent" path)
        Return 1 + the maximum of the left and right heights (the "straight" path) to the parent
    Call the DFS function on the root and return the global maximum diameter
```

**Generalization Strategy (Tree DFS - Global State):**

- **Trigger**: "Longest path between any two nodes", "Path doesn't have to pass through root".
- **Method**: Post-order DFS. The recursive function must return the *straight* path length to the parent, but simultaneously update a global variable with the *bent* path (left + right) at each node.

```java
class Solution {
    public int diameterOfBinaryTree(TreeNode root) {
        int[] maxDiameter = new int[1]; // Array used to pass primitive by reference
        calculateDepth(root, maxDiameter);
        return maxDiameter[0];
    }

    private int calculateDepth(TreeNode node, int[] maxDiameter) {
        if (node == null) return 0; // Base case: null nodes have 0 depth
        
        // 1. Recurse to the bottom to get the straight path depths of children
        int leftDepth = calculateDepth(node.left, maxDiameter);
        int rightDepth = calculateDepth(node.right, maxDiameter);
        
        // 2. Global Update: Calculate the "bent" path through this node
        int currentDiameter = leftDepth + rightDepth;
        if (currentDiameter > maxDiameter[0]) {
            maxDiameter[0] = currentDiameter;
        }
        
        // 3. Return: Send the longest "straight" path up to the parent
        return Math.max(leftDepth, rightDepth) + 1;
    }
}
```

### Prefix/Suffix Arrays: Product of Array Except Self (238)

**Question:** Given an integer array `nums`, return an array `answer` such that `answer[i]` is equal to the product of all the elements of `nums` except `nums[i]`. You must write an algorithm that runs in `O(n)` time and without using the division operation.

```
To find the product of all elements except self without division:
    Create a result array of the same length
    Do a left-to-right pass:
        Keep a running product of all elements to the left of i, and store it in result[i]
    Do a right-to-left pass:
        Keep a running product of all elements to the right of i, and multiply it by the existing value in result[i]
    Return the result array
```

**Generalization Strategy (Prefix/Suffix Arrays):**

- **Trigger**: "Product except self", "O(N) time without division", "State depends on everything to the left AND right".
- **Method**: Precompute prefix and suffix states. Do one pass left-to-right to gather "past" data, and one pass right-to-left to gather "future" data.
- **Prefix Sum vs. Dynamic Programming:** Prefix Sum is an **accumulator** (deterministic math like `+` or `*`) used for fast range querying, whereas DP is a **decision engine** (evaluating choices with `Math.max/min`) used to find optimal paths or combinations.

```java
class Solution {
    public int[] productExceptSelf(int[] nums) {
        int[] result = new int[nums.length];
        
        // 1. Left-to-right pass: store the product of everything to the LEFT of i
        int leftProduct = 1;
        for (int i = 0; i < nums.length; i++) {
            result[i] = leftProduct;
            leftProduct *= nums[i];
        }
        
        // 2. Right-to-left pass: multiply by the product of everything to the RIGHT of i
        int rightProduct = 1;
        for (int i = nums.length - 1; i >= 0; i--) {
            result[i] *= rightProduct;
            rightProduct *= nums[i];
        }
        
        return result;
    }
}
```

### Graph/Matrix - Multi-source BFS: Rotting Oranges (994)

**Question:** You are given an `m x n` `grid` where each cell can have one of three values: `0` (empty), `1` (fresh orange), or `2` (rotten orange). Every minute, any fresh orange that is 4-directionally adjacent to a rotten orange becomes rotten. Return the minimum number of minutes that must elapse until no cell has a fresh orange. If this is impossible, return `-1`.

```
To find the minimum time to rot all oranges:
    Scan the grid: count the fresh oranges and add ALL rotten oranges to a Queue
    If there are no fresh oranges, return 0
    Initialize a minutes counter to 0
    While the Queue is not empty and there are still fresh oranges:
        Increment the minutes counter
        Get the current size of the Queue (to process exactly one level/minute)
        For each orange in this level:
            Pop it from the Queue and check its 4 neighbors
            If a neighbor is fresh: make it rotten, decrement fresh count, and add it to the Queue
    If fresh count is 0, return minutes; otherwise return -1
```

**Generalization Strategy (Multi-source BFS):**

- **Trigger**: "Spread", "Simultaneous starting points", "Shortest time to infect all".
- **Method**: Multi-source BFS. Enqueue *all* initial targets before starting the `while` loop. Track levels (time/distance) by processing exactly `queue.size()` elements per iteration.

```java
class Solution {
    public int orangesRotting(int[][] grid) {
        int m = grid.length;
        int n = grid[0].length;
        Queue<int[]> queue = new LinkedList<>();
        int freshCount = 0;
        
        // 1. Scan the grid to find all initial rotten oranges and count fresh ones
        for (int i = 0; i < m; i++) {
            for (int j = 0; j < n; j++) {
                if (grid[i][j] == 2) {
                    queue.offer(new int[]{i, j});
                } else if (grid[i][j] == 1) {
                    freshCount++;
                }
            }
        }
        
        // If there are no fresh oranges to begin with, it takes 0 minutes
        if (freshCount == 0) return 0;
        
        int minutes = 0;
        int[][] directions = {{1, 0}, {-1, 0}, {0, 1}, {0, -1}};
        
        // 2. Multi-source BFS
        while (!queue.isEmpty() && freshCount > 0) {
            minutes++;
            int size = queue.size(); // Process exactly one level (minute) at a time
            
            for (int i = 0; i < size; i++) {
                int[] current = queue.poll();
                
                for (int[] dir : directions) {
                    int nextRow = current[0] + dir[0];
                    int nextCol = current[1] + dir[1];
                    
                    // If the neighbor is within bounds and is a fresh orange
                    if (nextRow >= 0 && nextRow < m && nextCol >= 0 && nextCol < n && grid[nextRow][nextCol] == 1) {
                        grid[nextRow][nextCol] = 2; // Make it rotten
                        freshCount--;               // Decrement fresh count
                        queue.offer(new int[]{nextRow, nextCol}); // Add to queue for next minute
                    }
                }
            }
        }
        
        // 3. If there are still fresh oranges left, they are unreachable
        return freshCount == 0 ? minutes : -1;
    }
}
```

### ~~Union-Find / Disjoint Set~~ / DFS: Number of Provinces (547)

**Question:** There are `n` cities. Some of them are connected, while some are not. A province is a group of directly or indirectly connected cities and no other cities outside of the group. You are given an `n x n` matrix `isConnected` where `isConnected[i][j] = 1` if the `i`th city and the `j`th city are directly connected, and `0` otherwise. Return the total number of provinces.

~~**Union-Find Approach (Not necessary for most interviews):**~~

~~`To find the number of isolated provinces:`~~
    ~~`Initialize a parent array where each city is its own parent (n provinces total)`~~
    ~~`Iterate through the adjacency matrix:`~~
        ~~`If city i is connected to city j:`~~
            ~~`Find the root parent of i and the root parent of j`~~
            ~~`If they have different root parents:`~~
                ~~`Union them (make one the parent of the other)`~~
                ~~`Decrement the total province count`~~
    ~~`Return the total province count`~~

~~**Generalization Strategy (Union-Find):**~~

~~- **Trigger**: "Connected components", "Groups/Provinces/Islands in an adjacency matrix", "Are these two nodes in the same network?".~~
~~- **Method**: Union-Find (Disjoint Set). Create a `find(x)` function to get the root parent, and a `union(x, y)` function to merge two sets.~~

```java
class Solution {
    public int findCircleNum(int[][] isConnected) {
        int n = isConnected.length;
        int[] parent = new int[n];
        int provinces = n; // Start by assuming every city is its own isolated province
        
        // 1. Initialize the parent array (each city is its own parent)
        for (int i = 0; i < n; i++) {
            parent[i] = i;
        }
        
        // 2. Iterate through the adjacency matrix
        for (int i = 0; i < n; i++) {
            for (int j = i + 1; j < n; j++) { // Only check top-right triangle to avoid redundant checks
                if (isConnected[i][j] == 1) {
                    
                    // 3. Find the root parents of both cities (inline while loops)
                    int rootI = i;
                    while (parent[rootI] != rootI) {
                        rootI = parent[rootI];
                    }
                    
                    int rootJ = j;
                    while (parent[rootJ] != rootJ) {
                        rootJ = parent[rootJ];
                    }
                    
                    // 4. If they have different root parents, Union them
                    if (rootI != rootJ) {
                        parent[rootI] = rootJ; // Make one the parent of the other
                        provinces--;           // We just merged two provinces into one
                    }
                }
            }
        }
        
        return provinces;
    }
}
```

**Iterative BFS Approach (Recommended for Interviews - Single Method, No Helper):**

```
To find the number of isolated provinces iteratively:
    Initialize a boolean array `visited` of size n to track visited cities
    Initialize a `provinces` counter to 0
    Initialize a queue for BFS traversal
    Iterate through each city i from 0 to n-1:
        If city i is not visited:
            Increment `provinces` counter
            Mark city i as visited and push i to queue
            While queue is not empty:
                curr = queue.poll()
                For every other city j from 0 to n-1:
                    If curr is connected to j AND j is not visited:
                        Mark j as visited
                        Push j to queue
    Return `provinces` counter
```

**Generalization Strategy (Iterative Traversal for Connected Components):**

- **Trigger**: "Connected components", "Groups/Provinces/Islands in an adjacency matrix".
- **Method**: Iterate through all nodes. When you find an unvisited node, increment your component counter and use an iterative queue to mark all reachable nodes as visited in a single method.

```java
class Solution {
    public int findCircleNum(int[][] isConnected) {
        int n = isConnected.length;
        boolean[] visited = new boolean[n];
        int provinces = 0;
        Queue<Integer> queue = new LinkedList<>();
        
        // 1. Iterate through every city
        for (int i = 0; i < n; i++) {
            // 2. If the city hasn't been visited, it's a new province
            if (!visited[i]) {
                provinces++;
                visited[i] = true;
                queue.offer(i);
                
                // 3. BFS traversal to mark all connected cities (entirely self-contained)
                while (!queue.isEmpty()) {
                    int curr = queue.poll();
                    for (int j = 0; j < n; j++) {
                        if (isConnected[curr][j] == 1 && !visited[j]) {
                            visited[j] = true;
                            queue.offer(j);
                        }
                    }
                }
            }
        }
        
        return provinces;
    }
}
```

### Doubly Linked List + Hash: LRU Cache (146)

**Question:** Design a data structure that follows the constraints of a Least Recently Used (LRU) cache. Implement the `LRUCache` class with `get(key)` and `put(key, value)` methods. If the number of keys exceeds the `capacity` from a put operation, evict the least recently used key. The functions `get` and `put` must each run in `O(1)` average time complexity.

```
To design an O(1) LRU Cache:
    Create a custom Node class with key, value, prev, and next pointers
    Initialize a HashMap mapping keys to Nodes
    Initialize a Doubly Linked List with dummy head and tail nodes
    For get(key):
        If key exists: move its Node to the head (most recently used) and return value
        Else: return -1
    For put(key, value):
        If key exists: update its value and move its Node to the head
        Else:
            Create a new Node, add it to the head, and add it to the HashMap
            If size exceeds capacity: remove the Node just before the dummy tail from both the list and the HashMap
```

**Generalization Strategy (Doubly Linked List + Hash):**

- **Trigger**: "O(1) cache", "Evict least recently used", "O(1) access AND O(1) reordering".
- **Method**: Combine a HashMap (for $O(1)$ lookup) with a custom Doubly Linked List (for $O(1)$ removal and insertion at the ends).

```java
class LRUCache {
    // 1. Define the custom Doubly Linked List Node
    class Node {
        int key;
        int value;
        Node prev;
        Node next;
        Node(int key, int value) {
            this.key = key;
            this.value = value;
        }
    }
    
    private int capacity;
    private Map<Integer, Node> map;
    private Node head;
    private Node tail;

    public LRUCache(int capacity) {
        this.capacity = capacity;
        this.map = new HashMap<>();
        
        // 2. Initialize dummy head and tail nodes to avoid null checks
        head = new Node(-1, -1);
        tail = new Node(-1, -1);
        head.next = tail;
        tail.prev = head;
    }
    
    public int get(int key) {
        if (!map.containsKey(key)) {
            return -1;
        }
        
        Node node = map.get(key);
        // Move the accessed node to the head (most recently used)
        removeNode(node);
        addToHead(node);
        
        return node.value;
    }
    
    public void put(int key, int value) {
        if (map.containsKey(key)) {
            // Update existing node and move to head
            Node node = map.get(key);
            node.value = value;
            removeNode(node);
            addToHead(node);
        } else {
            // Create new node
            Node newNode = new Node(key, value);
            map.put(key, newNode);
            addToHead(newNode);
            
            // If we exceeded capacity, evict the least recently used node (just before tail)
            if (map.size() > capacity) {
                Node lru = tail.prev;
                removeNode(lru);
                map.remove(lru.key);
            }
        }
    }
    
    // Helper function to remove a node from the linked list
    private void removeNode(Node node) {
        node.prev.next = node.next;
        node.next.prev = node.prev;
    }
    
    // Helper function to insert a node right after the dummy head
    private void addToHead(Node node) {
        node.next = head.next;
        node.prev = head;
        head.next.prev = node;
        head.next = node;
    }
}
```

