+++
title= "Never Pass Booleans"
date= "2026-02-01"
tags= ["post"]
draft= true
+++
Thesis: When possible, don't pass booleans into functions. The further you separate the boolean's instantiation with its usage the more opaque and hard to follow the control flow gets.

Are you sure you need a boolean?

Perhaps you're trying to make your functions do too much. All the function does should be reflected in a name to avoid misuse and keep the code maintainable.

Why not instead...

1. Use default values and pass in the replacement value instead of switching between possible states to select the correct value.

2. Refactor big, complicated functions into smaller, descriptive ones and calling the ones you need when you need them instead of passing in control flow flags to navigate your labyrinthian function

3. ???