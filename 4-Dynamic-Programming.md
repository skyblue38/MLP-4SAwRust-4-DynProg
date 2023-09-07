# 4. Dynamic Programming
## WORKFLOW
### 1. Modify `main` to perform dynamic programming.
``` rust
// Dynamic programming
println!("*** Dynamic programming ***");
run_algorithm(&dynamic_programming, &mut items, allowed_weight);
```
### 2. Write `dynamic_programming` function.
based on the following header:
``` rust
// Use dynamic programming to find a solution.
// Return the best assignment, value of that assignment,
// and the number of function calls we made.
fn dynamic_programming(items: &mut Vec<Item>, allowed_weight: i32) -> (Vec<Item>, i32, i32) {
```
* Make the function allocate the `solution_value` and `prev_weight` arrays so each has `num_items` rows and `allowed_weight + 1` columns
* Initialize row 0. Make variable `w` loop through the weights. If item 0 fits within the weight `w`, then set `solution_value` equal to the item’s value and set `prev_weight` equal to -1. If item 0 does not fit within the weight `w`, then set `solution_value` to 0 and set `prev_weight` equal to `w`
* Use a nested loop to initialise the vectors:
``` rust
 // Fill in the remaining table rows. 2(c)
 for i in 1..NUM_ITEMS as usize {
     for w in 0..=allowed_weight as usize {
        let value_not_in = solution_value[i-1][w];
        let this_weight = items[i].weight;
        let mut value_in = 0;
        let mut pw_index = 0usize;
        if w > this_weight as usize {
			pw_index = w - this_weight as usize;
		}
        value_in = items[i].value + solution_value[i-1][pw_index];
		if value_in > value_not_in && w as i32 > items[i].weight {
			solution_value[i][w] = value_in;
			prev_weight[i][w] = pw_index as i32;
		} else {
			solution_value[i][w] = value_not_in;
			prev_weight[i][w] = w as i32;
		}
     }
 }
```
* Next, all items were de-selected using the loop:
``` rust
    // Set all items.is_selected to false 2(d)
    for i in 0..NUM_ITEMS as usize {
		items[i].is_selected = false;
	}
```
* Finally, the solution was reconstructed and corresponding items were marked `is_selected`.
``` rust
    let mut back_i = NUM_ITEMS - 1;
    let mut back_w = allowed_weight;
    // Work backwards until we reach an initial solution.
    while back_i >= 0 {
        // Check prev_weight for the current solution.
        let prev_w = prev_weight[back_i as usize][back_w as usize];
        if back_w == prev_w {
            // We skipped item back_i.
            // Leave back_w unchanged.
        } else {
            // We added item back_i.
            items[back_i as usize].is_selected = true;   // Select this item in the solution.
            back_w = prev_w;                             // Move to the previous solution's weight.
        }
        back_i -= 1;    // Move to the previous row.
    }
```

### 3. Test Run
The complete program source code is available here: [m4-dyn-prog.rs](m4-dyn-prog.rs.md)

The standard test run with 20 items is shown below:
``` bash
[sysadmin@centos8s m4-dynprog]$ cargo run --release
   Compiling m4-dynprog v0.1.0 (/home/sysadmin/MLP/FSAwRust-4-Dynamic/m4-dynprog)
    Finished release [optimized] target(s) in 0.99s
     Running `target/release/m4-dynprog`
*** Parameters ***
No.items:       20
Total value:    93
Total weight:   144
Allowed weight: 72

*** Exhaustive Search ***
Elapsed: 221.397035ms
3(7, 9) 6(9, 5) 9(4, 4) 10(6, 4) 11(8, 8) 13(8, 9) 14(7, 5) 15(5, 7) 16(5, 7) 17(7, 7) 19(7, 7) 
Value: 73, Weight: 72, Calls: 1048576

*** Branch and Bound ***
Elapsed: 528.441µs
3(7, 9) 6(9, 5) 9(4, 4) 10(6, 4) 11(8, 8) 13(8, 9) 14(7, 5) 15(5, 7) 16(5, 7) 17(7, 7) 19(7, 7) 
Value: 73, Weight: 72, Calls: 43093

*** Rods Technique ***
Elapsed: 232.418µs
3(7, 9) 6(9, 5) 9(4, 4) 10(6, 4) 11(8, 8) 13(8, 9) 14(7, 5) 15(5, 7) 16(5, 7) 17(7, 7) 19(7, 7) 
Value: 73, Weight: 72, Calls: 3387

*** Rods Technique Sorted ***
Elapsed: 23.47µs
0(9, 5) 1(7, 5) 2(6, 4) 3(7, 7) 4(7, 7) 5(4, 4) 6(8, 8) 8(5, 7) 9(5, 7) 10(8, 9) 11(7, 9) 
Value: 73, Weight: 72, Calls: 361

*** Dynamic programming ***
Elapsed: 20.635µs
2(4, 6) 3(7, 9) 6(9, 5) 9(4, 4) 10(6, 4) 11(8, 8) 13(8, 9) 14(7, 5) 15(5, 7) 17(7, 7) 19(7, 7) 
Value: 72, Weight: 71, Calls: 1
```
Note that the *Dynamic Programing* technique seemed to find a different solution to the others... Item#2 was selected instead of item#16 ?  This requires further investigation!

