# 3. Rod's Technique
This algorithm extends the Branch and Bound method by pruning the search tree even further by recognising those items that are more valuable per unit volume or weigh less per unit cost and effectively "block" the inclusion of other items. This requires a change in the Item structure and the initialisation of a *blocked_by* data structure.
## WORKFLOW
### 1. Modify the *Item* structure
``` rust
struct Item {
    id: i32,
    value: i32,
    weight: i32,
    is_selected: bool,
    blocked_by: i32,
    block_list: Vec<i32>,
}
```
### 2. Modify `make_items` and `copy_items` functions.
It should set each item's id to its index position in the items vector, set `blocked_by` to -1, and set `blocked_items` to an empty vector. Similarly modify the `copy_items` function so it can handle the changes to the Item structure.
``` rust
// Make some random items.
fn make_items(prng: &mut Prng, num_items: i32, min_value: i32, max_value: i32,
        min_weight: i32, max_weight: i32) -> Vec<Item> {
    let mut items: Vec<Item> = Vec::with_capacity(num_items as usize);
    for i in 0..num_items {
        let item = Item {
            id: i,
            value: prng.next_i32(min_value, max_value),
            weight: prng.next_i32(min_weight, max_weight),
            is_selected: false,
            blocked_by: -1,
            block_list: Vec::new(),
        };
        items.push(item);
    }
    return items;
}
```
### 3. Modify `main` function
Added the following:
``` rust
// Rod's technique
if NUM_ITEMS > 60 {    // Only run Rod's technique if num_items is small enough.
    println!("Too many items for Rod's technique\n");
} else {
    println!("*** Rod's Technique ***");
    run_algorithm(&rods_technique, &mut items, allowed_weight);
}
```
### 4. Write  `make_block_list` function
``` rust
// Build the items' block lists.
fn make_block_lists(items: &mut Vec<Item>) {
    for i in 0..items.len() {
        items[i].block_list = Vec::new();
        for j in 0..items.len() {
            if i != j {
                if items[i].value >= items[j].value && items[i].weight <= items[j].weight {
                    let id = items[j].id;
                    items[i].block_list.push(id);
                }
            }
        }
    }
}
```
### 5. Create `rods_technique` and `do_rods_technique`
based on the exiting `branch_and_bound` functions.
### 6. Add `make_block_list` to initialisation in `rods_technique`
and call `do_rods_technique` instead of `do_branch_and_bound`.
### 7. Modify `do_rods_technique`
When adding the next item to the solution:
* Initialise the first test's results as NUL.
* If the current item is not blocked, add that item to the solution and recursively call the `do_...` function.
When NOT adding the item to the solution:
* scan the item's `block_list`. For each item in the `block_list` that is not already blocked, set that item's `blocked_by` field to the current item's `id`
*  Set the current item's `is_selected` field to `false` and recurse...
*  Repeat the first step, but this time if the item in he `blocked_list` was blobked by the current ite,, then unblock it by setting its `blocked_by` field to `-1`
### 8. Improvement
Arranging so that the Items with the longer block list appear earlier in the item vector will more quickly trim the search tree. To implement this, a new function called `rods_teechnique_sorted` was created, based on a copy of the exiting `rods_technique` function. This new version includes a change to the way `make_block_lists` function is used to construct the block list...
``` rust
// Make initial block lists.
make_block_lists(items);
// Sort so items with longer blocked lists come first.
items.sort_by(|a, b| b.block_list.len().cmp(&a.block_list.len()));
// Reset the items' IDs.
for i in 0..items.len() {
    items[i].id = i as i32;
}
// Rebuild the blocked lists with the new indices.
make_block_lists(items);
```
Note the rust code on the 4th line, that sorts the `items` vector in descending order of `block_list` length...
### 9. Test Run
The final version of the complete program is available HERE: [m3_rods_technique.rs](m3_rods_technique.rs.md)

The output or runs with a variety of numbers of items is shown below

