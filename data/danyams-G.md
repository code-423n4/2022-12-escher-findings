# Increment in For Loop Can Go Unchecked

# <= rather than <

# > rather than != 

# Does not Cache State Variables

# Unnecessary SLOAD 

- getPrice( ) unnecessarily loads state variables when called internally 
- the sale can be passed in via memory as it was already loaded when called 