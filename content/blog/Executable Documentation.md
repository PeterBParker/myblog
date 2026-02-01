+++
title= "Executable Documentation"
date= "2026-02-01"
tags= ["post"]
draft= true
+++
This attempts to help relieve the problem that devs may encounter in large repos where a change may have cascading effects on workflows and documentation. In those situations it is easy for documents, Makefiles, and scripts to grow stale and unusable.

1. Make documentation executable instead of static (a heavily commented and clearly written script instead of a markdown process doc)

2. Integrate it in your dev workflows and CI pipelines

3. If something with the process changes that would have necessitated a change to the documentation, the dev workflow/CI pipeline should break and draw attention to the required update

4.