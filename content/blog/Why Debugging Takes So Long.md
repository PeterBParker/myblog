+++
title= "Why Debugging Takes So Long"
date= "2026-02-01"
tags= ["post"]
draft= true
+++
1. We don't trust the error messages because they *can't* be correct (see the go library error saying "I'm hitting an outdated API server". Check the go.mod. It is the right version. Move on with debugging. Hours later come back because go.sum has an outdated version of the server due to a dependency of a dependency that the compiler was confusedly using.)

OR

2. We trust the error messages because *why* would they be wrong (example?)

  

Moral: Write accurate errors, dammit