To test the size limits of this implementation, increasingly larger numbers of items were tried, by varying the setting of the constant NUM_ITEMS. It was found that with this virtual machine setup (4GB RAM and 4GB Swap space, yielding about 7.2GB effective storage) the largest number of Items was about 15200. Any more than that produced a system *Out-Of-Memory* error which terminated the Rust application after about 40 seconds...

Output from the 15200 item run is shown below:
``` bash
[sysadmin@centos8s m4-dynprog]$ cargo run --release
   Compiling m4-dynprog v0.1.0 (/home/sysadmin/MLP/FSAwRust-4-Dynamic/m4-dynprog)
    Finished release [optimized] target(s) in 0.77s
     Running `target/release/m4-dynprog`
*** Parameters ***
No.items:       15200
Total value:    75580
Total weight:   98856
Allowed weight: 49428

Too many items for exhaustive search
Too many items for branch_and_bound search
Too many items for Rods technique
Too many items for Rods technique sorted
*** Dynamic programming ***
Elapsed: 39.305797046s
3(7, 9) 6(9, 5) 9(4, 4) 10(6, 4) 11(8, 8) 13(8, 9) 14(7, 5) 15(5, 7) 16(5, 7) 17(7, 7) 19(7, 7) 20(7, 5) 21(3, 4) 24(5, 5) 25(6, 6) 28(7, 8) 31(6, 8) 32(6, 5) 33(8, 9) 37(7, 6) 38(9, 4) 39(9, 8) 40(8, 4) 42(7, 9) 44(8, 7) 45(5, 6) 46(5, 6) 48(5, 7) 50(9, 9) 51(9, 5) 53(7, 9) 57(7, 6) 59(9, 5) 62(8, 5) 63(8, 6) 64(9, 6) 66(8, 6) 68(4, 5) 69(7, 7) 70(7, 5) 71(5, 7) 72(7, 5) 73(7, 5) 74(4, 5) 76(8, 5) 78(9, 4) 79(8, 9) 81(5, 7) 82(4, 5) 83(6, 4) 85(5, 6) 86(3, 4) 88(9, 7) 90(7, 4) 91(9, 8) 93(6, 7) 94(8, 8) 95(7, 5) 96(8, 7) 100(9, 5) ...
Value: 56313, Weight: 49427, Calls: 1
[sysadmin@centos8s m4-dynprog]$ 
```

## REFERENCES
* _Grokking Algorithms_ by Aditya Y. Bhargava (Manning, 2016). Chapter 9, [“Dynamic Programming”](https://livebook.manning.com/book/grokking-algorithms/chapter-9/point-17906-1-240-1)
* [How to solve the Knapsack Problem with dynamic programming](https://medium.com/@fabianterh/how-to-solve-the-knapsack-problem-with-dynamic-programming-eb88c706d3cf)
* [Optimizing the dynamic programming solution for the Knapsack Problem](https://medium.com/@fabianterh/optimizing-the-knapsack-problem-dynamic-programming-solution-for-space-complexity-c6bcdff3870b)
* [Partial solution by Rod Stephens - source code](https://lp-prod-resources.s3.amazonaws.com/1555/186642/2023-08-15-14-29-30/Stephens_RustAlgs%20Knapsack%20partial%20solution%204.rs)
