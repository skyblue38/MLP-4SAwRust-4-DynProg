# 1. Exhaustive Search
The exhaustive search approach will traverse the entire problem space and is therefore limited because it’s so slow. But it is usually easy to apply to a problem, so it’s often the first approach tried. Then, if it turns out that the program is too slow for the problem at hand, other techniques (described in the following milestones) can be tried instead.
## WORKFLOW
### 1. Random number generator - `Prng`
This has been implemented in previous projects, so it will be copied from there...
### 2. `main` function
``` rust
// START HERE
fn main() {
    // Prepare a Prng using the same seed each time.
    let mut prng = Prng { seed: 1337 };
    //prng.randomize();
    // Make some random items.
    let mut items = make_items(&mut prng, NUM_ITEMS, MIN_VALUE,
        MAX_VALUE, MIN_WEIGHT, MAX_WEIGHT);
    let allowed_weight = sum_weights(&mut items, true) / 2;
    // Display basic parameters.
    println!("*** Parameters ***");
    println!("# items:        {}", NUM_ITEMS);
    println!("Total value:    {}", sum_values(&mut items, true));
    println!("Total weight:   {}", sum_weights(&mut items, true));
    println!("Allowed weight: {}", allowed_weight);
    println!();
    // Exhaustive search
    if NUM_ITEMS > 25 {    // Only run exhaustive search if num_items is small enough.
        println!("Too many items for exhaustive search");
    } else {
        println!("*** Exhaustive Search ***");
        run_algorithm(&exhaustive_search, &mut items, allowed_weight);
    }
}
```
### 3. `make_items` function
``` rust
// Make some random items.
fn make_items(prng: &mut Prng, num_items: i32, min_value: i32, max_value: i32, min_weight: i32, max_weight: i32) -> Vec<Item> {
    let mut items: Vec<Item> = Vec::with_capacity(num_items as usize);
    for _ in 0..num_items {
        let item = Item {
            value: prng.next_i32(min_value, max_value),
            weight: prng.next_i32(min_weight, max_weight),
            is_selected: false,
        };
        items.push(item);
    }
    return items;
}
```
### 4. `copy_items` function
``` rust
// Return a copy of the items.
fn copy_items(items: &mut Vec<Item>) -> Vec<Item> {
    let mut new_items: Vec<Item> = Vec::with_capacity(items.len());
    for item in items {
        let new_item = Item {
            value: item.value,
            weight: item.weight,
            is_selected: item.is_selected,
        };
        new_items.push(new_item);
    }
    return new_items;
}
```
### 5. `sum_values` and `sum_weights` functions
``` rust
// Return the total value of the items.
// If add_all is true, add up all items.
// If add_all is false, only add up the selected items.
fn sum_values(items: &mut Vec<Item>, add_all: bool) -> i32 {
    if add_all {
        return items.iter().map(|item| item.value).sum();
    } else {
        return items.iter().filter(|item| item.is_selected).map(|item| item.value).sum();
    }
}
// Return the total weight of the items.
// If add_all is true, add up all items.
// If add_all is false, only add up the selected items.
fn sum_weights(items: &mut Vec<Item>, add_all: bool) -> i32 {
    if add_all {
        return items.iter().map(|item| item.weight).sum();
    } else {
        return items.iter().filter(|item| item.is_selected).map(|item| item.weight).sum();
    }
}
```
### 6. `solution_value` function
``` rust
// Return the value of this solution.
// If the solution is too heavy, return -1 so we prefer an empty solution.
fn solution_value(items: &mut Vec<Item>, allowed_weight: i32) -> i32 {
    // If the solution's total weight > allowed_weight,
    // return -1 so even an empty solution is better.
    if sum_weights(items, false) > allowed_weight { return -1; }
    // Return the sum of the selected values.
    return sum_values(items, false);
}
```
### 7. `print_selected` function
``` rust
// Print the selected items.
fn print_selected(items: &mut Vec<Item>) {
    let mut num_printed = 0;
    for i in 0..items.len() {
        if items[i].is_selected {
            print!("{}({}, {}) ", i, items[i].value, items[i].weight)
        }
        num_printed += 1;
        if num_printed > 100 {
            println!("...");
            return;
        }
    }
    println!();
}
```
### 8. `run_algorithm` function
``` rust
// Run the algorithm. Display the elapsed time and solution.
fn run_algorithm(alg: &dyn Fn(&mut Vec<Item>, i32) -> (Vec<Item>, i32, i32),
    items: &mut Vec<Item>, allowed_weight: i32)
{
    // Copy the items so the run isn't influenced by a previous run.
    let mut test_items = copy_items(items);
    let start = Instant::now();
    // Run the algorithm.
    let mut solution: Vec<Item>;
    let total_value: i32;
    let function_calls: i32;
    (solution, total_value, function_calls) = alg(&mut test_items, allowed_weight);

    let duration = start.elapsed();
    println!("Elapsed: {:?}", duration);

    print_selected(&mut solution);
    println!("Value: {}, Weight: {}, Calls: {}",
        total_value, sum_weights(&mut solution, false), function_calls);
    println!();
}
```
### 9. `exhaustive_search` function
``` rust
// Recursively assign values in or out of the solution.
// Return the best assignment, value of that assignment,
// and the number of function calls we made.
fn exhaustive_search(items: &mut Vec<Item>, allowed_weight: i32) -> (Vec<Item>, i32, i32) {
    return do_exhaustive_search(items, allowed_weight, 0);
}
```
### 10. `do_exhaustive_search` helper function
``` rust
fn do_exhaustive_search(items: &mut Vec<Item>, allowed_weight: i32, next_index: i32)
        -> (Vec<Item>, i32, i32) {
//	println!("Items {:?}\nAllowed weight: {}, nextItem: {}",items, allowed_weight, next_index);
    // See if we have a full assignment.
    if next_index >= items.len() as i32 {
        // Make a copy of this assignment.
        let mut items_copy = copy_items(items);
        // Return the assignment and its total value.
        let items_value = solution_value(&mut items_copy, allowed_weight);
        return (items_copy, items_value, 1);
    }
    // We do not have a full assignment.
    // Try adding the next item.
    items[next_index as usize].is_selected = true;
    // Recursively call the function.
    let solution_1: Vec<Item>;
    let total_value_1: i32;
    let function_calls_1: i32;
    let new_index = next_index + 1;
    (solution_1, total_value_1, function_calls_1) = do_exhaustive_search(items, allowed_weight, new_index);
    // Try not adding the next item.
    items[next_index as usize].is_selected = false;    
    // Recursively call the function.
    let solution_2: Vec<Item>;
    let total_value_2: i32;
    let function_calls_2: i32;
    (solution_2, total_value_2, function_calls_2) = do_exhaustive_search(items, allowed_weight, new_index);
    // Return the solution that is better.
    let function_calls = function_calls_1 + function_calls_2;
    if total_value_1 >= total_value_2 {
//		print_selected(&mut solution_1);
        return (solution_1, total_value_1, function_calls);
    } else {
//		print_selected(&mut solution_2);
        return (solution_2, total_value_2, function_calls);
    }
}
```
### 11. test run
When executed, the following output was produced:
``` text
[sysadmin@centos8s ~]$ cd MLP/FSAwRust-4-Dynamic/m1-xsearch
[sysadmin@centos8s m1-xsearch]$ cargo run --release
   Compiling m1-xsearch v0.1.0 (/home/sysadmin/MLP/FSAwRust-4-Dynamic/m1-xsearch)
    Finished release [optimized] target(s) in 0.64s
     Running `target/release/m1-xsearch`
*** Parameters ***
# items:        20
Total value:    93
Total weight:   144
Allowed weight: 72

*** Exhaustive Search ***
Elapsed: 189.417408ms
3(7, 9) 6(9, 5) 9(4, 4) 10(6, 4) 11(8, 8) 13(8, 9) 14(7, 5) 15(5, 7) 16(5, 7) 17(7, 7) 19(7, 7) 
Value: 73, Weight: 72, Calls: 1048576

[sysadmin@centos8s m1-xsearch]$ 
```
**Note**: This concurs with the expected output published in the project solution.

Partial solution provided with the Milestone specifications is available HERE: [m1-xsearch-partial.rs](m1-xsearch-partial.rs.md)

Final version of the complete program is available HERE: [m1-xsearch.rs](m1-xsearch.rs.md)

## REFERENCES
* [Wikipedia - Knapsack Problem](https://en.wikipedia.org/wiki/Knapsack_problem)
* [Rust Documentation - crate `rand`](https://docs.rs/rand/latest/rand)
* [Wikipedia - Pseudo-Random Number Generator](https://en.wikipedia.org/wiki/Pseudorandom_number_generator)
* [Wikipedia - Linear congruential generator](https://en.wikipedia.org/wiki/Linear_congruential_generator)


### (end)
