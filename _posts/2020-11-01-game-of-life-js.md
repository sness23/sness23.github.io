I am thinking of Conway's Game of Life recently.  The latest Adabox016 had a
16x32 display and one of the demos was the game of life.  They had a pretty nice
implementation in python, with the central part being:

    # Conway's "Game of Life" is played on a grid with simple rules, based
    # on the number of filled neighbors each cell has and whether the cell itself
    # is filled.
    #   * If the cell is filled, and 2 or 3 neighbors are filled, the cell stays
    #     filled
    #   * If the cell is empty, and exactly 3 neighbors are filled, a new cell
    #     becomes filled
    #   * Otherwise, the cell becomes or remains empty
    #
    # The complicated way that the "m1" (minus 1) and "p1" (plus one) offsets are
    # calculated is due to the way the grid "wraps around", with the left and right
    # sides being connected, as well as the top and bottom sides being connected.
    #
    # This function has been somewhat optimized, so that when it indexes the bitmap
    # a single number [x + width * y] is used instead of indexing with [x, y].
    # This makes the animation run faster with some loss of clarity. More
    # optimizations are probably possible.

     def apply_life_rule(old, new):
         width = old.width
         height = old.height
         for y in range(height):
             yyy = y * width
             ym1 = ((y + height - 1) % height) * width
             yp1 = ((y + 1) % height) * width
             xm1 = width - 1
             for x in range(width):
                 xp1 = (x + 1) % width
                 neighbors = (
                     old[xm1 + ym1] + old[xm1 + yyy] + old[xm1 + yp1] +
                     old[x   + ym1] +                  old[x   + yp1] +
                     old[xp1 + ym1] + old[xp1 + yyy] + old[xp1 + yp1])
                 new[x+yyy] = neighbors == 3 or (neighbors == 2 and old[x+yyy])
                 xm1 = x

They first setup all the variables they will need for the `x-1`, `x+1`, `y-1`
and `y+1` coordinates, other variations have these calculations inlined into the
neighbour calculation step and I find this way makes it much more clear for
learning.

They then calculate `neighbours`, which is the total number of neighbour cells
that have live cells in them.  They lay this out in a particularily nice way,
almost in the form of a picture:

    neighbors = (
        old[xm1 + ym1] + old[xm1 + yyy] + old[xm1 + yp1] +
        old[x   + ym1] +                  old[x   + yp1] +
        old[xp1 + ym1] + old[xp1 + yyy] + old[xp1 + yp1])


In this formula, picture these as a grid, with the central element missing,
which is the cell we are calculating for.  This formatting doesn't work with
automatic text formatters, but is a really nice teaching tool that really helped
me figure this out finally.

Now, this is just the most naive implementation of the Game of Life, there are
multitudes of ways to speed it up, including new projects that demo it on SIMD
and on the GPU.

I rewrote this in node:

    const width = 40
    const height = 30
    let gen = 0;

    function update(a,b) {
        for (y = 0; y < height; y++) {
            yyy = y * width;
            ym1 = ((y + height - 1) % height) * width
            yp1 = ((y + 1) % height) * width
            xm1 = width - 1
            for (x = 0; x < width; x++) {
                xp1 = (x + 1) % width
                neighbours = (a[xm1 + ym1] + a[xm1 + yyy] + a[xm1 + yp1] +
                              a[x + ym1]                  + a[x + yp1]   +
                              a[xp1 + ym1] + a[xp1 + yyy] + a[xp1 + yp1])
                if (neighbours == 3 || (neighbours == 2 && a[yyy+x])) {
                    b[yyy+x] = 1
                } else {
                    b[yyy+x] = 0
                }
                xm1 = x
            }
        }
    }

    function print(a) {
        console.clear()
        for (y = 0; y < height; y++) {
            yyy = y * width;
            for (x = 0; x < width; x++) {
                c = a[yyy+x] == 0 ? " " : "+"
                process.stdout.write(c)
            }
        }
    }

    curr = Array.from({length: width * height},
                      () => Math.floor(Math.random() * 2))
    next = Array.from({length: width * height},
                      () => Math.floor(Math.random() * 2))
    while (true) {
        update(curr,next)
        print(next)
        gen++

        update(next,curr)
        print(curr)
        gen++
    }