Note that the output of *Rods Technique sorted* contains the same answers but the items are in a different order:
``` bash
[sysadmin@centos8s ~]$ cd MLP/FSAwRust-4-Dynamic/m3-rods-technique/
[sysadmin@centos8s m3-rods-technique]$ cargo run --release
   Compiling m3-rods-technique v0.1.0 (/home/sysadmin/MLP/FSAwRust-4-Dynamic/m3-rods-technique)
    Finished release [optimized] target(s) in 0.98s
     Running `target/release/m3-rods-technique`
*** Parameters ***
No.items:       20
Total value:    93
Total weight:   144
Allowed weight: 72

*** Exhaustive Search ***
Elapsed: 216.618297ms
3(7, 9) 6(9, 5) 9(4, 4) 10(6, 4) 11(8, 8) 13(8, 9) 14(7, 5) 15(5, 7) 16(5, 7) 17(7, 7) 19(7, 7) 
Value: 73, Weight: 72, Calls: 1048576

*** Branch and Bound ***
Elapsed: 582.147µs
3(7, 9) 6(9, 5) 9(4, 4) 10(6, 4) 11(8, 8) 13(8, 9) 14(7, 5) 15(5, 7) 16(5, 7) 17(7, 7) 19(7, 7) 
Value: 73, Weight: 72, Calls: 43093

*** Rods Technique ***
Elapsed: 191.783µs
3(7, 9) 6(9, 5) 9(4, 4) 10(6, 4) 11(8, 8) 13(8, 9) 14(7, 5) 15(5, 7) 16(5, 7) 17(7, 7) 19(7, 7) 
Value: 73, Weight: 72, Calls: 3387

*** Rods Technique Sorted ***
Elapsed: 23.466µs
0(9, 5) 1(7, 5) 2(6, 4) 3(7, 7) 4(7, 7) 5(4, 4) 6(8, 8) 8(5, 7) 9(5, 7) 10(8, 9) 11(7, 9) 
Value: 73, Weight: 72, Calls: 361
```

``` bash
*** Parameters ***
No.items:        32
Total value:    140
Total weight:   223
Allowed weight: 111

Too many items for exhaustive search
*** Branch and Bound ***
Elapsed: 103.470213ms
2(4, 6) 3(7, 9) 6(9, 5) 9(4, 4) 10(6, 4) 11(8, 8) 13(8, 9) 14(7, 5) 15(5, 7) 16(5, 7) 17(7, 7) 19(7, 7) 20(7, 5) 21(3, 4) 24(5, 5) 25(6, 6) 27(3, 5) 28(7, 8) 
Value: 108, Weight: 111, Calls: 9130515

*** Rods Technique ***
Elapsed: 5.333968ms
2(4, 6) 3(7, 9) 6(9, 5) 9(4, 4) 10(6, 4) 11(8, 8) 13(8, 9) 14(7, 5) 15(5, 7) 16(5, 7) 17(7, 7) 19(7, 7) 20(7, 5) 21(3, 4) 24(5, 5) 25(6, 6) 27(3, 5) 28(7, 8) 
Value: 108, Weight: 111, Calls: 84355

*** Rods Technique Sorted ***
Elapsed: 153.317µs
0(9, 5) 1(7, 5) 2(7, 5) 3(6, 4) 4(7, 7) 5(7, 7) 6(4, 4) 7(5, 5) 8(6, 6) 9(8, 8) 10(3, 4) 11(7, 8) 12(4, 6) 13(5, 7) 14(5, 7) 15(3, 5) 18(8, 9) 19(7, 9) 
Value: 108, Weight: 111, Calls: 3617
```

``` bash
*** Parameters ***
No.items:        50
Total value:    232
Total weight:   339
Allowed weight: 169

Too many items for exhaustive search
Too many items for branch_and_bound search
*** Rods Technique ***
Elapsed: 605.336632ms
2(4, 6) 3(7, 9) 6(9, 5) 9(4, 4) 10(6, 4) 11(8, 8) 13(8, 9) 14(7, 5) 15(5, 7) 16(5, 7) 17(7, 7) 19(7, 7) 20(7, 5) 21(3, 4) 24(5, 5) 25(6, 6) 28(7, 8) 31(6, 8) 32(6, 5) 33(8, 9) 37(7, 6) 38(9, 4) 39(9, 8) 40(8, 4) 44(8, 7) 45(5, 6) 46(5, 6) 
Value: 176, Weight: 169, Calls: 6102467

*** Rods Technique Sorted ***
Elapsed: 1.020036ms
0(9, 4) 1(8, 4) 2(9, 5) 3(7, 5) 4(7, 5) 5(6, 4) 6(7, 6) 7(6, 5) 8(8, 7) 9(5, 5) 10(4, 4) 11(6, 6) 12(7, 7) 13(7, 7) 14(9, 8) 15(5, 6) 16(5, 6) 17(8, 8) 18(3, 4) 19(7, 8) 20(4, 6) 21(5, 7) 22(5, 7) 25(6, 8) 29(8, 9) 30(8, 9) 32(7, 9) 
Value: 176, Weight: 169, Calls: 21709
```

