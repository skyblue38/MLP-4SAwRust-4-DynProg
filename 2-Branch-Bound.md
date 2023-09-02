# 2. Branch and Bound
In this milestone, we use a technique called _branch and bound_ to prune some branches off of the decision tree. By skipping huge chunks of trees, this can solve larger problems than the exhaustive search method.
## WORKFLOW
### 1. Create `branch_and_bound` functions
Add more parameters to the exhaustive search functions and save as `branch_and_bound` and `do_branch_and_bound` respectively.
### 2. Modify `branch_and_bound` function
Make the function create the following variables for the initial empty solution:
* best_value (0)
* current_value (0)
* current_weight (0)
* remaining_value (`sum_values` of all item weights)

and pass those values to `do_branch_and_bound`
### 3. Modify `do_branch_and_bound` function
* (a) Add the new parameters `best_value`, `current_value`, `current_weight`, and `remaining_value`.
* (b) If we have a full assignment, return it as in the previous milestone.
* (c) If `current_value + remaining_value <= best_value` then return a blank solution vector (so we continue to use the previous best solution).
* (d) If `current_weight + items[next_index].weight <= allowed_weight`, then recursively try adding the next item to the solution. Pass in values for the `current_value`, `current_weight`, and `remaining_value` parameters that are adjusted for adding the next item to the solution. Save the results of the recursive call in variables `test1_solution`, `test1_value`, and `test1_calls`.
* (e) If `current_weight + items[next_index].weight <= allowed_weight` is not true, then set the variables `test1_solution`, `test1_value`, and `test1_calls` to represent a blank solution with 0 value and 1 function call.
* (f) If `current_value + remaining_value - items[next_index].value > best_value`, then recursively try not adding the next item to the solution. Don't change `current_value` or `current_weight`, but subtract the omitted item's value from the `remaining_value` parameter that you pass into the recursive call. Save the results of the recursive call in variables `test2_solution`, `test2_value`, and `test2_calls`.
* (g) If `current_value + remaining_value - items[next_index].value > best_value` is not true, then set the variables `test2_solution`, `test2_value`, and `test2_calls` to represent a blank solution with 0 value and 1 function call.
* (h) Return the better of the two recursive solutions as before.
### 4. Modify `main` function
Copy the `main` program's code that calls `exhaustive_search` and make it call `branch_and_bound` instead, with additional parameters included in the `do_branch+and_bound` calls. Only run branch and bound if the number of items is no greater than 40.
``` rust
// Branch and bound
if NUM_ITEMS > 40 {    // Only run branch and bound if num_items is small enough.
    println!("Too many items for branch and bound\n");
} else {
    println!("*** Branch and Bound ***");
    run_algorithm(&branch_and_bound, &mut items, allowed_weight);
}
```
### 5. Test run
The complete working program code is available HERE: [m2-branch-bound.rs](m2-branch-bound.rs.md)

When compiled and executed, the following results were displayed:
``` bash
[sysadmin@centos8s ~]$ cd MLP/FSAwRust-4-Dynamic/m2-branch-bound/
[sysadmin@centos8s m2-branch-bound]$ cargo run --release
   Compiling m2-branch-bound v0.1.0 (/home/sysadmin/MLP/FSAwRust-4-Dynamic/m2-branch-bound)
    Finished release [optimized] target(s) in 0.68s
     Running `target/release/m2-branch-bound`
*** Parameters ***
No.items:       20
Total value:    93
Total weight:   144
Allowed weight: 72

*** Brnach and Bound ***
Elapsed: 103.886204ms
3(7, 9) 6(9, 5) 9(4, 4) 10(6, 4) 11(8, 8) 13(8, 9) 14(7, 5) 15(5, 7) 16(5, 7) 17(7, 7) 19(7, 7) 
Value: 73, Weight: 72, Calls: 532061
```

### (end)

?