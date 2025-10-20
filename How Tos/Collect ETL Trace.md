```shell
Wpr -start {{profile}} -filemode
# ...
# Do the action in app
# ...
Wpr -stop output.etl
```

# Specific profile cheat sheet 

```shell
# Basic CPU sampling with call stacks (lightweight)
wpr -start CPU -start ReferenceSet -filemode
wpr -stop trace.etl

# High-frequency CPU sampling (frame-by-frame detail)
wpr -start "CPU Usage (Precise)" -filemode
wpr -stop trace.etl


# CPU + I/O + Network + Heap events (system-wide performance)
wpr -start CPU -start ReferenceSet -start DiskIO -start FileIO -start Network -start Heap -filemode
wpr -stop trace.etl

# CPU + Memory allocations only (useful for leaks or GC impact)
wpr -start CPU -start Heap -filemode
wpr -stop trace.etl

# CPU + GPU + Display (UI / rendering frame analysis)
wpr -start CPU -start GPU -start Display -start DWM -filemode
wpr -stop trace.etl

# General system activity (balanced preset)
wpr -start GeneralProfile -filemode
wpr -stop trace.etl

# Using a custom WPRP profile (full control over providers)
wpr -start "C:\Path\To\CustomProfile.wprp" -filemode
wpr -stop trace.etl
```