``` bash
*** Parameters ***
No.items:       100
Total value:    500
Total weight:   654
Allowed weight: 327

Too many items for exhaustive search
Too many items for branch_and_bound search
Too many items for Rods technique

*** Rods Technique Sorted ***
Elapsed: 39.049464ms
0(9, 4) 1(9, 4) 2(8, 4) 3(9, 5) 4(9, 5) 5(9, 5) 6(8, 5) 7(8, 5) 8(7, 4) 9(7, 5) 10(7, 5) 11(7, 5) 12(7, 5) 13(7, 5) 14(7, 5) 15(9, 6) 16(6, 4) 17(6, 4) 18(8, 6) 19(8, 6) 20(6, 5) 21(7, 6) 22(7, 6) 23(9, 7) 24(5, 5) 25(8, 7) 26(8, 7) 27(4, 4) 28(6, 6) 29(7, 7) 30(7, 7) 31(7, 7) 32(5, 6) 33(5, 6) 34(5, 6) 35(4, 5) 36(4, 5) 37(4, 5) 38(3, 4) 39(3, 4) 40(9, 8) 41(9, 8) 42(6, 7) 43(8, 8) 44(8, 8) 45(3, 5) 54(7, 8) 64(9, 9) 65(8, 9) 66(8, 9) 67(8, 9) 69(7, 9) 70(7, 9) 71(7, 9) 
Value: 373, Weight: 327, Calls: 621359
```

``` bash
*** Parameters ***
No.items:       200
Total value:    962
Total weight:   1312
Allowed weight: 656

Too many items for exhaustive search
Too many items for branch_and_bound search
Too many items for Rods technique

*** Rods Technique Sorted ***
Elapsed: 1.630641043s
0(9, 4) 1(9, 4) 2(9, 4) 3(9, 4) 4(8, 4) 5(9, 5) 6(9, 5) 7(9, 5) 8(9, 5) 9(9, 5) 10(9, 5) 11(9, 5) 12(7, 4) 13(7, 4) 14(7, 4) 15(8, 5) 16(8, 5) 17(8, 5) 18(6, 4) 19(7, 5) 20(7, 5) 21(7, 5) 22(7, 5) 23(7, 5) 24(6, 4) 25(7, 5) 26(6, 4) 27(6, 4) 28(7, 5) 29(6, 4) 30(7, 5) 31(6, 4) 32(9, 6) 33(9, 6) 34(9, 6) 35(8, 6) 36(8, 6) 37(8, 6) 38(8, 6) 39(8, 6) 40(5, 4) 41(6, 5) 42(6, 5) 43(7, 6) 44(7, 6) 45(7, 6) 46(7, 6) 47(9, 7) 48(9, 7) 49(9, 7) 50(5, 5) 51(8, 7) 52(8, 7) 53(8, 7) 54(5, 5) 55(5, 5) 56(4, 4) 57(4, 4) 58(6, 6) 59(6, 6) 60(7, 7) 61(7, 7) 62(7, 7) 63(7, 7) 64(7, 7) 65(5, 6) 66(5, 6) 67(5, 6) 68(5, 6) 69(5, 6) 70(4, 5) 71(4, 5) 72(4, 5) 73(3, 4) 74(3, 4) 75(3, 4) 76(3, 4) 77(6, 7) 78(9, 8) 79(9, 8) 80(8, 8) 81(8, 8) 82(8, 8) 83(8, 8) 84(8, 8) 85(5, 7) 86(5, 7) 87(5, 7) 88(5, 7) 89(5, 7) 90(5, 7) 91(4, 6) 92(4, 6) 93(3, 5) 98(7, 8) 99(7, 8) 100(7, 8) ...
Value: 739, Weight: 656, Calls: 15973275
```

### (end)