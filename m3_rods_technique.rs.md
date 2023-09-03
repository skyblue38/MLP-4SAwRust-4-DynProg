``` rust
// Manning Learning Project - Four Small Algorithms with Rust
// Project 4 - Dynamic Programming
// Milestone 3 - Rod's Technique - based on Partial Solution by Rod Stephens
// Completed by Chris Freeman (03SEP23)
use std::time::{Instant};
use std::time::{SystemTime, UNIX_EPOCH};
const NUM_ITEMS: i32 = 20;    // A reasonable value for exhaustive search.
const MIN_VALUE: i32 = 1;
const MAX_VALUE: i32 = 10;
const MIN_WEIGHT: i32 = 4;
const MAX_WEIGHT: i32 = 10;
#[derive(Debug)]
struct Item {
	id: i32,
    value: i32,
    weight: i32,
    is_selected: bool,
    blocked_by: i32,
    block_list: Vec<i32>
}
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
    // Branch and Bound search
    if NUM_ITEMS > 48 {  // Only run branch_and_bound if num_items is small enough.
        println!("Too many items for branch_and_bound search");
    } else {
        println!("*** Branch and Bound ***");
        run_algorithm(&branch_and_bound, &mut items, allowed_weight);
    }
    // Rod's technique
    if NUM_ITEMS > 60 {    // Only run Rod's technique if num_items is small enough.
        println!("Too many items for Rod's technique\n");
    } else {
        println!("*** Rod's Technique ***");
        run_algorithm(&rods_technique, &mut items, allowed_weight);
    }
    // Rod's technique sorted
    if NUM_ITEMS > 250 {    // Only run Rod's technique sorted if num_items is small enough.
        println!("Too many items for Rod's technique sorted");
    } else {
        println!("*** Rod's Technique Sorted ***");
        run_algorithm(&rods_technique_sorted, &mut items, allowed_weight);
    }
}

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
// Return the value of this solution.
// If the solution is too heavy, return -1 so we prefer an empty solution.
fn solution_value(items: &mut Vec<Item>, allowed_weight: i32) -> i32 {
    // If the solution's total weight > allowed_weight,
    // return -1 so even an empty solution is better.
    if sum_weights(items, false) > allowed_weight { return -1; }
    // Return the sum of the selected values.
    return sum_values(items, false);
}
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
// Traverse the whole solution space Recursively assigning values in or out of the solution.
// Return the best assignment, value of that assignment,
// and the number of function calls made.
fn exhaustive_search(items: &mut Vec<Item>, allowed_weight: i32) -> (Vec<Item>, i32, i32) {
	return do_exhaustive_search(items, allowed_weight, 0);
}
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
    (solution_1, total_value_1, function_calls_1) = 
        do_exhaustive_search(items, allowed_weight, new_index);
    // Try not adding the next item.
    items[next_index as usize].is_selected = false;    
    // Recursively call the function.
	let solution_2: Vec<Item>;
    let total_value_2: i32;
    let function_calls_2: i32;
    (solution_2, total_value_2, function_calls_2) = 
        do_exhaustive_search(items, allowed_weight, new_index);
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
// Recursively assign values in or out of the solution.
// Dont continue searching a branch that cannot meet weight limit.
// Return the best assignment, value of that assignment,
// and the number of function calls we made.
fn branch_and_bound(items: &mut Vec<Item>, allowed_weight: i32) -> (Vec<Item>, i32, i32) {
	let best_value = 0i32;
	let current_value = 0i32;
	let current_weight = 0i32;
	let remaining_value = sum_values(items, true);
	return do_branch_and_bound(items, allowed_weight, 0,
		best_value, current_value, current_weight, remaining_value);
}
fn do_branch_and_bound(items: &mut Vec<Item>, allowed_weight: i32, next_index: i32,
		mut best_value: i32, current_value: i32, current_weight: i32, remaining_value: i32)
        -> (Vec<Item>, i32, i32) {
    // See if we have a full assignment. (b)
    if next_index >= items.len() as i32 {
        // Make a copy of this assignment.
        let items_copy = copy_items(items);
        // Return the assignment and its total value.
        return (items_copy, current_value, 1);
    }
    // if no value improvement is possible, stop searching this branch and backtrack (c)
    if current_value + remaining_value <= best_value {
        // Make a copy of this assignment.
        return (Vec::new(), 0, 1);
	}
    // Define saved solutions for later comparison
    let solution_1: Vec<Item>;
    let total_value_1: i32;
    let function_calls_1: i32;
    let solution_2: Vec<Item>;
    let total_value_2: i32;
    let function_calls_2: i32;
    // We do not have a full assignment.
    // If allowed weight wont be exceeded, try adding the next item (d)
    if current_weight + items[next_index as usize].weight <= allowed_weight {
		items[next_index as usize].is_selected = true;
        // Recursively call the function with adjusted vlaues.
        (solution_1, total_value_1, function_calls_1) = 
            do_branch_and_bound(items, allowed_weight, next_index+1, best_value, 
           		current_value + items[next_index as usize].value, 
           		current_weight + items[next_index as usize].weight, 
           		remaining_value - items[next_index as usize].value);
        if total_value_1 > best_value {
			best_value = total_value_1;
		}
    } else {
        // stop searching this branch and backtrack (e)
        // Return the assignment and zero value.
        solution_1 = Vec::new();
        total_value_1 = 0;
        function_calls_1 = 1;
	}
    // If best_value is improved by NOT adding the next item (f)
    items[next_index as usize].is_selected = false;
    // Recursively call the function.
    (solution_2, total_value_2, function_calls_2) = 
        do_branch_and_bound(items, allowed_weight, next_index+1, 
            best_value, current_value, current_weight, 
           	remaining_value - items[next_index as usize].value);
    // Return the solution that is better (h)
    if total_value_1 >= total_value_2 {
    // print_selected(&mut solution_1);
        return (solution_1, total_value_1, function_calls_1 + function_calls_2 + 1);
    } else {
    // print_selected(&mut solution_2);
        return (solution_2, total_value_2, function_calls_1 + function_calls_2 + 1);
    }
}
// Recursively assign values in or out of the solution.
// Dont continue searching a branch that cannot meet weight limit
// or where the next item is blocked by a previous selection.
// Return the best assignment, value of that assignment,
// and the number of function calls we made.
fn rods_technique(items: &mut Vec<Item>, allowed_weight: i32) -> (Vec<Item>, i32, i32) {
	let best_value = 0i32;
	let current_value = 0i32;
	let current_weight = 0i32;
	let remaining_value = sum_values(items, true);
	// Make initial block lists.
    make_block_lists(items);
	return do_rods_technique(items, allowed_weight, 0,
		best_value, current_value, current_weight, remaining_value);
}

// Recursively assign values in or out of the solution.
// Dont continue searching a branch that cannot meet weight limit
// or where the next item is blocked by a previous selection.
// To speed pruning of search tree, order items by decreasing length of block list
// Return the best assignment, value of that assignment,
// and the number of function calls we made.
fn rods_technique_sorted(items: &mut Vec<Item>, allowed_weight: i32) -> (Vec<Item>, i32, i32) {
	let best_value = 0i32;
	let current_value = 0i32;
	let current_weight = 0i32;
	let remaining_value = sum_values(items, true);
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
	return do_rods_technique(items, allowed_weight, 0,
		best_value, current_value, current_weight, remaining_value);
}

fn do_rods_technique(items: &mut Vec<Item>, allowed_weight: i32, next_index: i32,
		mut best_value: i32, current_value: i32, current_weight: i32, remaining_value: i32)
        -> (Vec<Item>, i32, i32) {
    // See if we have a full assignment. (b)
    if next_index >= items.len() as i32 {
        // Make a copy of this assignment.
        let items_copy = copy_items(items);
        // Return the assignment and its total value.
        return (items_copy, current_value, 1);
    }
    // if no value improvement is possible, stop searching this branch and backtrack (c)
    if current_value + remaining_value <= best_value {
        // Make a copy of this assignment.
        return (Vec::new(), 0, 1);
	}
    // Define saved solutions for later comparison
    let mut solution_1 = Vec::new();
    let mut total_value_1 = 0i32;
    let mut function_calls_1 = 1i32;
    let solution_2: Vec<Item>;
    let total_value_2: i32;
    let function_calls_2: i32;
    // We do not have a full assignment.
    // If not blocked and allowed weight wont be exceeded, try adding the next item (d)
    if items[next_index as usize].blocked_by == -1 &&
       current_weight + items[next_index as usize].weight <= allowed_weight {
		items[next_index as usize].is_selected = true;
        // Recursively call the function with adjusted vlaues.
        (solution_1, total_value_1, function_calls_1) = 
            do_rods_technique(items, allowed_weight, next_index+1, best_value, 
           		current_value + items[next_index as usize].value, 
           		current_weight + items[next_index as usize].weight, 
           		remaining_value - items[next_index as usize].value);
        if total_value_1 > best_value {
			best_value = total_value_1;
		}
    } else {
        // stop searching this branch and backtrack (e)
        // Return the assignment and zero value.
        solution_1 = Vec::new();
        total_value_1 = 0;
        function_calls_1 = 1;
	}
    // If best_value is improved by NOT adding the next item (f)
    // first, apply blocks to anything in the next_item's list
    let this_id = items[next_index as usize].id;
    for i in 0..items[next_index as usize].block_list.len() {
		let block_id = items[next_index as usize].block_list[i] as usize;
		if items[block_id].blocked_by == -1 {
			items[block_id].blocked_by = this_id;
		}
	}
    items[next_index as usize].is_selected = false;
    // Recursively call the function.
    (solution_2, total_value_2, function_calls_2) = 
        do_rods_technique(items, allowed_weight, next_index+1, 
            best_value, current_value, current_weight, 
           	remaining_value - items[next_index as usize].value);
    // Lastley, clear blocks applied above
    let this_id = items[next_index as usize].id;
    for i in 0..items[next_index as usize].block_list.len() {
		let block_id = items[next_index as usize].block_list[i] as usize;
		if items[block_id].blocked_by == this_id {
			items[block_id].blocked_by = -1;
		}
	}
    // Return the solution that is better (h)
    if total_value_1 >= total_value_2 {
    // print_selected(&mut solution_1);
        return (solution_1, total_value_1, function_calls_1 + function_calls_2 + 1);
    } else {
    // print_selected(&mut solution_2);
        return (solution_2, total_value_2, function_calls_1 + function_calls_2 + 1);
    }
}

// Return a copy of the items.
fn copy_items(items: &mut Vec<Item>) -> Vec<Item> {
    let mut new_items: Vec<Item> = Vec::with_capacity(items.len());
    for item in items {
        let new_item = Item {
			id: item.id,
            value: item.value,
            weight: item.weight,
            is_selected: item.is_selected,
            blocked_by: -1,
            block_list: Vec::new(),
        };
        new_items.push(new_item);
    }
    return new_items;
}
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
// ************
// *** Prng ***
// ************
struct Prng {
    seed: u32,
}
impl Prng {
    fn new() -> Self {
        let mut prng = Self { seed: 0 };
        prng.randomize();
        return prng;
    }
    fn randomize(&mut self) {
        let millis = SystemTime::now()
            .duration_since(UNIX_EPOCH)
            .expect("Time went backwards")
            .as_millis();
        self.seed = millis as u32;
    }
    // Return a pseudorandom value in the range [0, 2147483647].
    fn next_u32(&mut self) -> u32 {
        self.seed = self.seed.wrapping_mul(1_103_515_245).wrapping_add(12_345);
        self.seed %= 1 << 31;
        return self.seed;
    }
    // Return a pseudorandom value in the range [0.0, 1.0).
    fn next_f64(&mut self) -> f64 {
        let f = self.next_u32() as f64;
        return f / (2147483647.0 + 1.0);
    }
    // Return a pseudorandom value in the range [min, max).
    fn next_i32(&mut self, min: i32, max: i32) -> i32 {
        let range = (max - min) as f64;
        let result = min as f64 + range * self.next_f64();
        return result as i32;
    }
}
```