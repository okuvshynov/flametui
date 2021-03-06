# Flame Graphs for terminal

![Example](/samples/sqlite_sample.png)

This is an implementation of flame graphs right in the terminal.

For an overview of flame graphs and original implementation, check [Brendan Gregg's website](http://www.brendangregg.com/flamegraphs.html) 

While not as powerful as GUI tools, having chart right in the terminal is more convenient in case of iterative process, like:
1. run a profile on remote machine
2. visualize and study the profile
3. make changes in code/workload/app-level settings/profiling settings/...
4. repeat

In specific case of Linux perf flamereport can be seen as an alternative to running 'perf report'.

## Flame graphs overview
Please check http://www.brendangregg.com/flamegraphs.html

## Usage example

### With provided sample

```$ python flame.py < samples/osx_dtrace```

### reproducing provided sample on Mac OS

1. Run dtrace with -q quiet flag:

```sudo dtrace -q -x stackframes=100 -n 'profile-99 /arg0/ { @[stack()] = count(); } tick-10s { exit(0); }' -o dtrace_stacks```

2. Get https://github.com/brendangregg/FlameGraph (needed for stack folding)

```$ git clone https://github.com/brendangregg/FlameGraph.git```

3. Now collapse stacks:

```$ ./FlameGraph/stackcollapse.pl < dtrace_stacks > stacks```

4. Run:

```$ python ./flametui/flame.py < stacks```

### With Linux perf

1. Get https://github.com/brendangregg/FlameGraph (needed for stack folding)

```$ git clone https://github.com/brendangregg/FlameGraph.git```

2. Do system-wide profile:

```$ perf record -F 999 -a -g -- sleep 10```

3. Collapse stacks:

```$ perf script | ./FlameGraph/stackcollapse-perf.pl > stacks```

4. View visualization

```$ python ./flametui/flame.py < ./stacks```


## Interactive commands
* left/h - select the block to the left
* right/l - select the block to the right
* up/k - select the block above
* down/j - select the block below
* f - focus. Zoom into selected frame; makes it fit whole width, hides the siblings.
* p - pin to top. This zooms into selected frame (like 'f') and hides parent frames. This is useful for deep traces, as there's no scrolling functionality.
* F - hard focus. Zoom into all frames with title of selected frame, pin them to top.
* x - eXclude frame. Removes the selected frame, its children and shrinks parents accordingly. This is useful when the graph is dominated by few large but not particularly interesting frames; It provides a more convenient view compared to focusing on 'smaller but more interesting' frames individually.
* r - reset focus. Resets the focus but keeps exculded frames excluded.
* R - hard reset, brings everything to default view; Useful after exclusions.
* single mouse click - select the block
* double mouse click - zoom into the block. Equivalent to 'f'
* / - search
* n/N - select next/prev block within the highlighted set of views
* q - quit

## Output description

Simple selection with auto-highlight matching frames

![Example](/samples/sqlite_selection.png)

Selecting a cell with multiple frames in it. Pressing 'n' on previous example would bring us here:

![Example](/samples/sqlite_multiselect.png)

In case 'x' was used to hide some of the less interesting frames, status will show how much was excluded:

![Example](/samples/sqlite_exclusion.png)


Graph element examples
* '[foo---]' -- individual frame. 
* '-' - also individual frame. 
* '+' - aggregated frame view which contains several frames. Selecting it will show the details in the status area. Might have hidden descendents. 

## Known limitations
* No color-coding for parts of stack traces (kernel/userspace/vm/native/etc.)