It runs pretty quick on the command line even at resolutions of 400x300.  It
does die after a while though with the following stack trace:

    <--- Last few GCs --->

    [271070:0x21a39b0]    27049 ms: Mark-sweep 1398.6 (1423.9) -> 1398.2 (1424.4) MB, 554.2 / 0.0 ms  (average mu = 0.127, current mu = 0.065) allocation failure scavenge might not succeed
    [271070:0x21a39b0]    27608 ms: Mark-sweep 1398.9 (1424.4) -> 1398.4 (1424.9) MB, 552.0 / 0.0 ms  (average mu = 0.073, current mu = 0.013) allocation failure scavenge might not succeed


    <--- JS stacktrace --->

    ==== JS stack trace =========================================

        0: ExitFrame [pc: 0x3ae4ec05452b]
        1: StubFrame [pc: 0x3ae4ec07b3bd]
    Security context: 0x0c6b00d2ee11 <JSObject>
        2: /* anonymous */ [0x3568bbf5d7c9] [net.js:1] [bytecode=0x314a3213e449 offset=0](this=0x35e2685ea2d1 <WriteStream map = 0xb0e9bae3261>,writev=0x0f198df85c31 <false>,data=0x0f198df89d69 <String[1]:  >,encoding=0x0c6b00d72a51 <String[4]: utf8>,cb=0x35e2685f5261 <JSBoundFunction (BoundTargetFunction 0x28c1749e1649)>)
     ...

    FATAL ERROR: Ineffective mark-compacts near heap limit Allocation failed - JavaScript heap out of memory
     1: 0x7fc69563046c node::Abort() [/lib/x86_64-linux-gnu/libnode.so.64]
     2: 0x7fc6956304b5  [/lib/x86_64-linux-gnu/libnode.so.64]
     3: 0x7fc69585ce6a v8::Utils::ReportOOMFailure(v8::internal::Isolate*, char const*, bool) [/lib/x86_64-linux-gnu/libnode.so.64]
     4: 0x7fc69585d0e1 v8::internal::V8::FatalProcessOutOfMemory(v8::internal::Isolate*, char const*, bool) [/lib/x86_64-linux-gnu/libnode.so.64]
     5: 0x7fc695bf7c66  [/lib/x86_64-linux-gnu/libnode.so.64]
     6: 0x7fc695c09043 v8::internal::Heap::PerformGarbageCollection(v8::internal::GarbageCollector, v8::GCCallbackFlags) [/lib/x86_64-linux-gnu/libnode.so.64]
     7: 0x7fc695c09930 v8::internal::Heap::CollectGarbage(v8::internal::AllocationSpace, v8::internal::GarbageCollectionReason, v8::GCCallbackFlags) [/lib/x86_64-linux-gnu/libnode.so.64]
     8: 0x7fc695c0b91d v8::internal::Heap::AllocateRawWithLigthRetry(int, v8::internal::AllocationSpace, v8::internal::AllocationAlignment) [/lib/x86_64-linux-gnu/libnode.so.64]
     9: 0x7fc695c0b975 v8::internal::Heap::AllocateRawWithRetryOrFail(int, v8::internal::AllocationSpace, v8::internal::AllocationAlignment) [/lib/x86_64-linux-gnu/libnode.so.64]
    10: 0x7fc695bd7dda v8::internal::Factory::NewFillerObject(int, bool, v8::internal::AllocationSpace) [/lib/x86_64-linux-gnu/libnode.so.64]
    11: 0x7fc695e6331e v8::internal::Runtime_AllocateInNewSpace(int, v8::internal::Object**, v8::internal::Isolate*) [/lib/x86_64-linux-gnu/libnode.so.64]
    12: 0x3ae4ec05452b
    [1]    271070 abort (core dumped)  node nodelife.js

